FROM php:7.3.11-fpm-alpine3.10 as runtime

WORKDIR /var/www

    # install deps
RUN apk add --no-cache --upgrade \
        ca-certificates \
        openssl \
        curl \
        git \
        # dependency of the php intl-extension
        icu \
        # dependency of the php gd-extension
        freetype \
        libjpeg-turbo \
        libpng \
        # dependency of php zip-extension
        libzip \
        ssmtp \
    # needed for gd extension configuration
    && apk add --no-cache --upgrade --virtual .build \
        $PHPIZE_DEPS \
        # depdency of the php intl-extension
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
    && docker-php-ext-install -j$(getconf _NPROCESSORS_ONLN) \
        pdo_mysql \
        intl \
        opcache \
        pcntl \
        gd \
        bcmath \
        zip \
    # remove all (non-hidden) files and dirs in /var/www/
    && rm -rf /var/www/* \
    && apk del .build \
    && rm -rf /tmp/*

FROM runtime as builder

ENV COMPOSER_ALLOW_SUPERUSER 1
ENV COMPOSER_VERSION 1.9.0
ENV COMPOSER_SHA256 c9dff69d092bdec14dee64df6677e7430163509798895fbd54891c166c5c0875
ENV XDEBUG_VERSION 2.8.0
ENV PATH /root/.composer/vendor/bin:$PATH

    # install composer
RUN curl -fsSL https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer --version=$COMPOSER_VERSION \
    && echo "$COMPOSER_SHA256 */usr/local/bin/composer" | sha256sum -c - \
    && composer --ansi --version --no-interaction \
    # install prestissimo composer plugin (paralell downloading of packages - remove this when Composer 2 is released!)
    && composer global require hirak/prestissimo:0.3.9 \
    # Needed for xdebug extension configuration (is removed later on, see --virtual)
    && apk add --no-cache --upgrade --virtual .build \
        $PHPIZE_DEPS \
    && pecl install xdebug-$XDEBUG_VERSION \
    && docker-php-ext-enable xdebug \
    && apk del .build \
    && rm -rf /tmp/*
