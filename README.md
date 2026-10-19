# Ubuntu Nginx web server

Run `docker compose up -d --build` and open http://localhost:8080.
Place your static website files in `web` in this project root. Changes are served immediately.

The `nginx` service uses Ubuntu 24.04 and Ubuntu-packaged Nginx.
Configuration lives in `nginx/default.conf`; rebuild after changing it.

## Configuration

Set HTTP_PORT to change the published port (default 8080).
Set UBUNTU_VERSION to change the Ubuntu base (default 24.04).
These values can be placed in a local `.env` file, for example:

```
HTTP_PORT=80
UBUNTU_VERSION=24.04
```

## Upgrades

Run `docker compose build --pull --no-cache nginx` then `docker compose up -d nginx`.
This refreshes the Ubuntu base and installs the available Nginx/security updates.
Your website remains in the host `web` directory across rebuilds and container replacement.
Review Ubuntu and Nginx compatibility before changing Ubuntu versions.

## Operations

- Logs: `docker compose logs -f nginx`
- Configuration check: `docker compose exec nginx nginx -t`
- Stop and remove containers: `docker compose down`

The default setup serves static files over HTTP. HTTPS and application runtimes require additional configuration.

## Production deployment

`compose.production.yaml` is standalone; do not combine it with the development Compose file.
The Dockerfile includes `web` in the image. Local development still uses the bind mount.
Build the image only after the complete website build has been copied into `web`.

Have GitHub Actions build and push this Dockerfile to GHCR, tagged with the Git commit SHA.
On the production server, put the exact image reference in `.env`:

```
WEB_IMAGE=ghcr.io/your-owner/210tools:your-commit-sha
HTTP_PORT=80
HTTP_BIND_ADDRESS=0.0.0.0
```

Deploy with:

```
docker compose -f compose.production.yaml pull
docker compose -f compose.production.yaml up -d --wait
```

Authenticate the server to GHCR first if the image is private.
To roll back, set WEB_IMAGE to the previous release tag and run the deployment commands again.
When using a reverse proxy on the same host, set HTTP_BIND_ADDRESS=127.0.0.1 and
HTTP_PORT=8080. Configure HTTPS at the reverse proxy or load balancer.
The production service checks HTTP availability and rotates container logs.
