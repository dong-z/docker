ARG ODOO_VERSION=11.0

FROM odoo:11.0
LABEL maintainer="dong@freshoo.cn"

USER root

# Add python pip mirrors
RUN mkdir -p ~/.pip \
    && echo "[global]" > ~/.pip/pip.conf \
    && echo "index-url=http://mirrors.aliyun.com/pypi/simple/" >> ~/.pip/pip.conf \
    && echo "[install]" >> ~/.pip/pip.conf \
    && echo "trusted-host=mirrors.aliyun.com" >> ~/.pip/pip.conf

# Add debian mirrors
RUN sed -i 's/deb\.debian\.org/mirrors.aliyun.com/g' /etc/apt/sources.list

# [Optional] Add sudo support for the non-root user
RUN set -x; apt-get update \
    && apt-get install -y sudo \
    && echo odoo ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/odoo \
    && chmod 0440 /etc/sudoers.d/odoo

RUN set -x ; apt-get update \
    && apt-get install -y git \
    && git clone -b ${ODOO_VERSION%.*}.0 --single-branch --depth=1 https://github.com/odoo/runbot.git /mnt/runbot \
    # && pip3 --no-cache-dir install -r /mnt/runbot/requirements.txt \
    && pip3 --no-cache-dir install matplotlib==3.0.2 \
    && chown odoo:odoo -R /mnt/runbot/runbot/static/ \
    && sudo sed -i -e "s;addons_path.*;addons_path = /mnt/extra-addons,/mnt/runbot;" /etc/odoo/odoo.conf

# # Install base files
# RUN set -x ; \
#     apt-get update \
#     && apt-get install -y --no-install-recommends \
#     apt-transport-https \
#     build-essential \
#     ca-certificates \
#     curl \
#     fonts-freefont-ttf \
#     fonts-noto-cjk \
#     gawk \
#     gnupg \
#     libldap2-dev \
#     libsasl2-dev \
#     libxslt1-dev \
#     node-less \
#     python \
#     python-dev \
#     python-pip \
#     python3 \
#     python3-dev \
#     python3-pip \
#     python3-setuptools \
#     python3-wheel \
#     sed \
#     sudo \
#     unzip \
#     xfonts-75dpi \
#     zip \
#     zlib1g-dev

# # Python3 support
# RUN set -x; apt-get update \
#     && apt-get install -y --no-install-recommends libffi-dev libpq-dev \
#     && curl -sSL https://npm.taobao.org/mirrors/python/3.7.3/Python-3.7.3.tgz -o Python-3.7.3.tgz \
#     && tar xvf Python-3.7.3.tgz \
#     && cd Python-3.7.3 \
#     && ./configure --enable-optimizations \
#     && make -j8 \
#     && make altinstall \
#     && ln -s /usr/local/bin/python3.7 /usr/bin/python3.7 \
#     #
#     # Make python3 as default
#     && update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.7 1 \
#     && update-alternatives --install /usr/bin/python python /usr/bin/python3.7 1 \
#     && update-alternatives --install /usr/bin/pip pip /usr/bin/pip3 2

# RUN curl -sSL https://bootstrap.pypa.io/get-pip.py -o get-pip.py \
#     && python3 get-pip.py \
#     && rm get-pip.py

# # Install Google Chrome
# RUN curl -sSL http://nightly.odoo.com/odoo.key | apt-key add - \
#     && echo "deb http://nightly.odoo.com/deb/bionic ./" > /etc/apt/sources.list.d/google-chrome.list \
#     && apt-get update \ 
#     && apt-get install -y -qq google-chrome-stable

# # Install phantomjs
# RUN curl -sSL https://bitbucket.org/ariya/phantomjs/downloads/phantomjs-2.1.1-linux-x86_64.tar.bz2  -o /tmp/phantomjs.tar.bz2 \
#     && tar xvfO /tmp/phantomjs.tar.bz2 phantomjs-2.1.1-linux-x86_64/bin/phantomjs > /usr/local/bin/phantomjs \
#     && chmod +x /usr/local/bin/phantomjs \
#     && rm -f /tmp/phantomjs.tar.bz2

