# nginx-cache

Small Docker Compose project that runs an **NGINX reverse proxy with caching** in front of an upstream API.

## Quick start

1. Create your environment file:

```bash
cp .env.example .env
```

2. Edit `.env` and set:
   - `API_TARGET`: upstream base URL (example: `https://some.api.com/`)
   - `API_HOST`: host header to send to the upstream (example: `some.api.com`)
   - `PORT` (optional): local port to expose the proxy on (default: `8080`)

3. Start the proxy:

```bash
docker compose up -d
```

4. Send requests via the proxy:
   - Proxy listens on `http://localhost:${PORT:-8080}`

## How caching works

- Only **GET** requests are cached (non-GET requests bypass cache).
- Cache key includes the `Authorization` header:
  - Requests with different `Authorization` values are cached separately.
- You can bypass caching by sending `X-Purge: <anything>` (this **bypasses** cache for that request; it does not actively delete cached entries).

## Configuration

The container is configured via environment variables (see `compose.yaml`):

- `PORT` (default: `8080`): local port that maps to container port `80`
- `CACHE_VALID_TIME` (default: `30m`): cache TTL for `200`/`302`
- `CACHE_MAX_SIZE` (default: `2g`): max cache size on disk
- `CACHE_INACTIVE` (default: `120m`): how long unused items stay in cache

Cached responses are stored in the Docker volume `cache_data`.

## Debugging / verifying

The proxy adds helpful response headers:

- `X-Cache-Status`: `MISS`, `HIT`, `BYPASS`, etc.
- `X-Proxy-Target`: the configured upstream target

View logs:

```bash
docker compose logs -f
```

Stop and remove the container:

```bash
docker compose down
```

To also remove the cache volume:

```bash
docker compose down -v
```
