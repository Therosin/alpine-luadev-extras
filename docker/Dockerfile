FROM frolvlad/alpine-gxx

LABEL maintainer="Theros - theros@svaltek.xyz"

# copy LuaSpace Scripts into Container
COPY ./LuaSpace /usr/LuaSpace
RUN chmod 777 -R /usr/LuaSpace

#Install Lua
ENV LUA_VERSION=5.1.5

RUN set -ex \
    \
    && apk add --no-cache --virtual .build-deps \
    ca-certificates \
    openssl \
    make \
    gcc \
    libc-dev \
    \
    && wget -c https://www.lua.org/ftp/lua-${LUA_VERSION}.tar.gz \
    -O - | tar -xzf - \
    \
    && cd lua-${LUA_VERSION} \
    && make -j"$(nproc)" posix \
    && make install \
    && cd .. \
    && rm -rf lua-${LUA_VERSION} \
    \
    && apk del .build-deps

# Install LuaRocks
ENV LUAROCKS_VERSION=3.3.0

RUN set -ex \
    \
    && apk add --no-cache \
    ca-certificates \
    openssl \
    curl \
    \
    && apk add --no-cache --virtual .build-deps \
    make \
    gcc \
    libc-dev \
    \
    && wget https://luarocks.github.io/luarocks/releases/luarocks-${LUAROCKS_VERSION}.tar.gz \
    -O - | tar -xzf - \
    \
    && cd luarocks-${LUAROCKS_VERSION} \
    && ./configure --with-lua=/usr/local \
    && make build \
    && make install \
    && cd .. \
    && rm -rf luarocks-${LUAROCKS_VERSION} \
    \
    && apk del .build-deps \
    # Add Some usefull tools
    && echo Install PreReqs \
    && apk add unzip curl wget tmux \
    && echo Installing Lua Modules \
    && luarocks install moonpick \
    && luarocks install busted

# Install golang
RUN apk add --no-cache git make musl-dev go
# Configure Go
ENV GOROOT /usr/lib/go
ENV GOPATH /go
ENV PATH /go/bin:$PATH

RUN mkdir -p ${GOPATH}/src ${GOPATH}/bin \
# Install Some Usefull golang tools
	&& echo Installing Golang Utils \
    && go get -u github.com/Masterminds/glide/... \
    && go get github.com/yudai/gotty\
    # Compile and install Algernon
    &&echo Installing Algernon \
    && mkdir -p /usr/src \
    && cd /usr/src \
    && git clone https://github.com/xyproto/algernon \
    && cd algernon \
    && go build -mod=vendor \
    && echo Finished....

# This Dockerfile adds a non-root user with sudo access. Use the "remoteUser"
# property in devcontainer.json to use it. On Linux, the container user's GID/UIDs
# will be updated to match your local UID/GID (when using the dockerFile property).
# See https://aka.ms/vscode-remote/containers/non-root-user for details.
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID

# Set to false to skip installing zsh and Oh My ZSH!
ARG INSTALL_ZSH="true"

# Location and expected SHA for common setup script - SHA generated on release
ARG COMMON_SCRIPT_SOURCE="https://raw.githubusercontent.com/microsoft/vscode-dev-containers/master/script-library/common-alpine.sh"
ARG COMMON_SCRIPT_SHA="dev-mode"

# Install git, bash, dependencies, and add a non-root user
RUN apk update \
    && apk add --no-cache wget nano coreutils ca-certificates ruby-full \
	&& echo 'Set disable_coredump false' >> /etc/sudo.conf \
    #
    # Verify git, common tools / libs installed, add/modify non-root user, optionally install zsh
    && wget -q -O /tmp/common-setup.sh $COMMON_SCRIPT_SOURCE \
    && if [ "$COMMON_SCRIPT_SHA" != "dev-mode" ]; then echo "$COMMON_SCRIPT_SHA /tmp/common-setup.sh" | sha256sum -c - ; fi \
    && /bin/ash /tmp/common-setup.sh "$INSTALL_ZSH" "$USERNAME" "$USER_UID" "$USER_GID" \
    && rm /tmp/common-setup.sh \
	# update glibc
	&& apk --no-cache add ca-certificates wget \
	&& wget -q -O /etc/apk/keys/sgerrand.rsa.pub https://alpine-pkgs.sgerrand.com/sgerrand.rsa.pub \
	&& wget https://github.com/sgerrand/alpine-pkg-glibc/releases/download/2.30-r0/glibc-2.30-r0.apk \
	&& apk --no-cache add glibc-2.30-r0.apk \
	&& apk --no-cache add bash build-base curl file git gzip ncurses ruby ruby-dbm ruby-etc ruby-irb ruby-json sudo clang clang-dev ninja \
	&& apk --update add grep
	

RUN echo install and configure antigen... \
	&& echo export PATH=/usr/LuaSpace:$PATH >> /home/vscode/.zshrc \
    && mkdir -p /home/vscode/.antigen \
    && curl -L git.io/antigen > /home/vscode/.antigen/antigen.zsh \
    && echo source /home/vscode/.antigen/antigen.zsh >> /home/vscode/.zshrc \
    && echo antigen init /home/vscode/.antigenrc >> /home/vscode/.zshrc \
	&& echo 'LuaSpace-MOTD' >> /home/vscode/.zshrc \
    && curl -o  /home/vscode/.antigenrc https://gitlab.com/svaltek/DevTools/luaspace-vscode/snippets/1936507/raw \
	&& chown -R vscode:vscode /home/vscode/.antigen && chown vscode:vscode /home/vscode/.antigenrc
	
USER vscode
RUN echo 'Fixing Wndows Line Endings Bug ref: https://stackoverflow.com/questions/36994298/oh-my-zsh-command-not-found-m' \
	&& git config --global core.autocrlf false \
	&& find /home/vscode -name "*.sh" | xargs  dos2unix -u \
	&& find /home/vscode -name "*.zsh" | xargs  dos2unix -u \
	&& mkdir -p ~/.z \
	&& echo Installing sumneko lua language server \
	&& mkdir -p ~/src && cd ~/src \
	&& git clone https://github.com/sumneko/vscode-lua \
	&& cd vscode-lua \
	&& git submodule update --init --recursive \
	&& cd server/3rd/luamake \
	&& ninja -f ninja/linux.ninja \
	&& cd ../../ \
	&& 3rd/luamake/luamake rebuild
	
# Done Revert to root User for VSCode first run
USER root