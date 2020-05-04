`TODO: This page will probably be dedicated to the API of how to build a storage adapter, not a list of storage adapters (except official ones like RSE, since it has its own API that needs explaining), a list of storage adapters should be included in the community/awesome/ecosystem page along with everything else so it is easy to find!` 

To permanently store all GUN's data, your GUN server app must define which storage engine to use. By default a GUN server will store the data in a file, but this is just meant to be used during development. It is not a sound solution for production.

However, GUN is different to most other databases when it comes to dev vs prod setup in that the only distinction is that a production setup can maybe include a persistent storage option (e.g. AWS S3, IPFS...) for a participating relay peer. That's not mandatory though, browsers peers with their localStorage storage are suffiecent more often than not, doesn't matter if you're still in development or already running your app in production with customers.


There are several storage engines that you can choose:

# Included Storage Engines

## localStorage

In browser, GUN app data is stored in the browser's localStorage, but is primarily treated as cache.

## File

Great for development, but not meant for production.

Works in nodejs and can be activated as follows: 

`var gun = Gun({ file: 'db/data.json', web: server, radisk: false, localStorage: true });`

## Radix

The Radix Storage Engine (RSE) is the default.

~~RSE will become the new default and will be production ready, however it is not the default yet (although you can make it the default by just turning off the current default with `Gun({localStorage: false})` in NodeJS.~~

## S3

[Temporarily located here](Using-Amazon-S3-for-Storage) (do not link to this, we'll be making it redirect)

Works with the RSE.

## IPFS

In the future We foresee GUN using RSE with IPFS/Filecoin bindings not S3, as RSE will allow for easy relay-peer storage-sharding (although I do not recommend people do sharding by default).

# Third Party Storage Solutions

## Level

https://github.com/PsychoLlama/gun-level

## File System (Alt1)

https://github.com/d3x0r/gun-file
https://npmjs.org/packages/gun-file
reqires: https://npmjs.org/packages/json-6  https://github.com/d3x0r/json6

Streams graph to file; works with a single node at a time to prevent string overflows and memory bloat by assembling the whole graph as a single object.

Delays output until the graph is write-idle, or a maximum number of updates has happened.  This makes it very performant by interfering very little with the working time of Gun.

The currently community-agreed upon custom storage engine is @d3x0r 's. 

## Sqlite/ODBC (SACK)

https://github.com/d3x0r/gun-db
https://npmjs.org/packages/gun-db
reqires: https://npmjs.org/packages/sack.vfs  (https://github.com/d3x0r/sack.vfs)

Defaults to a Sqlite db file.  Items are written as they are posted for change to the Gun database.  This means the Sqlite database is kept as up to date as possible.  Unknown values, when requested are requested one node at a time; as opposed to a file solution which requires loading the whole graph at startup.

ODBC access is available for using PostgreSQL, MySQL, MSSQL, etc using an ODBC driver.

Sqlite database may be stored in an encrypted VFS file.

## Flint

Allows for easy integration into using other databases as a storage engine, many of the systems below were built with this: https://github.com/sjones6/gun-flint .

We do not recommend using the Mongo/MySQL/etc. adapters, even though that is possible, simply because that is overkill and not the future - more like a toy example of integration.

## MongoDB

https://github.com/sjones6/gun-mongo

## MySQL

https://github.com/sjones6/gun-mysql

## Cassandra

https://github.com/lmangani/gun-cassandra

## Elasticsearch

https://github.com/lmangani/gun-elastic

## LMDB

https://github.com/notabugio/gun-lmdb

