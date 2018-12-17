# Docker container spec for building the stable branch of go-ethereum.
#
# The build process it potentially longer running but every effort was made to
# produce a very minimalistic container that can be reused many times without
# needing to constantly rebuild.
FROM ubuntu:16.04

# Build go-ethereum on the fly and delete all build tools afterwards
RUN \
  apt-get update                                              && \
  apt-get install -y bash jq go git make gcc musl-dev maven      \
  	  ca-certificates linux-headers openjdk-8-jdk flex opam      \
      pkg-config libmpfr-dev autoconf libtool pandoc             \
      zlib1g-dev                                              && \
  git clone --depth 1                                            \
  https://github.com/SomniaStellarum/go-ethereum              && \
  (cd go-ethereum && make geth)                               && \
  (cd go-ethereum                                             && \
  echo "{}"                                                      \
  | jq ".+ {\"repo\":\"$(git config --get remote.origin.url)\"}" \
  | jq ".+ {\"branch\":\"$(git rev-parse --abbrev-ref HEAD)\"}"  \
  | jq ".+ {\"commit\":\"$(git rev-parse HEAD)\"}"               \
	> /version.json)                                            && \
  cp go-ethereum/build/bin/geth /geth                         && \
  apt-get install -y python3                                  && \
  mkdir kevm && cd kevm                                       && \
  git clone --depth 1                                            \
  https://github.com/kframework/evm-semantics                 && \
  cd evm-semantics && env && make deps && make                && \
  apt-get remove go git make gcc musl-dev linux-headers

ARG KEVM_PORT
ENV KEVM_PORT ${KEVM_PORT:-8888}

ARG KEVM_HOST
ENV KEVM_HOST ${KEVM_HOST:-0.0.0.0}

ARG KEVM_DEBUG
ENV KEVM_DEBUG ${KEVM_DEBUG:-}

RUN LD_LIBRARY_PATH=./.build/local/lib ./.build/vm/kevm-vm $KEVM_PORT $KEVM_HOST $KEVM_DEBUG

RUN cd ../
RUN rm -rf /go-ethereum

# Inject the startup script
ADD geth.sh /geth.sh
RUN chmod +x /geth.sh

# Export the usual networking ports to allow outside access to the node
EXPOSE 8545 8546 30303 30303/udp

ENTRYPOINT ["/geth.sh"]
