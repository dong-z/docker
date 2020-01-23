# This Dockerfile adds a non-root 'node' user with sudo access. However, for Linux,
# this user's GID/UID must match your local user UID/GID to avoid permission issues
# with bind mounts. Update USER_UID / USER_GID if yours is not 1000. See
# https://aka.ms/vscode-remote/containers/non-root-user for details.
ARG USERNAME=node
ARG USER_UID=1000
ARG USER_GID=$USER_UID

ARG NODEJS_VERSION=12

FROM node:${NODEJS_VERSION}
LABEL maintainer="dong@freshoo.cn"

ARG USERNAME
ARG USER_UID
ARG USER_GID

# Avoid warnings by switching to noninteractive
ENV DEBIAN_FRONTEND=noninteractive

# Add debian mirrors
RUN sed -i 's/deb\.debian\.org/mirrors.aliyun.com/g' /etc/apt/sources.list

# Configure apt and install packages
RUN apt-get update \
    && apt-get -y install --no-install-recommends apt-utils dialog 2>&1 \ 
    #
    # Verify git and needed tools are installed
    && apt-get -y install git iproute2 procps \
    #
    # Remove outdated yarn from /opt and install via package 
    # so it can be easily updated via apt-get upgrade yarn
    && rm -rf /opt/yarn-* \
    && rm -f /usr/local/bin/yarn \
    && rm -f /usr/local/bin/yarnpkg \
    && apt-get install -y curl apt-transport-https lsb-release \
    && curl -sS https://dl.yarnpkg.com/$(lsb_release -is | tr '[:upper:]' '[:lower:]')/pubkey.gpg | apt-key add - 2>/dev/null \
    && echo "deb https://dl.yarnpkg.com/$(lsb_release -is | tr '[:upper:]' '[:lower:]')/ stable main" | tee /etc/apt/sources.list.d/yarn.list \
    && apt-get update \
    && apt-get -y install --no-install-recommends yarn \
    #
    # Add npm, yarn mirrors
    && npm config set registry 'https://registry.npm.taobao.org' \
    && yarn config set registry 'https://registry.npm.taobao.org' \
    #
    # Install tslint, eslint and typescript globally
    && npm install -g tslint eslint typescript \
    #
    # [Optional] Update a non-root user to match UID/GID - see https://aka.ms/vscode-remote/containers/non-root-user.
    && if [ "$USER_GID" != "1000" ]; then usermod --g $USER_GID node; fi \
    && if [ "$USERNAME" != "node" ]; then usermod -l $USERNAME node \
        && usermod -c $USERNAME $USERNAME \
        && groupmod -n $USERNAME node \
        && usermod -md /home/$USERNAME $USERNAME \
        && chown -R $USERNAME:$USERNAME $HOME; fi \
    && if [ "$USER_UID" != "1000" ]; then usermod --u $USER_UID node; fi \
    && chown ${USER_UID}:${USER_GID} /usr/local/lib/node_modules \
    # [Optional] Add sudo support for non-root user
    && apt-get install -y sudo \
    && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME \
    && chmod 0440 /etc/sudoers.d/$USERNAME \
    #
    # Clean up
    && apt-get autoremove -y \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/*

# Switch back to dialog for any ad-hoc use of apt-get
ENV DEBIAN_FRONTEND=

USER $USERNAME