FROM php:8.3.1-fpm

COPY ./configs/php.ini /usr/local/etc/php

RUN docker-php-ext-install pdo_mysql && \
    pecl install xdebug