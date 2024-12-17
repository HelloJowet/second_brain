# delta-rs

This guide provides an introduction to using the [Delta Lake Rust library (delta-rs)](https://github.com/delta-io/delta-rs).

## Install dependencies

```toml
[dependencies]
deltalake = { version = "0.22.3", features = ["datafusion", "s3"] }
tokio = "1.42.0"
```

## Initial setup

```rust
use deltalake::{DeltaOps, DeltaTableBuilder, DeltaTableError};
use std::env;

fn configure_s3() {
    // Set S3 configuration options using environment variables
    env::set_var("AWS_ENDPOINT_URL", "http://localhost:5561");
    env::set_var("AWS_REGION", "us-east-1");
    env::set_var("AWS_ACCESS_KEY_ID", "admin");
    env::set_var("AWS_SECRET_ACCESS_KEY", "password");
    env::set_var("AWS_ALLOW_HTTP", "true");
    env::set_var("AWS_S3_ALLOW_UNSAFE_RENAME", "true");

    // Register AWS S3 handlers for Delta Lake operations
    deltalake::aws::register_handlers(None);
}

/// Builds a `DeltaOps` instance for the specified Delta table.
/// Enabling operations such as creating, reading and writing data in the Delta Lake format.
async fn get_delta_ops(table_name: &str, load_state: bool) -> Result<DeltaOps, DeltaTableError> {
    let delta_table_builder = DeltaTableBuilder::from_uri(format!("s3://data-lakehouse/{}", table_name));
    let delta_table = match load_state {
        // Load the existing table state
        true => delta_table_builder.load().await?,
        // Build the table without loading existing state
        false => delta_table_builder.build()?,
    };

    Ok(DeltaOps::from(delta_table))
}

#[tokio::main()]
async fn main() {
    configure_s3()
}
```

If the table doesn't exist yet, the `load_state` parameter in `get_delta_ops` should be set to `false`, as setting it to `true` would attempt to read a non-existent state, resulting in an error. On the other hand, if you want to read from an existing table, `load_state` must be set to `true` to successfully load the data; otherwise, the load operation will fail.

## Create table

```rust
use deltalake::kernel::DataType;
use deltalake::{DeltaTable, DeltaTableError};

// ...

async fn create_table(table_name: &str) -> Result<DeltaTable, DeltaTableError> {
    let delta_ops = get_delta_ops(table_name, false)?;

    let table = delta_ops
        .create()
        .with_table_name(table_name)
        .with_column("id", DataType::INTEGER, false, Default::default())
        .with_column("name", DataType::STRING, false, Default::default())
        .await?;

    Ok(table)
}

#[tokio::main()]
async fn main() {
    configure_s3()

    let table_name = "employee";
    create_table(&table_name).await.expect("Table creation failed");
}
```

## Insert

```rust
use deltalake::arrow::array::{Int32Array, RecordBatch, StringArray};
use deltalake::arrow::datatypes::{DataType as ArrowDataType, Field, Schema as ArrowSchema};
use deltalake::protocol::SaveMode;
use deltalake::{DeltaTable, DeltaTableError};
use std::sync::Arc;

// ...

async fn insert(table_name: &str, save_mode: SaveMode) -> Result<DeltaTable, DeltaTableError> {
    // Define the schema for the record
    let schema = Arc::new(ArrowSchema::new(vec![
        Field::new("int", ArrowDataType::Int32, false),
        Field::new("string", ArrowDataType::Utf8, true),
    ]));

    // Create a employee record
    let ids = Int32Array::from(vec![1, 2, 3]);
    let names = StringArray::from(vec!["Tom", "Tim", "Titus"]);
    let employee_record = RecordBatch::try_new(schema, vec![Arc::new(ids), Arc::new(names)]).unwrap();

    let delta_ops = get_delta_ops(table_name, false)?;
    // Insert record
    let table = delta_ops.write(vec![employee_record]).with_save_mode(save_mode).await?;

    Ok(table)
}

#[tokio::main()]
async fn main() {
    // ...

    let table_name = "employee";
    insert(&table_name, SaveMode::Append).await.expect("Insert failed");
}
```

The default save mode for the `delta_ops.write` function is `SaveMode::Append`. To overwrite existing data instead of appending, use `SaveMode::Overwrite`.

## Read

```rust
use deltalake::arrow::array::RecordBatch;
use deltalake::operations::collect_sendable_stream;
use deltalake::DeltaTableError;

// ...

async fn read_01(table_name: &str) -> Result<Vec<RecordBatch>, DeltaTableError> {
    let delta_ops = get_delta_ops(table_name, true).await?;

    let (_, stream) = delta_ops.load().await?;
    let employee_records: Vec<RecordBatch> = collect_sendable_stream(stream).await?;

    Ok(employee_records)
}

#[tokio::main()]
async fn main() {
    // ...

    let table_name = "employee";
    let employee_records = read_01(&table_name).await.expect("Read failed");
    println!("employee_records: {:?}", employee_records);
}
```

or

```rust
use deltalake::{DeltaTable, DeltaTableError};

async fn read_02(table_path: &str) -> Result<DeltaTable, DeltaTableError> {
    let table = deltalake::open_table(table_path).await?;

    Ok(table)
}

#[tokio::main()]
async fn main() {
    // ...

    let table_path = "s3://data-lakehouse/employee";
    let table = read_02(table_path).await.expect("Read failed");
    println!("{:?}", table);
}
```
