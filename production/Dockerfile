FROM dunglas/frankenphp:1.8.0-builder-php8.4-bookworm AS builder

# Copy xcaddy in the builder image
COPY --from=caddy:builder /usr/bin/xcaddy /usr/bin/xcaddy

# CGO must be enabled to build FrankenPHP
ENV CGO_ENABLED=1 XCADDY_SETCAP=1 XCADDY_GO_BUILD_FLAGS="-ldflags '-w -s'"
RUN xcaddy build \
	--output /usr/local/bin/frankenphp \
	--with github.com/dunglas/frankenphp=./ \
	--with github.com/dunglas/frankenphp/caddy=./caddy/ \
	--with github.com/dunglas/caddy-cbrotli \
	--with github.com/dunglas/mercure/caddy \
	--with github.com/dunglas/vulcain/caddy \
	--with github.com/WeidiDeng/caddy-cloudflare-ip

# Replace the official binary by the one contained your custom modules
FROM dunglas/frankenphp:1.8-bookworm AS runner
COPY --from=builder /usr/local/bin/frankenphp /usr/local/bin/frankenphp

ENV GODEBUG=cgocheck=0
ENV GOMEMLIMIT=512MiB

ARG USER=1000
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

RUN echo 'memory_limit=512M' >> /usr/local/etc/php/conf.d/memory-limit.ini
RUN install-php-extensions yaml bcmath gmp mbstring pdo_mysql gd redis igbinary opcache gd excimer zip filter openssl ctype amqp session dom xml tokenizer simplexml curl simdjson
RUN apt update && apt install -y wkhtmltopdf zip unzip vim
USER ${USER}