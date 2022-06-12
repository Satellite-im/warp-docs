# QEP2P Mesh

## Overview

The `Mesh` interface is used to manage RTC communication with groups of multiple peers on the system.
Mesh peers announce themselves to the group and automatically establish a full mesh topology between peers using RTC signaling.

**Example**:

1. Host `A` starts a call for the group, dialing `B` and `C`.
2. User `B` accepts the call from `A` first, extends offer to `C`.
3. User `C` accepts the call from `A` last, verifies that `B` has joined, and automatically accepts the call from `B` also.

## Methods

`Mesh::subscribe` listen for P2P events on the mesh connection.

`Mesh::unsubscribe` stop listening for P2P events on the mesh connection.

`Mesh::publish` publish a P2P event on the mesh connection.

`Mesh::stream_start` start a media stream of the given type and stream it to the given mesh over the mesh connection.

`Mesh::stream_stop` stop a media stream of the given type and remove it from the mesh connection.

`Mesh::stream_subscribe` subscribe to changes to streams on the mesh connection.

`Mesh::stream_unsubscribe` unsubscribe from changes to streams on the mesh connection.

### Accepting a Call

```rust
// client shows ui on incoming call...
mesh.subscribe("call:incoming", onIncomingCall)
// user interacts to answer...
mesh.call_answer()
// when complete, user interacts to hang up...
mesh.call_hangup()
```

### Managing Media Streams

```rust
// media can be enabled before or after call starts
mesh.stream_start('video')
mesh.stream_start('audio')
mesh.stream_start('screen')
mesh.stream_stop('video')
```

### Observing for Changes

Changes to streams can be observed using the `stream_subscribe` method

```rust
mesh.stream_subscribe(onStreamChange)
mesh.stream_unsubscribe(onStreamChange)
```
