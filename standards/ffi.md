# FFI Standard Overview

Each module has an adapter struct (eg ConstellationAdapter) that contains a trait object wrapped behind a mutex and a atomic reference counter (aka Arc). This points to the extension that implements the module traits. The extension but import cbindgen as a build dependency. This is used to generate the C (or C++) header. In addition to cbindgen, the extension must support both rlib, staticlib and cdylib.

* FFI “constructor” functions should contain pointers to the adapter as c_void. 

  * While you can use the adapter struct as a pointer for for your extension, having it as a void pointer would ensure compatibility.

* FFI “constructor” functions name should be prefixed in the following format `<module>_<extension name>`. (eg constellation_fs_ipfs). This is to prevent conflicts and be able to easily identify the extension constructor.
	
* FFI “constructor” functions should return a void pointer of the adapter struct. It should NOT return anything else such as a pointer to the box trait object itself, otherwise it would result in a segfault when using the pointer with corresponding function. 

  * Purpose of this is due to the fact that a trait object itself is a fat pointer (a pointer that contains 2 pointers, one for the vtable from the trait and another for data from the struct) and while rust has a understanding of a fat pointer, C does not understand it out of the box. Using a struct to house the object would make it a bit easier at a cost of another box (which is not that much)

The extension does not need to create its own free/destroy functions unless there is something explicit it needs done prior to consuming the pointer and dropping it (though any clean up probably would need to be done by manually implementing Drop for the extension struct).



When using the FFI functions, it would be best to use the free functions from the warp header itself if at all possible. Eg directory_free should be use over free in C (or any related functions for freeing a pointer). This would allow for any explicit clean up prior to being dropped since using “free” would just free the pointer altogether without any additional calls.

## Constructor Example

***Note: This is imported from `warp-fs-memory`***

```rust
pub mod ffi {
    use crate::MemorySystem;
    use std::ffi::c_void;
    use warp::constellation::ConstellationAdapter;
    use warp::sync::{Arc, Mutex};

    #[allow(clippy::missing_safety_doc)]
    #[no_mangle]
    pub unsafe extern "C" fn constellation_fs_memory_new() -> *mut c_void {
        let obj = Box::new(ConstellationAdapter::new(Arc::new(Mutex::new(Box::new(
            MemorySystem::new(),
        )))));
        Box::into_raw(obj) as *mut ConstellationAdapter as *mut c_void
    }
}

```



Notes

* Extensions generally dont need to have their own functions as warp has functions specifically for dropping/freeing. 
* If at all possible extensions should also test against other targets such as aarch64, etc., to ensure compatibility.