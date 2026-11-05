# Local image instructions

These instructions build and run the backend image locally. They do not use
GHCR, registry tokens, or an internet image registry.

## Build the backend image

From the backend source repository:

```bash
cd /home/patisto/Projects/online-job-recruitment-platform
docker build -t online-job-backend:1.0.0 .
```

The tag must match `BACKEND_IMAGE_TAG` in the deployment repository's `.env`.
The default local configuration uses:

```env
BACKEND_IMAGE_REPOSITORY=online-job-backend
BACKEND_IMAGE_TAG=1.0.0
```

Verify that Docker has the image:

```bash
docker image inspect online-job-backend:1.0.0
```

## Start the complete local stack

From the deployment repository:

```bash
cd /home/patisto/Projects/online-job-recruitment-platform-deployment
cp .env.example .env
```

Replace the `REPLACE_*` values in `.env`, then start the services:

```bash
docker compose up -d --build
docker compose ps
curl --fail http://localhost:8080/actuator/health
```

`--build` builds the MinIO helper images in this repository. It does not build
the backend; Compose uses the local image created in the previous step.

## Update the local backend image

After changing the backend source code, choose a new local tag:

```bash
cd /home/patisto/Projects/online-job-recruitment-platform
docker build -t online-job-backend:1.0.1 .
```

Update the deployment repository `.env`:

```env
BACKEND_IMAGE_REPOSITORY=online-job-backend
BACKEND_IMAGE_TAG=1.0.1
```

Restart only the backend:

```bash
cd /home/patisto/Projects/online-job-recruitment-platform-deployment
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

## Optional future registry publishing

This local workflow does not require GHCR. If the image later needs to run on
another machine, publish it through a registry or transfer it as an image
archive; that is a separate distribution step.
