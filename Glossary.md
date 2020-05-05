Gun borrows some terms from graph theory and mesh networking, then defines some of its own. Here you can find a summary of what they mean.

#### Node
An object within a graph. It can contain primitive values, but not other objects (only pointers to other nodes). Also known as a [vertex](https://en.wikipedia.org/wiki/Vertex_(graph_theory)) in graph theory.

#### Pseudo-merge/Union
An intelligent merge between two objects. Unlike `Object.assign`, it uses the [HAM](https://github.com/amark/gun/wiki/Conflict-Resolution-with-Guns) conflict resolution engine to ensure updates are merged commutatively.

#### Pseudo-node/Key Node
A special type of node in gun used for adding secondary indices (via the [`.key()`](https://github.com/amark/gun/wiki/API-(v0.3.x)#key) method). It provides a list of unique IDs to pseudo-merge into an aggregate node.

#### Soul
A synonym for an object [Universally Unique Identifier](https://en.wikipedia.org/wiki/Universally_unique_identifier). Each node in gun has one (named "`#`" in the object metadata).

#### Graph
An object that contains unique nodes.

#### Peer
A single device on a mesh network. Usually takes both roles of client and server.

#### Mesh
A collaborative network where each peer is responsible for forwarding others' messages (and responding to requests if it has the data). Refers to the concept of a [mesh network](https://en.wikipedia.org/wiki/Mesh_networking).

#### Partition
When one group of peers can't communicate another, such as two servers losing connection between each other, but still serving clients.

#### Universe
The sum total of all nodes and graphs across every peer in the application.
