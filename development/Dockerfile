FROM dunglas/frankenphp:1.9.0-builder-php8.4-bookworm AS builder

# Copy xcaddy in the builder image
COPY --from=caddy:builder /usr/bin/xcaddy /usr/bin/xcaddy

# CGO must be enabled to build FrankenPHP
RUN CGO_ENABLED=1 \
XCADDY_GO_BUILD_FLAGS="-ldflags='-w -s' -tags=nobadger,nomysql,nopgx" \
CGO_CFLAGS=$(php-config --includes) \
CGO_LDFLAGS="$(php-config --ldflags) $(php-config --libs)" \
xcaddy build \
	--output /usr/local/bin/frankenphp \
	--with github.com/dunglas/frankenphp=./ \
	--with github.com/dunglas/frankenphp/caddy=./caddy/ \
	--with github.com/dunglas/caddy-cbrotli \
	--with github.com/dunglas/mercure/caddy \
	--with github.com/dunglas/vulcain/caddy \
	--with github.com/WeidiDeng/caddy-cloudflare-ip


FROM dunglas/frankenphp:1.9-bookworm AS runner
COPY --from=builder /usr/local/bin/frankenphp /usr/local/bin/frankenphp

ARG USER=1000
ENV GODEBUG=cgocheck=0
ENV GOMEMLIMIT=512MiB
ENV GOMAXPROCS=0

WORKDIR /app
VOLUME /etc/secrets
VOLUME /var/applog
VOLUME /etc/caddy/

RUN \
        groupadd -g ${USER} runner; \
        useradd runner -u ${USER} -g ${USER}; \
        # Add additional capability to bind to port 80 and 443
        setcap CAP_NET_BIND_SERVICE=+eip /usr/local/bin/frankenphp; \
        # Give write access to /data/caddy and /config/caddy
        chown -R ${USER}:${USER} /data/caddy && chown -R ${USER}:${USER} /config/caddy

RUN install-php-extensions yaml bcmath gmp mbstring pdo_mysql gd redis igbinary opcache apcu gd zip filter openssl ctype amqp session dom xml tokenizer simplexml curl xdebug excimer simdjson intl

# Xdebug configuration for development
RUN echo 'xdebug.mode=develop,debug' >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN echo 'xdebug.client_host=host.docker.internal' >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN echo 'xdebug.start_with_request=yes' >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN echo 'xdebug.log=/tmp/xdebug.log' >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini

# Performance configuration
RUN echo 'memory_limit=512M' >> /usr/local/etc/php/conf.d/memory-limit.ini

# OPcache configuration for development (less aggressive caching for development)
RUN echo 'opcache.enable=1' >> /usr/local/etc/php/conf.d/opcache.ini \
    && echo 'opcache.enable_cli=1' >> /usr/local/etc/php/conf.d/opcache.ini \
    && echo 'opcache.memory_consumption=128' >> /usr/local/etc/php/conf.d/opcache.ini \
    && echo 'opcache.interned_strings_buffer=8' >> /usr/local/etc/php/conf.d/opcache.ini \
    && echo 'opcache.max_accelerated_files=4000' >> /usr/local/etc/php/conf.d/opcache.ini \
    && echo 'opcache.revalidate_freq=1' >> /usr/local/etc/php/conf.d/opcache.ini \
    && echo 'opcache.validate_timestamps=1' >> /usr/local/etc/php/conf.d/opcache.ini \
    && echo 'opcache.save_comments=1' >> /usr/local/etc/php/conf.d/opcache.ini

# APCu configuration for development
RUN echo 'apc.enabled=1' >> /usr/local/etc/php/conf.d/apcu.ini \
    && echo 'apc.shm_size=64M' >> /usr/local/etc/php/conf.d/apcu.ini \
    && echo 'apc.ttl=7200' >> /usr/local/etc/php/conf.d/apcu.ini \
    && echo 'apc.enable_cli=1' >> /usr/local/etc/php/conf.d/apcu.ini
RUN apt update && apt install -y wkhtmltopdf zip unzip vim
RUN mkdir /etc/secrets && echo 'dummy' > /etc/secrets/dev.key
USER ${USER}
