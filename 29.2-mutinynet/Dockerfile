# Bitcoin Inquisition 29.2 with the Mutinynet patch (configurable signet block
# time), built from https://github.com/benthecarman/bitcoin at the commit the
# deployed Mutinynet runs.
#
# NOTE: In contrast to the release images in this repository, this image does
# NOT download and verify official release binaries. Instead it packages
# binaries that were built on the host and copied to the ./bin directory, see
# README.md in this directory for the build steps. The image is only meant for
# testing.
#
# The binaries are statically linked against everything but the C and C++
# runtime libraries of the build host, so the base image must match the
# distribution the binaries were built on.
FROM ubuntu:26.04

ARG UID=101
ARG GID=101

RUN groupadd --gid ${GID} bitcoin \
  && useradd --create-home --no-log-init -u ${UID} -g ${GID} bitcoin \
  && apt-get update -y \
  && apt-get install -y --no-install-recommends gosu \
  && apt-get clean \
  && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# The version the binaries report and the commit they were built from.
ENV BITCOIN_VERSION=29.2
ENV BITCOIN_COMMIT=2fda7bfc027e
ENV BITCOIN_DATA=/home/bitcoin/.bitcoin
ENV PATH=/opt/bitcoin-${BITCOIN_VERSION}/bin:$PATH

COPY bin/bitcoind bin/bitcoin-cli bin/bitcoin-tx bin/bitcoin-util \
  bin/bitcoin-wallet /opt/bitcoin-${BITCOIN_VERSION}/bin/

COPY docker-entrypoint.sh /entrypoint.sh

VOLUME ["/home/bitcoin/.bitcoin"]

EXPOSE 8332 8333 18332 18333 18443 18444 38333 38332

ENTRYPOINT ["/entrypoint.sh"]

# Make sure the binaries run in this image and carry the Mutinynet patch.
RUN bitcoind -version \
  | grep "Bitcoin Inquisition daemon version v${BITCOIN_VERSION}" \
  && bitcoind -help | grep -q -- "-signetblocktime"

CMD ["bitcoind"]
