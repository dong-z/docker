# This Dockerfile adds a non-root 'ubuntu' user with sudo access. However, for Linux,
# this user's GID/UID must match your local user UID/GID to avoid permission issues
# with bind mounts. Update USER_UID / USER_GID if yours is not 1000. See
# https://aka.ms/vscode-remote/containers/non-root-user for details.
ARG USERNAME=ubuntu
ARG USER_UID=1000
ARG USER_GID=$USER_UID

FROM ubuntu:bionic
LABEL maintainer="dong@freshoo.cn"

# Avoid warnings by switching to noninteractive
ENV DEBIAN_FRONTEND=noninteractive

# Generate locale C.UTF-8 for postgres and general locale data
ENV LANG C.UTF-8

ARG USERNAME
ARG USER_UID
ARG USER_GID

ENV NON_ROOT_USERNAME=$USERNAME
ENV NON_ROOT_USER_UID=$USER_UID
ENV NON_ROOT_USER_GID=$USER_GID

# Configure apt and install packages
RUN apt-get update \
    && apt-get -y install --no-install-recommends apt-utils dialog 2>&1 \
    #
    # Verify git, process tools, lsb-release (common in install instructions for CLIs) installed
    && apt-get -y install git iproute2 procps lsb-release locales curl python3-wheel \
        python3-setuptools python3-pip python3-yaml python3-venv \
    #
    # Generate locales
    && locale-gen en_US.UTF-8 \
    && update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8 \
    #
    # Create a non-root user to use if preferred - see https://aka.ms/vscode-remote/containers/non-root-user.
    && groupadd --gid $NON_ROOT_USER_GID $NON_ROOT_USERNAME \
    && useradd -s /bin/bash --uid $NON_ROOT_USER_UID --gid $NON_ROOT_USER_GID -m $NON_ROOT_USERNAME \
    #
    # [Optional] Add sudo support for the non-root user
    && apt-get install -y sudo \
    && echo $NON_ROOT_USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$NON_ROOT_USERNAME \
    && chmod 0440 /etc/sudoers.d/$NON_ROOT_USERNAME \
    #
    # Make python3 as default
    && update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.6 1 \
    && update-alternatives --install /usr/bin/python python /usr/bin/python3.6 1 \
    && update-alternatives --install /usr/bin/pip pip /usr/bin/pip3 2

# Install odoo apt dependencies
RUN set -x; apt-get update \
        && apt-get -y install \
            libcups2-dev \
            build-essential \
            libzip-dev \
            python-dev \
            python3-dev \
            libjpeg-dev \
            libjpeg8-dev \
            libpng-dev \
            libldap2-dev \
            libsasl2-dev \
            libssl-dev \
            libxml2-dev \
            libxslt1-dev \
            nodejs \
        #
        # Install npm and extra package
        && apt-get -y install npm \
        && npm install -g less \
        && npm install -g less-plugin-clean-css \
        && npm install -g jshint \
        && npm install -g eslint

# Install latest postgresql-client
RUN set -x; \
        echo 'deb https://apt.postgresql.org/pub/repos/apt/ bionic-pgdg main' > /etc/apt/sources.list.d/pgdg.list \
        && export GNUPGHOME="$(mktemp -d)" \
        && repokey='B97B0AFCAA1A47F044F244A07FCC7D46ACCC4CF8' \
        && gpg --batch --keyserver keyserver.ubuntu.com --recv-keys "${repokey}" \
        && gpg --batch --armor --export "${repokey}" > /etc/apt/trusted.gpg.d/pgdg.gpg.asc \
        && gpgconf --kill all \
        && rm -rf "$GNUPGHOME" \
        && apt-get update  \
        && apt-get install -y postgresql-client

# Install wkhtmltopdf
RUN curl -o wkhtmltox.deb -sSL https://github.com/wkhtmltopdf/wkhtmltopdf/releases/download/0.12.5/wkhtmltox_0.12.5-1.bionic_amd64.deb \
    && echo 'f1689a1b302ff102160f2693129f789410a1708a wkhtmltox.deb' | sha1sum -c - \
    && apt-get install -y --no-install-recommends ./wkhtmltox.deb \
    && rm -rf wkhtmltox.deb

# Clean up
RUN apt-get autoremove -y \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/*

# Copy Odoo configuration file
COPY ./odoo.conf /etc/odoo/
RUN chown $NON_ROOT_USERNAME /etc/odoo/odoo.conf

# Mount /var/lib/odoo to allow restoring filestore and /mnt/extra-addons for users addons
RUN mkdir -p /var/lib/odoo \
    && chown -R $NON_ROOT_USERNAME /var/lib/odoo \
    && mkdir -p /mnt/extra-addons \
    && chown -R $NON_ROOT_USERNAME /mnt/extra-addons
VOLUME ["/var/lib/odoo", "/mnt/extra-addons"]

# Expose odoo services
EXPOSE 8069 8071 8072

# Set the default config file
ENV ODOO_RC /etc/odoo/odoo.conf

# Switch non-root user
USER $NON_ROOT_USERNAME

# Switch back to dialog for any ad-hoc use of apt-get
ENV DEBIAN_FRONTEND=
