# FrankenPHP Cool Build

A collection of Docker images for PHP development and production using FrankenPHP and related tooling. All images are based on PHP 8.4.

## Images Overview

| Image | Base | Purpose |
|-------|------|---------|
| `builder` | `php:8.4-cli-alpine3.21` | CI/CD builds, dependency installation |
| `development` | `dunglas/frankenphp:1.10.1-php8.4-bookworm` | Local development with Xdebug |
| `production` | `dunglas/frankenphp:1.10.1-php8.4-bookworm` | Production deployment with JIT |
| `async-supervised` | `php:8.4-cli-alpine3.21` | Background workers with Supervisor |

---

## Builder

A lightweight Alpine-based image optimized for running Composer and npm/yarn builds in CI/CD pipelines.

### Use Cases
- Running `composer install` / `composer update`
- Building frontend assets with npm/yarn
- CI/CD pipeline build stages
- Ephemeral build containers (`docker run --rm`)

### Build Arguments

| Argument | Default | Description |
|----------|---------|-------------|
| `USER` | `1000` | UID for the container user |

### Environment Variables

| Variable | Value | Description |
|----------|-------|-------------|
| `HOME` | `/cache` | Home directory for caching |
| `COMPOSER_HOME` | `/cache/composer` | Composer home directory |
| `COMPOSER_CACHE_DIR` | `/cache/composer/cache` | Composer cache location |
| `COMPOSER_PROCESS_TIMEOUT` | `600` | Composer process timeout (seconds) |

### Volumes

| Path | Purpose |
|------|---------|
| `/app` | Application source code |
| `/cache` | Persistent cache for Composer/npm |

### PHP Configuration

- Memory limit: 2048M
- OPcache CLI enabled with 256M memory
- Realpath cache: 4096K with 600s TTL

### Composer Optimizations

- Prefers dist installations
- Autoloader optimization enabled
- Classmap authoritative mode
- APCu autoloader enabled

### Usage Example

```bash
# With named volume for cache persistence
docker run --rm \
  -v $(pwd):/app \
  -v builder-cache:/cache \
  builder composer install

# With host directory cache
docker run --rm \
  -v $(pwd):/app \
  -v ~/.builder-cache:/cache \
  builder yarn install
```

### Included Tools
- Composer
- Node.js + npm
- Yarn 4 (via corepack)
- Git, zip, unzip, vim, openssh-client

### PHP Extensions
yaml, bcmath, gmp, mbstring, pdo_mysql, gd, redis, igbinary, zip, filter, openssl, ctype, session, dom, xml, tokenizer, simplexml, curl, ssh2, apcu, intl, opcache

---

## Development

A FrankenPHP-based development image with Xdebug, custom Caddy modules, and development-friendly PHP settings.

### Use Cases
- Local development environment
- Debugging with Xdebug
- Running tests with code coverage
- Development web server

### Build Arguments

| Argument | Default | Description |
|----------|---------|-------------|
| `USER` | `1000` | UID for the container user |
| `XDEBUG_IDEKEY` | `PHPIDE` | Xdebug IDE key |
| `XDEBUG_TRIGGER` | `PHPIDE` | Xdebug trigger value |
| `XDEBUG_CLIENT_HOST` | `host.docker.internal` | Xdebug client host |

### Environment Variables

| Variable | Value | Description |
|----------|-------|-------------|
| `GODEBUG` | `cgocheck=0` | Go runtime debug settings |
| `GOMEMLIMIT` | `512MiB` | Go memory limit |
| `GOMAXPROCS` | `0` | Go max processors (auto) |

### Volumes

| Path | Purpose |
|------|---------|
| `/app` | Application source code (workdir) |
| `/etc/secrets` | Secret files (e.g., encryption keys) |
| `/var/applog` | Application logs |
| `/etc/caddy/` | Caddy configuration |

### PHP Configuration

| Setting | Value | Notes |
|---------|-------|-------|
| `memory_limit` | 512M | |
| `opcache.validate_timestamps` | 1 | Checks file changes |
| `opcache.revalidate_freq` | 0 | Checks on every request |
| `opcache.memory_consumption` | 128M | |
| `apc.shm_size` | 64M | |
| `apc.ttl` | 7200 | 2 hour TTL |
| `realpath_cache_ttl` | 120 | Lower for development |

### Xdebug Configuration

| Setting | Value |
|---------|-------|
| `xdebug.mode` | develop,debug,coverage |
| `xdebug.start_with_request` | trigger |
| `xdebug.log` | /tmp/xdebug.log |
| `xdebug.log_level` | 0 |

### Custom Caddy Modules
- `caddy-cbrotli` - Brotli compression
- `mercure` - Real-time updates
- `vulcain` - HTTP/2 Server Push
- `caddy-cloudflare-ip` - Cloudflare IP handling

### Included Tools
- wkhtmltopdf
- zip, unzip, vim

### PHP Extensions
yaml, bcmath, gmp, mbstring, pdo_mysql, gd, redis, igbinary, opcache, apcu, zip, filter, openssl, ctype, session, dom, xml, tokenizer, simplexml, curl, xdebug, excimer, simdjson, intl