# # Install wkhtml
# RUN curl -sSL https://github.com/wkhtmltopdf/wkhtmltopdf/releases/download/0.12.5/wkhtmltox_0.12.5-1.stretch_amd64.deb -o /tmp/wkhtml.deb \
#     && apt-get update \
#     && dpkg --force-depends -i /tmp/wkhtml.deb \
#     && apt-get install -y -f --no-install-recommends \
#     && rm /tmp/wkhtml.deb

# # Install rtlcss (on Debian stretch)
# RUN curl -sSL https://deb.nodesource.com/gpgkey/nodesource.gpg.key | apt-key add - \
#     && echo "deb https://deb.nodesource.com/node_8.x stretch main" > /etc/apt/sources.list.d/nodesource.list \
#     && apt-get update \
#     && apt-get install -y nodejs

# RUN npm install -g rtlcss

# # Install es-check tool
# RUN npm install -g es-check

# # Install for migration scripts
# RUN apt-get update \
#     && apt-get install -y python3-markdown

# # Install flamegraph.pl
# ADD https://raw.githubusercontent.com/brendangregg/FlameGraph/master/flamegraph.pl /usr/local/bin/flamegraph.pl
# RUN chmod +rx /usr/local/bin/flamegraph.pl

# # Install Odoo Debian dependencies
# ADD https://raw.githubusercontent.com/odoo/odoo/10.0/debian/control /tmp/p2-control
# ADD https://raw.githubusercontent.com/odoo/odoo/master/debian/control /tmp/p3-control
# RUN pip install -U setuptools wheel \
#     && apt-get update \
#     && sed -n '/^Depends:/,/^[A-Z]/p' /tmp/p2-control /tmp/p3-control | awk '/^ [a-z]/ { gsub(/,/,"") ; print }' | sort -u | sed 's/python-imaging/python-pil/'| sed 's/python-pypdf/python-pypdf2/' | DEBIAN_FRONTEND=noninteractive xargs apt-get install -y -qq \
#     && apt-get clean \
#     && rm -rf /var/lib/apt/lists/*

# # Install Odoo requirements for python2 and python3 not fullfilled by Debian dependencies
# ADD https://raw.githubusercontent.com/odoo/odoo/master/requirements.txt /root/p3-requirements.txt
# ADD https://raw.githubusercontent.com/odoo/odoo/10.0/requirements.txt /root/p2-requirements.txt
# RUN pip install --no-cache-dir -r /root/p2-requirements.txt coverage flanker==0.4.38 pylint==1.7.2 phonenumbers redis \
#     && pip3 install --no-cache-dir -r /root/p3-requirements.txt coverage websocket-client astroid==2.0.4 pylint==1.7.2 phonenumbers pyCrypto dbfread==2.0.7 firebase-admin==2.17.0 flamegraph


# =================================================================================================================================

# RUN apt-get update || true && \
#     apt-get install -y python-matplotlib emacs-nox git net-tools tree python-pip python3-pip file nginx python-dev sudo htop locales locales-all wget fonts-dejavu && \
#     pip install gevent psycogreen && \
#     #update werkzeug to make phantomjs work. See http://odoo-development.readthedocs.io/en/latest/dev/tests/js.html#regular-phantom-js-tests
#     pip install werkzeug --upgrade && \
#     apt-get install -y npm python-lxml libxml2-dev libxslt1-dev && \
#     # Extra package for pylint-odoo plugin
#     npm install -g jshint

# # install phantomjs (based on https://hub.docker.com/r/cmfatih/phantomjs/~/dockerfile/ )
# ENV PHANTOMJS_VERSION 1.9.8

# RUN \
#   apt-get install -y libfreetype6 libfontconfig && \
#   mkdir -p /srv/var && \
#   wget -q --no-check-certificate -O /tmp/phantomjs-$PHANTOMJS_VERSION.tar.gz https://github.com/ariya/phantomjs/archive/$PHANTOMJS_VERSION.tar.gz && \
#   tar -xzf /tmp/phantomjs-$PHANTOMJS_VERSION.tar.gz -C /tmp && \
#   rm -f /tmp/phantomjs-$PHANTOMJS_VERSION.tar.gz && \
#   mv /tmp/phantomjs-$PHANTOMJS_VERSION/ /srv/var/phantomjs && \
#   ln -s /srv/var/phantomjs/bin/phantomjs /usr/bin/phantomjs && \
#   git clone https://github.com/n1k0/casperjs.git /srv/var/casperjs && \
#   ln -s /srv/var/casperjs/bin/casperjs /usr/bin/casperjs && \
#   apt-get autoremove -y && \
#   apt-get clean all

