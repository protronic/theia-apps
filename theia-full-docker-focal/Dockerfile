FROM ubuntu:20.04

ENV DEBIAN_FRONTEND noninteractive

#Common deps
RUN apt-get update && apt-get -y install curl xz-utils wget gpg build-essential

#Install node and yarn
#From: https://github.com/nodejs/docker-node/blob/6b8d86d6ad59e0d1e7a94cec2e909cad137a028f/8/Dockerfile

# gpg keys listed at https://github.com/nodejs/node#release-keys
RUN set -ex \
    && for key in \
    4ED778F539E3634C779C87C6D7062848A1AB005C \
    B9E2F5981AA6E0CD28160D9FF13993A75599653C \
    94AE36675C464D64BAFA68DD7434390BDBE9B9C5 \
    B9AE9905FFD7803F25714661B63B535A4C206CA9 \
    77984A986EBC2AA786BC0F66B01FBB92821C587A \
    71DCFD284A79C3B38668286BC97EC7A07EDE3FC1 \
    FD3A5288F042B6850C66B31F09FE44734EB7990E \
    8FCCA13FEF1D0C2E91008E09770F7A9A5AE15600 \
    C4F0DFFF4E8C1A8236409D08E73BC641CC11F4C8 \
    DD8F2338BAE7501E3DD5AC78C273792F7D83545D \
    A48C2BEE680E841632CD4E44F07496B3EB3C1762 \
    ; do \
    gpg --batch --keyserver ipv4.pool.sks-keyservers.net --recv-keys "$key" || \
    gpg --batch --keyserver pool.sks-keyservers.net --recv-keys "$key" || \
    gpg --batch --keyserver pgp.mit.edu --recv-keys "$key" || \
    gpg --batch --keyserver keyserver.pgp.com --recv-keys "$key" || \
    gpg --batch --keyserver ha.pool.sks-keyservers.net --recv-keys "$key" ; \
    done

ARG NODE_VERSION=10.15.3
ENV NODE_VERSION $NODE_VERSION

RUN ARCH= && dpkgArch="$(dpkg --print-architecture)" \
    && case "${dpkgArch##*-}" in \
    amd64) ARCH='x64';; \
    ppc64el) ARCH='ppc64le';; \
    s390x) ARCH='s390x';; \
    arm64) ARCH='arm64';; \
    armhf) ARCH='armv7l';; \
    i386) ARCH='x86';; \
    *) echo "unsupported architecture"; exit 1 ;; \
    esac \
    && curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/node-v$NODE_VERSION-linux-$ARCH.tar.xz" \
    && curl -SLO --compressed "https://nodejs.org/dist/v$NODE_VERSION/SHASUMS256.txt.asc" \
    && gpg --batch --decrypt --output SHASUMS256.txt SHASUMS256.txt.asc \
    && grep " node-v$NODE_VERSION-linux-$ARCH.tar.xz\$" SHASUMS256.txt | sha256sum -c - \
    && tar -xJf "node-v$NODE_VERSION-linux-$ARCH.tar.xz" -C /usr/local --strip-components=1 --no-same-owner \
    && rm "node-v$NODE_VERSION-linux-$ARCH.tar.xz" SHASUMS256.txt.asc SHASUMS256.txt \
    && ln -s /usr/local/bin/node /usr/local/bin/nodejs

ENV YARN_VERSION 1.13.0

RUN set -ex \
    && for key in \
    6A010C5166006599AA17F08146C2130DFD2497F5 \
    ; do \
    gpg --batch --keyserver ipv4.pool.sks-keyservers.net --recv-keys "$key" || \
    gpg --batch --keyserver pool.sks-keyservers.net --recv-keys "$key" || \
    gpg --batch --keyserver pgp.mit.edu --recv-keys "$key" || \
    gpg --batch --keyserver keyserver.pgp.com --recv-keys "$key" || \
    gpg --batch --keyserver ha.pool.sks-keyservers.net --recv-keys "$key" ; \
    done \
    && curl -fSLO --compressed "https://yarnpkg.com/downloads/$YARN_VERSION/yarn-v$YARN_VERSION.tar.gz" \
    && curl -fSLO --compressed "https://yarnpkg.com/downloads/$YARN_VERSION/yarn-v$YARN_VERSION.tar.gz.asc" \
    && gpg --batch --verify yarn-v$YARN_VERSION.tar.gz.asc yarn-v$YARN_VERSION.tar.gz \
    && mkdir -p /opt/yarn \
    && tar -xzf yarn-v$YARN_VERSION.tar.gz -C /opt/yarn --strip-components=1 \
    && ln -s /opt/yarn/bin/yarn /usr/local/bin/yarn \
    && ln -s /opt/yarn/bin/yarn /usr/local/bin/yarnpkg \
    && rm yarn-v$YARN_VERSION.tar.gz.asc yarn-v$YARN_VERSION.tar.gz

