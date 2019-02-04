# Docker container spec for building the stable branch of go-ethereum.
#
# The build process it potentially longer running but every effort was made to
# produce a very minimalistic container that can be reused many times without
# needing to constantly rebuild.
FROM alpine:latest


# Build go-ethereum on the fly and delete all build tools afterwards
RUN apk add --update bash jq go git make gcc musl-dev              \
  	  ca-certificates linux-headers maven openjdk8 flex            \
      opam mpfr-dev autoconf libtool zlib-dev python3

RUN wget https://github.com/jgm/pandoc/releases/download/2.5/pandoc-2.5-linux.tar.gz
RUN tar xvzf pandoc-2.5-linux.tar.gz --strip-components 1 -C /usr/local

ENV GOPATH /gopath
ENV PATH   $GOPATH/bin:$PATH

COPY . $GOPATH/src/github.com/ethereum/go-ethereum
RUN cd $GOPATH/src/github.com/ethereum/go-ethereum && make geth
RUN cd $HOME
RUN mkdir kevm
RUN cd kevm
RUN git clone --depth 1                                          \
  https://github.com/kframework/evm-semantics                 && \
  cd evm-semantics && env && make deps && make

ARG KEVM_PORT
ENV KEVM_PORT ${KEVM_PORT:-8888}

ARG KEVM_HOST
ENV KEVM_HOST ${KEVM_HOST:-0.0.0.0}

ARG KEVM_DEBUG
ENV KEVM_DEBUG ${KEVM_DEBUG:-}

RUN LD_LIBRARY_PATH=./.build/local/lib ./.build/vm/kevm-vm $KEVM_PORT $KEVM_HOST $KEVM_DEBUG

RUN cd ../
# Inject the startup script
ADD geth.sh /geth.sh
RUN chmod +x /geth.sh

# Export the usual networking ports to allow outside access to the node
EXPOSE 8545 8546 30303 30303/udp

ENTRYPOINT ["/geth.sh"]
