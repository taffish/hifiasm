ARG HIFIASM_VERSION=0.25.0
ARG HIFIASM_INTERNAL_VERSION=0.25.0-r726
ARG HIFIASM_SOURCE_SHA256=51633138865207a9d41630da9377d46e4921ad4fc5facaa1740ceccae8611f1f
ARG BIOCONDA_RECIPES_COMMIT=7231b605452eb5c582f779e9f935e93f810c6740
ARG HIFIASM_AARCH64_PATCH_SHA256=5ca39318bd76d24d3b82ba655f5cc029ecc4fb24c80a471e30ac4985c32fa82d
ARG DEBIAN_MIRROR=http://deb.debian.org/debian
ARG DEBIAN_SECURITY_MIRROR=http://deb.debian.org/debian-security

FROM debian:12-slim AS builder

ARG HIFIASM_VERSION
ARG HIFIASM_INTERNAL_VERSION
ARG HIFIASM_SOURCE_SHA256
ARG BIOCONDA_RECIPES_COMMIT
ARG HIFIASM_AARCH64_PATCH_SHA256
ARG DEBIAN_MIRROR
ARG DEBIAN_SECURITY_MIRROR

ENV DEBIAN_FRONTEND=noninteractive

RUN set -eux; \
    sed -i \
      -e "s|http://deb.debian.org/debian-security|${DEBIAN_SECURITY_MIRROR}|g" \
      -e "s|http://deb.debian.org/debian|${DEBIAN_MIRROR}|g" \
      /etc/apt/sources.list.d/debian.sources; \
    for attempt in 1 2 3 4 5 6; do \
      if apt-get -o Acquire::Retries=3 update \
       && apt-get -o Acquire::Retries=3 install -y --no-install-recommends \
            build-essential \
            ca-certificates \
            curl \
            patch \
            zlib1g-dev; then \
        break; \
      fi; \
      if [ "$attempt" = "6" ]; then exit 1; fi; \
      apt-get clean; \
      rm -rf /var/lib/apt/lists/*; \
      sleep 5; \
    done; \
    rm -rf /var/lib/apt/lists/*

WORKDIR /tmp/build/hifiasm

RUN set -eux; \
    curl --retry 5 --retry-delay 2 --retry-all-errors -fsSL \
      "https://github.com/chhylp123/hifiasm/archive/refs/tags/${HIFIASM_VERSION}.tar.gz" \
      -o hifiasm.tar.gz; \
    echo "${HIFIASM_SOURCE_SHA256}  hifiasm.tar.gz" | sha256sum -c -; \
    tar -xzf hifiasm.tar.gz --strip-components=1; \
    curl --retry 5 --retry-delay 2 --retry-all-errors -fsSL \
      "https://raw.githubusercontent.com/bioconda/bioconda-recipes/${BIOCONDA_RECIPES_COMMIT}/recipes/hifiasm/linux-aarch64-support.patch" \
      -o linux-aarch64-support.patch; \
    echo "${HIFIASM_AARCH64_PATCH_SHA256}  linux-aarch64-support.patch" | sha256sum -c -; \
    patch -p1 < linux-aarch64-support.patch; \
    arch="$(uname -m)"; \
    case "${arch}" in \
      x86_64|amd64) arch_opts="-msse4.2 -mpopcnt" ;; \
      aarch64|arm64) arch_opts="" ;; \
      *) echo "Unsupported build architecture: ${arch}" >&2; exit 1 ;; \
    esac; \
    sed -i.bak 's/^CXXFLAGS=.*//' Makefile; \
    make -j"$(nproc)" \
      CXXFLAGS="-g -O3 ${arch_opts} -fomit-frame-pointer -Wall"; \
    mkdir -p /opt/hifiasm/bin /opt/hifiasm/share/doc/hifiasm /opt/hifiasm/share/man/man1; \
    install -m 0755 hifiasm /opt/hifiasm/bin/hifiasm; \
    install -m 0644 README.md LICENSE /opt/hifiasm/share/doc/hifiasm/; \
    install -m 0644 hifiasm.1 /opt/hifiasm/share/man/man1/hifiasm.1; \
    /opt/hifiasm/bin/hifiasm 2>&1 | grep -F "Usage: hifiasm" >/dev/null; \
    /opt/hifiasm/bin/hifiasm --version | grep -Fx "${HIFIASM_INTERNAL_VERSION}" >/dev/null

FROM debian:12-slim

ARG HIFIASM_VERSION
ARG HIFIASM_INTERNAL_VERSION
ARG DEBIAN_MIRROR
ARG DEBIAN_SECURITY_MIRROR

ENV DEBIAN_FRONTEND=noninteractive

RUN set -eux; \
    sed -i \
      -e "s|http://deb.debian.org/debian-security|${DEBIAN_SECURITY_MIRROR}|g" \
      -e "s|http://deb.debian.org/debian|${DEBIAN_MIRROR}|g" \
      /etc/apt/sources.list.d/debian.sources; \
    for attempt in 1 2 3 4 5 6; do \
      if apt-get -o Acquire::Retries=3 update \
       && apt-get -o Acquire::Retries=3 install -y --no-install-recommends \
            bash \
            libgcc-s1 \
            libstdc++6 \
            zlib1g; then \
        break; \
      fi; \
      if [ "$attempt" = "6" ]; then exit 1; fi; \
      apt-get clean; \
      rm -rf /var/lib/apt/lists/*; \
      sleep 5; \
    done; \
    rm -rf /var/lib/apt/lists/*

COPY --from=builder /opt/hifiasm/bin/hifiasm /usr/local/bin/hifiasm
COPY --from=builder /opt/hifiasm/share/doc/hifiasm/ /usr/local/share/doc/hifiasm/
COPY --from=builder /opt/hifiasm/share/man/man1/hifiasm.1 /usr/local/share/man/man1/hifiasm.1

RUN set -eux; \
    hifiasm 2>&1 | grep -F "Usage: hifiasm" >/dev/null; \
    hifiasm --version | grep -Fx "${HIFIASM_INTERNAL_VERSION}" >/dev/null; \
    ldd "$(command -v hifiasm)" | grep -F "libz" >/dev/null

RUN set -eux; \
    mkdir -p /tmp/hifiasm-check; \
    cd /tmp/hifiasm-check; \
    printf '@r1\nACGTACGTACGTACGTACGTACGTACGTACGTACGTACGTACGTACGTACGTACGTACGTACGT\n+\nFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF\n@r2\nCGTACGTACGTACGTACGTACGTACGTACGTACGTACGTACGTACGTACGTACGTACGTACGTA\n+\nFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF\n@r3\nGTACGTACGTACGTACGTACGTACGTACGTACGTACGTACGTACGTACGTACGTACGTACGTAC\n+\nFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF\n' > reads.fq; \
    hifiasm -o tiny -t1 -f0 reads.fq > tiny.out 2> tiny.log; \
    grep -F "[M::ha_analyze_count]" tiny.log >/dev/null; \
    test -s tiny.log; \
    cd /; \
    rm -rf /tmp/hifiasm-check

WORKDIR /root

ENV TAFFISH_ENV=TAFFISH
ENV TAFFISH_NAME=hifiasm
