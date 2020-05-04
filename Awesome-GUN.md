# ✨Awesome Gun✨
[![Awesome](https://awesome.re/badge.svg)](https://awesome.re)

> [GUN](https://github.com/amark/gun) is a realtime, decentralized, offline-first, graph protocol to sync the web.

- [Apps](#apps)
- [Plugins / Helpers](#plugins--helpers)
- [Framework Integrations](#framework-integrations)
- [Alternative Implementations](#alternative-implementations)
- [Storage Adapters](#storage-adapters)
- [Deployment](#deployment)

---

## Apps

- [GUN Kanban](https://github.com/nmaro/gun-kanban/) [demo](https://gun-kanban.nmaro.now.sh/)
  - A P2P KanBan!
- [gun-poll](https://github.com/nmaro/gun-poll/) [demo](https://gun-poll.nmaro.now.sh/)
  - Polls
- [gun-calendar](https://github.com/nmaro/gun-calendar/) [demo](https://gun-calendar.nmaro.now.sh/)
  - A calendar / scheduling app
- [gun-editor](https://github.com/nmaro/gun-editor/) [demo](https://gun-editor.nmaro.now.sh/)
  - An experimental collaborative editor (partial implementation)
- [NotABug](https://github.com/notabugio/notabug)
  - A decentralized link aggregator with reddit’s classic, open source UI
- [Iris](https://github.com/irislib/iris)
  - Iris is a social networking application that stores and indexes everything on the devices of its users and connects directly with peers who run the application - no corporate gatekeepers needed.
- [LiveCodeStream](https://github.com/QVDev/codestreamer)
  - Broadcast with ease, straight from your browser
- [PushToTalk](https://github.com/QVDev/GunPushToTalk)
  - A Push to Talk demo with GUN db
- [Maskbook](https://github.com/DimensionDev/Maskbook)
  - Encrypt your posts & chats on You-Know-Where. Allow only your friends to decrypt.
- [streams](https://github.com/nmaro/streams)
  - An experimental nanoblogging engine / anonymous group chat / outliner / productivity tool
- [GiveMeCredit](https://github.com/glensimister/givemecredit)
  - A Social media application designed to give communities political independence
- [Trace](https://github.com/PsychoLlama/Trace) 
  - ~ By [PsychoLlama](https://github.com/PsychoLlama) - **[live demo](http://trace.gundb.io)**
- [Connect-4](https://github.com/PsychoLlama/connect-four) 
  - ~ By [PsychoLlama](https://github.com/PsychoLlama) - **[live demo](https://gun-c4.herokuapp.com/)**
- [TodoMVC](https://github.com/jveres/todomvc) 
  - ~ By [jveres](https://github.com/jveres) - **[live demo](http://todos.loqali.com/)**
- [creeper.im](https://creeper.im)
  - Send public messages to nearby people.
- [GoShareData](https://github.com/zubairq/GoShareData)
  - An enterprise system for data sharing database data with colleagues 
- [JS-Trafficlights](https://github.com/niccord/js-trafficlights)
  - Trafficlights manager for limited resource access, like VPNs or remote desktops
- [SENTINL](https://github.com/sirensolutions/sentinl)
  - Clustering the Kibi/Kibana Alerting & Reporting Application _(watcher)_ ~ By [lmangani](https://github.com/lmangani) and [Sergey Bondarenko](https://github.com/sergibondarenko)
- [paStash](https://github.com/sipcapture/paStash)
  - Spaghetti I/O Event Processing, Interpolation, Correlation with Gun persistence ~ By [lmangani](https://github.com/lmangani)
- [fastify-gundb](https://github.com/lmangani/fastify-gundb)
  - Fastify is a Fast and low overhead modular web framework, supporting persistence layer using Gun ~ By [lmangani](https://github.com/lmangani)
- [PiTherm](https://github.com/paullj1/PiTherm)
  - Raspberry Pi thermostat application using Gun to sync client UIs ~ By [paullj1](https://github.com/paullj1)
- [Gewd Notes](https://gewd-notes.surge.sh/#/)
  - Workflowy clone using gun ~ By [negue](https://github.com/negue)

## Plugins / Helpers

- [GunStreamer](https://github.com/QVDev/GunStreamer)
  - Streaming component for Gun db.
- [gun-inspector](https://github.com/nmaro/gun-inspector) [demo](https://gun-inspector.nmaro.now.sh/)
  - Gun Node visualizer and editor
- [visualGraph](https://github.com/dletta/visualGraph)
  - Gun Utility to Traverse Graph and use d3.js to visualize
- [gun-graphin](https://github.com/lmangani/gun-graphin)
  - Gun Graph visualization and Navigation using [Graphin](https://github.com/antvis/Graphin) library
- [Gun-scape](https://github.com/lmangani/gun-scape)
  - GunDB Cytoscape Graph Visualizer + Live Editor
- [CRDT Continuous Sequence](https://github.com/nmaro/crdt-continuous-sequence)
  - Makes CRDT sets orderable
- [gunjs trust share key](https://github.com/Lightnet/gunjstrustsharekey)
  - gun.js Trust and Share Key functions simple.
- [polyGun](https://github.com/lmangani/polyGun-letsencrypt)
  - GunDB Polyglot Server [ https/wss + letsencrypt ]
- [gun-host](https://github.com/qxip/gun-host)
  - GunDB HTTP/HTTPS Server and API  ~ By [Sergey Bondarenko](https://github.com/sergibondarenko)
- [gun-kibana-graph](https://github.com/lmangani/kibana_graph)
  - Visualization Plugin to display Gun graphs in Kibana ~ By [lmangani](https://github.com/lmangani)
- [gun-synclist](https://www.npmjs.com/package/gun-synclist) Get an Array with full nodes on init, after that only the changed - full - node, its soul and its index. ~ By [Stefdv](https://github.com/stefdv)
 - [gun-tag](https://www.npmjs.com/package/gun-tag)
Tag, untag, filter and find intersects on nodes  ~ By [Stefdv](https://github.com/stefdv)
- [gun-schema](https://github.com/gundb/gun-schema)
  - Schema validation module for node and the browser ~ By [RangerMauve](https://github.com/RangerMauve)
- [labelmaker](https://github.com/PsychoLlama/labelmaker)
  - Organize and index data into groups with tags ~ By [PsychoLlama](https://github.com/PsychoLlama)
- [onward](https://github.com/gundb/onward)
  - Listen for changes on an entire document at once ~ By [amark](https://github.com/amark)
- [reticle](https://github.com/PsychoLlama/Reticle)
  - Namespace your data, preventing naming conflicts and adding clear separation between apps ~ By [PsychoLlama](https://github.com/PsychoLlama)
- [synchronous](https://github.com/gundb/synchronous)
  - One way deep (document oriented) object data binding, similar to [onward](https://github.com/gundb/onward) ~ By [amark](https://github.com/amark)
- [gun-ws](https://npmjs.org/packages/gun-ws) [on Github](https://github.com/d3x0r/gun-ws)
  - Extended websocket provider for Gun.  Initially meant to provide a way to connect multiple servers to a single 
  - Gun instance ~ By [d3x0r](https://github.com/d3x0r)
    - Uses 'ws' npm package for websockets.  Should either migrate to [sack.vfs](https://www.npmjs.com/package/sack.vfs#websocket-module)
- [load](https://github.com/IMGNRY/load)
  - Load full document structures to Gun ~ By [IMGNRY](https://github.com/IMGNRY)
- [gun-most](https://github.com/ctrlplusb/gun-most)
  - Extends gunDB with the ability to chain into most.js observables ~ By [ctrlplusb](https://github.com/ctrlplusb)
- [graphql-gun](https://github.com/brysgo/graphql-gun)
  - Augmented graphql query interface for Gun
- [bullet](https://github.com/bugs181/bullet)
  - Alternative syntax to Gun, promise support, utility modules, and easy Gun adapter events.
- [gun-appendOnly](https://github.com/GoodDollar/gun-appendOnly)
  - Support append only node keys on a first-come-first-served basis and allow modification to key content based on SEA users
- [gun-hue](https://github.com/PsychoLlama/gun-hue)
  - Explore and change your hue lighting with gun! ~ By [PsychoLlama](https://github.com/PsychoLlama)
- [gun-ui-lcd](https://github.com/Stefdv/gun-ui-lcd) 
  - Component to attach an html lcd to Gun ~ By [Stefdv](https://github.com/Stefdv)

## Framework Integrations

- [svelte gundb example](https://github.com/ak5/svelte-gun-example)
  - This is a project template for Svelte apps with the gun database.
- [sapper/svelte gundb example](https://gitlab.com/trokster/svelte-sapper-gun-template)
  - This is a project template for Sapper/Svelte apps with the gun database. The server is served alongside the page and accessigne at location.origin + "/gun". Sapper is a framework designed with svelte to facilitate offline first functionality and all svelte goodness.
- [Weir](https://github.com/rm-rf-etc/weir)
  - Auto-generate react component data bindings to gundb nodes
- [vue-gun](https://github.com/sjones6/vue-gun)
  - Vue plugin to add Gun integration ~ By [sjones6](https://github.com/sjones6)
- [gun-react-native](https://github.com/sjones6/gun-react-native)
  - A distribution of Gun designed to work with React Native out of the box ~ By [sjones6](https://github.com/sjones6)
- [as](https://github.com/amark/gun/wiki/as)
  - Mark's personal UI Framework with deep assumptions and opinionated bindings to GUN ~ By [amark](https://github.com/amark)
- [gun-realm](https://github.com/sjones6/gun-realm)
  - A React Native RealmJS adapter for GunJS ~ By [sjones6](https://github.com/sjones6)
- [gun-ui-gauge](https://github.com/Stefdv/gun-ui-gauge)
  - (Still under development to fit the latest Gun version ( 0.9.x ) )
  - Add a GunDB controlled gauge to your webpage ~ By [Stefdv](https://github.com/Stefdv)

## Alternative Implementations

- Typescript: [ChainGun](https://github.com/chain-gun)
- Java: [JGUN](https://github.com/ChronosX88/JGUN), [Gun.Java](https://github.com/anubiann00b/Gun.Java) (not implemented fully)  
- Python: [PyGUNDB](https://github.com/xmonader/pygundb)  
- Golang: [GUNDB Port Go](https://github.com/miguelmota/gundb-port-go), [Esgopeta](https://github.com/cretz/esgopeta)
 
## Storage Adapters

 - [gun-level](https://github.com/gundb/gun-level)
A LevelDB adapter that works with the levelDB module ecosystem ~ By [PsychoLlama](https://github.com/PsychoLlama)

 - [gun-level-browser-indexedDB](https://github.com/zrrrzzt/choo-gun-leveldb-test)
Example of LevelDB binding with indexedDB in the browser with GUN and choo ~ By [zrrrzzt](https://github.com/zrrrzzt)

 - [gun-mongo](https://github.com/sjones6/gun-mongo)
A MongoDB Adapter that stores entire nodes in Mongo documents ~ By [sjones6](https://github.com/sjones6)

 - [gun-mongo-key](https://github.com/sjones6/gun-mongo-key)
A MongoDB Adapter that stores nodes as key:value pairs in Mongo documents ~ By [sjones6](https://github.com/sjones6)

 - [gun-db](https://npmjs.org/packages/gun-db)[(on Github)](https://github.com/d3x0r/gun-db)
Sqlite/ODBC storage adapter for Gun ~ By [d3x0r](https://github.com/d3x0r)
    - Depends on [sack.vfs](https://npmjs.org/packages/sack.vfs#sqlite-interface).  Can provide encryption for storage.

 - [gun-file](https://npmjs.org/packages/gun-file)[(on Github)](https://github.com/d3x0r/gun-db)
Local Storage adapter for Gun ~ By [d3x0r](https://github.com/d3x0r)
    - Depends on [JSON6](https://npmjs.org/packages/json-6)

 - [gun-cassandra](https://github.com/lmangani/gun-cassandra)
Experimental Cassandra / Elassandra persistence layer for Gun ~ By [lmangani](https://github.com/lmangani)

 - [gun-elastic](https://github.com/lmangani/gun-elastic)
Experimental Elasticsearch persistence layer for Gun/Flint ~ By [lmangani](https://github.com/lmangani)

 - [gun-flint](https://github.com/sjones6/gun-flint)
Micro-framework for building Gun adapters ~ By [sjones6](https://github.com/sjones6)

 - [gun-adapter-perf](https://github.com/sjones6/gun-adapter-perf)
https://github.com/sjones6/gun-adapter-perf ~ By [sjones6](https://github.com/sjones6)
 

## Examples

Examples, demos and tools demonstrating Gun features and capabilities

 - [GitHub Examples](https://github.com/amark/gun/tree/master/examples)
Original, up to date Gun examples

 - [gun-scape](https://github.com/lmangani/gun-scape)
GunDB Cytoscape Graph Visualizer with [Live Editor](https://goo.gl/qDTBx7)

 - [Next.js](https://github.com/zrrrzzt/gun-next-example)
Example of how to get started with Next.js and GUN!

 - [GunStore.io](https://github.com/lmangani/gunstore.io) 
Storing data in Gun DB by sending simple HTTP requests & custom URLs

- [gundb-react-todolist](https://github.com/treinberger/gundb-react-todolist)
Simple todo-list using gun and react-gun

## Deployment

 - [arsenal-dock](https://github.com/sjones6/arsenal-dock)
A Docker container for running GunJS

## Archive

The old pages are: (these will be archived after migrated to this page)

 - [Projects](https://github.com/amark/gun/wiki/Projects)
 - [Modules](https://github.com/amark/gun/wiki/Modules)
 - [Snippets (v0.6.x)](https://github.com/amark/gun/wiki/Snippets-(v0.6.x))
 - [Snippets (v0.3.x)](https://github.com/amark/gun/wiki/Snippets-(v0.3.x))