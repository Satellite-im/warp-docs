# Hooks

## Overview

Hooks allow us to create subscription points for extensions of the platform to listen upon to know when a module does something. They are simple to implement into your module as well as used for external APIs.

## Usage

#### Creating a Hook

Each hook must be registered with the system, you can do so by calling the `create` method outlined below.

```rust
use warp::hooks::hooks::{Hook, Hooks};
use warp::module::Module;

fn main() {
    let mut system = Hooks::default();
    let hook = system.create(Module::FileSystem, "new_file").unwrap();
}
```

#### Triggering a Hook

You can emit the hook to all subscribers by `triggering` the hook. Again shown below.

```rust
use warp::hooks::hooks::{Hook, Hooks};
use warp::module::Module;

fn main() {
    let mut system = Hooks::default();
    let hook = system.create(Module::FileSystem, "new_file").unwrap();
    system.trigger("filesystem::new_file", &hook, &data);
}
```

#### Subscribing to Hook Triggers

You may subscribe to be notified via an `Fn` closure when a hook is triggered as well.

```rust
use warp::hooks::hooks::{Hook, Hooks};
use warp::module::Module;

fn main() {
    let mut system = Hooks::default();
    system.create(Module::FileSystem, "new_file").unwrap();
    system.subscribe("filesystem::new_file", |hook, data| {
        // Hook and Hook data provided in this scope
    })?;
}
```

#### Getting Registered Hooks

Lastly, it can be useful to see which hooks are currently registered in the system. Getting those are also very straightforward.

```rust
use warp_hooks::hooks::{Hook, Hooks};
use warp_module::Module;

fn main() {
    let mut system = Hooks::default();
    system.create(Module::FileSystem, "new_file").unwrap();
    let hooks = system.hooks();
}
```
