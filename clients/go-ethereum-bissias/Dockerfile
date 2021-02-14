# Docker container spec for building the master branch of go-ethereum.
#
# The build process it potentially longer running but every effort was made to
# produce a very minimalistic container that can be reused many times without
# needing to constantly rebuild.

#### BASED ON ETHEREUM DOCKER FILE ####
FROM golang:1.15-alpine as builder
ARG branch=master

RUN apk add --no-cache git make gcc musl-dev linux-headers git

RUN git clone https://github.com/bissias/go-ethereum /go-ethereum
RUN cd /go-ethereum && git checkout $branch
RUN cd /go-ethereum && make geth

# Pull Geth into a second stage deploy alpine container
FROM alpine:latest

RUN apk add --no-cache ca-certificates

COPY --from=builder /go-ethereum/build/bin/geth /usr/local/bin/
# If you want to maintain access to /go-ethereum, comment the line above 
# and uncomment the line below
#RUN cp /go-ethereum/build/bin/geth /usr/local/bin/

EXPOSE 8545 8546 30303 30303/udp
########################################

RUN apk add --update bash curl jq

# Build go-ethereum on the fly and delete all build tools afterwards
# RUN \
#   apk add --update bash curl jq go git make gcc musl-dev              \
#         ca-certificates linux-headers                           && \
#   git clone --depth 1 --branch $branch  https://github.com/ethereum/go-ethereum && \
#   (cd go-ethereum && make geth)                               && \
#   (cd go-ethereum                                             && \
#   echo "{}"                                                      \
#   | jq ".+ {\"repo\":\"$(git config --get remote.origin.url)\"}" \
#   | jq ".+ {\"branch\":\"$(git rev-parse --abbrev-ref HEAD)\"}"  \
#   | jq ".+ {\"commit\":\"$(git rev-parse HEAD)\"}"               \
#   > /version.json)                                            && \
#   cp go-ethereum/build/bin/geth /geth                         && \
#   apk del go git make gcc musl-dev linux-headers              && \
#   rm -rf /go-ethereum && rm -rf /var/cache/apk/*

RUN /usr/local/bin/geth console --exec 'console.log(admin.nodeInfo.name)' --maxpeers=0 --nodiscover --dev 2>/dev/null | head -1 > /version.txt

# Inject the startup script
ADD geth.sh /geth.sh
ADD mapper.jq /mapper.jq
RUN chmod +x /geth.sh

# Inject the enode id retriever script
ADD enode.sh /enode.sh
RUN chmod +x /enode.sh

ADD genesis.json /genesis.json

# Export the usual networking ports to allow outside access to the node
EXPOSE 8545 8546 8547 30303 30303/udp

# Generate the ethash verification caches
RUN \
 /usr/local/bin/geth makecache     1 ~/.ethereum/geth/ethash && \
 /usr/local/bin/geth makecache 30001 ~/.ethereum/geth/ethash

ENTRYPOINT ["/geth.sh"]
