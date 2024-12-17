# delta-rs

This guide provides an introduction to using the Delta Lake Rust library (delta-rs).

## Install dependencies

```toml
[dependencies]
deltalake = { version = "0.22.3", features = ["s3"] }
```

## Create table

```rust
use deltalake::kernel::DataType;
use deltalake::{DeltaOps, DeltaTable, DeltaTableBuilder, DeltaTableError};
use std::env;

/// Builds a `DeltaOps` instance for the specified Delta table.
/// Enabling operations such as creating, reading, and writing data in the Delta Lake format.
fn get_delta_ops(table_name: &str) -> Result<DeltaOps, DeltaTableError> {
    let delta_table = DeltaTableBuilder::from_uri(format!("s3://data-lakehouse/{}", table_name)).build()?;

    Ok(DeltaOps::from(delta_table))
}

async fn create_table(table_name: &str) -> Result<DeltaTable, DeltaTableError> {
    let delta_ops = get_delta_ops(table_name)?;

    let table = delta_ops
        .create()
        .with_table_name("employee")
        .with_column("id", DataType::INTEGER, false, Default::default())
        .with_column("name", DataType::STRING, false, Default::default())
        .await?;

    Ok(table)
}

#[tokio::main()]
async fn main() {
    let table_name = "employee";

    // Set S3 configuration options using environment variables
    env::set_var("AWS_ENDPOINT_URL", "http://localhost:5561");
    env::set_var("AWS_REGION", "us-east-1");
    env::set_var("AWS_ACCESS_KEY_ID", "admin");
    env::set_var("AWS_SECRET_ACCESS_KEY", "password");
    env::set_var("AWS_ALLOW_HTTP", "true");
    env::set_var("AWS_S3_ALLOW_UNSAFE_RENAME", "true");

    // Register AWS S3 handlers for Delta Lake operations.
    deltalake::aws::register_handlers(None);

    create_table(&table_name).await.expect("Table creation failed");
}
```