#Developer tools

## Git and sudo (sudo needed for user override)
RUN \
  apt-get update && \
  apt-get -y dist-upgrade && \
  apt-get install -y software-properties-common && \
  add-apt-repository universe && \
  add-apt-repository multiverse && \
  apt-get update && \
  apt-get -y install git sudo

#LSPs

#GO
ENV GO_VERSION 1.14.2
ENV GOPATH=/usr/local/go-packages
ENV GO_ROOT=/usr/local/go
ENV PATH $PATH:/usr/local/go/bin
ENV PATH $PATH:${GOPATH}/bin

RUN curl -sS https://storage.googleapis.com/golang/go$GO_VERSION.linux-amd64.tar.gz | tar -C /usr/local -xzf -

RUN go get -u -v github.com/mdempsky/gocode && \
    go get -u -v github.com/uudashr/gopkgs/cmd/gopkgs && \
    go get -u -v github.com/ramya-rao-a/go-outline && \
    go get -u -v github.com/acroca/go-symbols && \
    go get -u -v golang.org/x/tools/cmd/guru && \
    go get -u -v golang.org/x/tools/cmd/gorename && \
    go get -u -v github.com/fatih/gomodifytags && \
    go get -u -v github.com/haya14busa/goplay/cmd/goplay && \
    go get -u -v github.com/josharian/impl && \
    go get -u -v github.com/tylerb/gotype-live && \
    go get -u -v github.com/rogpeppe/godef && \
    go get -u -v github.com/zmb3/gogetdoc && \
    go get -u -v golang.org/x/tools/cmd/goimports && \
    go get -u -v github.com/sqs/goreturns && \
    go get -u -v winterdrache.de/goformat/goformat && \
    go get -u -v golang.org/x/lint/golint && \
    go get -u -v github.com/cweill/gotests/... && \
    go get -u -v github.com/alecthomas/gometalinter && \
    go get -u -v honnef.co/go/tools/... && \
    go get -u -v github.com/mgechev/revive && \
    go get -u -v github.com/sourcegraph/go-langserver && \
    go get -u -v github.com/go-delve/delve/cmd/dlv && \
    go get -u -v github.com/davidrjenni/reftools/cmd/fillstruct && \
    go get -u -v github.com/godoctor/godoctor

ENV GO111MODULE=on
RUN go get -u -v  github.com/golangci/golangci-lint/cmd/golangci-lint@v1.26.0

#Java
ENV JAVA_VERSION jdk-14.0.1+7

RUN set -eux; \
    ARCH="$(dpkg --print-architecture)"; \
    case "${ARCH}" in \
       aarch64|arm64) \
         ESUM='81ca31ad90f9bd789a2ca1753d6d83d10f4927876b4a4b9f4b1c4c8cbce85feb'; \
         BINARY_URL='https://github.com/AdoptOpenJDK/openjdk14-binaries/releases/download/jdk-14.0.1%2B7/OpenJDK14U-jdk_aarch64_linux_hotspot_14.0.1_7.tar.gz'; \
         ;; \
       armhf|armv7l) \
         ESUM='458d091756500dc3013737aa182a14752b3d4ffc358d09532201874ffb8cae22'; \
         BINARY_URL='https://github.com/AdoptOpenJDK/openjdk14-binaries/releases/download/jdk-14.0.1%2B7/OpenJDK14U-jdk_arm_linux_hotspot_14.0.1_7.tar.gz'; \
         ;; \
       ppc64el|ppc64le) \
         ESUM='bfdd77112d81256d4e1a859a465dd4dcb670019a5d6cf8260c30e24a0e5947e4'; \
         BINARY_URL='https://github.com/AdoptOpenJDK/openjdk14-binaries/releases/download/jdk-14.0.1%2B7/OpenJDK14U-jdk_ppc64le_linux_hotspot_14.0.1_7.tar.gz'; \
         ;; \
       s390x) \
         ESUM='c13545924e92cb9d495282e95270f299a28d5466f9741c67791f131c38ebbd0c'; \
         BINARY_URL='https://github.com/AdoptOpenJDK/openjdk14-binaries/releases/download/jdk-14.0.1%2B7/OpenJDK14U-jdk_s390x_linux_hotspot_14.0.1_7.tar.gz'; \
         ;; \
       amd64|x86_64) \
         ESUM='9ddf9b35996fbd784a53fff3e0d59920a7d5acf1a82d4c8d70906957ac146cd1'; \
         BINARY_URL='https://github.com/AdoptOpenJDK/openjdk14-binaries/releases/download/jdk-14.0.1%2B7/OpenJDK14U-jdk_x64_linux_hotspot_14.0.1_7.tar.gz'; \
         ;; \
       *) \
         echo "Unsupported arch: ${ARCH}"; \
         exit 1; \
         ;; \
    esac; \
    curl -LfsSo /tmp/openjdk.tar.gz ${BINARY_URL}; \
    echo "${ESUM} */tmp/openjdk.tar.gz" | sha256sum -c -; \
    mkdir -p /opt/java/openjdk; \
    cd /opt/java/openjdk; \
    tar -xf /tmp/openjdk.tar.gz --strip-components=1; \
    rm -rf /tmp/openjdk.tar.gz;

