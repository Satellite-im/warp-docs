# File

The `File` is a representation of the files uploaded to the `Constellation`. A `File` should reference an accessible location to retrieve the file that has been uploaded. 


### Creating a `File`

```rust
use warp::constellation::file::File;
    
fn main() { 
    let file = File::new("test.txt");
    assert_eq!(file.name().as_str(), "test.txt");
}
```

### Setting the description to a `File`

```rust
use warp::constellation::file::File;
    
fn main() { 
    let mut file = File::new("test.txt");
    file.set_description("Test File");

    assert_eq!(file.description().as_str(), "test file");
}
```

### Setting the size to a `File`

```rust
use warp::constellation::{file::File, item::Item};

fn main() {
    let mut file = File::new("test.txt");
    file.set_size(100000);

    assert_eq!(file.size(), 100000);
}
```

### Hashing a `File`

```rust
use std::io::Cursor;
use warp::constellation::file::{File, Hash};

fn main() {
    file.hash_mut().hash_from_file("test.txt").unwrap();

    assert_eq!(file.hash().sha256(), Some(String::from("QmdR1iHsUocy7pmRHBhNa9znM8eh8Mwqq5g5vcw8MDMXTt")))
}

```