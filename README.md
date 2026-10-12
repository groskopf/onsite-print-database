# onsite-print-database

A PostgreSQL image for onsite-print, published to GitHub Container Registry (GHCR).
The Dockerfile currently uses the stock `postgres:18-alpine` image.

## Set the database password

The password is a Docker secret read from `secrets/postgres_password`. This file
is ignored by git and must be created before the first start:

```sh
openssl rand -base64 32 > secrets/postgres_password
```

Postgres only reads the password the first time it initializes the `postgres-data`
volume. Changing the file later has no effect unless the volume is recreated.

The database and user are both named `onsite_print`. Postgres is published on
`127.0.0.1:5432` only. Other Compose services can reach it at `postgres:5432`.

## Publish a version

Create a GitHub personal access token (classic) with `write:packages` permission
and access to the `groskopf` namespace. Set `CR_PAT` securely in your environment
(do not commit it). If the token belongs to a different GitHub user, also set
`GHCR_USERNAME` to that user's login; it defaults to `groskopf`.

The `.env` file contains the image version, e.g. `IMAGE_TAG=1.0.0`. Update it
with a new tag for each release. Keep it limited to this single setting; do not
store credentials there.

```sh
bash docker-push.sh
```

The script logs in to GHCR, builds the Dockerfile with the latest base image, and
pushes `ghcr.io/groskopf/onsite-print-database:<IMAGE_TAG>`. It stops if any step fails.

## Run

```sh
docker compose up -d            # build locally
docker compose up -d --no-build # use the published image
```

## Test the publishing script

```sh
bash tests/docker-push-test.sh
```

These tests use a mocked Docker CLI and do not build or upload images.
GitHub Actions runs them on pushes and pull requests against this repo's own
publishing script; no credentials or sibling repositories are needed.
