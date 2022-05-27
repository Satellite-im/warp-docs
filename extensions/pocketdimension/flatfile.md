## Flatfile Extension Overview

## Importing extension into cargo project

In your cargo project add the following

```
[dependencies]
warp = { git = "https://github.com/Satellite-im/Warp" }
warp-extensions = { git = "https://github.com/Satellite-im/Warp", features = ["pd_flatfile"] }
```

## Starting Extension

```rust
	use warp::pocket_dimension::PocketDimension;
	use warp_extensions::pd_flatfile::FlatfileStorage;

	let mut system = FlatfileStorage::new_with_index_file("</path/to/directory>", "index-file")?;
```

### Testing Flatfile Extension
***Note: This uses serde to deserialize and serialize in the examples***

#### Add Content
```rust
	use warp::pocket_dimension::PocketDimension;
	use warp_extensions::pd_flatfile::FlatfileStorage;
	use serde::{Serialize, Deserialize};

	#[derive(Serialize, Deserialize)]
	struct Instruction {
		pub instr_type: u8,
		pub message: String
	}

	let data = DataObject::new(DataType::Unknown, Instruction { instr_type: 1, message: "PING".into() })?;

	system.add_data(DataType::Unknown, &data)?;
```

***TODO: Show how to cache a file directly***

#### Retrieve Content
```rust
	use warp::pocket_dimension::PocketDimension;
	use warp_extensions::pd_flatfile::FlatfileStorage;
	use serde::{Serialize, Deserialize};

	#[derive(Serialize, Deserialize)]
	struct Instruction {
		pub instr_type: u8,
		pub message: String
	}

	let mut query = QueryBuilder::default();
	query.where("instr_type", 1)?;

	let objects = system.get_data(DataType::Unknown, Some(&query))?;

	let object = objects.last().unwrap();

	let instruction = object.payload::<Instruction>()?;

	assert_eq!(instruction.instr_type, 1);
	assert_eq!(instruction.message, String::from("PING"));


```

***TODO: Show how to retrieve a cached file***