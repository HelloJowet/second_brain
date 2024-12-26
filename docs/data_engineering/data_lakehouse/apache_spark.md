# ⭐ Apache Spark

## Deployment

### Docker compose

```yaml
services:
  spark_master:
    restart: always
    image: bitnami/spark:3.5
    ports:
      - 8080:8080
      - 7077:7077
    hostname: spark-master
    environment:
      - SPARK_MODE=master
      - SPARK_RPC_AUTHENTICATION_ENABLED=no
      - SPARK_RPC_ENCRYPTION_ENABLED=no
      - SPARK_LOCAL_STORAGE_ENCRYPTION_ENABLED=no
      - SPARK_SSL_ENABLED=no
      - SPARK_WORKER_WEBUI_PORT=8081

  spark_worker:
    restart: always
    image: bitnami/spark:3.5
    ports:
      - 8081:8081
    environment:
      - SPARK_MODE=worker
      - SPARK_MASTER_URL=spark://spark-master:7077
      - SPARK_WORKER_MEMORY=8G
      - SPARK_WORKER_CORES=4
      - AWS_ACCESS_KEY_ID=user
      - AWS_SECRET_ACCESS_KEY=password
      - AWS_REGION=us-east-1
    depends_on:
      - spark_master
```
