# delta-rs

## Create table

```rust
use deltalake::{kernel::DataType, DeltaOps};

#[tokio::main()]
async fn main() {
    let delta_ops = DeltaOps::new_in_memory();

    let table = delta_ops
        .create()
        .with_table_name("table_01")
        .with_column("id", DataType::INTEGER, false, Default::default())
        .with_column("name", DataType::STRING, false, Default::default())
        .await
        .expect("Table creation failed");
}
```

## Insert data

```rust
use deltalake::arrow::array::{Int32Array, RecordBatch, StringArray};
use deltalake::arrow::datatypes::{DataType as ArrowDataType, Field, Schema as ArrowSchema};
use deltalake::DeltaOps;
use std::sync::Arc;

#[tokio::main()]
async fn main() {
    // ...

    let schema = Arc::new(ArrowSchema::new(vec![
        Field::new("id", ArrowDataType::Int32, false),
        Field::new("name", ArrowDataType::Utf8, true),
    ]));

    // Create employee records
    let ids = Int32Array::from(vec![1, 2, 3]);
    let names = StringArray::from(vec!["Tom", "Tim", "Titus"]);
    let employee_record = RecordBatch::try_new(schema, vec![Arc::new(ids), Arc::new(names)]).unwrap();

    // Insert records
    let table = DeltaOps(table).write(vec![employee_record]).await.expect("Insert failed");
}
```

> The Arrow Rust array primitives are _very_ fickle and so creating a direct transformation is quite tricky in Rust, whereas in Python or another loosely typed language it might be simpler.

[(source)](https://github.com/delta-io/delta-rs/blob/99e39ca1ca372211cf7b90b62d33878fa961881c/crates/deltalake/examples/recordbatch-writer.rs#L156)

The default save mode for the `delta_ops.write` function is `SaveMode::Append`. To overwrite existing data instead of appending, use `SaveMode::Overwrite`:

```rust
use deltalake::protocol::SaveMode;
use deltalake::DeltaOps;

#[tokio::main()]
async fn main() {
    // ...

    let table = DeltaOps(table)
        .write(vec![employee_record])
        .with_save_mode(SaveMode::Overwrite)
        .await
        .expect("Insert failed");
}
```

## Load table

Open table:

```rust

#[tokio::main()]
async fn main() {
    // ...

    let table = deltalake::open_table("s3://data-lakehouse/employee").await.expect("Load failed");
}
```

Load table data:

```rust
use deltalake::operations::collect_sendable_stream;
use deltalake::DeltaOps;

#[tokio::main()]
async fn main() {
    // ...

    let (_, stream) = DeltaOps(table).load().await.expect("Load failed");
    let records = collect_sendable_stream(stream).await.unwrap();

    println!("{:?}", records)
}
```

### S3 storage

```rust
use std::collections::HashMap;

#[tokio::main()]
async fn main() {
    // Register AWS S3 handlers for Delta Lake operations
    deltalake::aws::register_handlers(None);

    let mut storage_options = HashMap::new();
    storage_options.insert("AWS_ENDPOINT_URL".to_string(), "http://localhost:5561".to_string());
    storage_options.insert("AWS_REGION".to_string(), "us-east-1".to_string());
    storage_options.insert("AWS_ACCESS_KEY_ID".to_string(), "admin".to_string());
    storage_options.insert("AWS_SECRET_ACCESS_KEY".to_string(), "password".to_string());
    storage_options.insert("AWS_ALLOW_HTTP".to_string(), "true".to_string());
    storage_options.insert("AWS_S3_ALLOW_UNSAFE_RENAME".to_string(), "true".to_string());

    let table = deltalake::open_table_with_storage_options("s3://data-lakehouse/employee", storage_options)
        .await
        .expect("Load failed");
}
```

You can set the storage option parameters as environment variables too.

S3 requires a locking provider by default ([more information](https://delta-io.github.io/delta-rs/usage/writing/writing-to-s3-with-locking-provider/)). If you don't want to use a locking provider, you can disable it by setting the `AWS_S3_ALLOW_UNSAFE_RENAME` variable to `true`.

## Time travel

To load the previous state of a table, you can use the `open_table_with_version` function:

```rust
let version = 1;
let mut table = deltalake::open_table_with_version("s3://data-lakehouse/employee", version).await.expect("Load failed");
```

If the table is already loaded and you want to change the version number, just use the `load_version` function.

```rust
table.load_version(2).await.expect("Load failed");
```
