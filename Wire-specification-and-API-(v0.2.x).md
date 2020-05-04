# Gun Wire Methods

Before diving into documentation, here is a quick sample of what the simplest (not well designed) driver looks like:

(upcoming v0.5.x API)
```javascript
Gun.on('put', function(at){
    for(var soul in at.graph){ // graph is the DELTA CHANGE of each node, but when we flush to disk we want the full node. The assumption here is that what we have in memory is the full node.
        localStorage[soul] = JSON.stringify(at.gun.__.graph[soul]);
    }
    at.cb(null, {ok: "saved"}); // call the ACK callback.
});

Gun.on('get', function(at){
    at.cb(null, JSON.parse(localStorage[at.lex.soul])); // call the reply callback.
});
```

(v0.3.x API)
```javascript
var driver = {};

Gun.on('opt').event(function(gun, opt){
    // Note: This opt event gets called every time options update and for different instances of gun.

    gun.__.opt.wire.put = put;
    function put(graph, cb, o){
        for(var soul in graph){ // graph is the DELTA CHANGE of each node, but when we flush to disk we want the full node. The assumption here is that what we have in memory is the full node.
            localStorage[soul] = JSON.stringify(gun.__.graph[soul]);
        }
        cb(null, {ok: "saved"}); // call the ACK callback.
    }

    gun.__.opt.wire.get = get;
    function get(lex, cb, o){
        var soul = lex[Gun._.soul]; // only lookup the soul.
        var node = JSON.parse(localStorage[soul]); // grab it from localStorage.
        cb(null, node); // the current version of GUN
        cb(null, Gun.is.node.soul.ify({}, soul)); // has a streaming format for responses
        cb(null, {}); // which we are simplifying in the future.
    }
});
```


#Documentation
Documentation for each of gun's wire methods, `.get`, `.put` and `.key`.

## put(graph, callback, options)

### graph

The graph is a container with nodes going one level deep, each keyed by a unique ID.

### callback(error, okay)

The callback takes two arguments, `error` and `okay`. It *must* be invoked at some point, either with error or success, as gun's asynchronous engine depends on the response to continue execution.
- **errors** should be passed into the callback as an object `{err: new Error(msg)}`. If any data fails to persist, the entire `put` is considered a failure and should error out immediately.
- **okay** is an object `{ok: true}` passed as the second parameter.

### Object options

No options currently available.

## get(name, callback, options)

The responsiblity of get is to find the data requested and stream it back to the client.

## name

`get` can handle two types of requests: **[keys](#key)** and **[relations](#relation)**.

#### Key

A key is a graph containing relation objects, each pointing to other objects. `get`'s responsibility is to extract each soul, find it's corresponding object, and stream back a graph containing those objects, keyed by their souls.

#### Relation

A relation is an object that points to another object, or *references* that object. Relations are always objects containing only one property named `"#"`, who's value is a `soul` ([more detail](GUN’s-Data-Format-(JSON))). When passed a relation, `get` should return a graph containing the object it points to, keyed by it's soul.

### callback(error, okay)

The `get` operation, whether successful or not, should invoke the callback with the appropriate arguments.

#### Error

If your function irrevocably fails, invoke the callback with an error object `{ err: new Error(msg) }` and quit the operation.

#### Success/Streaming

The `get` method is expected to stream data back to the server, allowing three seperate types of responses:

##### object stream
```javascript
callback(null, {
  [soul]: { /* your object */ }
})
```
If you have a thousand properties on an object, you can break it into chunks and send one at a time. Respond with a graph containing the soul of the node and the data that belongs to it. *The node does not need to be complete*.

##### object termination
```javascript
callback(null, {
  [soul]: Gun.union.pseudo(soul)
})
```

This is gun's special termination sequence. It does *not* end the stream, it simply ends the node. Above we mentioned that your node could be broken into pieces and streamed. This is what we use to tell gun the entire node has been sent.

##### stream termination
```javascript
callback(null, {})
```

Once you've sent all the data requested, terminate the connection by sending an **empty object**.

### options

No options currently available.

## key(name, soul, callback)

`.key` takes a string as a name and a soul to associate with that name. Keys should be able to hold entire graphs of souls, storing them as relation objects, producable on demand from `.get`.

### name

The name can be any string. It is what will be passed into `.get` to return the data set.

### soul

By default, the soul is a unique 24-digit alphanumeric string that points to an object: `I1447469384953RQ7KG4`. It's what gun uses as a "link" to that object.

### callback(error, okay)

The function to invoke once we have finished.

**Error:** if the data fails to save, we send an error into the callback as the first argument: `callback({ err: new Error(msg) })`.


Once we're certain the key's been written, we invoke the callback with an object as the second parameter: `callback(null, { ok: true })`.

