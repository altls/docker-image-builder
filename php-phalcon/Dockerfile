FROM php:7.3-fpm-alpine
# ustc mirror
RUN set -eux && sed -i 's/dl-cdn.alpinelinux.org/mirrors.ustc.edu.cn/g' /etc/apk/repositories

WORKDIR /var/www

######## Modules Already included in this image:
# Core ctype curl date dom fileinfo filter ftp hash iconv json
# libxml mbstring mysqlnd openssl pcre PDO pdo_sqlite Phar posix
# readline Reflection session SimpleXML sodium SPL sqlite3
# standard tokenizer xml xmlreader xmlwriter zlib
####################################################

RUN apk add --no-cache --virtual .build-deps ${PHPIZE_DEPS}

############### phalcon package compile ##################
ARG PHALCON_VERSION=3.4.5
ARG PHALCON_EXT_PATH=php7/64bits

COPY ./phalcon-${PHALCON_VERSION}.tar.gz ${PWD}

RUN set -xe && \
        # Compile Phalcon
        tar xzf ${PWD}/phalcon-${PHALCON_VERSION}.tar.gz

RUN set -xe && \
    docker-php-ext-install -j $(getconf _NPROCESSORS_ONLN) ${PWD}/cphalcon-${PHALCON_VERSION}/build/${PHALCON_EXT_PATH} && \
        # Remove all temp files
        rm -r \
            ${PWD}/phalcon-${PHALCON_VERSION}.tar.gz \
            ${PWD}/cphalcon-${PHALCON_VERSION}
############### phalcon package compile ##################


RUN apk add --no-cache \
    # common tools
    git openssh \
    # zip bz2 pgsql modules
    libzip-dev bzip2-dev \
    # gd
    freetype-dev libjpeg-turbo-dev libpng-dev \
    # redis yaml packages
    hiredis-dev yaml-dev \
    # rdkafka
    librdkafka-dev

# Pecl packages
RUN pecl channel-update pecl.php.net \
    && pecl install redis-5.0.2 \
    && pecl install xdebug-3.0.1 \
    && pecl install yaml-2.0.4 \
    && pecl install rdkafka \
    && pecl install xlswriter \
    # && pecl install phalcon \
    # docker-php-ext-configure
    && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ --with-png-dir=/usr/include \
    && docker-php-ext-configure zip --with-libzip \
    # docker-php-ext-install
    && docker-php-ext-install \
        zip sockets bcmath bz2 gd pcntl opcache \
        pdo_mysql mysqli \
    # docker-php-ext-enable
    && docker-php-ext-enable redis xdebug yaml rdkafka xlswriter

RUN apk del .build-deps

RUN set -ex \
    && rm -rf /var/cache/apk/* /tmp/* /var/tmp/* ${HOME}/.cache \
    && rm -rf /usr/src/* \
    && rm -rf /usr/local/include/* /usr/include/*

# Composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin/ --filename=composer

CMD php-fpm