ENV JAVA_HOME=/opt/java/openjdk \
    PATH="/opt/java/openjdk/bin:$PATH"
RUN apt-get update && apt-get -y install maven gradle


#C/C++
# public LLVM PPA, development version of LLVM
RUN wget -O - https://apt.llvm.org/llvm-snapshot.gpg.key | apt-key add - && \
    echo "deb http://apt.llvm.org/focal/ llvm-toolchain-focal main" > /etc/apt/sources.list.d/llvm.list && \
    apt-get update && apt-get install -y clang-tools-11 clangd-11 clang-tidy-11 && \
    ln -s /usr/bin/clangd-11 /usr/bin/clangd && \
    ln -s /usr/bin/clang-tidy-11 /usr/bin/clang-tidy

# Install latest stable CMake
RUN wget "https://cmake.org/files/v3.17/cmake-3.17.0-Linux-x86_64.sh" && \
    chmod a+x cmake-3.17.0-Linux-x86_64.sh && \
    ./cmake-3.17.0-Linux-x86_64.sh --prefix=/usr/ --skip-license && \
    rm cmake-3.17.0-Linux-x86_64.sh

#Python 2-3
RUN apt-get update \
    && apt-get install -y python \
    && curl https://bootstrap.pypa.io/get-pip.py | python \
    && apt-get install -y python3-dev python3-pip \
    && pip install --upgrade pip --user \
    && pip3 install --upgrade pip --user \
    && pip3 install python-language-server flake8 autopep8 ;

#PHP
RUN apt-get -y install php curl php-cli php-mbstring unzip

# https://getcomposer.org/doc/faqs/how-to-install-composer-programmatically.md
# https://linuxconfig.org/how-to-install-php-composer-on-debian-linux
RUN curl -s -o composer-setup.php https://getcomposer.org/installer \
    && php composer-setup.php --install-dir=/usr/local/bin --filename=composer \
    && rm composer-setup.php

#Ruby
RUN apt-get -y install ruby ruby-dev zlib1g-dev && \
    gem install solargraph


# Rust
ENV RUSTUP_HOME=/usr/local/rustup \
    CARGO_HOME=/usr/local/cargo \
    PATH=/usr/local/cargo/bin:$PATH

ENV USER root

RUN curl -fsSL https://sh.rustup.rs | sh -s -- -y \
    && rustup update \
    && rustup component add rls rust-src rust-analysis

# Embedded Rust
RUN npm install --global xpm@latest
RUN xpm install --global @xpack-dev-tools/arm-none-eabi-gcc@latest
RUN apt-get -y install pkg-config openssl libssl-dev
RUN cargo install cargo-generate
RUN cargo install cargo-clone
RUN rustup target add thumbv6m-none-eabi thumbv7m-none-eabi thumbv7em-none-eabi thumbv7em-none-eabihf

# Angular
RUN npm install -g @angular/cli

## User account
RUN adduser --disabled-password --gecos '' theia && \
    adduser theia sudo && \
    echo '%sudo ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers;

RUN chmod g+rw /home && \
    mkdir -p /home/project && \
    mkdir -p /usr/local/cargo && \
    mkdir -p /usr/local/go && \
    mkdir -p /usr/local/go-packages && \
    chown -R theia:theia /usr/local/cargo && \
    chown -R theia:theia /usr/local/go && \
    chown -R theia:theia /usr/local/go-packages && \
    chown -R theia:theia /home/theia && \
    chown -R theia:theia /home/project;

#Theia
##Needed for node-gyp, nsfw build
RUN apt-get update && apt-get install -y python build-essential

USER theia
ARG version=latest
WORKDIR /home/theia
ADD $version.package.json ./package.json
ARG GITHUB_TOKEN
# using "NODE_OPTIONS=..." to avoid out-of-memory problem in CI
RUN yarn --cache-folder ./ycache && rm -rf ./ycache && \
    NODE_OPTIONS="--max_old_space_size=4096" yarn theia build ;\
    yarn theia download:plugins
EXPOSE 3000
ENV SHELL=/bin/bash \
    THEIA_DEFAULT_PLUGINS=local-dir:/home/theia/plugins

ENV GOPATH=/home/project
ENV PATH=$PATH:$GOPATH/bin
ENV LC_ALL=C.UTF-8
ENV LANG=C.UTF-8

ENTRYPOINT [ "yarn", "theia", "start", "/home/project", "--hostname=0.0.0.0" ]