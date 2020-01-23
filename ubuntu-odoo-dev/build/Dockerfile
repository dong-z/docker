# This Dockerfile adds a non-root 'ubuntu' user with sudo access. However, for Linux,
# this user's GID/UID must match your local user UID/GID to avoid permission issues
# with bind mounts. Update USER_UID / USER_GID if yours is not 1000. See
# https://aka.ms/vscode-remote/containers/non-root-user for details.
ARG USERNAME=ubuntu
ARG USER_UID=1000
ARG USER_GID=$USER_UID

ARG ODOO_VERSION=13.0
ARG ADDITIONAL_VERSION=13.0
ARG ODOO_SOURCE=download
ARG ODOO_DB_HOST=db
ARG ODOO_DB_PORT=5432
ARG ODOO_DB_USER=odoo
ARG ODOO_DB_PASSWORD=odoo
ARG IMAGE=freshoo/ubuntu-odoo-base:18.04

FROM ${IMAGE}
LABEL maintainer="dong@freshoo.cn"

USER root

ARG USERNAME
ARG USER_UID
ARG USER_GID

ARG ODOO_VERSION
ARG ADDITIONAL_VERSION
ARG ODOO_SOURCE
ARG ODOO_DB_HOST
ARG ODOO_DB_PORT
ARG ODOO_DB_USER
ARG ODOO_DB_PASSWORD

# Update Odoo db configuration
RUN sed -i -e "s/; db_host.*/db_host = ${ODOO_DB_HOST:-db}/" /etc/odoo/odoo.conf \
    && sed -i -e "s/; db_port.*/db_port = ${ODOO_DB_PORT:-5432}/" /etc/odoo/odoo.conf \
    && sed -i -e "s/; db_user.*/db_user = ${ODOO_DB_USER:-odoo}/" /etc/odoo/odoo.conf \
    && sed -i -e "s/; db_password.*/db_password = ${ODOO_DB_PASSWORD:-odoo}/" /etc/odoo/odoo.conf

# [Optional] Update a non-root user to match UID/GID - see https://aka.ms/vscode-remote/containers/non-root-user.
RUN if [ "$USER_GID" != "$NON_ROOT_USER_GID" ]; then groupmod -g $USER_GID $NON_ROOT_USERNAME; fi \
    && if [ "$USERNAME" != "$NON_ROOT_USERNAME" ]; then usermod -l $USERNAME $NON_ROOT_USERNAME \
        && usermod -c $USERNAME $USERNAME \
        && groupmod -n $USERNAME $NON_ROOT_USERNAME \
        && usermod -md /home/$USERNAME $USERNAME \
        && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME \
        && chmod 0440 /etc/sudoers.d/$USERNAME \
        && chown -R $USERNAME:$USERNAME $HOME \
        && chown $USERNAME /etc/odoo/odoo.conf \
        && chown -R $USERNAME /var/lib/odoo \
        && chown -R $USERNAME /mnt/extra-addons; fi \
    && if [ "$USER_UID" != "$NON_ROOT_USER_UID" ]; then usermod -u $USER_UID $USERNAME; fi

# [Optional] Update non-root user ENV
ENV NON_ROOT_USERNAME=$USERNAME
ENV NON_ROOT_USER_UID=$USER_UID
ENV NON_ROOT_USER_GID=$USER_GID

RUN mkdir -p /opt/odoo \
    && chown -R $USER_UID:$USER_GID /opt/odoo

USER $NON_ROOT_USERNAME

# Add Odoo sources and remove .git folder in order to reduce image size
RUN if [ "$ODOO_SOURCE" != "mount" ]; then git clone -b ${ODOO_VERSION%.*}.0 --single-branch \
        --depth=1 https://github.com/odoo/odoo.git /opt/odoo/odoo-${ODOO_VERSION%.*}.0\
        && sudo sed -i -e "s;addons_path.*;addons_path = /mnt/extra-addons,/opt/odoo/odoo-${ODOO_VERSION%.*}.0/addons,/opt/odoo/odoo-${ODOO_VERSION%.*}.0/odoo/addons;" /etc/odoo/odoo.conf \
        && rm -rf /opt/odoo/odoo-${ODOO_VERSION%.*}.0/.git ; fi

COPY ./pip_extra.txt /tmp/pip_extra.txt

# Install Odoo requirements for python3 from github.com/odoo/odoo/requirements.txt
RUN cd /tmp \
    #
    # Use venv for the specified odoo version
    && python3 -m venv ~/.venv/odoo${ODOO_VERSION%.*}-venv \
    && ~/.venv/odoo${ODOO_VERSION%.*}-venv/bin/pip3 --no-cache-dir install -r https://raw.githubusercontent.com/odoo/odoo/${ODOO_VERSION%.*}.0/requirements.txt \
    #
    # Install extra python dependencies
    && ~/.venv/odoo${ODOO_VERSION%.*}-venv/bin/pip3 --no-cache-dir install -r /tmp/pip_extra.txt \
    #
    # Install additional odoo version
    && if [ "${ADDITIONAL_VERSION%.*}" != "${ODOO_VERSION%.*}" ]; then \
        python3 -m venv ~/.venv/odoo${ADDITIONAL_VERSION%.*}-venv \
        && ~/.venv/odoo${ADDITIONAL_VERSION%.*}-venv/bin/pip3 --no-cache-dir install -r https://raw.githubusercontent.com/odoo/odoo/${ADDITIONAL_VERSION%.*}.0/requirements.txt \
        && ~/.venv/odoo${ADDITIONAL_VERSION%.*}-venv/bin/pip3 --no-cache-dir install -r /tmp/pip_extra.txt ; fi
