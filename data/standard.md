# Data Structure


## Data Structure

```rust
pub struct Data {
    /// ID of the Data Object
    id: Uuid,

    /// Version of the Data Object. Used in conjunction with `PocketDimension`
    version: u32,

    /// Timestamp of the Data Object upon creation
    timestamp: DateTime<Utc>,

    /// Size of the Data Object
    size: u64,

    /// Module that the Data Object and payload is utilizing
    data_type: DataType,

    /// Data that is stored for the Data Object.
    payload: Value,
}

```

## Data Type

```rust
pub enum DataType {
    Messaging,
    FileSystem,
    Accounts,
    Cache,
    Http,
    DataExport,
    Unknown,
}
```