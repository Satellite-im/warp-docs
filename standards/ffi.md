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

## FFIResult and FFIError for Result and Error Handling

```rust
#[repr(C)]
pub struct FFIResult<T> {
    pub data: *mut T,
    pub error: *mut FFIError,
}

#[repr(C)]
pub struct FFIError {
    pub error_type: *mut std::os::raw::c_char,
    pub error_message: *mut std::os::raw::c_char,
}
```

Due to C not supporting any Result-like functionality that can be found in Rust, we had to create a `FFIResult<T>`, which would allow us to create a `Result<T, E=warp::error::Error>` that be used in C. This struct contains 2 pointer with one containing the data of `*mut T` on a successful execution of a function, and the other containing a pointer to `FFIError` if there is an error. `FFIError` have 2 string pointers with one being a string of the error type (eg `Error::Unimplemented` would come up as `error_type = "Unimplemented"`), and the other being a string of the error message (eg `error_message = "Functionality is not yet implemented"`).

The function returns successful if `FFIResult::error` is `std::ptr::null_mut()` (or `NULL` in C). `FFIResult::data` is equal `NULL` even if successful especially if the function called did not return anything.

For us to utilize C specific types like `char`, `void`, etc., we would have to use those types as `T`. Example, if we were to take a `String` from rust and convert it to a `char *` that would be used as the result of a successful function, we would need to use `FFIResult<c_char>`, which in C, would look like `FFIResult_c_char` with the fields being the following in C

```C
struct FFIResult_c_char {
    char *data;
    struct FFIError *error;
};
```

This is because in Rust, we handle the conversion of a `String` to `c_char` using `CString`. For a function that returns `Result<(), Error>`, we would be utilziing `c_void` type in Rust (or `void` in C), which means we would use `FFIResult<c_void>` that would translate to `FFIResult_c_void`, creating a void pointer that would be `NULL` if the function is successful, otherwise would have a valid pointer for `FFIResult::error`. 

Integer types like `usize` in rust would translate to something like `FFIResult_usize`, containing a `uintptr_t` pointer. 

Other types within warp can be use in a similar way. Using `Tesseract` as an example, if we had a FFI function returning `FFIResult<Tesseract>`, this would translate to the following in C

```C
struct FFIResult_Tesseract {
    struct Tesseract *data;
    struct FFIError *error;
};
``` 

## FFIVec and FFIArray for Array handling

***TODO***

Notes

* Extensions generally dont need to have their own functions as warp has functions specifically for dropping/freeing. 
* If at all possible extensions should also test against other targets such as aarch64, etc., to ensure compatibility.
* While one can use a struct type in C as a pointer, it does not mean we can use it directly in C. Instead we would need a extern function that would use that pointer within.  