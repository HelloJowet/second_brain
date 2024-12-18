# Create Table

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
