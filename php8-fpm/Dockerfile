# See https://github.com/docker-library/php/blob/master/8.4/alpine3.22/fpm/Dockerfile
FROM php:8.4-fpm
ARG TIMEZONE

LABEL creator="Maxence POUTORD <maxence.poutord@gmail.com>"
LABEL maintainer="Julien DE CONTI <julien.de.conti@gmail.com>"

RUN apt-get update && apt-get install -y \
    openssl \
    git \
    unzip \
    libicu-dev

RUN mkdir -p /var/lib/php/session && chown -R www-data:www-data /var/lib/php

# Install Composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer \
    && composer --version

# Set timezone
RUN ln -snf /usr/share/zoneinfo/${TIMEZONE} /etc/localtime && echo ${TIMEZONE} > /etc/timezone \
    && printf '[PHP]\ndate.timezone = "%s"\n', ${TIMEZONE} > /usr/local/etc/php/conf.d/tzone.ini \
    && "date"

# Type docker-php-ext-install to see available extensions
# Install pdo mysql
RUN docker-php-ext-install pdo pdo_mysql

# Install symfony php-intl dependency
RUN docker-php-ext-configure intl && docker-php-ext-install intl

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

RUN echo 'alias sf="php bin/console"' >> ~/.bashrc

WORKDIR /var/www/symfony
