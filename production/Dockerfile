FROM dunglas/frankenphp:1.2-bookworm

ARG USER=runner
WORKDIR /app
VOLUME /etc/secrets
VOLUME /var/applog
VOLUME /etc/caddy/
RUN \
        groupadd -g 1000 runner; \
        useradd ${USER} -u 1000 -g 1000; \
        # Add additional capability to bind to port 80 and 443
        setcap CAP_NET_BIND_SERVICE=+eip /usr/local/bin/frankenphp; \
        # Give write access to /data/caddy and /config/caddy
        chown -R ${USER}:${USER} /data/caddy && chown -R ${USER}:${USER} /config/caddy

RUN echo 'memory_limit=512M' >> /usr/local/etc/php/conf.d/memory-limit.ini
RUN install-php-extensions yaml bcmath gmp mbstring pdo_mysql gd redis igbinary opcache gd excimer zip filter openssl ctype amqp session dom xml tokenizer simplexml curl
RUN apt update && apt install -y wkhtmltopdf
USER ${USER}