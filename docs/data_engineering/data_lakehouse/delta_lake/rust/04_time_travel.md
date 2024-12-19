description: delta-rs, rust, time travel

# Time travel

To load the previous state of a table, you can use the `open_table_with_version` function:

```rust
let version = 1;
let mut table = deltalake::open_table_with_version("s3://data-lakehouse/employee", version).await.expect("Load failed");
```

If the table is already loaded and you want to change the version number, just use the `load_version` function.

```rust
table.load_version(2).await.expect("Load failed");
```
