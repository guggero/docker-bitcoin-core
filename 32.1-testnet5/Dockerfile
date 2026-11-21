# Bitcoin Core with testnet5 (BIP 95) support, built from the not yet merged
# pull request https://github.com/bitcoin/bitcoin/pull/35861.
#
# NOTE: In contrast to the release images in this repository, this image does
# NOT download and verify official release binaries, because none exist yet.
# Instead it packages binaries that were built on the host and copied to the
# ./bin directory, see README.md in this directory for the build steps. This is
# a temporary shortcut and the image is only meant for testing.
#
# The binaries are dynamically linked against the libraries of the build host,
# so the base image must match the distribution the binaries were built on.
FROM ubuntu:26.04

ARG UID=101
ARG GID=101

# The runtime libraries the host built binaries are linked against: ZeroMQ for
# the notification interface and SQLite for descriptor wallets.
RUN groupadd --gid ${GID} bitcoin \
  && useradd --create-home --no-log-init -u ${UID} -g ${GID} bitcoin \
  && apt-get update -y \
  && apt-get install -y --no-install-recommends gosu libsqlite3-0 libzmq5 \
  && apt-get clean \
  && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# The version the pull request branch reports and the commit the binaries were
# built from.
ENV BITCOIN_VERSION=32.1
ENV BITCOIN_COMMIT=ee505609676a
ENV BITCOIN_DATA=/home/bitcoin/.bitcoin
ENV PATH=/opt/bitcoin-${BITCOIN_VERSION}/bin:$PATH

COPY bin/bitcoin bin/bitcoind bin/bitcoin-cli bin/bitcoin-tx \
  /opt/bitcoin-${BITCOIN_VERSION}/bin/

COPY docker-entrypoint.sh /entrypoint.sh

VOLUME ["/home/bitcoin/.bitcoin"]

# Testnet5 uses 18352 for RPC and 18353 for P2P, the remaining ports are the
# same as in the release images.
EXPOSE 8332 8333 18332 18333 18352 18353 18443 18444 38333 38332

ENTRYPOINT ["/entrypoint.sh"]

# Make sure the binaries run in this image and know about testnet5.
RUN bitcoind -version | grep "Bitcoin Core daemon version v${BITCOIN_VERSION}" \
  && bitcoind -help | grep -q -- "-testnet5"

CMD ["bitcoind"]
