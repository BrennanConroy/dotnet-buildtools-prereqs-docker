FROM arm64v8/alpine:3.15
# Install .NET and test dependencies
RUN apk update && apk add --no-cache \
    bash \
    coreutils \
    curl \
    icu-data \
    icu-libs \
    iputils \
    krb5-libs \
    lldb \
    llvm \
    lttng-ust \
    musl-locales \
    openssl \
    python3 \
    sudo \
    tzdata

RUN apk update && apk add --no-cache && \
    apk add \
        cargo \
        libffi-dev \
        linux-headers \
        python3-dev \
        openssl-dev && \
    ln -sf /usr/bin/python3 /usr/bin/python && \
    curl https://bootstrap.pypa.io/get-pip.py -o ./get-pip.py --fail --silent --show-error && \
    python3 ./get-pip.py && rm ./get-pip.py && \
    python3 -m pip install --upgrade pip==22.2.2 && \
    python3 -m pip install virtualenv==20.16.5 && \
    pip download --no-deps helix-scripts --index-url https://dnceng.pkgs.visualstudio.com/public/_packaging/helix-client-prod/pypi/simple && \
    pip install ./helix_scripts-*-py3-none-any.whl && \
    apk del \
        cargo \
        libffi-dev \
        linux-headers \
        python3-dev \
        openssl-dev

# build MsQuic as we don't have Alpine packages (yet)
RUN apk update && apk add --no-cache && \
    apk add \
        cmake \
        g++ \
        gcc \
        git \
        numactl-dev \
        linux-headers \
        lttng-ust-dev \
        make \
        musl-dev \
        openssl-dev \
        perl && \
    cd /tmp && \
    git clone --depth 1 --single-branch --branch main --recursive https://github.com/dotnet/msquic && \
    cd msquic/src/msquic && \
    cmake -B build/linux/arm64_openssl \
       -DQUIC_OUTPUT_DIR=/tmp/msquic/src/msquic/artifacts/bin/linux/arm64_Release_openssl \
       -DCMAKE_BUILD_TYPE=Release \
       -DCMAKE_TARGET_ARCHITECTURE=arm64 \
       -DQUIC_TLS=openssl \
       -DQUIC_ENABLE_LOGGING=true \
       -DQUIC_USE_SYSTEM_LIBCRYPTO=true \
       -DQUIC_BUILD_TOOLS=off \
       -DQUIC_BUILD_TEST=off \
       -DQUIC_BUILD_PERF=off && \
    cmake --build build/linux/arm64_openssl  --config Release && \
    cp artifacts/bin/linux/arm64_Release_openssl/libmsquic.so.* artifacts/bin/linux/arm64_Release_openssl/libmsquic.lttng.so.* /usr/lib && \
    rm -rf /tmp/msquic && \
    apk del \
        cmake \
        g++ \
        gcc \
        git \
        linux-headers \
        lttng-ust-dev \
        make \
        musl-dev \
        openssl-dev \
        perl

# Needed for runtime tests to pass
ENV LANG=en-US.UTF-8
RUN echo export LANG=${LANG}  >> /etc/profile.d/locale.sh

# create helixbot user and give rights to sudo without password
# Alpine does not support long options
RUN /usr/sbin/adduser -D -g '' -G adm -s /bin/bash -u 1001 helixbot && \
    /usr/sbin/adduser -D -g '' -G adm -s /bin/bash -u 1000 helixbot2 && \
    chmod 755 /root && \
    echo "helixbot ALL=(ALL)       NOPASSWD: ALL" > /etc/sudoers.d/helixbot &&  \
    echo "helixbot2 ALL=(ALL)      NOPASSWD: ALL" > /etc/sudoers.d/helixbot2

USER helixbot

RUN python3 -m virtualenv /home/helixbot/.vsts-env
