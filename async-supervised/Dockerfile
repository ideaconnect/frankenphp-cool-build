FROM php:8.4-cli-alpine3.22

ARG USER=1000

ENV HOME=/cache
VOLUME "/app"
VOLUME "/cache"
VOLUME "/var/applog"
VOLUME "/etc/secrets"
WORKDIR "/app"
RUN apk update
RUN mkdir /var/log/supervisor
RUN apk add --no-cache curl zip jq zlib zlib-dev openssh-client git openssh supervisor bash vim
ADD https://github.com/mlocati/docker-php-extension-installer/releases/latest/download/install-php-extensions /usr/local/bin/
RUN chmod +x /usr/local/bin/install-php-extensions && install-php-extensions ssh2 intl xml opcache pdo_mysql iconv gmp bcmath zip sockets redis igbinary apcu pcntl amqp gd imagick simdjson
RUN apk add --update nodejs npm
RUN npm install --global yarn
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
RUN echo 'memory_limit=1024M' > /usr/local/etc/php/conf.d/memory_limit.ini
RUN echo 'default_socket_timeout=-1' > /usr/local/etc/php/conf.d/timeouts.ini
# Optimizations for long-running processes
RUN echo 'opcache.enable=1' > /usr/local/etc/php/conf.d/opcache.ini
RUN echo 'opcache.memory_consumption=256' >> /usr/local/etc/php/conf.d/opcache.ini
RUN echo 'opcache.max_accelerated_files=10000' >> /usr/local/etc/php/conf.d/opcache.ini
RUN echo 'opcache.validate_timestamps=0' >> /usr/local/etc/php/conf.d/opcache.ini
RUN echo 'opcache.revalidate_freq=0' >> /usr/local/etc/php/conf.d/opcache.ini
RUN echo 'opcache.fast_shutdown=1' >> /usr/local/etc/php/conf.d/opcache.ini
# APCu for application caching
RUN echo 'apc.shm_size=128M' > /usr/local/etc/php/conf.d/apcu.ini
RUN echo 'apc.ttl=0' >> /usr/local/etc/php/conf.d/apcu.ini
# Realpath cache for filesystem performance
RUN echo 'realpath_cache_size=4096K' > /usr/local/etc/php/conf.d/realpath.ini
RUN echo 'realpath_cache_ttl=600' >> /usr/local/etc/php/conf.d/realpath.ini
# Process control optimizations
RUN echo 'max_execution_time=0' > /usr/local/etc/php/conf.d/execution.ini
RUN echo 'max_input_time=-1' >> /usr/local/etc/php/conf.d/execution.ini
RUN curl -1sLf 'https://dl.cloudsmith.io/public/symfony/stable/setup.alpine.sh' | bash
RUN apk add symfony-cli
RUN chown -R ${USER}:${USER} /cache

USER ${USER}
CMD ["/usr/bin/supervisord"]