# WASM Standard Overview

Warp utilizes wasm-bindgen for handling of the exports of wasm compatible functions. This also includes in the generation of code via wasm-pack (or wasm-bindgen-cli) that would be use on the browser. In similar fashion to FFI, extensions should be prefixed with name in <module>_<extension> name in their “constructor” functions. This function would return the adapter.

There are a few things to note:

1. WASM, while does bridge the gap between native languages and the web, is limited to what it can and cannot do. Eg, due to the browser sandbox, it is unable to read and write directly to the filesystem (there is a “virtual filesystem” but thats irrelevant here) or perform direct network calls from the system (since all networking is handled by the browser).

2. wasm-bindgen is not full feature complete. There are times where you would require to use a JsValue or the like in place of say HashMap despite javascript (or typescript) having something similar. 

3. Incombatible functions that do compile on wasm32 target may panic or throw an error at runtime if it is not caught by the compiler (or wasm-pack). 

With that said, the extensions that wish to be compatible with wasm-bindgen:

1. Must not utilize any dependency that specific to the system (eg tokio would not be compatible with wasm32 at this time due to mio dependency not supporting wasm32 so in its place under wasm32 target, futures should be used unless one wish to use a wasm compat async crate).

2. Should avoid any incompatible imports (eg std::io::Read, std::io::Write, etc). Such imports should be gated behind `#[cfg(not(target_arch="wasm32"))]` to ensure it builds and executes successfully.

   a. You are able to use `cfg-if` crate if that would make it easier. 

3. Should utilize wasm-bindgen, js-sys, web-sys or related crates where needed. These crates make it easier to interact with dom and perform various of task on the browser. It also gives utility functions to convert some items to others. Eg Futures in rust are considered as a Promise in Js, though for traits functions utilizing async (through async-trait or similar) shouldnt need to worry about external conversion. 

4. May want to try to keep the functionality of the extension within the trait functions. This would ensure the need of not having any irrelevant types or exports that cannot be used. The constructor function may have such exports such as C-style enums[1], etc.

5. May require to import javascript functions into rust (eg [Wasm-bindgen guide](https://rustwasm.github.io/docs/wasm-bindgen/reference/attributes/on-js-imports/constructor.html) ).

    a. Instances where crates do not have any compatible wasm functions may sometime require importing the javascript functions into rust to utilize functions in the browser. An example would be ipfs. Also rust-ipfs is incomplete, it is not compatible with wasm at this time so one would need to import functions from the js-ipfs library to interact with it in wasm while say rust-libp2p has some support for wasm so only enabling the feature would be needed to be able to utilize that crate in wasm32, assuming there is no other crate blocking compiling for wasm.

   
## Constructor Example

***Note: This is imported from `warp-fs-memory`***

```rust
#[wasm_bindgen]
pub fn constellation_fs_memory_new() -> ConstellationAdapter {
    ConstellationAdapter::new(Arc::new(Mutex::new(Box::new(MemorySystem::new()))))
}
```


Notes:

[1]: Enums in rust must be C-style for it to be exported by wasm-bindgen. If its a enum with tuples, it would not be compatible and a workaround is to wrap a struct around the enum and export that struct with any relevant functions (eg getters, setters, etc). Eg

Incompatible  (would error with an incompatible C-style enum)
```rust
 enum MyEnum {
  MyString(String),
  MyInt(i64)
 }
 
 impl MyEnum {
  fn new_string(string: String) -> MyEnum {
    MyEnum::MyString(string)
  }
  
  fn new_int(int64: i64) -> MyEnum {
    MyEnum::MyInt(int64)
  }
  
  fn get_string(&self) -> Option<String> {
    match self {
      MyEnum::MyString(string) => Some(string),
      MyEnum::MyInt(_) => None
    }
  }
  
  fn get_int(&self) -> Option<i64> {
    match self {
      MyEnum::MyString(_) => None,
      MyEnum::MyInt(i) => Some(i)
    }
  }
}
```

Solution

```rust
struct MyStruct(MyEnum);

enum MyEnum {
  MyString(String),
  MyInt(i64)
}

impl MyStruct {
  fn new_string(string: String) -> MyStruct {
    MyStruct(MyEnum::MyString(string))
  }
  
  fn new_int(int64: i64) -> MyStruct {
    MyStruct(MyEnum::MyInt(int64))
  }
  
  fn get_string(&self) -> Option<String> {
    match self.0 {
      MyEnum::MyString(string) => Some(string),
      MyEnum::MyInt(_) => None
    }
  }
  
  fn get_int(&self) -> Option<i64> {
    match self.0 {
      MyEnum::MyString(_) => None,
      MyEnum::MyInt(i) => Some(i)
    }
  }
} 
```

Note that if youre not exporting the enum to be used by the browser then tuple-style enums are fine to use within Rust. This may change in the future where wasm-bindgen may be able to take such style and translate it into javascript to be used.

[2]: Crates that are compatible with `no_std` can be easily used within wasm but would require a wrapper around them for direct from the browser. See https://github.com/Satellite-im/Warp/blob/main/warp/src/crypto/signature.rs for an example of us wrapping around ed25519_dalek, which is compatible with no_std. There may be cases where no_std may not be supported by wasm (eg custom allocator, specific calls to the system thats outside of the scope of wasm32, etc).