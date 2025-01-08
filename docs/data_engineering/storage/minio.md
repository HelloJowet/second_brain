# 🪣 Minio

## Docker compose

`docker_compose.yaml`:

```yaml
services:
  storage_s3:
    restart: always
    image: quay.io/minio/minio:RELEASE.2024-10-29T16-01-48Z
    ports:
      - 5560:5560
      - 5561:5561
    hostname: storage-s3
    environment:
      MINIO_ROOT_USER: admin
      MINIO_ROOT_PASSWORD: password
    command: server /data --console-address ":5560" --address=":5561"
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:5560/minio/health/live"]
      interval: 5s
      timeout: 5s
      retries: 5

  storage_s3_initial_setup:
    image: minio/mc:RELEASE.2024-10-29T15-34-59Z
    depends_on:
      storage_s3:
        condition: service_healthy
    volumes:
      - ./docker_entrypoint.sh:/docker_entrypoint.sh:z
    entrypoint:
      - /docker_entrypoint.sh
```

`docker_entrypoint.sh`:

```sh
#!/bin/bash

# Set up alias for MinIO
mc alias set minio http://storage-s3:5561 admin password;

# Create buckets
mc mb minio/data-lakehouse;
```
