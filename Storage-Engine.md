[THIS PAGE HAS BEEN MOVED TO /STORAGE](Storage) (since this page is very new and thus not linked much, it will be deleted, DO NOT LINK TO IT)

In browser (client) GUN apps data is stored in the browser's localStorage, but that is only for caching.

To permanently store all GUN's data, your GUN server app must define which storage engine to use.

By default a GUN server will store the data in a file, but this is just meant to be used during development. It is not a sound solution for production.

There are several storage engines that you can choose:
* File: this is default, but not meant for production.
* RSE (Radix Storage Engine): will soon become the default and then be a good choice for production.
* ...

The RSE (Radix Storage Engine) will become the new default and will be production ready, however it is not the default yet (although you can make it the default by just turning off the current default with `Gun({localStorage: false})` in NodeJS.

The currently community-agreed upon custom storage engine is @d3x0r 's. We do not recommend using the Mongo/MySQL adapters, even though that is possible, simply because that is overkill and not the future - more like a toy example of integration.

In the future We foresee GUN using RSE with IPFS/Filecoin bindings not S3, as RSE will allow for easy relay-peer storage-sharding (although I do not recommend people do sharding by default).