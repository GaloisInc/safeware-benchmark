# Table of contents

1. [Table of Contents](#table-of-contents)
2. [Quick start](#quick-start)
3. [Introduction](#introduction)
4. [Executables](#executables)
    1. [Bulk ORE operations](#bulk-ore-operations)
    2. [Single-shot MIFE operations](#single-shot-mife-operations)
    3. [Base input creation](#base-input-creation)
5. [Data](#data)
6. [Code](#code)

# Quick start

The commands below are the quickest way to get started. They will get you set up with a virtual-machine-like setup that includes our complete toolset, and run the tools to compare the ciphertexts we have provided against each other.

1. Install [docker](https://www.docker.com/products/overview).
2. Navigate to the directory containing this README.
3. `docker build -t ore-ggh docker` to install dependencies and build the ORE tools.
4. `docker run -v $PWD/mount:/mount ore-ggh eval-all` to compare all pairs of ciphertexts.

Can't find the docker daemon during `docker build`? On `systemd` systems, run `systemctl start docker` as root.  
Permission problems during `docker build`? Add yourself to the `docker` group or use `sudo`.  
Permission problems during `docker run`? On SELinux systems, add `:z` to the mount instruction:

    $ docker run -v $PWD/mount:/mount:z -t ore-ggh eval-all

# Introduction

We provide:
* a [`Dockerfile`](ore-ggh/docker/Dockerfile) that fetches the code for our tools (see [Code](#code)), builds them, and creates a convenient environment for executing them (see [Executables](#executables)),
* a [directory full of ciphertexts](ore-ggh/mount) (see [Data](#data)), and
* a [short script to aid in creating your own ciphertexts](ore-ggh/docker/generate-private-data) (see [Executables](#executables)).

The [`Dockerfile`](ore-ggh/docker/Dockerfile) is a set of instructions to [docker](https://www.docker.com/) which will set up a docker image -- which is somewhat akin to a virtual machine snapshot -- that includes the complete toolset needed to explore and recreate our ORE data. This snapshot includes all dependencies needed for the tools to work correctly, so that any system that supports docker will be able to get our tools working with a minimum of fuss. (Of course, if you prefer, you may peek in the [`Dockerfile`](ore-ggh/docker/Dockerfile) yourself as a guide to setting things up on your bare metal!)

In this document, we will use `$` to indicate a prompt on the host machine and `#` to indicate a prompt in a docker container that is based on the image we provide.

To build the image, use `docker build` and pass the directory containing the `Dockerfile`. For example, in the directory containing this tutorial, you can run

    $ docker build -t ore-ggh docker

to build the image. The `-t ore-ggh` arguments give a convenient, human-readable name to the resulting image; the `docker` argument is the directory containing the `Dockerfile`.

Once you have built a docker image, you can start a docker container to run any single process using the `docker run` command. For example, to poke around interactively, you could use:

    $ docker run -it ore-ggh /bin/bash

The `-i` and `-t` options keep stdin open and allocate a TTY, both needed for the shell to be useful; `ore-ggh` chooses the image we built earlier. Each time you `docker run` something, it will start afresh from the image -- and in particular, changes to the filesystem or environment will not be shared between invocations. To set up a persistent part of the filesystem, you can map a directory into a container's filesystem with the `-v` option. For example, try

    $ docker run -itv /path/in/host/system:/path/in/docker/container ore-ggh /bin/bash

then run

    # ls -l /path/in/docker/container

at the prompt provided. In particular, this is useful for mounting the directory of ciphertexts we provide without doubling the disk space needed.

See the [docker documentation](https://docs.docker.com/) and man pages for further information.

# Executables

We provide a collection of tools for creating and using MIFE ciphertexts, some of which are specialized to make ORE-style usage more convenient. This section discusses the purpose and usage of each of these tools.

## Bulk ORE operations

There are three main top-level shell scripts for ORE: [`keygen-all`](ore-ggh/docker/keygen-all), [`encrypt-all`](ore-ggh/docker/encrypt-all), and [`eval-all`](ore-ggh/docker/eval-all). These run the three steps of the ORE protocol: `keygen-all` produces public/private keypairs that support a given size and number of plaintexts using a specified multilinear map backend; `encrypt-all` takes a collection of plaintexts and produces the corresponding ciphertexts; and `eval-all` takes a collection of ciphertexts and runs all pairwise comparisons, printing the result of each.

Our distribution package includes the data needed to run `eval-all`; during experimentation, you may wish to use the `keygen-all` and `encrypt-all` scripts to create smaller datasets with lower security parameters to reduce the computation time and disk space needed for your tests, but these scripts are not needed to compare two ciphertexts.

All three scripts accept the same arguments:

* `--base DIR` (default `/mount`)

    The script will look in `DIR` for information on which jobs to run. The scripts look for a specific directory structure to decide what to do; your base directory should contain subdirectories of the following form:

        DIR/BACKEND/SECPARAM/

    Here `BACKEND` is one of `clt` or `gghlite` (case sensitive), and `SECPARAM` is a positive number. The ciphertexts we provide are already in this directory structure. The script will crawl through all subdirectories of this form and create some jobs for each. Within each subdirectory:

    | Script | Consumes | Produces | Jobs created |
    | ------ | -------- | -------- | ------------ |
    | `keygen-all` | `public/template.json`\*, `private/seed.bin`\* (if it exists) | `private/mife.priv`, `public/mife.pub` | one job |
    | `encrypt-all` | `private/plain.txt`\*, `private/seed.bin`\*, `private/mife.priv`, `public/template.json`, `public/mife.pub` | subdirectories in `database/` containing ciphertexts | one job per line in `private/plain.txt` |
    | `eval-all` | `database/*`, `public/template.json`, `public/mife.pub` | output on stdout | one job for each ordered pair of ciphertexts in `database/` (including self pairings)

    The asterisks indicate files that are consumed by these scripts but not created by them; see [Base input creation](#base-input-creation) for information on creating `public/template.json`, `private/seed.bin`, and `private/plain.txt`. All three scripts will leave the contents of stdout and stderr for each job in the `logs/` directory.

* `-j NUM` (default varies by script)

    Run up to `NUM` jobs in parallel. The scripts will spawn up to `NUM` expensive subprocesses (`keygen` for `keygen-all`, `encrypt` for `encrypt-all`, or `eval` for `eval-all`) at a time, waiting for all of them to finish before continuing. Higher numbers should let the bulk operation finish more quickly... provided you've got the memory.

* `--help` Show some usage instructions and exit successfully without running any jobs.

The scripts exit once all jobs have finished; exit code 0 indicates all jobs finished successfully, and exit code 1 indicates some job did not.

**Note:** The `Dockerfile` included in this directory was used to generate the GGH-based ciphertexts distributed with the benchmark, and differs slightly from the `Dockerfile` used to generate the CLT-based ciphertexts. It is recommended that you do not use this docker image's `eval-all` to compare the CLT-based ciphertexts distributed with this benchmark.

## Single-shot MIFE operations

The top-level ORE scripts discussed in [Bulk ORE operations](#bulk-ore-operations) dispatch to our implementation of generic multi-input functional encryption (MIFE) as provided by the three executables `keygen`, `encrypt`, and `eval`. These tools provide the three basic operations of the MIFE protocol: `keygen` produces public/private keypairs that support a given function and number of plaintexts using a specified multilinear map backend; `encrypt` takes a single plaintext input to each argument of this function and produces a corresponding ciphertext; and `eval` takes one ciphertext per function argument and evaluates the function.

The command line parameters:

| Short | Argument | Meaning | `keygen` | `encrypt` | `eval` |
| :---: | -------- | ------- | :------: | :-------: | :----: |
| `-h`  | none | Print some usage information, then exit | ✓ | ✓ | ✓ |
| `-C`  | none | Use the CLT13 multilinear map backend | ✓ | ✓ | ✓ |
| `-u`  | directory | Location of the public key | ✓ | ✓ | ✓ |
| `-r`  | directory | Location of the private key | ✓ | ✓ | |
| `-d`  | directory | Location of the ciphertexts | | ✓ | ✓ |
| `-n`  | number | Generate keys that support databases of up to 2^n ciphertexts | ✓ | | |
| `-s`  | number | Security parameter | ✓ | | |
| `-a`  | number | Choose the partition used when encoding matrix elements for this plaintext | | ✓ | |
| `-i`  | arbitrary text | A unique name to store the ciphertext under in the database; use this name in the mandatory argument to `eval` to select the ciphertext created | | ✓ | |
| none | JSON array of strings | Mandatory; the plaintext to be encrypted, specified by selecting a symbol from each step of the matrix branching program (see [Matrix branching programs](#matrix-branching-programs) for details) | | ✓ | |
| none | JSON object | Mandatory; select a ciphertext for each of the function's arguments. Each named argument to the function should be a key in the object, whose value is a directory in the database (see [Matrix branching programs](#matrix-branching-programs) for details on named arguments) | | | ✓ |

Passing `-h` to each program will give further details about defaults for each option and the files used by each step of the MIFE protocol.

## Base input creation

We have three tools for easing the generation of input data to our MIFE tools. We do not expect these tools to be the source of vulnerabilities, but include them for completeness and to encourage experimentation with variants of the released benchmark.

The `cryfsm` tool takes a function written in the high-level language [cryptol](http://cryptol.net/) and converts it into our low-level JSON representation for matrix branching programs (see [Matrix branching programs](#matrix-branching-programs) for details). This JSON is a suitable contents for the `template.json` file used by our MIFE tools. The docker image we provide includes further resources for using and understanding the `cryfsm` tool in the `/inst/cryfsm` directory:

* `README.md` includes a tutorial for using cryfsm, flag reference, and information on the format of the JSON it produces. We duplicate the discussion of the JSON format in [Matrix branching programs](#matrix-branching-programs) so that this document is somewhat self-contained.
* `examples/ore-clt.cry` and `examples/ore-gghlite.cry` are the cryptol files used to generate the `template.json` files distributed with this benchmark. You can regenerate `template.json` from one of these files by running

        # cryfsm /inst/cryfsm/examples/ore-clt.cry -o ore-clt.json

    (and similarly for `ore-gghlite`). You can generate templates for smaller plaintext sizes easily by modifying the `base` and `len` arguments in the cryptol files, which specify the numeric base used internally to represent plaintexts and the number of digits in the plaintexts, respectively.

Unlike standard matrix branching programs, which branch on a bit at each step of the program, our programs branch on an arbitrary alphabet at each step. The `numfsm` tool takes human-readable base-10 numbers and a matrix branching program and converts the numbers into strings over the alphabet used in the matrix branching program. These strings serve as suitable plaintext inputs to the `encrypt` tool. For example, if you have generated `ore-clt.json` using `cryfsm` as above, one could observe the following output:

    # numfsm 123456789012 ore-clt.json
    ["000","000001","001011","011010","010100","100001","001100","100010","010101","101101","101010","010100","100000","000010","010010","010000","000"]

This output contains binary-coded base-6 digits, with digits chunked in two different ways corresponding to the two arguments to the comparison operator. One can also give different values for the two different arguments to the comparison operator; for example:

    # numfsm '{"a":2821109907455,"b":000000000000}' ore-clt.json
    ["101","000000","101101","000000","101101","000000","101101","000000","101101","000000","101101","000000","101101","000000","101101","000000","101"]

Here we can see every left argument to the comparison operation acts as if it has only `5` digits, while every right argument acts as if it has only `0` digits.

The `numfsm` tool always accepts two arguments, `SOURCE` and `TARGET`. The `SOURCE` is either a single base-10 number (in which case all argument positions to the function are populated with this number) or a JSON object with function argument positions as keys and numbers as values. The `TARGET` is a filename containing a matrix branching program in JSON format. See [Matrix branching programs](#matrix-branching-programs) for details of the format as well as a discussion of function argument positions. It prints a JSON array of strings, one string per step in the matrix branching program (and in the alphabet of the matching step), to stdout.

Finally, the [`generate-private-data`](ore-ggh/docker/generate-private-data) script can be used to create random seeds and choose base-10 numbers to serve as the plaintexts for ORE. Other than `-j`, it accepts the same arguments as the [Bulk ORE operations](#bulk-ore-operations) and expects an identical directory structure. It uses `/dev/urandom` to produce a `private/seed.bin` and `private/plain.txt` in each subdirectory it visits. It is hardcoded to produce 1024 arbitrary bytes of seed information and ten plaintexts in the range `0` to `999999999999`.

# Data

The [`mount`](ore-ggh/mount) directory contains a collection of ORE databases. There are subdirectories named after the multilinear map and security parameter used to generate the database, as described in [Bulk ORE operations](#bulk-ore-operations). Each pairing of multilinear map and security parameter contains a `database` directory with ten ciphertexts, a public key in `public/mife.pub`, and a matrix branching program to execute on the ciphertexts in `public/template.json`. We discuss the format of these files in turn below.

## Ciphertexts

The `database` directory contains one subdirectory per ciphertext; in our released package, the ciphertexts we produced were sorted using the `eval` operation and then numbered `0` through `9`. Each ciphertext contains two directories, named `a` and `b`, containing encoded matrices to be used when considering that ciphertext as the left argument to the comparison operator or the right argument, respectively. The encoded matrices are stored in files `0.bin`, `1.bin`, `2.bin`, and so on in a custom binary format. For details on the binary format, you can read the source. Here are some good starting places in the docker image:

* `/inst/mife/mife/mife_io.c` contains the generic skeleton of the format in the `fwrite_mmap_enc_mat` and `fread_mmap_enc_mat` functions.
* `/inst/libmmap/mmap/mmap_clt.c` contains the format for CLT-based encoded matrix elements in the `clt_enc_fread_wrapper` and `clt_enc_fwrite_wrapper` functions.
* `/inst/libmmap/mmap/mmap_gghlite.c` contains the format for GGH-based encoded matrix elements in the `gghlite_enc_fread_raw` and `gghlite_enc_fprint_raw` macros.

## Public keys

The file `public/mife.pub` contains the public key in a custom binary format. Details on the binary format are available in the source:

* `/inst/clt13/src/clt13.c` contains the format for CLT-based public keys in the `clt_pp_fread` and `clt_pp_fsave` functions.
* `/inst/libmmap/mmap/mmap_gghlite.c` contains the format for GGH-based public keys in the `fread_gghlite_params` and `fwrite_gghlite_params` functions.

## Matrix branching programs

The file `public/template.json` contains a description of the matrix branching program that represents the function being evaluated on the ciphertexts. The format is quite general: it allows arbitrary-arity functions with named parameters, and branches on an arbitrary alphabet at each step of the program (unlike the standard presentation which always branches on a single bit).

Our matrix branching programs are structured as a sequence of steps. Each step is associated with one of the named parameters of the function being evaluated, and contains a mapping from symbols to matrices. One matrix is chosen for each step, and the matrix product is computed as usual. A two-dimensional array of strings -- with the same size as the matrix product -- is consulted to choose the output of the function: all strings at non-zero positions in the matrix product are output.

These data are stored in a simple JSON structure. The top level object contains two fields:

* `steps`: a JSON array of steps. Each step is an object containing at least one field:
    * `position`: a string naming the function input which determines the symbol to be used to branch on in this step.
    * All other fields: the field name is a symbol, and the field value is a matrix (a rectangular array of arrays containing elements that are either `0` or `1`). All matrices in a single step have the same size, and the sizes are compatible from step to step (the number of columns in step `i` is the number of rows in step `i+1`). In the programs produced by `cryfsm`, all symbols are bitstrings -- strings containing only the characters `0` and `1` -- and the matrices in the first step will always have exactly one row.
* `outputs`: A rectangular array of arrays of strings. For programs produced by `cryfsm`, there will be exactly one `1` in the matrix product for any given input; if this `1` is in row `r` and column `c` of the matrix product, then the string in row `r` and column `c` of `outputs` gives the cryptol value output by the original function.

See [Base input creation](#base-input-creation) for information on using `cryfsm` to produce a matrix branching program in this format from a cryptol function.

# Code

The docker image contains a number of git repositories in `/inst` containing the code for our tools; or you can browse these repositories via the [5GenCrypto](https://github.com/5GenCrypto) organization on github. An overview of the architecture of our tools is given in [5Gen: A Framework for Prototyping Applications Using Multilinear Maps and Matrix Branching Programs](http://eprint.iacr.org/2016/619). We also briefly review the purpose of each repository here.

* `cryfsm`: Compile cryptol functions to finite state machines, then pretty-print them as matrix branching programs. This also provides the `numfsm` executable. See [Base input creation](#base-input-creation) for details on using these tools.
* `libaesrand`: A thin wrapper around OpenSSL's AES functions that gives a simplified secure random number generation API.
* `clt13`: An implementation of [Practical Multilinear Maps over the Integers](https://eprint.iacr.org/2013/183).
* `gghlite-flint`: A fork of [gghlite-flint](https://bitbucket.org/malb/gghlite-flint) that includes disk serialization and deserialization, uses secure random number generation, and has some optimization tweaks (see the paper [5Gen: A Framework for Prototyping Applications Using Multilinear Maps and Matrix Branching Programs](http://eprint.iacr.org/2016/619) for details). Implements [GGHLite: More Efficient Multilinear Maps from Ideal Lattices](https://eprint.iacr.org/2014/487).
* `libmmap`: This library provides a consistent API for using multilinear maps, and provides wrappers that instantiate that API with functions from the `gghlite-flint` and `clt13` repositories.
* `mife`: Implements the protocol described in [Semantically Secure Order-Revealing Encryption: Multi-Input Functional Encryption Without Obfuscation](https://eprint.iacr.org/2014/834). See [Single-shot MIFE operations](#single-shot-mife-operations) for details on using these tools.
