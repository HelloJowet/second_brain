# delta-rs

This guide provides an introduction to using the Delta Lake Rust library (delta-rs).

## Install dependencies

```toml
[dependencies]
deltalake = { version = "0.22.3", features = ["s3"] }
tokio = "1.42.0"
```

## Create table

```rust
use deltalake::kernel::DataType;
use deltalake::operations::create::CreateBuilder;
use std::collections::HashMap;

#[tokio::main()]
async fn main() {
    // Register AWS S3 handlers for Delta Lake operations.
    deltalake::aws::register_handlers(None);

    // Configure S3 storage parameters
    let mut storage_options = HashMap::new();
    storage_options.insert("AWS_ENDPOINT_URL".to_string(), "http://localhost:5561".to_string());
    storage_options.insert("AWS_REGION".to_string(), "us-east-1".to_string());
    storage_options.insert("AWS_ACCESS_KEY_ID".to_string(), "admin".to_string());
    storage_options.insert("AWS_SECRET_ACCESS_KEY".to_string(), "password".to_string());
    storage_options.insert("AWS_ALLOW_HTTP".to_string(), "true".to_string());
    storage_options.insert("AWS_S3_ALLOW_UNSAFE_RENAME".to_string(), "true".to_string());

    // Create table
    CreateBuilder::new()
        .with_location("s3://data-lakehouse/".to_string())
        .with_storage_options(storage_options)
        .with_column("id", DataType::INTEGER, false, Default::default())
        .with_column("name", DataType::STRING, false, Default::default())
        .await
        .expect("Table creation failed");
}

```
