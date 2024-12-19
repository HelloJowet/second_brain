description: delta-rs, rust, insert data

# Insert data

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

## Overwrite

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
