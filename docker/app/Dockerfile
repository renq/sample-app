# BASE image
FROM php:8.1.8-fpm-buster as base

CMD ["php-fpm", "-F"]
ENV WKHTMLTOPDF_PATH /usr/local/bin/wkhtmltopdf

RUN apt-get update \
    && apt-get install -y --no-install-recommends libzip-dev unzip libicu-dev mariadb-client libpq-dev libmemcached-dev \
    && docker-php-ext-install opcache pdo_mysql pdo_pgsql intl zip sockets \
    && pecl install memcache-8.0 \
    && docker-php-ext-enable memcache \
    && docker-php-source delete \
    && rm -rf /var/lib/apt/lists/* \
    && rm -rf /tmp/*

COPY ./docker/app/zz-custom.ini /usr/local/etc/php/conf.d/zz-custom.ini
COPY ./docker/app/wait-for-it.sh /usr/local/bin/wait-for-it.sh


# Composer
FROM composer:2 as composer


# Base development image (base image + development tools)
FROM base as dev

RUN apt-get update \
    && apt-get install -y --no-install-recommends poppler-utils \
    git-all \
    && pecl install xdebug-3.1.5 pcov \
    && docker-php-ext-enable xdebug pcov \
    && docker-php-source delete \
    && rm -rf /var/lib/apt/lists/* \
    && rm -rf /tmp/*

COPY --from=composer /usr/bin/composer /usr/bin/composer

WORKDIR /var/www/application

ENV APP_DEBUG=1 \
    COMPOSER_MEMORY_LIMIT=-1

ARG APP_USER_ID=1000
ARG APP_GROUP_ID=1000

COPY ./docker/app/zz-dev.ini /usr/local/etc/php/conf.d/zz-dev.ini
RUN usermod -u $APP_USER_ID www-data --shell /bin/bash && groupmod -g $APP_GROUP_ID www-data

USER www-data:www-data


# BUILD image. Build is used for composer and installing dependencies for the production.
FROM base as build

COPY --from=composer /usr/bin/composer /usr/bin/composer
RUN mkdir /composer && chown www-data:www-data /composer
ENV COMPOSER_HOME /composer

WORKDIR /var/www/application
COPY --chown=www-data:www-data composer.json composer.lock /var/www/application/
RUN chown www-data:www-data /var/www/ -R
USER www-data:www-data

RUN composer install --prefer-dist --no-scripts --no-interaction --no-progress --no-suggest --no-dev --no-cache --optimize-autoloader


# PROD image
FROM base as prod

WORKDIR /var/www/application

ENV APP_ENV=prod \
    APP_DEBUG=0 \
    SYMFONY_ENV=prod

USER www-data:www-data

COPY --chown=www-data:www-data . /var/www/application/
COPY --from=build --chown=www-data:www-data /var/www/application/vendor /var/www/application/vendor

RUN bin/console --env prod cache:warmup \
    && bin/console --env prod assets:install

VOLUME /var/www/application/var
