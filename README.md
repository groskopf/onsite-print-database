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

Publishing requires a `major.minor.patch` tag without leading zeros. The script
publishes the current version, then increments the patch in `.env` only after
the push succeeds (`0.0.1` -> `0.0.2`). Failures leave the version unchanged.
Commit the updated `.env`; its new version is the next release, not yet published.
Use the printed published tag for deployment. Do not publish concurrently or
reuse stale version files: automatic increments do not prevent registry tag
overwrites from stale checkouts or interrupted-push retries.

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

Alternatively, trigger the **Publish release** GitHub Actions workflow from
the Actions tab (`workflow_dispatch` only, runs from `main`). It reads the
pending `IMAGE_TAG` from `.env`, builds and pushes the image to GHCR using the
built-in `GITHUB_TOKEN`, tags the published commit on `main` with that version
(no `v` prefix), and commits the incremented `.env` back to `main`. This
requires a one-time repository setting: allow the GitHub Actions bot to
bypass `main`'s branch protection for direct pushes, since the workflow pushes
a tag and a commit straight to `main` without a pull request.

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
