Think these docs could be improved? Contribute to the wiki! Or [comment](https://github.com/amark/gun/issues/70).

**Core API**
 - [Gun constructor](#gun)
 - [gun.put](#put)
 - [gun.get](#get)
 - [gun.opt](#opt)
 - [gun.chain](#chain)
 - [gun.back](#back)

**Main API**
 - [gun.on](#on)
 - [gun.once](#once) ([gun.val](#val))
 - [gun.set](#set)
 - [gun.map](#map)

**User API (authenticated)**
 - [gun.user.create](https://gun.eco/docs/User#create)
 - [gun.user.auth](https://gun.eco/docs/User#auth) TBD
 - [gun.user.leave](https://gun.eco/docs/User#leave) TBD
 - [gun.user.delete](https://gun.eco/docs/User#delete) TBD
 - [gun.user.recall](https://gun.eco/docs/User#recall) TBD
 - [gun.user.alive](https://gun.eco/docs/User#alive) TBD
 - [gun.user.trust](https://gun.eco/docs/User#trust) TBD
 - [gun.user.grant](https://gun.eco/docs/User#grant) TBD
 - [gun.user.secret](https://gun.eco/docs/User#secret) TBD

**Extended API**
 - [gun.path](#path)
 - [gun.not](#not)
 - [gun.open](#open)
 - [gun.load](#load)
 - [gun.then](#then)
 - [gun.bye](#bye)
 - [gun.later](#later)
 - [gun.unset](#unset)

**Utils**
 - [Gun.node](#utilities)


----


# -- Core API --


## <a name="gun"></a>Gun(options)

<a href="https://youtu.be/zvo6jC1OA3Y" title="GUN constructor"><img src="http://img.youtube.com/vi/zvo6jC1OA3Y/0.jpg" width="425px"></a><br>
Used to create a new gun database instance.

```javascript
var gun = Gun(options)
```
> **note:** `Gun` works with or without the `new` operator.

### Options

 - no parameters `undefined` creates a local datastore using the default persistence layer, either localStorage or [Radisk](https://gun.eco/docs/Storage#radix).

 - passing a URL `string` creates the above local datastore that also tries to sync with the URL.

   - or you can pass in an `array` of URLs to sync with multiple peers.

 - the previous options are actually aggregated into an `object`, which you can pass in yourself.

   - `options.peers` is an object, where the URLs are properties, and the value is an empty object.

   - `options.radisk` (boolean, default: `true`) creates and persists local (nodejs) data using [Radisk](https://gun.eco/docs/Storage#radix).

   - `options.localStorage` (boolean, default: `true`) persists local (browser) data to localStorage.

   - `options.uuid` allows you to override the default 24 random alphanumeric soul generator with
      your own function.

   - `options['module name']` allows you to pass options to a 3rd party module. Their project README
     will likely list the exposed options.
     [Here is a list of such modules...](https://github.com/amark/gun/wiki/Modules)

### Examples
Sync with one peer
```javascript
Gun('http://yourdomain.com/gun')
```

Sync with many peers
```javascript
Gun(['http://server1.com/gun', 'http://server2.com/gun'])
```

Working with modules
```javascript
Gun({
  // Amazon S3 (comes bundled)
  s3: {
    key: '',
    secret: '',
    bucket: ''
  },

  // simple JSON persistence (bundled)
  // meant for ease of getting started
  // NOT meant for production
  file: 'file/path.json',

  // set your own UUID function
  uuid: function () {...}
})
```



## <a name="put"></a>gun.put(data, callback)

<a href="https://youtu.be/QLg-Z-y5sVo" title="GUN put"><img src="http://img.youtube.com/vi/QLg-Z-y5sVo/0.jpg" width="425px"></a><br>

Save data into gun, syncing it with your connected peers.

It has two parameters, and only the first is required:

 1. the `data` to save
 2. an optional `callback`, invoked on each acknowledgment

`gun.get('key').put({hello: "world"}, function(ack){})`

You do not need to re-save the entire object every time, gun will automatically merge your data into what already exists as a "partial" update.

### Allowed types

`.put` restricts the input to a specific subset:

 - `objects`: [partials](Partials-and-Circular-References), [circular](Partials-and-Circular-References), and nested
 - `strings`
 - `numbers`
 - `booleans`
 - `null`

Other values, like `undefined`, `NaN`, `Infinity`, `arrays`, will be rejected.

> Traditional arrays are dangerous in real-time apps. Use [gun.set](#set) instead.

> **Note:** when using `.put`, if any part of the chain does not exist yet, it will implicitly create it as an empty object.
```javascript
gun.get('something').get('not').get('exist').get('yet').put("Hello World!");
// `.put` will if needed, backwards create a document
// so "Hello World!" has a place to be saved.
```

### Callback(ack)
  
 - `ack.err`, if there was an error during save.
 - `ack.ok`, if there was a success message (none is required though).

The `callback` is fired for each peer that responds with an error or successful persistence message, including the local cache. Acknowledgement can be slow, but the write propagates across networks as fast as the pipes connecting them.

If the error property is undefined, then the operation succeeded, although the exact values are left up to the module developer.

### Examples

Saving objects
```javascript
gun.get('key').put({
  property: 'value',
  object: {
    nested: true
  }
})
```

Saving primitives
```javascript
// strings
gun.get('person').get('name').put('Alice')

// numbers
gun.get('IoT').get('temperature').put(58.6)

// booleans
gun.get('player').get('alive').put(true)
```

Using the callback
```javascript
gun.get('survey').get('submission').put(submission, function(ack){
  if(ack.err){
    return ui.show.error(ack.err)
  }
  ui.show.success(true)
})
```

### Chain context
`gun.put` does not change the gun context.
```javascript
gun.get('key').put(data) /* same context as */ gun.get('key')
```

### Unexpected behavior

You cannot save primitive values at the root level.
```javascript
Gun().put("oops"); // error
Gun().get("odd").put("oops"); // error
```
All data is normalized to a parent node.
```javascript
Gun().put({foo: 'bar'}); // internally becomes...
Gun().get(randomUUID).put({foo: 'bar'});

Gun().get('user').get('alice').put(data); // internally becomes...
Gun().get('user').put({'alice': data});
// An update to both user and alice happens, not just alice.
```
You can save a gun chain reference,
```javascript
var ref = Gun().put({text: 'Hello world!'})
Gun().get('message').get('first').put(ref)
```
But you cannot save it inline, yet.
```javascript
var sender = Gun().put({name: 'Tom'})
var msg = Gun().put({
  text: 'Hello world!',
  sender: sender // this will fail
})
// however
msg.get('sender').put(sender) // this will work
``` 
Be careful saving deeply nested objects,
```javascript
Gun().put({
  foo: {
    bar: {
      lol: {
        yay: true
      }
    }
  }
}):
```
For the most part, gun will handle this perfectly fine. It will attempt to automatically merge every nested object as a partial. However, if it cannot find data (due to a network failure, or a peer it has never spoken with) to merge with it will generate new random UUIDs. You are unlikely to see this in practice, because your apps will probably save data based on user interaction (with previously loaded data). But if you do have this problem, consider giving each one of your sub-objects a deterministic ID.




## <a name="get"></a>gun.get(key)
Where to read data from.

<a href="https://youtu.be/wNrIrrLffs4" title="GUN get"><img src="http://img.youtube.com/vi/wNrIrrLffs4/0.jpg" width="425px"></a><br>

It takes two parameters:

 - `key`
 - `callback`

`gun.get('key').get('property', function(ack){})`

You will usually be using [gun.on](#on) or [gun.once](#once) to actually retrieve your data, not this `callback` (it is intended for more low level control, for module and extensions).

### Key
The `key` is the ID or property name of the data that you saved from earlier (or that will be saved later).

> Note that if you use `.put` at any depth after a `get` it first reads the data and then writes, merging the data as a partial update.

```javascript
gun.get('key').put({property: 'value'})

gun.get('key').on(function(data, key){
  // {property: 'value'}, 'key'
})
```

### Callback(ack)

 - `ack.put`, the raw data.
 - `ack.get`, the key, ID, or property name of the data.

The callback is a listener for read errors, not found, and updates. It may be called multiple times for a single request, since gun uses a reactive streaming architecture. Generally, you'll find [`.not`](#not), [`.on`](#on), and [`.once`](#once) as more convenient for every day use. Skip to those!

```javascript
gun.get(key, function(ack){
  // called many times
})
```

### Examples

Retrieving a key
```javascript
// retrieve all available users
gun.get('users').map().on(ui.show.users)
```

Using the callback
```javascript
gun.get(key, function(ack){
  if(ack.err){
    server.log(error)
  } else
  if(!ack.put){
    // not found
  } else {
    // data!
  }
})
```

### Chain context
Chaining multiple `get`s together changes the context of the chain, allowing you to access, traverse, and navigate a graph, node, table, or document.

> Note: For users upgrading versions, prior to v0.5.x `get` used to always return a context from the absolute root of the database. If you want to go back to the root, either save a reference `var root = Gun();` or now use [`.back(-1)`](#back).

```javascript
gun.get('user').get('alice') /* same context as */ gun.get('users').get('alice')
```

### Unexpected behavior

Most callbacks in gun will be called multiple times.




## <a name="opt"></a> gun.opt(options)
Change the configuration of the gun database instance.

The `options` argument is the same object you pass to the [constructor](#gun). The `options`'s properties replace those in the instance's configuration but `options.peers` are **added** to peers known to the gun instance.

### Examples
Create the gun instance.
```javascript
gun = Gun('http://yourdomain.com/gun')
```
Change UUID generator:
```javascript
gun.opt({
  uuid: function () {
    return Math.floor(Math.random() * 4294967296);
  }
});
```
Add more peers:
```javascript
gun.opt({peers: ['http://anotherdomain.com/gun']})
/* Now gun syncs with ['http://yourdomain.com/gun', 'http://anotherdomain.com/gun']. */
```



## <a name="back"></a>gun.back(amount)

Move up to the parent context on the chain.

Every time a new chain is created, a reference to the old context is kept to go `back` to.

### Amount

The number of times you want to go back up the chain. `-1` or `Infinity` will take you to the root.

### Examples
Moving to a parent context
```javascript
gun.get('users')
  /* now change the context to alice */
  .get('alice')
  .put(data)
  /* go back up the chain once, to 'users' */
  .back().map(...)
```

Another example
```javascript
gun.get('player').get('game').get('score').back(1)
// is the same as...
gun.get('player').get('game')
```

### Chain context
The context will always be different, returning you to the
```javascript
gun.get('key').get('property')
/* is not the same as */
gun.get('key').get('property').back()
```





# -- Main API --



## <a name="on"></a> gun.on(callback, option)

<a href="https://youtu.be/SEneRvDQysE" title="GUN on"><img src="http://img.youtube.com/vi/SEneRvDQysE/0.jpg" width="425px"></a><br>

Subscribe to updates and changes on a node or property in realtime.

### Callback(data, key)
Once initially and whenever the property or node you're focused on changes, this callback is immediately fired with the data as it is at that point in time.

Since gun streams data, the callback will probably be called multiple times as new chunk comes in.

To remove a listener call .off() on the same property or node.

 > Note: If data is a node (object), it may have `_` meta property on it. If you [delete](https://gun.eco/docs/Delete) an item, you might get a `null` tombstone.

### Option
Currently, the only option is to filter out old data, and just be given the changes. If you're listening to a node with 100 fields, and just one changes, you'll instead be passed a node with a single property representing that change rather than the full node every time.

**Longhand syntax**
```javascript
// add listener to foo
gun.get('foo').on(callback, {
  change: true
})

// remove listener to foo
gun.get('foo').off()
```

**Shorthand syntax**
```javascript
// add listener to foo
gun.get('foo').on(callback, true)

// remove listener to foo
gun.get('foo').off()
```

### Examples
Listening for updates on a key
```javascript
gun.get('users').get(username).on(function(user){
  // update in real-time
  if (user.online) {
    view.show.active(user.name)
  } else {
    view.show.offline(user.name)
  }
})
```

Listening to updates on a field
```javascript
gun.get('lights').get('living room').on(function(state, room){
  // update the UI when the living room lights change state
  view.lights[room].show(state)
})
```
### IMPORTANT
:exclamation: There's a 'bug' when dealing with multiple levels.
```
gun.get('home').get('lights').on(cb,true);
gun.get('home').get('lights').put({state:'on'})       // BAD fires twice
gun.get('home').get('lights').get('state').put('on')  // GOOD fires once
```
### Chain Context
`gun.on` does not change the chain context.
```javascript
gun.get(key).on(handler) /* is the same as */ gun.get(key)
```

### Unexpected behavior

Data is only 1 layer deep, a full document is not returned (see the [gun.open](#open) extension for that), this helps keep things fast.

It will be called many times.



## <a name="once"></a> gun.once(callback, option)

<a href="https://youtu.be/k-CkP43-uJo" title="GUN once"><img src="http://img.youtube.com/vi/k-CkP43-uJo/0.jpg" width="425px"></a><br>

Get the current data without subscribing to updates. Or `undefined` if it cannot be found.

### Option

 - `wait` controls the asynchronous timing (see unexpected behavior, below). `gun.get('foo').once(cb, {wait: 0})`

### Callback(data, key)
The data is the value for that chain at that given point in time. And the key is the last property name or ID of the node.

 > Note: If data is a node (object), it may have `_` meta property on it. If you [delete](https://gun.eco/docs/Delete) an item, you might get a `null` tombstone. If the data cannot be found, `undefined` may be called back.

### Examples
```javascript
gun.get('peer').get(userID).get('profile').once(function(profile){
  // render it, but only once. No updates.
  view.show.user(profile)
})
```

Reading a property
```javascript
gun.get('IoT').get('temperature').once(function(number){
  view.show.temp(number)
})
```

### Chain Context
`gun.once` does not currently change the context of the chain, but it is being discussed for future versions that it will - so try to avoid chaining off of `.once` for now. This feature is now in experimental mode with `v0.6.x`, but only if `.once()` is not passed a callback. A useful example would be `gun.get('users').once().map().on(cb)` this will tell gun to get the current users in the list and subscribe to each of them, but not any new ones. Please test this behavior and recommend suggestions.

### Unexpected behavior

`.once` is synchronous and immediate (at extremely high performance) if the data has already been loaded.

`.once` is asynchronous and on a **debounce timeout** while data is still being loaded - so it may be called completely out of order compared to other functions. This is intended because gun streams partials of data, so `once` avoids firing immediately because it may not represent the "complete" data set yet. You can control this timeout with the `wait` option.

`once` fires again if you update that node from within it.

Example for `once` firing again with an update from within:

    node.once(function(data, key) {
      node.get('something').put('something')
    }

Data is only 1 layer deep, a full document is not returned (see the [gun.load](#open) extension for that), this helps keep things fast.



## <a name="set"></a>gun.set(data, callback)

Add a unique item to an unordered list.

`gun.set` works like a mathematical set, where each item in the list is unique. If the item is added twice, it will be merged. This means only objects, for now, are supported.

_To remove items from a set, see [gun.unset](#unset)._

### Data
Data should be a gun reference or an object.
```javascript
var user = gun.get('alice').put({name: "Alice"});
gun.get('users').set(user);
```

### Callback
The callback is invoked exactly the same as `.put`, since `.set` is just a convenience wrapper around `.put`.

### Examples

```javascript
var gun = Gun();
var bob = gun.get('bob').put({name: "Bob"});
var dave = gun.get('dave').put({name: "Dave"});

dave.get('friends').set(bob);
bob.get('friends').set(dave);
```
The "friends" example is perfect, since the set guarantees that you won't have duplicates in your list.

### Chain Context
`gun.set` changes the chain context, it returns the item reference.
```javascript
gun.get('friends') /* is not the same as */ gun.get('friends').set(friend)
```



## <a name="map"></a>gun.map(callback)

<a href="https://youtu.be/F2FSMsxMSic" title="GUN map"><img src="http://img.youtube.com/vi/F2FSMsxMSic/0.jpg" width="425px"></a><br>

Map iterates over each property and item on a node, passing it down the chain, behaving like a forEach on your data. It also subscribes to every item as well and listens for newly inserted items. It accepts one argument:

 - a `callback` function that transforms the data as it passes through. If the data is transformed to `undefined` it gets filtered out of the chain.
- the `callback` gets two arguments (value,key) and will be called once for each key value pair in the objects that are returned from `map`.

> Note: As of `v0.6.x` the transform function is in experimental mode. Please play with it and report bugs or suggestions on how it could be improved to be more useful.

### Examples
Iterate over an object
```javascript
/*
  where `stats` are {
    'new customers': 35,
    'returning': 65
  }
*/
gun.get('stats').map().on(function(percent, category) {
  pie.chart(category, percent)
})
```
The first call to the above will be with (35,'new customers') and the second will be with (65,'returning').

Or `forEach`ing through every user.
```javascript
/*
{
  user123: "Mark",
  user456: "Dex",
  user789: "Bob"
}
*/
gun.get('users').map().once(function(user, id){
  ui.list.user(user);
});
```
The above will be called 3 times.

Here's a summary of .map() behavior depending on where it is on the chain:
- users.map().on(cb) subscribes to changes on every user and to users as they are added.
- users.map().once(cb) gets each user once, including ones that are added over time.
- users.once().map().on(cb) gets the user list once, but subscribes to changes on each of those users (not added ones).
- users.once().map().once(cb) gets the user list once, gets each of those users only once (not added ones).

Iterate over and only return matching property
```javascript
/*
{
  user123: "Mark",
  user456: "Dex",
  user789: "Bob"
}
*/
gun.get('users').map(user => user.name === 'Mark'? user : undefined).once(function(user, id){
  ui.list.user(user);
});
```

Will only return user123: "Mark", as it was the only match.

### Chain context
`.map` changes the context of the chain to hold many chains simultaneously. Check out this example:
```javascript
gun.get('users').map().get('name').on(cb);
```
Everything after the `map()` will be done for every item in the list, such that you'll get called with each name for every user in the list. This can be combined in really expressive and powerful ways.
```javascript
gun.get('users').map().get('friends').map().get('pet').on(cb);
```
This will give you each pet of every friend of every user!

```javascript
gun.get(key).map() /* is not the same as */ gun.get(key)
```





# -- Extended API --



## <a name="path"></a>gun.path(key)

<a href="https://youtu.be/UDZGVYLNLAU" title="GUN path"><img src="http://img.youtube.com/vi/UDZGVYLNLAU/0.jpg" width="425px"></a><br>

> Warning: This extension was removed from core, you probably shouldn't be using it!

> Warning: Not included by default! You must include it yourself via `require('gun/lib/path.js')` or `<script src="https://cdn.jsdelivr.net/npm/gun/lib/path.js"></script>`!

Path does the same thing as `get` but has some conveniences built in.

### Key
The key `property` is the name of the field to move to.

```javascript
// move to the "themes" field on the settings object
gun.get('settings').path('themes')
```

Once you've changed the context, you can read, write, and `path` again from that field. While you can just chain one `path` after another, it becomes verbose, so there are two shorthand styles:

 - dot format
 - array format

Here's dot notation in action:
```javascript
// verbose
gun.get('settings').path('themes').path('active')

// shorthand
gun.get('settings').path('themes.active')

// which happens to be the the same as
gun.get('settings').get('themes').get('active')
```


And the array format, which really becomes useful when using variables instead of literal strings:
```javascript
gun.get('settings').path(['themes', themeName])
```

### Unexpected behavior
The dot notation can do some strange things if you're not expecting it. Under the hood, everything is changed into a string, including floating point numbers. If you use a decimal in your path, it will split into two paths...
```javascript
gun.path(30.5)
// interprets to
gun.path(30).path(5)
```

This can be especially confusing as the chain might never resolve to a value.

> Note: For users upgrading from versions prior to v0.5.x, `path` used to be necessary - now it is purely a convenience wrapper around `get`.

### Examples
Navigating to a property
```javascript
/*
  where `user` is {
    name: 'Bob'
  }
*/
gun.get('user').path('name')
```
Once you've focused on the `name` property, you can chain other methods like [`.put`](#put) or [`.on`](#on) to interact with it.

Moving through multiple properties
```javascript
/*
  where `user` is {
    name: { first: 'bob' }
  }
*/
gun.get('user').path('name').path('first')
// or the shorthand...
gun.get('user').path('name.first')
```

### Chain context
`gun.path` creates a new context each time it's called, and is always a result of the previous context.
```javascript
gun.get('API').path('path').path('chain')
/* is different from */
gun.get('API').path('path')
/* and is different from */
gun.get('API')
```



## <a name="not"></a> gun.not(callback)

> Warning: Not included by default! You must include it yourself via `require('gun/lib/not.js')` or `<script src="https://cdn.jsdelivr.net/npm/gun/lib/not.js"></script>`!
Handle cases where data can't be found.

If you need to know whether a property or key exists, you can check with `.not`. It will consult the connected peers and invoke the callback if there's reasonable certainty that none of them have the data available.

> **Warning:** `.not` has no guarantees, since data could theoretically exist on an unrelated peer that we have no knowledge of. If you only have one server, and data is synced through it, then you have a pretty reasonable assurance that a `not` found means that the data doesn't exist yet. Just be mindful of how you use it.

### Callback(key)
If there's reason to believe the data doesn't exist, the callback will be invoked. This can be used as a check to prevent implicitly writing data (as described in [`.put`](#put)).

### Key
The name of the property or key that could not be found.

### Examples
Providing defaults if they aren't found
```javascript
// if not found
gun.get('players/3').not(function(key){
  // put in an object and key it
  gun.get(key).put({
    active: false
  });
}).on(handler)
// listen for changes on that key
```

Setting a property if it isn't found
```javascript
gun.get('chat').get('enabled').not(function(key){
  this.put(false)
})
```

### Chain context
`.not` does not change the context of the chain.
```javascript
gun.get(key).not(handler) /* is the same as */ gun.get(key)
```



## <a name="open"></a> gun.open(callback)

> Warning: Not included by default! You must include it yourself via `require('gun/lib/open.js')` or `<script src="https://cdn.jsdelivr.net/npm/gun/lib/open.js"></script>`!

Open behaves very similarly to [gun.on](#on), except it gives you **the full depth of a document** on every update. It also works with graphs, tables, or other data structures. Think of it as opening up a live connection to a document.

> Note: This will automatically load everything it can find on the context. This may sound convenient, but may be unnecessary and excessive - resulting in more bandwidth and slower load times for larger data. It could also result in your entire database being loaded, if your app is highly interconnected.

### Callback(data)

The callback has 1 parameter, and will get called every time an update happens anywhere in the full depth of the data.

 - `data`. Unlike most of the API, `open` does not give you a node. It gives you a copy of your data with all metadata removed. Updates to the callback will return the same data, with changes modified onto it.

### Examples
```javascript
// include .open
gun.get('person/mark').open(function(mark){
  mark; // {name: "Mark Nadal", pet: {name: "Frizzles", species: "kitty", slave: {...}}}
});

var human = {
  name: "Mark Nadal",
  pet: {
    name: "Frizzles",
    species: "kitty" // for science!
  }
};
human.pet.slave = human;

gun.get('person/mark').put(human);
```

### Chain context
`.open` does not change the context.

```javascript
gun.get('company/acme').open(cb).get('employees').map().once(cb)
```

### Unexpected behavior

If you do not use a schema with `.open(cb)` it can only best guess and approximate whether the data is fully loaded or not. As a result, do not assume all the data will be available on the first callback - it may take several calls for things to fully load, so code defensively! By default, it waits 1ms after each piece of data it receives before triggering the callback. You can change the default by passing an option like `.open(cb, {wait: 99})` which forces it to wait 99ms before triggering (which is the default [gun.once](#once) has).



## <a name="load"></a> gun.load(cb, opt)

> Warning: Not included by default! You must include it yourself via `require('gun/lib/load.js')` or `<script src="https://cdn.jsdelivr.net/npm/gun/lib/open.js"></script><script src="https://cdn.jsdelivr.net/npm/gun/lib/load.js"></script>`!

Loads the full object once. It is the same as `open` but with the behavior of [`once`](#once).



## <a name="then"></a> gun.then(cb)

> Warning: Not included by default! You must include it yourself via `require('gun/lib/then.js')` or `<script src="https://cdn.jsdelivr.net/npm/gun/lib/then.js"></script>`!

Returns a promise for you to use.

> Note: a gun chain is not promises! You must include and call `.then()` to promisify a gun chain!

### cb(resolved)

`cb` is a function that has 1 parameter.

`resolved` is the data.



## .promise(cb)

`.then(cb)` has a cousin of `.promise(cb)` which behaves the same way except that `resolved` is an object with:

```javascript
resolved = {
  put: data,
  get: key,
  gun: ref // if applicable
}
```

In case you need more context or metadata.

### Chain context

It is no longer a gun chain, but you can chain promises off of it!

### Examples

```javascript
Promise.race([
    gun.get('alice').then(), // must be called!
    gun.get('bob').then() // must be called!
])
```

Or use it with `async`!

```javascript
async function get(name) {
     var node = await gun.get(name).then();
     return node;
};
```

### Unexpected behavior

A gun chain is **not** already a promise, you must call `then()` to make it a promise.



## <a name="bye"></a> gun.bye()

> Warning: Not included by default! You must include it yourself via `require('gun/lib/bye.js')` or `<script src="https://cdn.jsdelivr.net/npm/gun/lib/bye.js"></script>`!

`bye` lets you change data after that browser peer disconnects. This is useful for games and status messages, that if a player leaves you can remove them from the game or set a user's status to "away".

> Note: This requires a server side component, and therefore must be included there as well in order for this to work. In the future it should be generalizable to P2P settings, however may not be as reliable. 

### Chain context

It returns a **special chain context** with **only 1 method on it** of `put`. It currently **does not support chaining**, however in the future we hope to make it more chainable. Keep this in mind until then.

### Examples

```javascript
gun.get('marknadal').get('status').put("I'm online!");

gun.get('marknadal').get('status').bye().put("I'm offline. :(");
```

Even though this is **written entirely in the browser**, the `put` will get called **on the server** when this browser tab disconnects.

> Note: A user might have multiple tabs open on your website, just because they close 1 tab does not mean they are now offline. We recommend you use `.bye()` for data related to browser tab sessions, and then aggregate those sessions together to determine if the the user as a whole is online or offline.

```javascript
var player = gun.get('kittycommando1337');

gun.get('game').get('players').set(player);

gun.get('game').get('players').get('kittycommando1337').bye().put(null);
```

This deletes the player from the game when they go offline or disconnect from the server. It does not delete the player, just whether they are a player of the game or not.

### Unexpected behavior

`bye()` is in experimental alpha, please report any problems or bugs you have with it. Note again that it does not fire immediately, and it does not get run from the browser. It makes the data change on the server when that browser tab disconnects.



## <a name="later"></a> gun.later(cb, seconds)

> Warning: Not included by default! You must include it yourself via `require('gun/lib/later.js')` or `<script src="/gun/lib/open.js"></script><script src="https://cdn.jsdelivr.net/npm/gun/lib/later.js"></script>`!

Say you save some data, but want to do something with it later, like expire it or refresh it. Well, then `later` is for you! You could use this to easily implement a TTL or similar behavior.

### cb(data, key)

 - `data`, is a safe snapshot of what you saved, including full depth documents or circular graphs, without any of the metadata.
 - `key` is the name of the data.
 - `this` is the gun reference that it was called with.

### seconds

Is the number of seconds you want to wait before firing the callback.

### Chain context

It returns itself, as in `gun.later() === gun`.

### Examples

See a full working [example here, at jsbin](http://jsbin.com/veyezoxuhe/edit?html,js,console)!

```javascript
gun.get('foo').put(data).later(function(data, key){
  this.get('session').put(null); // expire data!
}, 2); // 2 seconds!
```

### Unexpected behavior

1. Exact timing is not guaranteed! Because it uses `setTimeout` underneath. Further, after the timeout, it must then open and load the snapshot, this will likely add at least `1`ms to the delay. **Experimental**: If this is problematic, please report it, as we can modify the implementation of `later` to be more precise.)

2. If a process/browser has to restart, the timeout will not be called. **Experimental**: If this behavior is needed, please report it, as it could be added to the implementation.



## <a name="unset"></a> gun.unset(node)

> Warning: Not included by default! You must include it yourself via `require('gun/lib/unset.js')` or `<script src="https://cdn.jsdelivr.net/npm/gun/lib/unset.js"></script>`!

After you save some data in an unordered list, you may need to remove it. 

```JavaScript
let gun = new Gun();
let machines = gun.get('machines');
let machine = gun.get('machine/tesseract');
machine.put({faces: 24, cells: 8, edges: 32});
// let's add machine to the list of machines;
machines.set(machine);
// now let's remove machine from the list of machines
machines.unset(machine);
```





# <a name="utilities"></a> -- Gun utils --

While running, Gun provides several high-level utility functions for querying and manipulating our components.

Note the capital "G" in `Gun`, as opposed to an instance variable called `gun`.



## Gun.node.is(data)

Returns true if `data` is a gun node, otherwise false.



## Gun.node.soul(data)

Returns `data`'s `gun` ID (instead of manually grabbing its metadata i.e. `data["_"]["#"]`, which is faster but could change in the future).

Returns `undefined` if `data` is not correct gun data.



## Gun.node.ify(json)

Returns a "gun-ified" variant of the `json` input by injecting a new gun ID into the metadata field.