# Table of contents

1. [Table of Contents](#table-of-contents)
2. [Quick start](#quick-start)
3. [Introduction](#introduction)
4. [Data](#data)
5. [Executables](#executables)
6. [Code](#code)

# Quick start

The commands below are the quickest way to get started. They will get you set up
with a virtual-machine-like setup that includes our complete toolset, and run
the tools to compare the ciphertexts we have provided against each other.

1. Install [docker](https://www.docker.com/products/overview).
2. Navigate to the directory containing this README.
3. `docker build -t obf docker` to install dependencies and build the obfuscation tools.
4. `docker run -v $PWD/mount:/mount obf eval-all` to evaluate each obfuscated function on a random input.

Can't find the docker daemon during `docker build`? On `systemd` systems, run `systemctl start docker` as root.  
Permission problems during `docker build`? Add yourself to the `docker` group or use `sudo`.  
Permission problems during `docker run`? On SELinux systems, add `:z` to the mount instruction:

    $ docker run -itv $PWD/mount:/mount obf /bin/bash

# Introduction

We provide:
* a [`Dockerfile`](obf/docker/Dockerfile) that fetches the code for our tools (see [Code](#code)), builds them, and creates a convenient environment for executing them (see [Executables](#executables)),
* a [directory full of obfuscated programs](obf/mount) (see [Data](#data)).

The [`Dockerfile`](obf/docker/Dockerfile) is a set of instructions to [docker](https://www.docker.com/) which will set up a docker image -- which is somewhat akin to a virtual machine snapshot -- that includes the complete toolset needed to explore and recreate our ORE data. This snapshot includes all dependencies needed for the tools to work correctly, so that any system that supports docker will be able to get our tools working with a minimum of fuss. (Of course, if you prefer, you may peek in the [`Dockerfile`](obf/docker/Dockerfile) yourself as a guide to setting things up on your bare metal!)

In this document, we will use `$` to indicate a prompt on the host machine and `#` to indicate a prompt in a docker container that is based on the image we provide.

To build the image, use `docker build` and pass the directory containing the `Dockerfile`. For example, in the directory containing this tutorial, you can run

    $ docker build -t obf docker

to build the image. The `-t obf` arguments give a convenient, human-readable name to the resulting image; the `docker` argument is the directory containing the `Dockerfile`.

Once you have built a docker image, you can start a docker container to run any single process using the `docker run` command. For example, to poke around interactively, you could use:

    $ docker run -it obf /bin/bash

The `-i` and `-t` options keep stdin open and allocate a TTY, both needed for the shell to be useful; `obf` chooses the image we built earlier. Each time you `docker run` something, it will start afresh from the image -- and in particular, changes to the filesystem or environment will not be shared between invocations. To set up a persistent part of the filesystem, you can map a directory into a container's filesystem with the `-v` option. For example, try

    $ docker run -itv /path/in/host/system:/path/in/docker/container obf /bin/bash

then run

    # ls -l /path/in/docker/container

at the prompt provided.

See the [docker documentation](https://docs.docker.com/) and man pages for further information.

# Data

The [`mount`](obf/mount) directory contains a collection of obfuscated point
functions. Each function is named

    BACKEND/BASE/LENGTH/NUM

where `BACKEND` is either `clt` or `gghlite`, `BASE` is the number base used
when creating the point function, `LENGTH` is the number of digits in the
point, and `NUM` is number identifying the function. The function is itself a
directory. Inside this directory, the `i`th encoded matrices are stored in
files `i.0`, `i.1`, etc.  `i.nrows` and `i.ncols` contains the number of rows
and columns of those matrices, respectively, and `i.input` maps those matrices
to the specific input bit they correspond to.  Finally, `params` contains the
parameter information for the multilinear map.

# Executables

To evaluate an obfuscated program, run

    # obfuscator obf --load-obf path-to-obfuscated-program --eval input --base b --mmap mmap

where `path-to-obfuscated-program` is self explanatory, `b` is the base used to
encode the secret point, `mmap` is `CLT` or `GGH`, and `input` denotes the
input (in base `b`) to evaluate. For the functions we release, the directory
structure tells which mmap to use, which base is appropriate, and how long of
an input to choose; see [Data](#data) for details.

# Code

The docker image contains a number of git repositories in `/inst` containing the code for our tools; or you can browse these repositories via the [5GenCrypto](https://github.com/5GenCrypto) organization on github. An overview of the architecture of our tools is given in [5Gen: A Framework for Prototyping Applications Using Multilinear Maps and Matrix Branching Programs](http://eprint.iacr.org/2016/619). We also briefly review the purpose of each repository here.

* `cryfsm`: Compile cryptol functions to finite state machines, then pretty-print them as matrix branching programs.
* `libaesrand`: A thin wrapper around OpenSSL's AES functions that gives a simplified secure random number generation API.
* `clt13`: An implementation of [Practical Multilinear Maps over the Integers](https://eprint.iacr.org/2013/183).
* `gghlite-flint`: A fork of [gghlite-flint](https://bitbucket.org/malb/gghlite-flint) that includes disk serialization and deserialization, uses secure random number generation, and has some optimization tweaks (see the paper [5Gen: A Framework for Prototyping Applications Using Multilinear Maps and Matrix Branching Programs](http://eprint.iacr.org/2016/619) for details). Implements [GGHLite: More Efficient Multilinear Maps from Ideal Lattices](https://eprint.iacr.org/2014/487).
* `libmmap`: This library provides a consistent API for using multilinear maps, and provides wrappers that instantiate that API with functions from the `gghlite-flint` and `clt13` repositories.
* `obfuscation`: Implements the program obfuscator.