# # fix access issue with nginx
# RUN touch /var/log/nginx/error.log && \
#     touch /var/log/nginx/access.log && \
#     chown odoo:odoo -R /var/log/nginx && \
#     chown odoo:odoo -R /var/lib/nginx/

# # python3 support
# # https://unix.stackexchange.com/questions/332641/how-to-install-python-3-6
# RUN wget https://www.python.org/ftp/python/3.6.3/Python-3.6.3.tgz && \
#     tar xvf Python-3.6.3.tgz && \
#     cd Python-3.6.3 && \
#     ./configure --enable-optimizations && \
#     make -j8 && \
#     make altinstall && \
#     rm /usr/bin/python3 && \
#     ln -s /usr/local/bin/python3.6 /usr/bin/python3

# RUN wget https://bootstrap.pypa.io/get-pip.py && \
#     python3 get-pip.py && \
#     pip --version && \
#     pip3 --version && \
#     rm get-pip.py

# RUN apt-get install -y libldap2-dev libsasl2-dev libffi-dev libssl-dev python-pandas && \
#     pip3 install -r https://raw.githubusercontent.com/odoo/odoo/12.0/requirements.txt || true && \
#     pip3 install -r https://raw.githubusercontent.com/odoo/odoo/11.0/requirements.txt || true && \
#     pip install -r https://raw.githubusercontent.com/odoo/odoo/10.0/requirements.txt || true


# ENV BUILD_DATE=2016_11_03

# RUN git clone -b 8.0 https://github.com/it-projects-llc/runbot-addons.git /mnt/runbot-addons && \
#     pip install --upgrade pip setuptools && \
#     pip install --upgrade pylint && \
#     pip install --upgrade git+https://github.com/oca/pylint-odoo.git && \
#     git clone -b runbot-docker https://github.com/yelizariev/odoo-extra.git /mnt/odoo-extra

# # grant access to work dir
# RUN chown odoo:odoo -R /mnt/odoo-extra/runbot/static/

# RUN true && \
#     # always close cron db connnections
#     sed -i "s/if len(db_names) > 1:/if True:/" /usr/lib/python2.7/dist-packages/openerp/service/server.py && \
#     # auto_reload
#     sed -i -e "s/auto_reload = True/; auto_reload = True/" /etc/odoo/openerp-server.conf && \
#     # limits:
#     sed -i -e "s/; limit_time_cpu.*/limit_time_cpu = 300/" /etc/odoo/openerp-server.conf && \
#     sed -i -e "s/; limit_time_real.*/limit_time_real = 600/" /etc/odoo/openerp-server.conf && \
#     # addons_path:
#     sed -i -e "s;addons_path.*;addons_path = /mnt/odoo-extra,/mnt/extra-addons,/mnt/runbot-addons,/usr/lib/python2.7/dist-packages/openerp/addons;" /etc/odoo/openerp-server.conf

# VOLUME ["/mnt/odoo-extra", "/mnt/runbot-addons"]

# CMD ["openerp-server", "--database=runbot", "--db-filter=^runbot$", "--workers=2"]

RUN true \
    # always close cron db connnections
    && sed -i "s/if len(db_names) > 1:/if True:/" /usr/lib/python3/dist-packages/odoo/service/server.py \
    # auto_reload
    && sed -i -e "s/auto_reload = True/; auto_reload = True/" /etc/odoo/odoo.conf \
    # limits:
    && sed -i -e "s/; limit_time_cpu.*/limit_time_cpu = 300/" /etc/odoo/odoo.conf \
    && sed -i -e "s/; limit_time_real.*/limit_time_real = 600/" /etc/odoo/odoo.conf

CMD ["odoo", "--database=runbot", "--db-filter=^runbot$", "--workers=2"]

USER odoo