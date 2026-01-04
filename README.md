# PDF Service

A production ready PDF generation service powered by [Gotenberg](https://gotenberg.dev/), with API key authentication via Caddy.

## Overview

This setup provides:

- **Gotenberg** — Converts HTML, URLs, and Office files to PDF using headless Chromium
- **Caddy** — Reverse proxy handling SSL termination and API key authentication

## Quick Start

1. Clone this repo
2. Update `Caddyfile` with your domain
3. Add your API keys to `api_keys.caddy`
4. Start the containers:

```bash
docker compose up -d
```

Caddy automatically provisions SSL certificates via Let's Encrypt.

## Configuration

### API Keys

Add or remove keys in `api_keys.caddy`:

```
map {header.X-Api-Key} {is_authorized} {
    "your-api-key-here"     "true"
    "another-api-key"       "true"
    
    default "false"
}
```

You can generate secure keys using this:

```bash
openssl rand -hex 16
```

After updating, hot reload Caddy without downtime:

```bash
docker compose exec caddy caddy reload --config /etc/caddy/Caddyfile
```

### Domain

Update the domain in `Caddyfile`:
```
pdf.example.com {
    # ...
}
```

## Usage

Send requests with your API key in the `X-Api-Key` header:
```bash
# Health check
curl https://pdf.example.com/health

# Generate PDF from HTML
curl -X POST \
  -H "X-Api-Key: your-api-key-here" \
  -F "files=@index.html;filename=index.html" \
  https://pdf.example.com/forms/chromium/convert/html \
  -o output.pdf
```

See [Gotenberg documentation](https://gotenberg.dev/docs/routes) for all available endpoints.

## PHP Integration

Use the official [gotenberg-php](https://github.com/gotenberg/gotenberg-php) package to integrate with your PHP application:

```bash
composer require gotenberg/gotenberg-php
```

For API key authentication, add a custom HTTP client that includes the `X-Api-Key` header with each request.

## Resource Limits

Default limits in `docker-compose.yml`:

| Service   | Memory | CPU |
|-----------|--------|-----|
| Caddy     | 256MB  | -   |
| Gotenberg | 2GB    | 2   |

Adjust based on your workload.

## Notes

- LibreOffice routes are disabled by default. Enable in `docker-compose.yml` if you need Office file conversion.
- Gotenberg is not exposed directly—all traffic goes through Caddy.

