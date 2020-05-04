Gun exposes a pluggable interface that enables you to write custom storage adapters to persist your Gun database. Writing storage adapters is a non-trivial enterprise that requires some careful consideration of Gun's data format, request/response API, and wire spec.

This document walks through how to wire up a Gun adapter and handle `get` and `put` requests.

# Use RAD Instead

> Note: It is recommended you use [RAD's storage interface](./RAD#api) instead.

For all `v0.2020.x` & `v0.2019.x` and above, please use RAD instead. It is much simpler.

Below is for old and historic documentation purposes:

***


# Hook Into Gun `<= v0.2019.x` IO Events

> Note: It is recommended you use [RAD's storage interface](./RAD#api) instead.

Please see warning note above.

Gun includes an event system that you can hook into in order to receive `get` and `put` request.

```javascript
var Gun = require('gun/gun');

// `db` param passed in is the gun instance
// that is being created.
Gun.on('create', function(db) {

  // This line is critical to allow other
  // extensions to register as well.
  this.to.next(db);

  // Register IO listeners with gun context
  db.on('get', function(request) {

    // same as above.
    this.to.next(request);

    // read data, etc.
  });
  db.on('put', function(request) {

    // same as above.
    this.to.next(request);

    // write data, etc.
  });
});
```
# Storing data (`put` requests) 

## Request Formats
The context received in the above example `put` example can take a few shapes. Here's a basic example:

```javascript
{
  gun: ...,
  '#': 'AB312C', // sort of like a write 'id', used by Gun for deduplication and tracking
  '@': 'ACW352', // Not always present, but if so, it corresponds to the acknowledgment for a write request.
  put: {
    nodeKey1: {
      // metadata, stored under key '_'
      '_': {
        '#': 'nodeKey1',
        '>': {
          prop1: 12345678910, // state used in conflict resolution
          prop2: 12345678910
        }
      },
      prop1: 'This the value for prop1. It could a string, number, boolean, or null',
      prop2: false
    },
    nodeKey2: ...
    nodeKey3: ...
  },
  ...
}
```
The context received contains a `put` key which contains a node delta to be written as well as some metadata that Gun uses in various ways (chiefly the conflict resolution algorithm). You must store the metadata and the actual values.

**Warning: all `put`s are Node delta/diffs and not full nodes. If you treat a delta like a full node, you could have data loss!!**

Here is a more complex example that includes node relationships, omitting the metadata for the sake of clarity:

```javascript
put: {
    nodeKey1: {
        '_': ...
        prop1: 'This the value for prop1. It could a string, number, boolean, or null',
        prop2: {
            '#': 'nodeKey2' // a reference to the node that has the key 'nodeKey2'
        }
    },
    nodeKey2: {
        '_': ...,
        prop1: {
            '#': 'nodeKey2'; // a reference to the node that has the key 'nodeKey1'
        }
    }
}
```
The format and mechanism that you store this is totally up to you. When reading data, Gun will expect the data to be returned in a very similar format.

## Acknowledge a write

Once you have handled the `put` request, you will want to let Gun know that the data has been processed successfully. This is called an acknowledgement or `ack` for short.

```javascript
db.on('put', function(request) {
  this.to.next(request);
  
  // grab the node delta
  var delta = request.put;
  var dedupId = request['#'];
  
  // Remember the delta is an object with multiple keys/nodes
  Storage.write(delta).then(function(err) {

    // acknowledge the write to the gun db instance
    db.on('in', {
      '@': dedupId,
      ok: !err,       // boolean value, optional
      err: err        // the error, if any; or null
    });
  });
});
```

# Retrieving data (`get` requests)

## `get` request format

1. Requesting a full node

A `get` request for an _entire node_ has this format:

```javascript
{
    '#': 'EUwDZUQio', // request dedupId
    get: {
        '#': 'nodeKey1' // the key for the node to retrieve
    },
    gun: ...
}
```

2. Requesting a single field

A `get` request for a _single field_ on a node has this format:

```javascript
{
    '#': 'EUwDZUQio', // request dedupId
    get: {
        '#': 'nodeKey1',  // the key for the node to retrieve
        '.': 'prop1'      // the field to retrieve
    },
    gun: ...
}
```

## Acknowledging a `get` / send back data

```javascript
db.on('get', function(request) {

  // same as above.
  this.to.next(request);

  // read data, etc.
  var dedupId = request['#'];
  var get = request.get;
  var key = get['#'];
  var field = get['.'];

  // Make sure to handle both whole node and field retrieval
  Storage.read(key, field).then(function(err, data) {

    // acknowledge the retrieval
    db.on('in', {
        '@': dedupId,
        put: data,
        err: err
    });
  });
});
```
In this instance, we assume that `data` being returned from storage has the following format:

```javascript
{
  nodeKey1: {
      // metadata
      '_': {
        '#': 'nodeKey1',
        '>': {
          prop1: 12345678910,
          prop2: 12345678910
        }
      },
      prop1: 'Value', // a simple value in a property
      prop2: {
          '#': 'nodeKey2' // a reference to another node
      }
    }
}
```

When no data is found, it is still important to acknowledge the `get` like so: 

```javascript
  Storage.read(key, field).then(function(err, data) {

    if (!err && !data) {
        db.on('in', {
            '@': dedupId,
            put: null,
            err: null
        });
    }
  });
```
This lets Gun know that the adapter successfully processed the request (no error) but that no data was found to return.

## Streaming Data back to Gun

In order to account for nodes that could overwhelm the process's memory, you can stream data back into Gun. Simply acknowledge the `get` request multiple times as the data comes in.

# Gun Helper Methods

Gun exposes a few helpful methods:

`Gun.graph.node(node)`: Format a single node as a valid graph. Useful in `acks` during `get` requests:

```javascript
var graph = Gun.graph.node({
    '_': {
       '#': 'nodeKey1',
       '>': {
        prop1: 12345678910,
       }
    },
    prop1: 'Value'
});

db.on('in', {
    '@': dedupId,
    put: graph,
    err: err
});
```
`Gun.state.to(node, field)`: Retrieve a field from a node and return it in a format that Gun recognizes. Useful when handling requests for a single field to pull that one field from a node.

```javascript
var node = Gun.state.to({
    '_': {
       '#': 'nodeKey1',
       '>': {
         prop1: 12345678910,
       }
    },
    prop1: 'Value'
}, 'prop1');

db.on('in', {
    '@': dedupId,
    put: Gun.graph.node(node),
    err: err
});
```

# Words of Caution

* If you're shipping your adapter as a package available via NPM, **DO NOT INCLUDE `GUN` IN YOUR PACKAGE DEPENDENCIES!**
* If you anticipate large nodes at all, be sure to account for these and enable `read` streaming. Otherwise, you risk crashes when nodes overwhelm the memory.
* A `get` request will result in a `write` request. Using the `dedupId`s you can filter out when these duplicate write requests come through. 

# Alternative to writing your own adapter from scratch

There are some community built adapters listed [here](https://github.com/amark/gun/wiki/Modules). One of these might fit your needs.

If you don't want or need to deal with the low-level concerns outlined above, you could also try [`gun-flint`](https://github.com/sjones6/gun-flint). This package abstracts away the low-level details while enabling non-trivial adapters.


