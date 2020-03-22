FROM php:7.3.16-fpm-alpine3.11 as runtime

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
        # dependency of php uv-extension
        libuv \
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
        # dependency of php uv-extension
        libuv-dev \
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
    && pecl install channel://pecl.php.net/uv-0.2.4 \
    && docker-php-ext-enable uv \
    && mv $PHP_INI_DIR/php.ini-production $PHP_INI_DIR/php.ini \
    && apk del .build \
    # remove all (non-hidden) files and dirs in /var/www/
    && rm -rf /var/www/* \
    && rm -rf /tmp/*

ARG VCS_REF
ARG VERSION
LABEL org.opencontainers.image.revision=$VCS_REF
LABEL org.opencontainers.image.version=$VERSION
LABEL org.opencontainers.image.title=php73-fpm
LABEL org.opencontainers.image.description="A PHP 7.3 based base image for development and production"
LABEL org.opencontainers.image.url=https://github.com/Ilyes512/docker-php73-fpm
LABEL org.opencontainers.image.documentation=https://github.com/Ilyes512/docker-php73-fpm/blob/master/README.md
LABEL org.opencontainers.image.vendor="ilyes512"
LABEL org.opencontainers.image.licenses=MIT
LABEL org.opencontainers.image.source=https://github.com/Ilyes512/docker-php73-fpm

FROM runtime as builder

ENV COMPOSER_ALLOW_SUPERUSER 1
ENV COMPOSER_VERSION 1.9.2
ENV COMPOSER_SHA256 58753998712ae435915a452d701ae28a9389653bbf36b3b6adf24e03d90a9467
ENV XDEBUG_VERSION 2.9.1
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
    && mv $PHP_INI_DIR/php.ini $PHP_INI_DIR/php.ini-production \
    && mv $PHP_INI_DIR/php.ini-development $PHP_INI_DIR/php.ini \
    && apk del .build \
    && rm -rf /tmp/*

ARG VCS_REF
ARG VERSION
LABEL org.opencontainers.image.revision=$VCS_REF
LABEL org.opencontainers.image.version=$VERSION
LABEL org.opencontainers.image.title=php73-fpm
LABEL org.opencontainers.image.description="A PHP 7.3 based base image for development and production"
LABEL org.opencontainers.image.url=https://github.com/Ilyes512/docker-php73-fpm
LABEL org.opencontainers.image.documentation=https://github.com/Ilyes512/docker-php73-fpm/blob/master/README.md
LABEL org.opencontainers.image.vendor="ilyes512"
LABEL org.opencontainers.image.licenses=MIT
LABEL org.opencontainers.image.source=https://github.com/Ilyes512/docker-php73-fpm
