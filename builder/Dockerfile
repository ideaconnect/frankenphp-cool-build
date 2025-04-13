FROM php:8.3-cli-alpine3.20

ARG USER_ID=1000

RUN mkdir /cache
RUN mkdir /app

VOLUME /cache
VOLUME /app

RUN chown -R 1000:1000 /app
RUN chown -R 1000:1000 /cache

ENV HOME=/cache
ADD --chmod=0755 https://github.com/mlocati/docker-php-extension-installer/releases/latest/download/install-php-extensions /usr/local/bin/
WORKDIR /app

RUN apk update && apk add nodejs npm git zip unzip
RUN npm install --global yarn
RUN install-php-extensions yaml bcmath gmp mbstring pdo_mysql gd redis igbinary gd zip filter openssl ctype amqp session dom xml tokenizer simplexml curl ssh2
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
RUN mkdir /etc/secrets && echo 'dummy' > /etc/secrets/dev.key
USER ${USER_ID}