FROM php:7.3.17-fpm-buster as runtime

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
        # dependency of php uv-extension
        libuv1 \
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
        # dependency of php uv-extension
        libuv1-dev \
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
    && pecl install channel://pecl.php.net/uv-0.2.4 \
    && docker-php-ext-enable uv \
    # purge packages that where only needed for building php extensions
    && apt-get purge --assume-yes \
        $PHPIZE_DEPS \
        libicu-dev \
        libfreetype6-dev \
        libjpeg62-turbo-dev \
        libpng-dev \
        libzip-dev \
        libuv1-dev \
    && mv $PHP_INI_DIR/php.ini-production $PHP_INI_DIR/php.ini \
    # remove all (non-hidden) files and dirs in /var/www/
    && rm -rf /var/www/* \
    && rm -rf /var/lib/apt/lists/* \
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
    && mv $PHP_INI_DIR/php.ini $PHP_INI_DIR/php.ini-production \
    && mv $PHP_INI_DIR/php.ini-development $PHP_INI_DIR/php.ini \
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
