# Backend image release instructions

The deployment repository consumes the backend image from GitHub Container
Registry (GHCR). The package must be public so a fresh machine can pull it
without registry credentials.

## Build and publish the backend image

From the backend source repository:

```bash
cd /home/patisto/Projects/online-job-recruitment-platform
VERSION=1.0.1
IMAGE=ghcr.io/codekaren777/online-job-recruitment-platform-backend

docker build -t "$IMAGE:$VERSION" .
echo "$GHCR_TOKEN" | docker login ghcr.io -u Patisto --password-stdin
docker push "$IMAGE:$VERSION"
```

`GHCR_TOKEN` must be a GitHub token with `write:packages` permission. Do not
commit the token or put it in `.env`.

## Make the package public

Open the package page after the first push:

```text
https://github.com/Patisto?tab=packages
```

Open the package settings, choose **Change visibility**, select **Public**, and
confirm the warning. A public package can be pulled without `docker login`.

## Deploy an updated image

On the machine running this repository, update `.env`:

```env
BACKEND_IMAGE_REPOSITORY=ghcr.io/codekaren777/online-job-recruitment-platform-backend
BACKEND_IMAGE_TAG=1.0.1
```

```bash
docker compose pull backend
docker compose up -d backend
docker compose ps
curl --fail http://localhost:8080/actuator/health
```

Flyway applies pending migrations during backend startup. Do not delete the
PostgreSQL or MinIO volumes during an ordinary update.

## Backups

Create a PostgreSQL backup:

```bash
docker compose exec -T db pg_dump \
  -U "$POSTGRES_USER" -d "$POSTGRES_DB" > recruitment-backup.sql
```

Back up the `minio_data` volume or MinIO objects separately. PostgreSQL
backups do not contain uploaded documents.
