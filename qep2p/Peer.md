# QEP2P Peer

## Overview

The `Peer` interface is used to manage RTC communication with a specific peer on the system.

## Methods

`Peer::subscribe` listen for P2P events on this RTC peer connection.

`Peer::unsubscribe` stop listening for P2P events on this RTC peer connection.

`Peer::publish` publish a P2P event on the peer connection.

`Peer::call_start` start a call and signal to the peer.

`Peer::call_answer` answer a call signaled by the peer.

`Peer::call_hangup` hangup/destroy an active call.

`Peer::stream_start` start a media stream of the given type and stream it to the given peer over the RTC peer connection.

`Peer::stream_stop` stop a media stream of the given type and remove it from the RTC peer connection.

`Peer::stream_subscribe` subscribe to changes to streams on the given RTC peer connection.

`Peer::stream_unsubscribe` unsubscribe from changes to streams on the given RTC peer connection.

### Accepting a Call

```rust
// client shows ui on incoming call...
peer.subscribe("call:incoming", onIncomingCall)
// user interacts to answer...
peer.call_answer()
// when complete, user interacts to hang up...
peer.call_hangup()
```

### Managing Media Streams

```rust
// media can be enabled before or after call starts
peer.stream_start('video')
peer.stream_start('audio')
peer.stream_start('screen')
peer.stream_stop('video')
```

### Observing for Changes

Changes to streams can be observed using the `stream_subscribe` method

```rust
peer.stream_subscribe(onStreamChange)
peer.stream_unsubscribe(onStreamChange)
```
