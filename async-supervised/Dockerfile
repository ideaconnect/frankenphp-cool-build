FROM php:8.4-cli-alpine3.22
ENV HOST_USER_ID=1000

ARG USER=${HOST_USER_ID}

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
RUN curl -1sLf 'https://dl.cloudsmith.io/public/symfony/stable/setup.alpine.sh' | bash
RUN apk add symfony-cli
RUN chown -R ${USER}:${USER} /cache
USER ${USER}