FROM alpine:latest

LABEL maintainer="Theros - theros@svaltek.xyz"

# copy LuaSpace Scripts into Container
COPY ./docker/LuaSpace /usr/LuaSpace
RUN chmod 777 -R /usr/LuaSpace

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
    && apk add --no-cache wget nano coreutils ca-certificates ruby-full\
    && echo 'Set disable_coredump false' >> /etc/sudo.conf \
    #
    # Verify git, common tools / libs installed, add/modify non-root user, optionally install zsh
    && wget -q -O /tmp/common-setup.sh $COMMON_SCRIPT_SOURCE \
    && if [ "$COMMON_SCRIPT_SHA" != "dev-mode" ]; then echo "$COMMON_SCRIPT_SHA /tmp/common-setup.sh" | sha256sum -c - ; fi \
    && /bin/ash /tmp/common-setup.sh "$INSTALL_ZSH" "$USERNAME" "$USER_UID" "$USER_GID" \
    && rm /tmp/common-setup.sh

# Install linuxBrew
RUN apk update \
    && apk --no-cache add bash curl file g++ git libc6-compat make ruby ruby-bigdecimal ruby-etc ruby-irb ruby-json ruby-test-unit\
    && apk --update add grep \
    && adduser -D -s /bin/bash linuxbrew \
    && echo 'linuxbrew ALL=(ALL) NOPASSWD:ALL' >>/etc/sudoers

USER linuxbrew
WORKDIR /home/linuxbrew
ENV PATH=/home/linuxbrew/.linuxbrew/bin:/home/linuxbrew/.linuxbrew/sbin:$PATH \
    SHELL=/bin/bash \
    USER=linuxbrew

RUN ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Linuxbrew/install/master/install)" \
    && HOMEBREW_NO_ANALYTICS=1 brew install -s patchelf \
    && HOMEBREW_NO_ANALYTICS=1 brew install --ignore-dependencies binutils gmp isl@0.18 libmpc linux-headers mpfr zlib \
    && (HOMEBREW_NO_ANALYTICS=1 brew install --ignore-dependencies gcc || true) \
    && HOMEBREW_NO_ANALYTICS=1 brew install glibc \
    && HOMEBREW_NO_ANALYTICS=1 brew postinstall gcc \
    && HOMEBREW_NO_ANALYTICS=1 brew remove patchelf \
    && HOMEBREW_NO_ANALYTICS=1 brew install -s patchelf \
    && HOMEBREW_NO_ANALYTICS=1 brew config

USER root
WORKDIR /home/vscode
# Configure Zsh and Antigen
RUN echo install and configure antigen... \
    && echo export PATH=/usr/LuaSpace:$PATH >> /home/vscode/.zshrc \
    && mkdir -p /home/vscode/.antigen \
    && mkdir -p /home/vscode/.z && chown vscode:vscode /home/vscode/.z \
    && curl -L git.io/antigen > /home/vscode/.antigen/antigen.zsh \
    && echo source /home/vscode/.antigen/antigen.zsh >> /home/vscode/.zshrc \
    && echo antigen init /home/vscode/.antigenrc >> /home/vscode/.zshrc \
    && echo 'LuaSpace-MOTD' >> /home/vscode/.zshrc \
    && curl -o  /home/vscode/.antigenrc https://gitlab.com/svaltek/DevTools/luaspace-vscode/snippets/1936507/raw \
    && chown -R vscode:vscode /home/vscode/.antigen && chown vscode:vscode /home/vscode/.antigenrc \
    && echo 'Fixing Wndows Line Endings Bug ref: https://stackoverflow.com/questions/36994298/oh-my-zsh-command-not-found-m' \
    && git config --global core.autocrlf false \
    && find /home/vscode -name "*.sh" | xargs  dos2unix -u \
    && find /home/vscode -name "*.zsh" | xargs  dos2unix -u \
    && echo '   All done....'
# Done Revert to root User for VSCode first run
USER root
