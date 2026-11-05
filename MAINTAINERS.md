# Maintainer release instructions

This file is for publishing a new backend image. It is not required for API
consumers.

## Build and publish

Run these commands from the backend source repository:

```bash
VERSION=1.0.1
IMAGE=ghcr.io/patisto/online-job-recruitment-platform-backend

echo "$GHCR_TOKEN" | docker login ghcr.io -u Patisto --password-stdin
docker build -t "$IMAGE:$VERSION" .
docker push "$IMAGE:$VERSION"
```

Use a new immutable version for every backend release. The registry package
must be public, or the target machine must authenticate to GHCR before pulling.

## Deploy an updated image

On the machine running this repository, update `.env`:

```env
BACKEND_IMAGE_TAG=1.0.1
```

Then run:

```bash
docker compose pull backend
docker compose up -d backend
docker compose ps
curl --fail http://localhost:8080/actuator/health
```

Flyway applies pending database migrations during backend startup. Do not
delete the database or MinIO volumes during an ordinary update.

## Backups

Create a PostgreSQL backup:

```bash
docker compose exec -T db pg_dump \
  -U "$POSTGRES_USER" -d "$POSTGRES_DB" > recruitment-backup.sql
```

Back up the `minio_data` volume or MinIO objects separately. PostgreSQL
backups do not contain uploaded documents.
