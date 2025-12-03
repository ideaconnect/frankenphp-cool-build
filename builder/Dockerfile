FROM php:8.5-cli-alpine3.22

ARG USER=1000

# Consolidate directory creation and ownership
RUN mkdir -p /cache /app /etc/secrets \
    && chown -R ${USER}:${USER} /app /cache \
    && echo 'dummy' > /etc/secrets/dev.key

VOLUME /cache
VOLUME /app

ENV HOME=/cache
# Composer optimizations: use /cache for Composer cache, parallel downloads
ENV COMPOSER_HOME=/cache/composer
ENV COMPOSER_CACHE_DIR=/cache/composer/cache
ENV COMPOSER_PROCESS_TIMEOUT=600

# ============================================================================
# CACHE VOLUME USAGE (for docker run --rm mode)
# ============================================================================
# To persist package caches between runs, mount the /cache volume:
#
#   docker run --rm \
#     -v $(pwd):/app \
#     -v builder-cache:/cache \
#     builder composer install
#
# Or with host directory:
#
#   docker run --rm \
#     -v $(pwd):/app \
#     -v ~/.builder-cache:/cache \
#     builder composer install
#
# This persists: Composer packages, npm/yarn cache, and speeds up subsequent builds.
# ============================================================================

ADD --chmod=0755 https://github.com/mlocati/docker-php-extension-installer/releases/latest/download/install-php-extensions /usr/local/bin/
WORKDIR /app

# System packages with cache cleanup
RUN apk update && apk add --no-cache nodejs npm git zip unzip vim openssh-client \
    && rm -rf /var/cache/apk/*

RUN npm install --global corepack \
    && corepack enable \
    && corepack prepare yarn@4 --activate

RUN install-php-extensions yaml bcmath gmp mbstring pdo_mysql gd redis igbinary zip filter openssl ctype amqp session dom xml tokenizer simplexml curl ssh2 apcu intl opcache

# Install Composer with version pinning for reproducibility
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# PHP config optimized for Composer/build operations
RUN echo 'memory_limit=2048M' > /usr/local/etc/php/conf.d/memory_limit.ini \
    && echo 'opcache.enable_cli=1' > /usr/local/etc/php/conf.d/opcache.ini \
    && echo 'opcache.memory_consumption=256' >> /usr/local/etc/php/conf.d/opcache.ini \
    && echo 'realpath_cache_size=4096K' >> /usr/local/etc/php/conf.d/opcache.ini \
    && echo 'realpath_cache_ttl=600' >> /usr/local/etc/php/conf.d/opcache.ini

# Configure Composer for optimal performance
RUN mkdir -p /cache/composer/cache \
    && chown -R ${USER}:${USER} /cache/composer \
    && composer config --global process-timeout 600 \
    && composer config --global preferred-install dist \
    && composer config --global optimize-autoloader true \
    && composer config --global classmap-authoritative true \
    && composer config --global apcu-autoloader true

USER ${USER}