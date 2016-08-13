FROM ubuntu:16.04

RUN apt-get -y update
RUN apt-get -y install software-properties-common
RUN add-apt-repository -y ppa:ubuntu-toolchain-r/test
RUN apt-get -y install git gcc-4.9 autoconf libtool make libgmp3-dev libmpfr-dev libmpfr4 libssl-dev python python-dev python-setuptools python-numpy python-networkx wget g++ ghc cabal-install alex happy libghc-async-dev libghc-base-compat-dev libghc-cracknum-dev libghc-data-binary-ieee754-dev libghc-gitrev-dev libghc-graphscc-dev libghc-heredoc-dev libghc-ieee754-dev libghc-monadlib-dev libghc-mtl-dev libghc-old-locale-dev libghc-old-time-dev libghc-presburger-dev libghc-primitive-dev libghc-quickcheck2-dev libghc-random-dev libghc-sbv-dev libghc-smtlib-dev libghc-stm-dev libghc-syb-dev libghc-text-dev libghc-tf-random-dev libghc-utf8-string-dev z3 libghc-ansi-terminal-dev libghc-deepseq-generics-dev libghc-transformers-compat-dev libghc-transformers-base-dev libghc-monad-control-dev libghc-data-default-dev libghc-fgl-dev libghc-hashable-dev libghc-monad-loops-dev libghc-optparse-applicative-dev libghc-universe-base-dev libghc-unordered-containers-dev libghc-attoparsec-dev libghc-aeson-dev libghc-lens-dev libghc-wl-pprint-text-dev libghc-graphviz-dev

WORKDIR /inst
RUN wget http://flintlib.org/flint-2.5.2.tar.gz
RUN tar xvf flint-2.5.2.tar.gz

WORKDIR /inst/flint-2.5.2
RUN ./configure
RUN make -j
RUN make install
RUN ldconfig

WORKDIR /inst
RUN git clone https://github.com/5GenCrypto/cryfsm.git

WORKDIR /inst/cryfsm
RUN cabal update
# might be slightly wrong dependencies -- but improves sharing between this
# Dockerfile and others we are shipping
RUN cabal install --constraint "cryptol==2.3.0" --constraint "sbv installed" --only-dependencies
RUN git checkout 2b494c7
RUN cabal install --constraint "cryptol==2.3.0" --constraint "sbv installed"

WORKDIR /inst
RUN git clone https://github.com/5GenCrypto/libaesrand.git

WORKDIR /inst/libaesrand
RUN git checkout 7243a4f
RUN autoreconf -i
RUN ./configure
RUN make -j
RUN make install

WORKDIR /inst
RUN git clone https://github.com/5GenCrypto/clt13.git

WORKDIR /inst/clt13
RUN git checkout 3377495
RUN mkdir -p build/autoconf
RUN autoreconf -i
RUN ./configure
RUN make -j
RUN make install

WORKDIR /inst
RUN git clone https://github.com/5GenCrypto/gghlite-flint.git

WORKDIR /inst/gghlite-flint
RUN git checkout c880e89
RUN autoreconf -i
RUN ./configure
RUN make -j
RUN make install

WORKDIR /inst
RUN git clone https://github.com/5GenCrypto/libmmap.git

WORKDIR /inst/libmmap
RUN git checkout 4214fed
RUN autoreconf -i
RUN ./configure
RUN make -j
RUN make install

WORKDIR /inst
RUN git clone https://github.com/5GenCrypto/mife.git

WORKDIR /inst/mife
RUN git checkout 336f0eb
RUN git submodule init
RUN git submodule update
RUN mkdir -p build/autoconf
RUN autoreconf -i
RUN ./configure
RUN make -j
RUN make install

ENV LD_LIBRARY_PATH /usr/local/lib
ENV PATH /root/.cabal/bin:$PATH

COPY             ore-utils /usr/local/bin/
COPY generate-private-data /usr/local/bin/
COPY            keygen-all /usr/local/bin/
COPY           encrypt-all /usr/local/bin/
COPY              eval-all /usr/local/bin/
