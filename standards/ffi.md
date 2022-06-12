# FFI Standard Overview

Each module has an adapter struct (eg `ConstellationAdapter`) that contains a trait object wrapped behind a mutex and a atomic reference counter (aka Arc). This points to the extension that implements the module traits. 

* FFI “constructor” functions should contain pointers to the adapter as c_void. 

  * While you can use the adapter struct as a pointer for for your extension, having it as a void pointer would ensure compatibility.

* FFI “constructor” functions name should be prefixed in the following format `<module>_<extension name>`. (eg `constellation_fs_ipfs`). This is to prevent conflicts and be able to easily identify the extension constructor.
	
* FFI “constructor” functions should return a pointer of the adapter struct. It should NOT return anything else such as a pointer to the box trait object itself, otherwise it would result in a segfault when using the pointer with corresponding function. 

  * Purpose of this is due to the fact that a trait object itself is a fat pointer (a pointer that contains 2 pointers, one for the vtable from the trait and another for data from the struct) and while rust has a understanding of a fat pointer, C does not understand it out of the box. Using a struct to house the object would make it a bit easier at a cost of another box (which is not that much). 

* Extensions would need to use `cbindgen` as a build dependency to generate to the headers have support for  `rlib`, `staticlib` and `cdylib`.

    * This is not required if you plan on typing the functions manually or will be importing it using libdl or similar shared library loader or will be using a tool to generate bindings for another language. 

The extension does not need to create its own free/destroy functions unless there is something explicit it needs done prior to consuming the pointer and dropping it (though any clean up probably would need to be done by manually implementing Drop for the extension struct). 



When using the FFI functions, it would be best to use the free functions from the warp header itself if at all possible. Eg `directory_free` should be use over free in C (or any related functions for freeing a pointer). This would allow for any explicit clean up prior to being dropped since using “free” would just free the pointer altogether without any additional calls or cleanup.

## Constructor Example

***Note: This is imported from `warp-fs-memory`***

Cargo.toml:
```toml
[lib]
crate-type = ["cdylib", "rlib", "staticlib"]

[build-dependencies]
cbindgen = "0.23"

```

build.rs:
```rust
extern crate cbindgen;

use cbindgen::Language;
use std::env;
use std::str::FromStr;

fn main() {
    let crate_dir = env::var("CARGO_MANIFEST_DIR").expect("CARGO_MANIFEST_DIR does not exist");
    let lang = Language::from_str(&env::var("CBINDGEN_LANG").unwrap_or_else(|_| String::from("C")))
        .unwrap_or(Language::C);

    cbindgen::Builder::new()
        .with_crate(crate_dir)
        .with_language(lang)
        .generate()
        .expect("Unable to generate bindings")
        .write_to_file("warp-fs-memory.h");
}

```

lib.rs:
```rust
pub mod ffi {
    use crate::MemorySystem;
    use std::ffi::c_void;
    use warp::constellation::ConstellationAdapter;
    use warp::sync::{Arc, Mutex};

    #[no_mangle]
    pub unsafe extern "C" fn constellation_fs_memory_new() -> *mut ConstellationAdapter {
        let obj = Box::new(ConstellationAdapter::new(Arc::new(Mutex::new(Box::new(
            MemorySystem::new(),
        )))));
        Box::into_raw(obj) as *mut ConstellationAdapter
    }

    // Not apart of the code but only to show about "freeing" or dropping the object
    #[no_mangle]
    pub unsafe extern "C" fn constellation_fs_memory_free(fs: *mut ConstellationAdapter){
        if fs.is_null() {
            return;
        }
        let _ = Box::from_raw(fs);
        // drops here though you can call `drop` manually 
    }
}

```



Notes

* Extensions generally dont need to have their own functions as warp has functions specifically for dropping/freeing. 
* If at all possible extensions should also test against other targets such as aarch64, etc., to ensure compatibility.