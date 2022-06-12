# QEP2P Interface

## Overview

The `QEP2P` interface is used to find and securely communicate with peers on the system. It manages RTC peer connections (used for voice and video streaming) and provides an interface to manage one-on-one or full-mesh group connections for voice and video calling.

## Methods

`QEP2P::subscribe` listen to P2P events on the global pubsub layer.

`QEP2P::unsubscribe` stop listening to P2P events on the global pubsub layer.

`QEP2P::publish` publish a P2P event on the global pubsub layer.

`QEP2P::get_peers` fetch the current list of connected [Peers](/qep2p/Peer.md).

`QEP2P::peer_connected` returns a boolean representing the RTC connection status of the provided [Peer](/qep2p/Peer.md).

`QEP2P::peer_connect` establish or retrieve an RTC connection to the provided [Peer](/qep2p/Peer.md).

`QEP2P::mesh_connected` returns a boolean representing the RTC connection status of the provided [Mesh](/qep2p/Mesh.md).

`QEP2P::mesh_connect` establish or retrieve a [Mesh](/qep2p/Mesh.md) connection between multiple provided peers.

### Finding Peers

Peers can be requested during client initialization using the `get_peers` method, and additional peer connection events can be followed using the subscribe method.

```rust
peers = QEP2P::get_peers()
QEP2P::subscribe("peer:connect", onPeerConnected)
```

### Connecting to a Peer

Peer connections are created with the `peer_connect` method and managed via the [Peer](/qep2p/Peer.md) interface.

```rust
peer = QEP2P::peer_connect(peerId)
peer.subscribe("peer:typing", onPeerTyping)
peer.call_start()
```

### Group Connections

Group connections are created with the `mesh_connect` method and managed via the [Mesh](/qep2p/Mesh.md) interface.

```rust
mesh = QEP2P::mesh_connect(meshId, peerIds);
mesh.subscribe("mesh:typing", onMeshPeerTyping)
mesh.call_start()
```
