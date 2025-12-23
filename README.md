# SSL-enabled Postgres DB image for Postgres 18

This repository contains the logic to build SSL-enabled Postgres images.

By default, when you deploy Postgres from the official Postgres template on
Railway, the image that is used is built from this repository!

[![Deploy on
Railway](https://railway.app/button.svg)](https://railway.com/deploy/postgres-18?referralCode=2ZivV-)

## Bump to 18

The issue with the official Railway image for PostgreSQL is that it lags behind and didn't introduce integration with 18.x versions. We fix that by forking the [official template](https://github.com/railwayapp-templates/postgres-ssl) and making additional changes.

## SSL

### Why SSL mod though?

The official Postgres image in Docker hub does not come with SSL baked in.

Since this could pose a problem for applications or services attempting to
connect to Postgres services, we decided to roll our own Postgres image with SSL
enabled right out of the box.

### How does it work?

The Dockerfiles contained in this repository start with the official Postgres
image as base. Then the `init-ssl.sh` script is copied into the
`docker-entrypoint-initdb.d/` directory to be executed upon initialization.

### Certificate expiry

By default, the cert expiry is set to 820 days. You can control this by
configuring the `SSL_CERT_DAYS` environment variable as needed.

### Certificate renewal

When a redeploy or restart is done the certificates expiry is checked, if it has
expired or will expire in 30 days a new certificate is automatically generated.

## Docker image tags

Images are automatically built weekly and tagged with multiple version levels
for flexibility:

- **Major version tags** (e.g., `:18`, `:17`, `:16`): Always points to the
  latest minor version for that major release
- **Minor version tags** (e.g., `:18.1`): Pins to specific minor
  version for stability
- **Latest tag** (`:latest`): Currently points to PostgreSQL 18

Example usage:

```bash
# Auto-update to latest minor versions (recommended for development)
docker run xlab/postgres-ssl:18

# Pin to specific minor version (recommended for production)
docker run xlab/postgres-ssl:18.1
```

## Caveats

### A note about ports

By default, this image is hardcoded to listen on port `5432` regardless of what
is set in the `PGPORT` environment variable. We did this to allow connections
to the postgres service over the `RAILWAY_TCP_PROXY_PORT`. If you need to
change this behavior, feel free to build your own image without passing the
`--port` parameter to the `CMD` command in the Dockerfile.
