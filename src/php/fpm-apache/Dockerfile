# This file is the work of https://github.com/shinsenter/php
################################################################################

ARG BASE_IMAGE=shinsenter/php
ARG PHP_VERSION=8.1

# base image and platform
FROM ${BASE_IMAGE}:${PHP_VERSION}-fpm as base

ARG APACHE_PPA_KEY="4F4EA0AAE5267A6C"
ARG APACHE_PPA_URL="http://ppa.launchpadcontent.net/ondrej/apache2/ubuntu"
ARG APACHE_SOURCES="/etc/apt/sources.list.d/ondrej-apache.list"

ARG APACHE_RUN_GROUP=webgroup
ARG APACHE_RUN_USER=webuser

# sets env variables
ENV DEBUG_MODE=false
ENV MSMTP_RELAY_SERVER_HOSTNAME=mailhog
ENV MSMTP_RELAY_SERVER_PORT=1025
ENV APACHE_DOCUMENT_ROOT=${APACHE_DOCUMENT_ROOT:-$WEBHOME}
ENV APACHE_MAX_CONNECTIONS_PER_CHILD=0
ENV APACHE_MAX_REQUEST_WORKERS=150
ENV APACHE_MAX_SPARE_THREADS=75
ENV APACHE_MIN_SPARE_THREADS=10
ENV APACHE_START_SERVERS=2
ENV APACHE_THREAD_LIMIT=64
ENV APACHE_THREADS_PER_CHILD=25

RUN echo "Installing Apache..." \
    \
    # adds ondrej repos for Apache2
    # && add-ppa $APACHE_PPA_URL $APACHE_SOURCES $APACHE_PPA_KEY \
    \
    # installs apache, msmtp
    && apt-update \
    && apt-install msmtp msmtp-mta \
    && apt-install apache2 \
    && service apache2 stop && update-rc.d apache2 disable \
    && rm -f ${APACHE_DOCUMENT_ROOT}/*.html \
    && a2enmod \
        actions autoindex deflate headers http2 \
        proxy proxy_fcgi \
        remoteip rewrite setenvif ssl unique_id \
    && echo "ServerName localhost" >>/etc/apache2/apache2.conf \
    && sed -i 's/LogFormat "%h %l %u %t \\\"%r\\\" %>s %O \\\"/LogFormat "%a %l %u %t \\\"%r\\\" %>s %O \\\"/' /etc/apache2/apache2.conf \
    && echo >>/etc/apache2/envvars \
    && echo "export APACHE_RUN_USER=${APACHE_RUN_USER}" >>/etc/apache2/envvars \
    && echo "export APACHE_RUN_GROUP=${APACHE_RUN_GROUP}" >>/etc/apache2/envvars \
    && ln -sf /dev/stdout /var/log/apache2/access.log \
    && ln -sf /dev/stdout /var/log/apache2/other_vhosts_access.log \
    && ln -sf /dev/stderr /var/log/apache2/error.log \
    \
    # cleanup
    && rm -f $APACHE_SOURCES \
    && apt-cleanup

################################################################################

# main image
FROM base

# adds config files
ADD etc/ /etc/
ADD index.html ${APACHE_DOCUMENT_ROOT}/index.html

# exposes ports
EXPOSE 80
EXPOSE 443
EXPOSE 443/udp

ENTRYPOINT ["/init"]

################################################################################