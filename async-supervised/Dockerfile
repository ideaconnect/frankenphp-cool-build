FROM php:8.5-cli-alpine3.22

ARG USER=1000

ENV HOME=/cache
VOLUME "/app"
VOLUME "/cache"
VOLUME "/var/applog"
VOLUME "/etc/secrets"
WORKDIR "/app"

# Step 1: Consolidate system packages into single layer with cache cleanup
RUN apk update && apk add --no-cache \
    curl zip jq zlib zlib-dev openssh-client git openssh \
    supervisor bash vim nodejs npm \
    && rm -rf /var/cache/apk/* \
    && mkdir -p /var/log/supervisor

# Step 2: Use ADD --chmod to save a layer
ADD --chmod=0755 https://github.com/mlocati/docker-php-extension-installer/releases/latest/download/install-php-extensions /usr/local/bin/
RUN install-php-extensions ssh2 intl xml opcache pdo_mysql iconv gmp bcmath zip sockets redis igbinary apcu pcntl gd imagick simdjson

# Step 5: Modernize yarn installation via corepack
RUN npm install --global corepack \
    && corepack enable \
    && corepack prepare yarn@4 --activate

RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# Step 4: Consolidate PHP config - memory and timeouts
RUN echo 'memory_limit=1024M' > /usr/local/etc/php/conf.d/memory_limit.ini \
    && echo 'default_socket_timeout=-1' > /usr/local/etc/php/conf.d/timeouts.ini

# Step 3 & 4: OPcache config with CLI enablement for long-running processes
RUN echo 'opcache.enable=1' > /usr/local/etc/php/conf.d/opcache.ini \
    && echo 'opcache.enable_cli=1' >> /usr/local/etc/php/conf.d/opcache.ini \
    && echo 'opcache.memory_consumption=256' >> /usr/local/etc/php/conf.d/opcache.ini \
    && echo 'opcache.interned_strings_buffer=16' >> /usr/local/etc/php/conf.d/opcache.ini \
    && echo 'opcache.max_accelerated_files=10000' >> /usr/local/etc/php/conf.d/opcache.ini \
    && echo 'opcache.validate_timestamps=0' >> /usr/local/etc/php/conf.d/opcache.ini \
    && echo 'opcache.revalidate_freq=0' >> /usr/local/etc/php/conf.d/opcache.ini \
    && echo 'opcache.fast_shutdown=1' >> /usr/local/etc/php/conf.d/opcache.ini \
    && echo 'opcache.enable_file_override=1' >> /usr/local/etc/php/conf.d/opcache.ini

# Step 3 & 4: APCu config with CLI enablement
RUN echo 'apc.enabled=1' > /usr/local/etc/php/conf.d/apcu.ini \
    && echo 'apc.shm_size=128M' >> /usr/local/etc/php/conf.d/apcu.ini \
    && echo 'apc.ttl=0' >> /usr/local/etc/php/conf.d/apcu.ini \
    && echo 'apc.enable_cli=1' >> /usr/local/etc/php/conf.d/apcu.ini

# Step 4: Realpath cache and execution settings consolidated
RUN echo 'realpath_cache_size=4096K' > /usr/local/etc/php/conf.d/performance.ini \
    && echo 'realpath_cache_ttl=600' >> /usr/local/etc/php/conf.d/performance.ini \
    && echo 'max_execution_time=0' >> /usr/local/etc/php/conf.d/performance.ini \
    && echo 'max_input_time=-1' >> /usr/local/etc/php/conf.d/performance.ini

RUN curl -1sLf 'https://dl.cloudsmith.io/public/symfony/stable/setup.alpine.sh' | bash \
    && apk add --no-cache symfony-cli

RUN chown -R ${USER}:${USER} /cache

# Step 6: Add healthcheck for orchestration
HEALTHCHECK --interval=30s --timeout=10s --retries=3 \
    CMD pgrep supervisord || exit 1

USER ${USER}
CMD ["/usr/bin/supervisord"]