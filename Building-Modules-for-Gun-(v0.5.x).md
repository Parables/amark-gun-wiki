> **Note:** As of this writing, the v0.5 branch is still in beta. The API is still subject to change.

In version 0.5, gun's module API is greatly improved. It holds many advantages over the previous model:
- 3rd party methods can stream data into other methods (like `.map` can stream into `.path`)
- Backend drivers play nicer with one another.
- Streamlined get/put plugin interface.
- Massive performance improvements.

This document is a work in progress, so we're gonna zip through some stuff. Lot's of code examples.

## Hooking Into Gun
The `Gun` constructor has several events your plugin can tap into:

- **`opt`:** fires whenever a new gun instance is created, or if the options change via `.opt()`.
- **`get`:** when any gun instance tries to read a value, this event is fired.
- **`put`:** when a new value is written to gun, this event fires.

Here's an example:

```js
Gun.on('opt', function (context) {
	console.log('Options changed!')
})
```

The common plugin pattern is to subscribe to the `opt` event, adding plugin-specific flags to the gun instance. Whenever `get` or `put` is fired, you check to see if the flags exist, and use them to answer the request.

Say you've got a database backend driver. On your project docs, you'd tell your users what option you're watching for. Let's say it's `opt.myDB`.

You'd subscribe to the `opt` event, so you know whenever the options change. If the `opt.myDB` wasn't passed, don't do anything, just return.

```js
Gun.on('opt', function (context) {

	// Get the plugin-specific options
	var options = context.opt.myDB

	// The user didn't activate this plugin.
	if (!options) {
		return
	}

	// Set plugin-specific flags.
	// We'll come back to this in a moment...
	context.gun._.myDB = options

	// ... plugin logic ...
})
```

In this case, a user would activate the plugin by passing your option:

```js
Gun({
	myDB: true,
})
```

That options object is passed directly to you. How you decide to interpret the input is up to your imagination.

Now, let's do something interesting and handle read requests!

```js
// Listen globally for any get request
Gun.on('get', function (context) {
	var gun = context.gun

	// Make sure the developer wants this
	// plugin to handle it's get requests.
	if (!gun._.myDB) {
		return
	}

	// Here's what the get request is:
	var request = context.get
})
```

The request is what gun calls a "lexical cursor". It's somewhat like a query. Here's a summary:

```js
var cursor = {

	// This specific key (optional, but usually provided)
	'#': 'key',

	// Anything matching this field (optional)
	'.': 'field name',

	// Anything matching this value (optional)
	'=': 'value',

	// Anything matching this state (optional)
	'>': 'state',
}
```

Cursors can get more complicated though. Each field can either point to a primitive (like above), or it can describe a more complex pattern.

Using a cursor, you could say "give me all fields that are greater than `95` in the object `performance/machine/8`".

```js
var cursor = {
	'#': 'performance/machine/8',

	// Value
	'=': {

		// "gt" selector
		'>': 95,
	},
}
```

Right now, the cursors aren't well known, and are usually kept pretty simple. Just something to keep in mind, a feature if you want to support it.

> Lexical cursors are still being developed, and while being mostly stable, are still subject to change. More documentation is coming.

So, for our example, let's keep things a bit simple and only look for `"key"` selectors.

```js
Gun.on('get', function (context) {
	// ... filter instances without our plugin ...

	var cursor = context.get

	// This *may* not exist.
	var key = cursor['#']

	// For simplicity, we're not worrying
	// about more complex cursors.
	if (!key) {
		return
	}

	// This is what gun uses to uniquely ID the request.
	var requestID = context['#']

	// Get the root chain context.
	var root = context.gun._.root

	// This is where you'd read from your database.
	// Naturally, we're looking at pseudocode.
	myDatabase.read(key).then(function (value) {

		// Now we tell gun what we found:
		// (send data into it's input channel)
		root.on('in', {

			// Tell gun what request you're
			// responding to.
			'@': requestID,

			// If we failed, we pass the error here.
			err: null,

			// Respond with a value:
			put: value,
		})

	})
})
```

Play around with the code above, `console.log` everything, just get a feel for how the API works and what it's expecting.

Onto writing values! Gun works in graphs, so your write handler will be passed a graph. We'll cover what it looks like in a moment...

```js
Gun.on('put', function (context) {
	var gun = context.gun

	// Once again, make sure they want you
	// to handle the write.
	if (!gun._.myDB) {
		return
	}

	// What gun wants us to write.
	var graph = context.put
})
```

Looking familiar? It's mostly the same process as `Gun.on('get')`, but that'll change shortly. We're gonna take a look at what gun's graph structure looks like.

> **Note:** The depiction below isn't &ast;perfectly&ast; accurate, but good enough for our purposes.

```js
// A graph is an object that contains other objects.
var graph = {

	// Each field is the unique ID of the node it points to.
	'user/123': {
		name: 'Alice',

		// Nodes cannot have nested objects,
		// only pointers to other objects.
		friend: { '#': 'user/456' },
	},

	'user/456': {
		name: 'Bob',
		friend: { '#': 'user/123' },
	},
}
```

We want to write each node, and make it discoverable by it's unique ID (sometimes called a "soul").

Let's go ahead and do that now.

```js
Gun.on('put', function (context) {
	// ... our code from before ...

	var graph = context.put

	// For each node in the graph...
	var writes = Object.keys(graph).map(function (key) {

		// Get a reference to the node...
		var node = graph[key]

		// And write it to the database.
		return myDatabase.write(key, node)
	})

	// Once we're all done...
	Promise.all(writes).then(function () {

		// Tell gun we were successful.
		Gun.on.ack(context, {

			// Failed? Report it here.
			err: null,

			// Otherwise you can provide a status.
			ok: 'optional message :)'
		})
	})
})
```

Congrats, you've just implemented a storage engine! Network drivers should work basically the same way.

> **Note:** This document is a work in progress. As v0.5 moves forward, this documentation should improve.

If you're curious about our plugin system, feel free to swing by our [Gitter channel](https://gitter.im/amark/gun) and send us your questions.
