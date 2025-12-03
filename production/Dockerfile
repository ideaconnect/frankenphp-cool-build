FROM dunglas/frankenphp:1.10.1-builder-php8.4-bookworm AS builder

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

# Replace the official binary by the one contained your custom modules
FROM dunglas/frankenphp:1.10.1-php8.4-bookworm AS runner
COPY --from=builder /usr/local/bin/frankenphp /usr/local/bin/frankenphp

ENV GODEBUG=cgocheck=0
ENV GOMEMLIMIT=512MiB
ENV GOMAXPROCS=0

ARG USER=1000
WORKDIR /app
VOLUME /etc/secrets
VOLUME /var/applog
VOLUME /etc/caddy/

# User setup and permissions
RUN groupadd -g ${USER} runner \
    && useradd runner -u ${USER} -g ${USER} \
    && setcap CAP_NET_BIND_SERVICE=+eip /usr/local/bin/frankenphp \
    && chown -R ${USER}:${USER} /data/caddy /config/caddy

# PHP extensions (removed duplicate gd)
RUN install-php-extensions yaml bcmath gmp mbstring pdo_mysql gd redis igbinary opcache apcu excimer zip filter openssl ctype session dom xml tokenizer simplexml curl simdjson intl amqp

# System packages with cache cleanup (removed vim for production)
RUN apt-get update && apt-get install -y --no-install-recommends \
    wkhtmltopdf zip unzip \
    && rm -rf /var/lib/apt/lists/*

# Memory configuration
RUN echo 'memory_limit=512M' > /usr/local/etc/php/conf.d/memory-limit.ini

# Production OPcache configuration (aggressive caching + JIT)
RUN echo 'opcache.enable=1' > /usr/local/etc/php/conf.d/opcache.ini \
    && echo 'opcache.enable_cli=1' >> /usr/local/etc/php/conf.d/opcache.ini \
    && echo 'opcache.memory_consumption=256' >> /usr/local/etc/php/conf.d/opcache.ini \
    && echo 'opcache.interned_strings_buffer=16' >> /usr/local/etc/php/conf.d/opcache.ini \
    && echo 'opcache.max_accelerated_files=20000' >> /usr/local/etc/php/conf.d/opcache.ini \
    && echo 'opcache.revalidate_freq=0' >> /usr/local/etc/php/conf.d/opcache.ini \
    && echo 'opcache.validate_timestamps=0' >> /usr/local/etc/php/conf.d/opcache.ini \
    && echo 'opcache.save_comments=1' >> /usr/local/etc/php/conf.d/opcache.ini \
    && echo 'opcache.fast_shutdown=1' >> /usr/local/etc/php/conf.d/opcache.ini \
    && echo 'opcache.enable_file_override=1' >> /usr/local/etc/php/conf.d/opcache.ini \
    && echo 'opcache.jit=1255' >> /usr/local/etc/php/conf.d/opcache.ini \
    && echo 'opcache.jit_buffer_size=128M' >> /usr/local/etc/php/conf.d/opcache.ini

# Production APCu configuration
RUN echo 'apc.enabled=1' > /usr/local/etc/php/conf.d/apcu.ini \
    && echo 'apc.shm_size=128M' >> /usr/local/etc/php/conf.d/apcu.ini \
    && echo 'apc.ttl=0' >> /usr/local/etc/php/conf.d/apcu.ini \
    && echo 'apc.enable_cli=1' >> /usr/local/etc/php/conf.d/apcu.ini

# Performance and limits configuration
RUN echo 'realpath_cache_size=4096K' > /usr/local/etc/php/conf.d/performance.ini \
    && echo 'realpath_cache_ttl=600' >> /usr/local/etc/php/conf.d/performance.ini \
    && echo 'max_execution_time=60' >> /usr/local/etc/php/conf.d/performance.ini \
    && echo 'max_input_vars=3000' >> /usr/local/etc/php/conf.d/performance.ini \
    && echo 'post_max_size=64M' >> /usr/local/etc/php/conf.d/performance.ini \
    && echo 'upload_max_filesize=64M' >> /usr/local/etc/php/conf.d/performance.ini

# Healthcheck for orchestration
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD curl -f http://localhost/health || exit 1

USER ${USER}
