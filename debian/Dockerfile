FROM php:7.3.13-fpm-buster as runtime

WORKDIR /var/www

    # install deps
    # hadolint ignore=SC2086
RUN apt-get update \
    && apt-get install --assume-yes --no-install-recommends \
        ca-certificates \
        openssl \
        curl \
        git \
        msmtp \
        # dependency of the php intl-extension
        libicu63 \
        # dependency of the php gd-extension
        libfreetype6 \
        libjpeg62-turbo \
        libpng16-16 \
        # dependency of php zip-extension
        libzip4 \
    # install packages that are needed for building php extensions
    && apt-get install --assume-yes --no-install-recommends \
        $PHPIZE_DEPS \
        # dependency of the php intl-extension
        libicu-dev \
        # dependencies of php gd-extension
        libfreetype6-dev \
        libjpeg62-turbo-dev \
        libpng-dev \
        # dependency of php zip-extension
        libzip-dev \
    # configure php gd-extension
    && docker-php-ext-configure gd \
        --with-jpeg-dir=/usr/include/ \
        --with-png-dir=/usr/include/ \
        --with-freetype-dir=/usr/include/ \
    # install php extensions
    && docker-php-ext-install -j "$(nproc --all)" \
        pdo_mysql \
        intl \
        opcache \
        pcntl \
        gd \
        bcmath \
        zip \
    # purge packages that where only needed for building php extensions
    && apt-get purge --assume-yes \
        $PHPIZE_DEPS \
        libicu-dev \
        libfreetype6-dev \
        libjpeg62-turbo-dev \
        libpng-dev \
        libzip-dev \
    # remove all (non-hidden) files and dirs in /var/www/
    && rm -rf /var/www/* \
    && rm -rf /var/lib/apt/lists/* \
    && rm -rf /tmp/*

FROM runtime as builder

ENV COMPOSER_ALLOW_SUPERUSER 1
ENV COMPOSER_VERSION 1.9.1
ENV COMPOSER_SHA256 1f210b9037fcf82670d75892dfc44400f13fe9ada7af9e787f93e50e3b764111
ENV XDEBUG_VERSION 2.9.0
ENV PATH /root/.composer/vendor/bin:$PATH

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

    # install composer and xdebug
    # hadolint ignore=SC2086
RUN curl -fsSL https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer --version="$COMPOSER_VERSION" \
    && echo "$COMPOSER_SHA256 */usr/local/bin/composer" | sha256sum -c - \
    && composer --ansi --version --no-interaction \
    # install prestissimo composer plugin (paralell downloading of packages - remove this when Composer 2 is released!)
    && composer global require hirak/prestissimo:0.3.9 \
    # Needed for xdebug extension configuration
    && apt-get update \
    && apt-get install --assume-yes --no-install-recommends \
        $PHPIZE_DEPS \
    && pecl install xdebug-$XDEBUG_VERSION \
    && docker-php-ext-enable xdebug \
    && apt-get purge --assume-yes $PHPIZE_DEPS \
    && rm -rf /var/lib/apt/lists/* \
    && rm -rf /tmp/*
