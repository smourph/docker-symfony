# See https://github.com/docker-library/php/blob/master/8.4/alpine3.22/fpm/Dockerfile
FROM php:8.4-fpm
ARG TIMEZONE

LABEL creator="Maxence POUTORD <maxence.poutord@gmail.com>"
LABEL maintainer="Julien DE CONTI <julien.de.conti@gmail.com>"

RUN apt-get update && apt-get install -y \
    openssl \
    git \
    unzip

RUN mkdir -p /var/lib/php/session && chown -R www-data:www-data /var/lib/php

# Set timezone
RUN ln -snf /usr/share/zoneinfo/${TIMEZONE} /etc/localtime && echo ${TIMEZONE} > /etc/timezone \
    && printf '[PHP]\ndate.timezone = "%s"\n', ${TIMEZONE} > /usr/local/etc/php/conf.d/tzone.ini \
    && "date"

# Type docker-php-ext-install to see available extensions
# Install pdo mysql
RUN docker-php-ext-install pdo pdo_mysql mysqli

# Install recommended PHP extensions for WordPress
# exif
RUN docker-php-ext-install exif
# zip
RUN apt-get update && apt-get install -y libzip-dev
RUN docker-php-ext-install zip
# gd
RUN apt-get update && apt-get install -y --no-install-recommends \
    libpng-dev \
    libjpeg-dev \
    libjpeg62-turbo-dev \
    libfreetype6-dev
RUN docker-php-ext-configure gd \
    --with-freetype=/usr/include/ \
    --with-jpeg=/usr/include/ \
    && docker-php-ext-install -j$(nproc) gd
# imagick
RUN apt-get update && apt-get install -y libmagickwand-dev --no-install-recommends
RUN pecl install imagick && docker-php-ext-enable imagick

# Install php-opcache
RUN docker-php-ext-install opcache && docker-php-ext-enable opcache

# install xdebug
RUN pecl install xdebug \
    && docker-php-ext-enable xdebug \
    && echo "error_reporting = E_ALL" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
    && echo "display_startup_errors = On" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
    && echo "display_errors = On" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
    && echo "xdebug.remote_enable=1" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
    && echo "xdebug.remote_connect_back=1" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
    && echo "xdebug.idekey=\"PHPSTORM\"" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
    && echo "xdebug.remote_port=9001" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini

WORKDIR /var/www/website

RUN chown -R www-data:www-data /var/www/website
RUN find /var/www/website -type d -exec chmod 0755 {} \;
RUN find /var/www/website -type f -exec chmod 644 {} \;
