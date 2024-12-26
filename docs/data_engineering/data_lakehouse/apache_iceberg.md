# 🧊 Apache Iceberg

## Rust package

### Dependencies

```toml
[dependencies]
iceberg = "0.4.0"
tokio = "1.42.0"
```

### Connect to Catalog

#### Memory Catalog

```toml
[dependencies]
iceberg-catalog-memory = "0.4.0"
```

```rust
use iceberg::io::FileIOBuilder;
use iceberg_catalog_memory::MemoryCatalog;

#[tokio::main()]
async fn main() {
    let file_io = FileIOBuilder::new("memory").build().expect("Failed to build file io");
    let catalog = MemoryCatalog::new(file_io, None);
}
```

### Create namespace

```rust
use std::collections::HashMap;

use iceberg::NamespaceIdent;

#[tokio::main()]
async fn main() {
    // ...

    let namespace_ident = NamespaceIdent::new("namespace_01".to_string());
    let namespace = catalog
        .create_namespace(&namespace_ident, HashMap::new())
        .await
        .expect("Failed to create namespace");
}
```

### Create table

```rust
use std::collections::HashMap;

use iceberg::{
    spec::{NestedField, PrimitiveType, Schema, Type},
    TableCreation,
};

#[tokio::main()]
async fn main() {
    // ...

    let table_schema = Schema::builder()
        .with_fields(vec![
            NestedField::optional(1, "foo", Type::Primitive(PrimitiveType::String)).into(),
            NestedField::required(2, "bar", Type::Primitive(PrimitiveType::Int)).into(),
            NestedField::optional(3, "baz", Type::Primitive(PrimitiveType::Boolean)).into(),
        ])
        .with_schema_id(1)
        .with_identifier_field_ids(vec![2])
        .build()
        .unwrap();

    let table_creation = TableCreation::builder()
        .name("table_01".to_string())
        .location("table_01".to_string())
        .schema(table_schema.clone())
        .properties(HashMap::from([("owner".to_string(), "Jonas Frei".to_string())]))
        .build();

    let table = catalog.create_table(&namespace_ident, table_creation).await.unwrap();

    println!("Table created: {:?}", table.metadata());
}
```

### Load table

```rust
use iceberg::TableIdent;

#[tokio::main()]
async fn main() {
    // ...

    let table_created = catalog
        .load_table(&TableIdent::from_strs(["namespace_01", "table_01"]).unwrap())
        .await
        .unwrap();
    println!("{:?}", table_created.metadata());
}
```
