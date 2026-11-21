# Bitcoin Inquisition for Mutinynet

An image for running a node on [Mutinynet](https://mutinynet.com), a custom
signet with 30 second blocks. It packages binaries built from
[benthecarman/bitcoin](https://github.com/benthecarman/bitcoin) at commit
`2fda7bfc027e` (release `mutinynet-inq-template-hash`), which is Bitcoin
Inquisition 29.2 plus `OP_TEMPLATEHASH` and the `-signetblocktime` option.

Unlike the release images in this repository, the binaries are **not**
downloaded and signature checked inside of the container. They are built on the
host and copied into `./bin`, which is not checked in. Do not use this image
for anything but testing.

## Building the binaries

The dependencies are built with Bitcoin Core's `depends` system and linked
statically. Only the C and C++ runtime libraries are linked dynamically, so the
binaries must be built on the distribution the `Dockerfile` uses as its base
image (currently Ubuntu 26.04).

```shell
cd ../../bitcoin
git remote add benthecarman https://github.com/benthecarman/bitcoin.git
git fetch benthecarman --tags
git worktree add --detach ../bitcoin-mutinynet 2fda7bfc027e

cd ../bitcoin-mutinynet
make -C depends -j"$(nproc)" NO_QT=1 NO_USDT=1 NO_BDB=1
cmake -B build-docker \
  --toolchain depends/x86_64-pc-linux-gnu/toolchain.cmake \
  -DBUILD_TESTS=OFF -DBUILD_BENCH=OFF -DBUILD_FUZZ_BINARY=OFF \
  -DBUILD_TX=ON -DBUILD_UTIL=ON -DBUILD_WALLET_TOOL=ON
cmake --build build-docker -j"$(nproc)"

cd ../docker-bitcoin-core/29.2-mutinynet
mkdir -p bin
cp ../../bitcoin-mutinynet/build-docker/bin/{bitcoind,bitcoin-cli,bitcoin-tx,bitcoin-util,bitcoin-wallet} bin/
strip bin/*
```

Update `BITCOIN_COMMIT` in the `Dockerfile` when building from a newer commit.

## Building the image

```shell
docker build -t guggero/bitcoin-core:29.2-mutinynet .
```

## Running

Mutinynet is a custom signet, so the challenge, the block time and a peer must
be given explicitly. It uses the default signet ports, `38333` for P2P and
`38332` for RPC.

```shell
docker run --rm -it --name bitcoind-mutinynet \
  -v bitcoin-mutinynet:/home/bitcoin/.bitcoin \
  -p 38332:38332 -p 38333:38333 \
  guggero/bitcoin-core:29.2-mutinynet \
  -signet -printtoconsole \
  -signetchallenge=512102f7561d208dd9ae99bf497273e16f389bdbd6c4742ddb8e6b216e64fa2928ad8f51ae \
  -signetblocktime=30 \
  -addnode=45.79.52.207:38333 \
  -rpcbind=0.0.0.0 -rpcallowip=0.0.0.0/0 \
  -rpcuser=lightning -rpcpassword=lightning
```
