# We need to build in a Linux environment to support C libraries, e.g. RocksDB.
# We use Debian instead of Alpine, so that we can use binary database packages
# instead of spending time compiling them.
FROM golang:1.15

RUN apt-get -qq update -y && apt-get -qq upgrade -y >/dev/null
RUN apt-get -qq install -y cmake sudo libgmp-dev libleveldb-dev librocksdb-dev >/dev/null

# Set up build directory /src/tenderxazab
ENV TENDERMINT_BUILD_OPTIONS badgerdb,boltdb,cleveldb,rocksdb
WORKDIR /src/tenderxazab

# Fetch dependencies separately (for layer caching)
COPY go.mod go.sum ./
RUN go mod download

# Copy Tenderxazab source
COPY . .
# Install BLS library
RUN make install-bls
# Build Tenderxazab and install into /usr/bin/tenderxazab
RUN make build && cp build/tenderxazab /usr/bin/tenderxazab
COPY test/e2e/docker/entrypoint* /usr/bin/
RUN cd test/e2e && make maverick && cp build/maverick /usr/bin/maverick
RUN cd test/e2e && make app && cp build/app /usr/bin/app

# Set up runtime directory. We don't use a separate runtime image since we need
# e.g. leveldb and rocksdb which are already installed in the build image.
WORKDIR /tenderxazab
VOLUME /tenderxazab
ENV TMHOME=/tenderxazab

EXPOSE 26656 26657 26660
ENTRYPOINT ["/usr/bin/entrypoint"]
CMD ["node"]
STOPSIGNAL SIGTERM
