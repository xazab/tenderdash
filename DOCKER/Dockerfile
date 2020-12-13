FROM golang:1.15-alpine AS build

RUN apk update && \
    apk upgrade && \
    apk add bash git gmp-dev sudo cmake build-base

COPY . /go/src/github.com/xazab/tenderxazab

WORKDIR /go/src/github.com/xazab/tenderxazab

RUN make install-bls
RUN make build-linux

FROM alpine:3.9
LABEL maintainer="developers@xazab.xyz"

# Tenderxazab will be looking for the genesis file in /tenderxazab/config/genesis.json
# (unless you change `genesis_file` in config.toml). You can put your config.toml and
# private validator file into /tenderxazab/config.
#
# The /tenderxazab/data dir is used by tenderxazab to store state.
ENV TMHOME /tenderxazab

# OS environment setup
# Set user right away for determinism, create directory for persistence and give our user ownership
# jq and curl used for extracting `pub_key` from private validator while
# deploying tenderxazab with Kubernetes. It is nice to have bash so the users
# could execute bash commands.
RUN apk update && \
    apk upgrade && \
    apk --no-cache add curl jq bash gmp libstdc++ && \
    addgroup tmuser && \
    adduser -S -G tmuser tmuser -h "$TMHOME"

# Run the container with tmuser by default. (UID=100, GID=1000)
USER tmuser

WORKDIR $TMHOME

# p2p, rpc and prometheus port
EXPOSE 26656 26657 26660

STOPSIGNAL SIGTERM

COPY --from=build /go/src/github.com/xazab/tenderxazab/build/tenderxazab /usr/bin/tenderxazab

# You can overwrite these before the first run to influence
# config.json and genesis.json. Additionally, you can override
# CMD to add parameters to `tenderxazab node`.
ENV PROXY_APP=kvstore MONIKER=dockernode CHAIN_ID=dockerchain

COPY ./DOCKER/docker-entrypoint.sh /usr/local/bin/

ENTRYPOINT ["docker-entrypoint.sh"]
CMD ["node"]

# Expose the data directory as a volume since there's mutable state in there
VOLUME [ "$TMHOME" ]
