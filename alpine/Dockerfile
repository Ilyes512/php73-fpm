FROM php:7.3.11-fpm-alpine3.10 as runtime

WORKDIR /var/www

    # install deps
    # hadolint ignore=SC2086
RUN apk add --no-cache --upgrade \
        ca-certificates \
        openssl \
        curl \
        git \
        msmtp \
        # dependency of the php intl-extension
        icu \
        # dependency of the php gd-extension
        freetype \
        libjpeg-turbo \
        libpng \
        # dependency of php zip-extension
        libzip \
    # install needed php extension configuration/installing packages
    && apk add --no-cache --upgrade --virtual .build \
        $PHPIZE_DEPS \
        # dependency of the php intl-extension
        icu-dev \
        # dependencies of php gd-extension
        freetype-dev \
        libjpeg-turbo-dev \
        libpng-dev \
        # dependency of php zip-extension
        libzip-dev \
    # configure php gd-extension
    && docker-php-ext-configure gd \
        --with-jpeg-dir=/usr/include/ \
        --with-png-dir=/usr/include/ \
        --with-freetype-dir=/usr/include/ \
    # install php extensions
    && docker-php-ext-install -j "$(getconf _NPROCESSORS_ONLN)" \
        pdo_mysql \
        intl \
        opcache \
        pcntl \
        gd \
        bcmath \
        zip \
    && apk del .build \
    # remove all (non-hidden) files and dirs in /var/www/
    && rm -rf /var/www/* \
    && rm -rf /tmp/*

FROM runtime as builder

ENV COMPOSER_ALLOW_SUPERUSER 1
ENV COMPOSER_VERSION 1.9.1
ENV COMPOSER_SHA256 1f210b9037fcf82670d75892dfc44400f13fe9ada7af9e787f93e50e3b764111
ENV XDEBUG_VERSION 2.8.0
ENV PATH /root/.composer/vendor/bin:$PATH

SHELL ["/bin/ash", "-eo", "pipefail", "-c"]

    # install composer and xdebug
    # hadolint ignore=SC2086
RUN curl -fsSL https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer --version="$COMPOSER_VERSION" \
    && echo "$COMPOSER_SHA256 */usr/local/bin/composer" | sha256sum -c - \
    && composer --ansi --version --no-interaction \
    # install prestissimo composer plugin (paralell downloading of packages - remove this when Composer 2 is released!)
    && composer global require hirak/prestissimo:0.3.9 \
    # Needed for xdebug extension configuration (is removed later on, see --virtual)
    && apk add --no-cache --upgrade --virtual .build \
        $PHPIZE_DEPS \
    && pecl install "xdebug-$XDEBUG_VERSION" \
    && docker-php-ext-enable xdebug \
    && apk del .build \
    && rm -rf /tmp/*
