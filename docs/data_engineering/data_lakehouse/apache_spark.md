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

## Python library

-> [Minio](../../dev_ops/services/minio.md) as local s3 service

### Apache Iceberg integration

```python
from pyspark.sql import SparkSession

spark = (
    SparkSession.builder.master('spark://localhost:7077')
    .config(
        'spark.jars.packages',
        'org.apache.iceberg:iceberg-spark-runtime-3.5_2.12:1.7.1,' 'org.apache.iceberg:iceberg-aws-bundle:1.7.1,' 'org.postgresql:postgresql:42.7.4',
    )
    .config('spark.sql.extensions', 'org.apache.iceberg.spark.extensions.IcebergSparkSessionExtensions')
    .config('spark.sql.catalog.my_catalog', 'org.apache.iceberg.spark.SparkCatalog')
    .config('spark.sql.catalog.my_catalog.type', 'hadoop')
    .config('spark.sql.catalog.my_catalog.type', 'jdbc')
    .config('spark.sql.catalog.my_catalog.uri', 'jdbc:postgresql://localhost:5500/postgres')
    .config('spark.sql.catalog.my_catalog.jdbc.user', 'postgres')
    .config('spark.sql.catalog.my_catalog.jdbc.password', 'postgres')
    .config('spark.sql.catalog.my_catalog.io-impl', 'org.apache.iceberg.aws.s3.S3FileIO')
    .config('spark.sql.catalog.my_catalog.warehouse', 's3://data-lakehouse')
    .config('spark.sql.catalog.my_catalog.s3.region', 'us-east-1')
    .config('spark.sql.catalog.my_catalog.s3.endpoint', 'http://YOUR_IP_ADDRESS:5561')
    .config('spark.sql.catalog.my_catalog.s3.access-key-id', 'admin')
    .config('spark.sql.catalog.my_catalog.s3.secret-access-key', 'password')
    .getOrCreate()
)

spark.sql('CREATE TABLE my_catalog.table (name string) USING iceberg;')
spark.sql("INSERT INTO my_catalog.table VALUES ('Alex'), ('Dipankar'), ('Jason')")
```

### Apache Iceberg + Sedona

```python
from sedona.spark import SedonaContext

spark = (
    SedonaContext.builder()
    .master('spark://localhost:7077')
    .config(
        'spark.jars.packages',
        # sedona
        'org.apache.sedona:sedona-spark-3.5_2.12:1.7.0,'
        'org.datasyslab:geotools-wrapper:1.7.0-28.5,'
        # iceberg
        'org.apache.iceberg:iceberg-spark-runtime-3.5_2.12:1.7.1,'
        'org.apache.iceberg:iceberg-aws-bundle:1.7.1,'
        'org.postgresql:postgresql:42.7.4',
    )
    .config('spark.sql.extensions', 'org.apache.iceberg.spark.extensions.IcebergSparkSessionExtensions')
    .config('spark.sql.catalog.my_catalog', 'org.apache.iceberg.spark.SparkCatalog')
    .config('spark.sql.catalog.my_catalog.type', 'jdbc')
    .config('spark.sql.catalog.my_catalog.uri', 'jdbc:postgresql://localhost:5500/postgres')
    .config('spark.sql.catalog.my_catalog.jdbc.user', 'postgres')
    .config('spark.sql.catalog.my_catalog.jdbc.password', 'postgres')
    .config('spark.sql.catalog.my_catalog.io-impl', 'org.apache.iceberg.aws.s3.S3FileIO')
    .config('spark.sql.catalog.my_catalog.warehouse', 's3://data-lakehouse')
    .config('spark.sql.catalog.my_catalog.s3.region', 'us-east-1')
    .config('spark.sql.catalog.my_catalog.s3.endpoint', 'http://YOUR_IP_ADDRESS:5561')
    .config('spark.sql.catalog.my_catalog.s3.access-key-id', 'admin')
    .config('spark.sql.catalog.my_catalog.s3.secret-access-key', 'password')
    .config('spark.sql.catalog.my_catalog.s3.path-style-access', 'true')
    .getOrCreate()
)

spark.sql('CREATE TABLE my_catalog.table8 (name string) USING iceberg;')
spark.sql("INSERT INTO my_catalog.table8 VALUES ('Alex'), ('Dipankar'), ('Jason')")
```
