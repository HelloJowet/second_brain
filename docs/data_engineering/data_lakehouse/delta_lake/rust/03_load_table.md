# Load table

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

or

```rust

#[tokio::main()]
async fn main() {
    // ...

    let table = deltalake::open_table("s3://data-lakehouse/employee").await.expect("Load failed");
}
```