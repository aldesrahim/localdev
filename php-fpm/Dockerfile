ARG PHP_VERSION=8.4

FROM php:${PHP_VERSION}-fpm-alpine

ADD --chmod=0755 https://github.com/mlocati/docker-php-extension-installer/releases/latest/download/install-php-extensions /usr/local/bin/

RUN apk --update add shadow

RUN install-php-extensions bcmath \
    bz2 \
    calendar \
    dba \
    exif \
    ffi \
    ftp \
    gd \
    gmp \
    igbinary \
    imagick \
    imap \
    intl \
    ldap \
    lz4 \
    mongodb \
    mysqli \
    opcache \
    pcntl \
    pdo_mysql \
    pdo_pgsql \
    pgsql \
    redis \
    shmop \
    soap \
    sockets \
    sysvmsg \
    sysvsem \
    sysvshm \
    xsl \
    zip

RUN install-php-extensions @composer

# Set working directory
WORKDIR /var/www

# Clean up
RUN rm -rf /tmp/* /var/cache/apk/*

EXPOSE 9000
CMD ["php-fpm"]
