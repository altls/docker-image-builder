FROM php:8.1-fpm-alpine3.16

######## Modules Already included in php image:
# bcmath bz2 calendar ctype curl dba dl_test dom enchant exif ffi fileinfo filter ftp
# gd gettext gmp hash iconv imap intl json ldap mbstring mysqli oci8 odbc opcache pcntl
# pdo pdo_dblib pdo_firebird pdo_mysql pdo_oci pdo_odbc pdo_pgsql pdo_sqlite pgsql phar
# posix pspell readline reflection session shmop simplexml snmp soap sockets sodium spl
# standard sysvmsg sysvsem sysvshm tidy tokenizer xml xmlreader xmlwriter xsl zend_test zip
####################################################

ARG ALPINE_MIRROR=mirrors.ustc.edu.cn
ARG COMPOSER_MIRROR="https://mirrors.aliyun.com/composer/"
ARG TZ="Asia/Shanghai"

USER root

WORKDIR /var/www

# package --need--> libs
# ldap => openldap
# imap => imap
# intl => icu-dev
# gd => libpng-dev
# zip => libzip-dev
# coreutils krb5-libs icu-libs libzip
ENV USER_TOOL_DEPS git curl nginx zip unzip libpng-dev imap-dev icu-dev libzip-dev librdkafka-dev yaml-dev
# openldap-dev

# 额外依赖，用后删除
ENV OTHER_DEPS gcc make libc-dev zlib-dev libxml2-dev libtool alpine-conf libstdc++


# 更新apk国内镜像
RUN [ "${ALPINE_MIRROR}" != "dl-cdn.alpinelinux.org" ] && sed -i 's/dl-cdn.alpinelinux.org/'${ALPINE_MIRROR}'/g' /etc/apk/repositories

# 设置php默认主ini
RUN cd ${PHP_INI_DIR} && cp php.ini-production php.ini

# 安装一些容器内工具，始终存在
RUN set -x \
    && apk update \
    && apk add --no-cache ${USER_TOOL_DEPS}

# 安装php本体及扩展所需依赖， 用后删除
RUN set -xe && apk add --no-cache --update --virtual .phpize-deps ${PHPIZE_DEPS}
RUN set -xe && apk add --no-cache --update --virtual .other-deps ${OTHER_DEPS}

# alpine-conf 设置时区
RUN /sbin/setup-timezone -z ${TZ}

# # 安装并启动自带的PHP扩展，位于/usr/src/php/ext。docker-php-ext-install 安装的扩展在安装完成后，会自动调用docker-php-ext-enable来启动安装的扩展
RUN docker-php-ext-install zip sockets soap gd bcmath mysqli pdo_mysql opcache intl imap
# ldap

# pecl 扩展
RUN pecl channel-update pecl.php.net \
    && pecl install msgpack && docker-php-ext-enable msgpack \
    && pecl install igbinary && docker-php-ext-enable igbinary \
    && pecl install redis && docker-php-ext-enable redis \
    && pecl install xdebug && docker-php-ext-enable xdebug \
    && pecl install yaml && docker-php-ext-enable yaml \
    && pecl install rdkafka && docker-php-ext-enable rdkafka \
    # && pecl install swoole && docker-php-ext-enable swoole \
    && rm -rf /tmp/pear

# 删除所有依赖文件
RUN apk del .other-deps .phpize-deps

RUN set -ex \
    && rm -rf /var/cache/apk/* /tmp/* /var/tmp/* ${HOME}/.cache \
    && rm -rf /usr/src/* \
    && rm -rf /usr/local/include/* /usr/include/* \
    && rm -rf /usr/libexec/*
    # 非root用户使用1024以下端口
    # && setcap 'cap_net_bind_service=+ep' /usr/local/bin/php

# Remove configuration files which are templated during the entrypoint command
RUN rm /usr/local/etc/php-fpm.d/zz-docker.conf /usr/local/etc/php-fpm.d/www.conf.default

RUN curl http://getcomposer.org/installer | php -- --install-dir=/usr/local/bin/ --filename=composer

RUN [ "${COMPOSER_MIRROR}" != "" ] && composer config -g repos.packagist composer ${COMPOSER_MIRROR}


# Add usabilla user and group
RUN set -x \
    && addgroup -g 1024 app \
    && adduser -u 1024 -D -G app app

RUN if [ ! -d /var/log ]; then mkdir /var/log; fi \
    && chown -R app:app /var/log \
    && chmod a+x /var/log/ \
    && if [ ! -d /var/run ]; then mkdir /var/run; fi \
    && chown -R app:app /var/run \
    && chmod a+x /var/run/

CMD ["/bin/sh", "-c", "nginx && php-fpm"]
EXPOSE 80