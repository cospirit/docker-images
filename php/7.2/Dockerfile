FROM php:7.2-fpm-alpine3.7

# persistent / runtime deps
ENV PHPIZE_DEPS \
    autoconf \
    cmake \
    file \
    g++ \
    gcc \
    libc-dev \
    pcre-dev \
    make \
    pkgconf \
    re2c

RUN apk update

RUN apk add --no-cache --virtual .persistent-deps \
    # for intl extension
    icu-dev \
    # for mcrypt extension
    libmcrypt-dev \
    # for databases
    postgresql-dev \
    mysql-client \
#    postgresql-client \
    # for gd extension
    freetype-dev \
    libjpeg-turbo-dev \
    libpng-dev \
    # zip extension
    libzip-dev \
    zip

# Install Requirements
RUN set -xe \
 && apk add --no-cache --virtual .build-deps \
    $PHPIZE_DEPS \
#    zlib-dev \
    wget

# opcache
RUN docker-php-ext-configure opcache --enable-opcache \
 && docker-php-ext-install -j$(nproc) opcache

# intl
RUN docker-php-ext-configure intl --enable-intl \
 && docker-php-ext-install -j$(nproc) \
     iconv \
     intl

# databases
RUN docker-php-ext-configure pdo_mysql --with-pdo-mysql \
 && docker-php-ext-configure pdo_pgsql --with-pdo-pgsql \
 && docker-php-ext-install -j$(nproc) \
    pdo_mysql \
    pdo_pgsql

# images
RUN docker-php-ext-configure gd \
    --with-gd \
    --with-freetype-dir=/usr/include/ \
    --with-jpeg-dir=/usr/include/ \
    --with-png-dir=/usr/include/ \
 && docker-php-ext-install -j$(nproc) exif gd

# mcrypt
#RUN pecl install mcrypt \
# && echo "extension=mcrypt.so" > /usr/local/etc/php/conf.d/mcrypt.ini \

# APCu
RUN pecl install apcu-5.1.11 \
 && pecl install apcu_bc-1.0.4 \
 && docker-php-ext-enable apcu --ini-name 10-docker-php-ext-apcu.ini \
 && docker-php-ext-enable apc --ini-name 20-docker-php-ext-apc.ini

# zip
RUN docker-php-ext-configure zip --with-libzip \
 && docker-php-ext-install -j$(nproc) zip

# clean packages and sources
RUN apk del .build-deps \
 && rm -rf /var/cache/apk/*

COPY conf/php.ini /usr/local/etc/php/php.ini

CMD ["php-fpm"]
