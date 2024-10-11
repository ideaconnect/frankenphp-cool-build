FROM dunglas/frankenphp:1.2-bookworm

ARG USER=runner

WORKDIR /app
VOLUME /etc/secrets
VOLUME /var/applog
VOLUME /etc/caddy/
RUN apt-get update && apt-get install -y chromium-driver

RUN \
        groupadd -g 1000 runner; \
        useradd ${USER} -u 1000 -g 1000; \
        # Add additional capability to bind to port 80 and 443
        setcap CAP_NET_BIND_SERVICE=+eip /usr/local/bin/frankenphp; \
        # Give write access to /data/caddy and /config/caddy
        chown -R ${USER}:${USER} /data/caddy && chown -R ${USER}:${USER} /config/caddy

RUN install-php-extensions yaml bcmath gmp mbstring pdo_mysql gd redis igbinary gd imagick zip filter openssl ctype amqp session dom xml tokenizer simplexml curl xdebug excimer

RUN echo 'xdebug.mode=develop,debug' >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN echo 'xdebug.client_host=host.docker.internal' >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN echo 'xdebug.start_with_request=yes' >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN echo 'xdebug.log=/tmp/xdebug.log' >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN echo 'memory_limit=512M' >> /usr/local/etc/php/conf.d/memory-limit.ini

RUN mkdir /etc/secrets && echo 'dummy' > /etc/secrets/dev.key
USER ${USER}
