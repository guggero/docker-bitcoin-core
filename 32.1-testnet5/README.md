# Bitcoin Core with testnet5 (BIP 95)

A temporary image for testing against testnet5 before a Bitcoin Core release
supports it. It packages binaries built from the pull request
[bitcoin/bitcoin#35861](https://github.com/bitcoin/bitcoin/pull/35861).

Unlike the release images in this repository, the binaries are **not**
downloaded and signature checked inside of the container. They are built on the
host and copied into `./bin`, which is not checked in. Do not use this image
for anything but testing.

## Building the binaries

The binaries are dynamically linked, so they must be built on the distribution
the `Dockerfile` uses as its base image (currently Ubuntu 26.04) with the
`libzmq3-dev` and `libsqlite3-dev` packages installed.

```shell
cd ../../bitcoin
git checkout pr/35861
cmake -B build-testnet5-docker -DCMAKE_BUILD_TYPE=Release \
  -DBUILD_TESTS=OFF -DBUILD_BENCH=OFF -DBUILD_FUZZ_BINARY=OFF \
  -DBUILD_GUI=OFF -DENABLE_IPC=OFF -DENABLE_WALLET=ON -DWITH_ZMQ=ON \
  -DBUILD_TX=ON -DBUILD_UTIL=OFF -DBUILD_WALLET_TOOL=OFF
cmake --build build-testnet5-docker -j"$(nproc)"

cd ../docker-bitcoin-core/32.1-testnet5
mkdir -p bin
cp ../../bitcoin/build-testnet5-docker/bin/{bitcoin,bitcoind,bitcoin-cli,bitcoin-tx} bin/
strip bin/*
```

Update `BITCOIN_COMMIT` in the `Dockerfile` when building from a newer commit.

## Building the image

```shell
docker build -t guggero/bitcoin-core:32.1-testnet5 .
```

## Running

Testnet5 uses port `18353` for P2P and `18352` for RPC. There are no DNS seeds
yet, so a peer must be added manually.

```shell
docker run --rm -it --name bitcoind-testnet5 \
  -v bitcoin-testnet5:/home/bitcoin/.bitcoin \
  -p 18352:18352 -p 18353:18353 \
  guggero/bitcoin-core:32.1-testnet5 \
  -testnet5 -printtoconsole \
  -rpcbind=0.0.0.0 -rpcallowip=0.0.0.0/0 \
  -rpcuser=lightning -rpcpassword=lightning \
  -addnode=<host>:18353
```
