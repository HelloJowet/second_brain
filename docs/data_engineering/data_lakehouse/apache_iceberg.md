# 🧊 Apache Iceberg

## Rust package

!!! bug

    The code below doesn't work. My feeling is that curently (19.12.2024) the `iceberg_rust` package isn't mature yet. For this reason is switched to [Delta Lake](../delta_lake/delta_rs.md).

Dependencies:

```toml
[dependencies]
iceberg-rust = "0.6.1"
iceberg-sql-catalog = "0.6.1"
object_store = "0.11.1"
tokio = "1.42.0"
```

Code:

```rust
use std::{collections::HashMap, error::Error, sync::Arc};

use iceberg_rust::{
    catalog::Catalog,
    object_store::ObjectStoreBuilder,
    spec::{
        schema::Schema as IcebergSchema,
        types::{PrimitiveType, StructField as IcebergStructField, StructType, Type},
    },
    table::Table as IcebergTable,
};
use iceberg_sql_catalog::SqlCatalog;
use object_store::aws::AmazonS3Builder;

struct Table {
    name: String,
    properties: Option<HashMap<String, String>>,
    struct_fields: Vec<IcebergStructField>,
}

impl Table {
    pub async fn to_iceberg_table(self, catalog: Arc<dyn Catalog>, namespace: &[String]) -> Result<IcebergTable, Box<dyn Error>> {
        let schema = IcebergSchema::builder()
            .with_fields(StructType::builder().fields(self.struct_fields).build().unwrap())
            .build()?;
        let properties = self.properties.unwrap_or(HashMap::new());
        let table = IcebergTable::builder()
            .with_name(self.name)
            .with_schema(schema)
            .with_properties(properties)
            .build(namespace, catalog)
            .await?;

        Ok(table)
    }
}

#[tokio::main]
async fn main() {
    let object_store = ObjectStoreBuilder::S3(
        AmazonS3Builder::new()
            .with_endpoint("http://localhost:5561")
            .with_region("us-east-1")
            .with_bucket_name("data-lakehouse")
            .with_access_key_id("admin")
            .with_secret_access_key("password")
            .with_virtual_hosted_style_request(true),
    );
    let catalog: Arc<dyn Catalog> = Arc::new(
        SqlCatalog::new(
            "postgresql://postgres:postgres@localhost:5500/data_lakehouse_catalog",
            "data_lakehouse_catalog",
            object_store,
        )
        .await
        .expect("Sql catalog creation failed"),
    );

    let table = Table {
        name: "table_01".to_owned(),
        properties: None,
        struct_fields: vec![
            IcebergStructField {
                id: 0,
                name: "id".to_owned(),
                required: true,
                field_type: Type::Primitive(PrimitiveType::Int),
                doc: None,
            },
            IcebergStructField {
                id: 1,
                name: "age".to_owned(),
                required: false,
                field_type: Type::Primitive(PrimitiveType::Int),
                doc: None,
            },
        ],
    };

    let namespace = &["namespace_01".to_owned()];
    table.to_iceberg_table(catalog.clone(), namespace).await.expect("Table creation failed");

    println!("{:?}", catalog);
}
```
