# **DARPA SAFEWARE Multilinear Map Benchmark (Version 1, August 2016)**
-------------------------------------------------

# Introduction
--------------

We present and invite competitive submissions for two benchmarks demonstrating
the existing state-of-the-art (as of August 2016) regarding multilinear maps
(mmaps) and their applications. Our two benchmarks implement applications of
multi-input functional encryption and program obfuscation,
respectively. Competitive submissions will use existing theory or novel
approaches to concretely break the security of one or more of the
benchmarks. Winners will be those entries that achieve a certain degree of
demonstrable security compromise with the best computational efficiency. This
document describes our benchmarks and the process of submitting a benchmark
entry.

Research into the (in)security of mmaps and their applications is a very active
and fast-moving area of cryptography. New mmap schemes are proposed frequently,
and shown subject to attack just as frequently. However, both the proposals and
attacks are typically theoretical, with relatively few concrete examples. The
benchmarks offered here aim to facilitate the research process by presenting
just such concrete opportunities to demonstrate cryptographic breaks for mmap
schemes. Such breaks will aid in the future development of more secure schemes,
and give us a better understanding of the particular _concrete_ security
parameter settings of the schemes.

Each benchmark uses mmaps for security. One benchmark implements order-revealing
encryption, while the other implements point-function obfuscation. We used [the
5Gen framework](https://github.com/5GenCrypto) to create these benchmarks.  5Gen
is an open-source framework for prototyping and experimenting with applications
of mmaps, with current support for both the CLT13 and GGHLite mmaps.  (See
[5Gen: A Framework for Prototyping Applications Using Multilinear Maps and
Matrix Branching Programs](https://eprint.iacr.org/2016/619) for details on this
framework and for implementation details and parameter settings for the mmaps we
consider.)

A successful competitive submission for a benchmark must demonstrate a certain
level of security compromise to be considered.  We also require that the
successful attack code be made available under an open-source license, with the
goal of building a repository of concrete attacks on the various security
schemes we consider here.  See [What Constitutes a
"Win"](#what-constitutes-a-win) for more details.

Some of our benchmarks have published theoretical attacks against them, and it
seems probable that new attacks may be published in the future.  However, the
goal of these benchmarks is to understand the mechanics and efficiency of viable
attacks at a _concrete_ level, with the hope that this will aid in both
designing and choosing appropriate security parameters in future
constructions. Thus, we believe that concrete implementations of known attacks
that demonstrate their (in)feasibility for specific parameter settings are as
important as any new (and possibly only theoretically applicable) attacks.

# Benchmarks
------------

We now discuss the individual benchmarks in more detail.  Throughout, we use λ
as the security parameter.  Namely, a scheme with λ-bits of security should
require Ω(2<sup>λ</sup>) CPU cycles.  For each benchmark, we consider various
parameter settings based on both the mmap and security parameter used.

## ORE Benchmark
----------------

The ORE benchmark works as follows.  For each setting, we have released ten
ciphertexts encrypted using ORE, as well as the public parameters required to
compare the order of any two ciphertexts.

**Success condition**: Recover one plaintext.

This benchmark provides two different settings to cover the
various mmaps and security parameters.  Throughout, we use a plaintext domain
size of 10<sup>12</sup>.

| Multilinear map |  λ  |
| :-------------: | :--: |
| GGHlite         | 40   |
| CLT             | 80   |

### Known Attacks

We briefly list the known attacks against ORE and the underlying mmaps.  [Miles
et al. (CRYPTO 2016)](https://eprint.iacr.org/2016/147) introduced _annihilation
attacks_ on mmap instantiations, and showed how these can be used to break the
ORE construction of [Boneh et al. (EUROCRYPT
2015)](https://eprint.iacr.org/2014/834), which is the basis for our
construction.  This attack currently only holds for ORE instantiated using
GGHLite, and it is unclear what the concrete performance of this attack is (and
this is something we are particularly interested in exploring from this
benchmark).

Another recent attack is that of
[Cheon et al.](https://eprint.iacr.org/2016/139), which can attack GGHLite
without any low-level encodings of 0.  However, it relies on a known plaintext,
which we do not release as part of this benchmark, and thus this attack does not
appear to be applicable to our setting.  More damaging is the attack of
[Albrecht et al. (CRYPTO 2016)](https://eprint.iacr.org/2016/127), which
suggests a polynomial attack for large multilinearity parameter κ.  As κ in our
case is relatively small in relation to λ, it seems this attack does not
immediately apply, although Albrecht et al. in Section 4.2 of the aforementioned
paper suggest that the constants are favorable to breaks for small κ.

As far as we are aware, there are no known attacks on ORE built from CLT13.

## Obfuscation Benchmark
------------------------

The obfuscation benchmark works as follows.  For each setting, for a given λ we
release an n-bit point function F, namely, a function that takes inputs of n
bits such that for all inputs _except one_ it holds that F(x) = 1.

**Success condition**: Recover the input x for which F(x) = 0.

For this benchmark, we release two different settings to cover the various
mmaps and security parameters; see below.<sup>[1](#footnotes)</sup>

| Multilinear map |  λ  |  n  |
| :-------------: | :-: | :-: |
| GGHLite         | 40  | 40  |
| CLT             | 80  | 80  |

### Known Attacks

Work of [Coron et al. (CRYPTO 2015)](https://eprint.iacr.org/2015/596) presented
an attack on a specific type of obfuscation using "simple" branching programs.
This attack relies on access to zero-encodings at the top level of the mmap, and
in general applies to our construction. However, our particular benchmark
obfuscations do not seem to be affected by this attack, as the only
zero-encoding at the top level is produced when given the hidden point as input,
and thus would imply a break in the first place.

In addition, the caveats regarding the attacks on GGHLite of [Cheon et
al.](https://eprint.iacr.org/2016/139) and [Albrecht et
al.](https://eprint.iacr.org/2016/127) mentioned above apply here as well.

Besides these works, we are aware of no other attacks on our construction.

# What Constitutes a "Win"
--------------------------

A submission will be accepted if it meets the following conditions:

* It meets the success condition for the relevant benchmark.
* The necessary recovered data that meets the success condition is provided along with the entry.
* The code used to meet the conditions is provided, along with any instructions necessary to build and run that code, under an open-source license.
* We can successfully re-create the security break to validate the claim.
* Consulting with the team that realized the submission is available to help if needed in re-creating the break.

# How to Participate
--------------------

The ciphertexts are available on [Dropbox](https://www.dropbox.com/sh/of90kypq8j1nayo/AABwcjxsFOLYfx4adQkZ8GLna?dl=0). Alternatively, please email _safeware.benchmark@galois.com_ to request the ciphertexts of interest to you, and we will prepare a USB key for you.

You may also email _safeware.benchmark@galois.com_ with the required submission information once you have developed a break.

# Declaring Winners
-------------------

At a time of our choosing, we will stop accepting new submissions for these
benchmarks. We will provide advance notice of the date on which submissions
close, with a likely close date of **December 31, 2016**. At this point, both
(1) the team that broke the benchmark first, and (2) the top team on the
leaderboard of each benchmark, will be declared the winners of that
benchmark. Each member of a winning team will receive **TBD**.

In addition, if there is a submission which does not fully break the benchmark
(for example, it only learns some portion of the secret) but otherwise presents
novel features, we may present a "judges prize" in this case.


# Leaderboard
-------------

We will maintain a leaderboard that displays:

* The identity of the initial team to submit a successful entry for each security setting of each benchmark.
* A list, ranked by normalized wall clock time, of successful entries for each setting of each benchmark.

To produce normalized rankings on an accurate [leaderboard](leaderboard.md), we
will reproduce all entries on a Dell PowerEdge R930 server with 4 16-core Intel
Xeon CPUs running at 2.5 GHz, with 2 TB RAM and 5 TB disk storage. We will of
course accept initial breaks that utilize code that cannot be run on our setup
(e.g., due to using distributed computing), but due to the difficulty of
comparing across setups, we will not add such breaks to the ranked list on the
leaderboard.

If a break occurs due to a software bug or other issue which is unrelated to the
underlying cryptographic constructs, we plan to release an updated benchmark
with the same parameters while still giving the team credit for the break.
However, we won't list that break as part of the leaderboard. This choice aims
to keep the focus on the actual crypto constructions, rather than potential
errors in the 5Gen framework prototype.<sup>[2](#footnotes)</sup>

# Future Benchmarks
-------------------

We plan to release new benchmarks periodically, as new (and presumably more
secure) constructions are developed and implemented.

# Questions/Comments?
---------------------

Please email _safeware.benchmark@galois.com_ with any questions and/or comments.

# Footnotes
-----------

<sup>1</sup> We understand that there are (much) more efficient ways to do point-function obfuscation.  The goal here though is to present a particular application of (general) obfuscation that is amenable to a benchmark and which is possible to produce given existing constructions.

<sup>2</sup> What constitutes a "software bug" versus an attack on the cryptographic construction may of course be subjective, and in these edge cases we plan to discuss our thoughts with the attacking team before making a decision.