---

## Production

A FrankenPHP-based production image with JIT compilation, aggressive caching, and security hardening.

### Use Cases
- Production web server deployment
- High-performance PHP applications
- Container orchestration (Kubernetes, Docker Swarm)

### Build Arguments

| Argument | Default | Description |
|----------|---------|-------------|
| `USER` | `1000` | UID for the container user |

### Environment Variables

| Variable | Value | Description |
|----------|-------|-------------|
| `GODEBUG` | `cgocheck=0` | Go runtime debug settings |
| `GOMEMLIMIT` | `512MiB` | Go memory limit |
| `GOMAXPROCS` | `0` | Go max processors (auto) |

### Volumes

| Path | Purpose |
|------|---------|
| `/app` | Application source code (workdir) |
| `/etc/secrets` | Secret files (e.g., encryption keys) |
| `/var/applog` | Application logs |
| `/etc/caddy/` | Caddy configuration |

### PHP Configuration

| Setting | Value | Notes |
|---------|-------|-------|
| `memory_limit` | 512M | |
| `max_execution_time` | 60 | seconds |
| `max_input_vars` | 3000 | |
| `post_max_size` | 64M | |
| `upload_max_filesize` | 64M | |
| `opcache.validate_timestamps` | 0 | No file checks (restart to reload) |
| `opcache.memory_consumption` | 256M | |
| `opcache.max_accelerated_files` | 20000 | |
| `opcache.jit` | 1255 | Tracing JIT |
| `opcache.jit_buffer_size` | 128M | |
| `apc.shm_size` | 128M | |
| `apc.ttl` | 0 | No TTL (permanent) |
| `realpath_cache_ttl` | 600 | 10 minutes |

### Healthcheck

The image includes a built-in healthcheck:
- **Endpoint**: `http://localhost/health`
- **Interval**: 30s
- **Timeout**: 10s
- **Start period**: 5s
- **Retries**: 3

> **Note**: Your application must implement a `/health` endpoint that returns HTTP 200.

### Custom Caddy Modules
- `caddy-cbrotli` - Brotli compression
- `mercure` - Real-time updates
- `vulcain` - HTTP/2 Server Push
- `caddy-cloudflare-ip` - Cloudflare IP handling

### Included Tools
- wkhtmltopdf
- zip, unzip

### PHP Extensions
yaml, bcmath, gmp, mbstring, pdo_mysql, gd, redis, igbinary, opcache, apcu, excimer, zip, filter, openssl, ctype, session, dom, xml, tokenizer, simplexml, curl, simdjson, intl

---

## Async Supervised

An Alpine-based PHP CLI image with Supervisor for running long-lived background processes and workers.

### Use Cases
- Queue workers (Laravel Horizon, Symfony Messenger)
- Background job processing
- Scheduled task runners
- WebSocket servers
- Any long-running PHP processes

### Build Arguments

| Argument | Default | Description |
|----------|---------|-------------|
| `USER` | `1000` | UID for the container user |

### Environment Variables

| Variable | Value | Description |
|----------|-------|-------------|
| `HOME` | `/cache` | Home directory |

### Volumes

| Path | Purpose |
|------|---------|
| `/app` | Application source code (workdir) |
| `/cache` | Cache directory |
| `/var/applog` | Application logs |
| `/etc/secrets` | Secret files |

### PHP Configuration

| Setting | Value | Notes |
|---------|-------|-------|
| `memory_limit` | 1024M | Higher for workers |
| `max_execution_time` | 0 | No limit |
| `max_input_time` | -1 | No limit |
| `default_socket_timeout` | -1 | No limit |
| `opcache.enable_cli` | 1 | CLI OPcache enabled |
| `opcache.validate_timestamps` | 0 | No file checks |
| `opcache.memory_consumption` | 256M | |
| `apc.shm_size` | 128M | |
| `apc.ttl` | 0 | No TTL |

### Healthcheck

The image includes a built-in healthcheck:
- **Check**: `pgrep supervisord`
- **Interval**: 30s
- **Timeout**: 10s
- **Retries**: 3

### Default Command

```bash
/usr/bin/supervisord
```

You must provide a Supervisor configuration file. Mount it to `/etc/supervisor/conf.d/` or provide a custom `supervisord.conf`.

### Usage Example

```yaml
# docker-compose.yml
services:
  worker:
    image: async-supervised
    volumes:
      - ./:/app
      - ./supervisor.conf:/etc/supervisor/conf.d/workers.conf
```

### Included Tools
- Supervisor
- Composer
- Node.js + npm + Yarn 4
- Symfony CLI
- Git, zip, jq, vim, bash, openssh

### PHP Extensions
ssh2, intl, xml, opcache, pdo_mysql, iconv, gmp, bcmath, zip, sockets, redis, igbinary, apcu, pcntl, gd, imagick, simdjson

---

## Building the Images

```bash
# Build all images
docker build -t builder ./builder
docker build -t development ./development
docker build -t production ./production
docker build -t async-supervised ./async-supervised

# Build with custom user ID
docker build --build-arg USER=1001 -t production ./production
```

## License

See [LICENSE](LICENSE) for details.
