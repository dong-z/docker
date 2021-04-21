FROM freshoo/ubuntu-odoo-base:20.04
LABEL maintainer="dong@freshoo.cn"

# Avoid warnings by switching to noninteractive
ENV DEBIAN_FRONTEND=noninteractive

USER root

# Set timezone (default Etc/UTC)
RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \
    && echo Asia/Shanghai > /etc/timezone

# Add ubuntu mirrors
RUN sed -i 's/archive\.ubuntu\.com/mirrors.aliyun.com/g' /etc/apt/sources.list

# Install Chinese fonts
RUN apt-get update && apt-get -y install ttf-wqy-zenhei ttf-wqy-microhei

# Add npm mirrors
RUN npm config set registry 'https://registry.npm.taobao.org'

USER $NON_ROOT_USERNAME

# Add python pip mirrors
RUN mkdir -p ~/.pip \
    && echo "[global]" > ~/.pip/pip.conf \
    && echo "index-url=http://mirrors.aliyun.com/pypi/simple/" >> ~/.pip/pip.conf \
    && echo "[install]" >> ~/.pip/pip.conf \
    && echo "trusted-host=mirrors.aliyun.com" >> ~/.pip/pip.conf

# Switch back to dialog for any ad-hoc use of apt-get
ENV DEBIAN_FRONTEND=
