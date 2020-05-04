> **Learn answers to all your questions in [THE INTERACTIVE TUTORIAL](Todo-Dapp)**! Or read along:

GUN is a small, easy, and fast data sync and storage system that runs everywhere JavaScript does. The aim of GUN is to let you focus on the data that needs to be stored, loaded, and shared in your app without worrying about servers, network calls, databases, or tracking offline changes or concurrency conflicts. This lets you build cool apps fast, like:

<div style="position: relative; padding-bottom: 56.25%;"><iframe src="https://www.youtube.com/embed/1ASrmQ-CwX4" frameborder="0" allowfullscreen style="border: 0px; position: absolute; width: 100%; height: 100%;"></iframe></div>

![](https://gun.eco/see/3dvr.gif "3D multiplayer VR games")
[![](https://gun.eco/see/aiml.gif "Distributed AI/ML")](https://github.com/cstefanache/cstefanache.github.io/blob/master/_posts/2016-08-02-gun-db-artificial-knowledge-sharing.md#gundb)
[![](https://gun.eco/see/gps.gif "Realtime GPS tracking")](http://gps.gundb.io/)
[![](https://gun.eco/see/dataviz.gif)](https://github.com/lmangani/gun-scape#gun-scape "Data Viz")
[![](https://gun.eco/see/p2p.gif "P2P dApps")](https://github.com/amark/gun/wiki/Auth)
[![](https://gun.eco/see/iot.gif "IoT sync")](https://github.com/Stefdv/gun-ui-lcd#okay-what-about-gundb-)

## Offline-First

When a browser peer asks for data, it'll merge the reply with its own data using a [CRDT](https://github.com/amark/gun/wiki/Conflict-Resolution-with-Guns), then cache the result. This means that:

- The next time the browser asks for that data, the reply is instant, even when offline.
- Data is replicated on each browser that asked for it.
- If your server fails, you can still recover your data from the browsers.

This makes the loss of important information nearly impossible, as all copies of the data would need to be destroyed before it is lost.

## Distributed

GUN is fully decentralized (peer-to-peer or multi-master), meaning that changes are not controlled by a centralized server. A server can be just another peer in the network, one that may have more reliable resources than a browser. You save data on one machine, and it will sync it to other peers without needing a complex consensus protocol. It just works.

---

### But how does it work?

_If you are feeling lost in buzzwords, the following description might give you a general idea about what's going on._

The gun graph database is stored across all peers participating in the network. Most data distribution scenarios that one could think of are possible to occur: Every peer might possess the complete graph, or only a subset of the complete graph and may posses data that does not exist on any other node (yet). The whole database is considered to be the union of all peers' graphs.

There is no theoretical limit for the total size of a gun graph. The amount of data that a peer has locally available is limited by the memory constraints of the host environment, like operating system, browser, etc. The amount of data that can be persisted beyond the running process depends on the storage engine. In a web browser it will usually be 5MB of localStorage. A relay peer with Radix file storage can persist much bigger amounts of data.

Relay peers are dedicated gun peers running on NodeJS. They will receive all data that is broadcasted to the network and be able to serve it on request. Normal peers running in a web browser will not start downloading the whole database but only the parts they request. A well designed gun application will only download relevant data, instead of draining the users bandwidth and RAM. This and the general unreliability of browser peers make it essential to have one or more relay peers running 24/7 for the operation of most real world use cases.

### Next Up?

- Follow [THE INTERACTIVE TUTORIAL](Todo-Dapp)!
