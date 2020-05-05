> WARNING: This documentation is out of date. We use a different storage adapter system now.

Use @sjones6 's excellent gun `flint` framework instead in the meanwhile (and even after) we update documentation:

https://github.com/sjones6/gun-flint

# Persistence

Making storage drivers for gun

There are a lot of methods and tools gun provides to manipulate data, but there are only four that have to do with saving reading and writing:

- [Get:](API-(v0.3.x)#get) fetch from the persistence layer
- [Put:](API-(v0.3.x)#put) save to the persistence layer
- [Key:](API-(v0.3.x)#key) name groups of data

We'll cover these operations and how they work, then at the end we'll see how to [expose those methods](#exposing-your-functions-to-gun) to gun. If you're looking for a more technical reading, take a look at the [wire spec](Wire-specification-and-API-(v0.2.x)).

#### standard arguments

There are two types of arguments that are included in most actions, `callback` and `options`.

**Callback:** the callback is a function that accepts two parameters, (`err`, `data`). It's part of gun, and will always be provided.
- **Errors:** when things go irrevocably wrong, send an error back: `callback({ err: new Error(msg) })`.
- **Success:** if everything goes according to plan, `null` is the customary error field, while either the data or `true` is passed in the second parameter: `callback(null, data)`.

**Options:** The options object is what is passed into the Gun constructor when creating an instance. Those options are passed into `.get`, `.put` and `.key`. If you want, you can provide your users with configurations and settings:
```javascript
new Gun({
  persistenceName: {
    path: 'path/to/data',
    errors: 'throw'
  }
})
```

> keep in mind it's perfectly fine not to expose any options

### saving data with .put

Every save operation in gun channels through `.put`. Gun formats the data before sending it to you, passing it as the first argument using gun's favorite data structure, the **graph**. Graphs are objects filled with other objects a single layer deep, who's property names are randomly generated strings we call "souls". It's pretty hard to visualize, so it can be handy to review [GUN’s Data Format](GUN’s-Data-Format-(JSON)) and [Partials & Circular References](Partials-and-Circular-References-(v0.2.x)) before continuing.

You will need to accept three parameters, `(graph, callback, options)`.

The graph you're passed will look something like this:
```javascript
{
  '5v8GrX2p23L3xw0IFZxWPiqr': {
    firstname: 'john',
    lastname: 'smithsner',
    pet: { '#': '3KZefLpzOksix3iPE08Cl0Jh' }
  },
  '3KZefLpzOksix3iPE08Cl0Jh': {
    name: 'pugsley'
  }
}
```

Ultimately, the way you choose to store the data is up to you, but it's easiest to loop over the object, saving the key (soul), and the value (stringified object). Since graphs are so ubiquitous in gun, it has an exposed method that loops over them for you, passing you each soul and object inside.

```javascript
Gun.is.graph(graph, function (node, soul) {
  var key = soul;
  var val = Gun.text.ify(node);
  database.save(key, val)
})
```

### naming data with .key

Keys are basically groups of data. You might want a group named "fabulous taco places", but regretfully, that's not a natural grouping of restaurants. Keys would let you gather those otherwise unrelated restaurants into one collection of taco joints (remember that gun works off of souls, so you'd just be grouping the *addresses* of the restaurants, not piling them all into one massive cart).

To key something, you'll need the name of the key, the soul that you'll be keying, and a callback to notify when you're finished: `(name, soul, callback)`

Knowing that we'll need to hold several references, we might benefit from storing this data as a graph:
```javascript
name: {
  // This object is what we call a "relation"
  [soul1]: {
    '#': soul1
  },
  [soul2]: {
    '#': soul2
  }
}
```

Gun exposes methods for iterating over graphs, making this structure easy to pull from when we eventually request the key. Once the write has finished (or errored out), go ahead and invoke the callback. Remember that the callback [expects arguments](#standard-arguments).

### retrieval with .get

There are two types of arguments that `.get` should accept: **keys** and **relations**.

> recap: relations are what gun uses to point to other objects, and they look like this: { "#": "WXsKe5CfM28I" }

Reading out a soul is fairly straightforward, but aren't keys a whole different animal? Well, let's take a look a the structure...
```javascript
// the graph your key refers to
{
  // each key points to a relation
  "13o59D49rOZUN": {
    '#': "13o59D49rOZUN"
  },
  "InBRDX4BRhC": {
    '#': "InBRDX4BRhC"
  }
}
```
Wait a minute, `.get` can take relations! If we just send each relation back into `.get`, then we've just finished all of our key logic!

We can use `Gun.is.soul(relation)` to figure out if it's a relation. If it is, the method will return the soul, otherwise it returns `false`.
```javascript
function get(name, callback, options) {
  var soul = Gun.is.soul(name);
  if (soul) {
    // we've been passed a relation!
  } else {
    // we've been passed a key
  }
}
```

Now, we can distinguish between keys and relations, and keys are just a collection of relations (which we can read back using `.get`). All that's left is reading out the soul and streaming it to the client.

> note: some databases consider "key not found" to be an error, while gun treats it as a new collection. Send null as the value `callback(null, null)`, letting gun know it's empty.

```javascript
if (soul) {
  yourDatabase.read(soul).then(function (node) {
    // now we send a graph back to the client
    
    // send this graph to the client
    callback(null, {
      [soul]: node
    })
    
    // This is our termination sequence
    // more on this in a second...
    callback(null, {
      [soul]: Gun.union.pseudo(soul)
    })
  })
}
```

So why `Gun.union.pseudo`? Well, since we're streaming out the data, we don't need to send the *entire object*. If we have thousands of properties we need to send, we can break it into chunks and stream one piece at a time. `pseudo` returns a *termination* value that essentially says "that's all we have for this node, let's move onto the next one", finishing out that object stream.

So let's step through what we've got:

```javascript
function get(name, callback, options) {
  // Is this a relation? If so, grab it's soul
  var soul = Gun.is.soul(name)
  
  if (soul) {
    yourDatabase.read(soul).then(function (node) {
      // send it back to the client!
      callback(null, {
        [soul]: node
      })
      
      // finish/terminate our object stream
      callback(null, {
        [soul]: Gun.union.pseudo(soul)
      })
    })
  } else {
    // If it's not a relation, it's a key
    yourDatabase.read(key).then(function (graph) {
      // keys are graphs filled with relations
      
      // loop over our graph
      Gun.obj.map(graph, function (relation) {
        // send it through get as a relation this time
        get(relation, callback);
      })
    })
  }
  // terminate the entire stream
  // this means that all your data has been sent
  callback(null, {});
}
```

So, to recap: `.get` can take two types of arguments, **keys** and **relations**. Relations are objects that link to other objects, and keys are graphs filled with those relations. When passed a relation, we extract it's soul, read the node it points to from the database, and stream it back to the client. If we are given a key, we read it's graph from persistence and repeat that process for every relation inside it. Now the only thing left to do is expose your functions to gun!

## exposing your functions to gun

Gun exposes hooks for each of the functions we've covered. They're basically events that will only fire when the method is called with *valid input*, so if you were to pass Infinity into `.put`, it's regarded as invalid and your function is never called, meaning you never have to worry about validation. Here's how you can subscribe:

```javascript
gun.opt({
  // override the default methods with your own
  hooks: {
    put: function () {...},
    key: function () {...},
    get: function () {...}
  }
})
```

Congrats, you're now subscribed to the gun instance! Wait a minute, the *instance?* Don't we want this used throughout gun? Yes. This is only half of the picture, and the other half is the gun `opt` event:

```javascript
// any time options are passed
Gun.on('opt').event(function () {
  // run this function
  this.opt({
    hooks: {
      put: opt.hooks.put || yourPutFunction,
      key: opt.hooks.key || yourKeyFunction,
      get: opt.hooks.get || yourGetFunction
    }
  }, true);
})
```
Since this function could potentially be run several times, we test to see if we've already set those methods, and if we have, use those instead of creating new ones.

> The `true` argument above is very important. It prevents the 'opt' event from firing again and spawning an infinite loop.

## final notes

It should be mentioned that you can override these methods on the server *and* on the client, meaning that you can exchange the localStorage engine with something else.

If you want succinct documentation for the `.get`, `.put` and `.key` contracts, you can find it in the [Wire specification and API](Wire-specification-and-API-(v0.2.x)).

That should be about everything you'd want to know about making a persistence layer. If you have any questions, submit an issue or post in our [gitter channel](https://gitter.im/amark/gun) and we'll do our best to answer them! Thanks for reading :)