With the news of the [Parse shutdown](http://blog.parse.com/announcements/moving-on/), we've decided to write a Parse-specific migration guide to [gunDB](https://github.com/amark/gun).

> **Note:** until gunDB reaches a stable 1.0, we are focusing our development solely on JavaScript platforms.

> **Note:** gun does not support push notifications or binary files.

## What is gunDB?
Gun is an easy, modular Javascript database that runs on both clients and Node.js servers. It's built from the ground up with real-time data in mind, while being offline friendly. By using graph structures under the hood, gun allows you to save any data you can express in a Javascript object (document, tabular, key-value, etc.) and read it back with an O(1) lookup.

## How is it different from Parse?
While Parse is a hosted solution with everything out of the box, gunDB is self-hosted and designed to be lightweight and modular, however both are designed primarily for client use. The most obvious difference between the two is gun's strong real-time tooling.

## API
How the Parse API compares to gun's.

### Setup
For Parse, they need the ID for your app, and the platform it's running on.
```javascript
Parse.initialize(app, platform)
```

Gun just needs the server you're connecting to, and can take some optional configuration.
```javascript
var db = new Gun(['http://your-server.com/gun'])
```

As the array implies, you can sync with more than one server. Just pass another URL.

> **Note:** if you don't have a server set up yet, you can use the shared community server: http://gunjs.herokuapp.com/gun

### Creating a target
Gun and Parse are similar when it comes to creating a database interface. Opening up a channel for Parse might look something like this:

```javascript
// open the "highscores" class
var Highscores = Parse.Object.extend('highscores')
var highscores = new Highscores()
```

Instead of extending "highscores", you'll retrieve the highscores object by one of its keys and modify it.
```javascript
// create a Gun instance
var gun = Gun(servers)

// get the node named "highscores"
var highscores = gun.get('highscores')
```

### Creating an object reference
The most comparable thing to Parse' classes is a gun "key", although the major difference is that you cannot subclass at this time, but you can globally extend gun instances by extending the `Gun.chain` (or `Gun.prototype`). This is being worked on for newer releases, and should be available soon.

**Parse**
```javascript
var Game = new Parse.Object.extend('game', {
	method: function () { /*...*/ }
})
var game = new Game();
```

**gun**
```javascript
var potato = Gun().get('game')
Gun.chain.method = function () { /*...*/ }
```

### Saving keys/values
Parse has the `.save` method, gun has the [`.put`](API-%28v0.3.x%29#put) method. Both work approximately the same, except `.put` can handle infinitely recursing objects.

**Parse**
```javascript
parse.save({
	field: value
})

var circular = {}
circular.circular = circular
parse.save(circular)
// Error: Too much recursion
```

**gun**
```javascript
// both happen immediately
gun.path('field').put(value)
// OR
gun.put({
	field: value
})

var circular = {}
circular.circular = circular
gun.put(circular)
// success
```

> **Note:** every value given to `.put` is treated as a [partial update](Partials-and-Circular-References-%28v0.2.x%29).

### Creating relation structures
Parse and gunDB have a very similar API in this manner. The easiest way to link data is by sending `gun.put` or `gun.set` a gun instance.

**Parse**
```javascript
// create two instances, parse1 and parse2
Parse.initialize(app, platform)
var Example = Parse.Object.extend('example')

var parse1 = new Example()
var parse2 = new Example()

parse1.set('parse2', parse2)
parse2.set('parse1', parse1)
```

**gun**
```javascript
var gun = new Gun('http://server.com/gun')
var gun1 = gun.get('gun1')
var gun2 = gun.get('gun2')

gun1.path('gun2').put(gun2);
gun2.path('gun1').put(gun1);
```

If you already have a reference to the object you're linking to, you can send it to `gun.put` and it works just the same.

```javascript
gun2.path('data').put(object)
```

Parse uses arrays to build one-to-many relationships, while gun has a native method called a `set`. They are essentially the same, except sets will not allow duplication.

Since Parse and gunDB use object pointers, both can support circular and many-to-one relationships.

### Removing data
Gun takes a different approach from most databases. Where Parse has `.destroy`, gun has `.put(null)`. This is an important distinction, since gun doesn't actually remove the node itself, but rather breaks the reference to it.

Think of it like Javascript garbage collection. If you have two objects, each referencing a third, and you delete one of the references, Javascript doesn't delete the object, just it's reference.

```javascript
// Create an object,
var third = {
	exists: true
}

// reference it from two others,
var first = {
	pointer: third
}
var second = {
	pointer: third
}

// and delete a reference from one of them.
// Javascript delete delete "third",
// just the reference to it. Same
// thing with gun.
delete first.pointer
```

This is the paradigm gun takes, removing "pointers" to other objects, not the objects themselves. However, unlike Javascript garbage collection, you aren't able to know how many strong references an object has before deleting the actual object, since other offline clients might still be using it.

If you want to learn more, gun has a [wiki page](Delete) on the topic.

### Queries
Instead of building another query language, gun lets you use the language you already know - Javascript.
Instead of doing an expensive database search every time you need data, we strongly encourage our users to index data on entry. It almost always improves performance drastically, and can have a strong impact on how your app is perceived. Since sometimes there's no way around performing a query, it's made fairly trivial to do it in vanilla javascript, just by using the API. Once again, we **strongly** recommend indexing the data as you run the query.

**example**
```javascript
// expensive query and indexing
// `.map` is essentially a `forEach`
function runQuery() {
	// potentially expensive query
	gun
		.get('peers').map()
		.path('friends').map()
		.path('birthday').val(function (birthday) {
			if (Date.parse(birthday) > value) this.key('query/' + name)
		});
}

gun.get('query/' + name)
	.not(runQuery) // only run if the result isn't found.
	.map(...) // ready to read!
```

### Fetching data
This is where gun really excels. Gun doesn't need a `.fetch` method, since the data has been updating in real time since you made the first request. While you can treat your data as a terminating value with `gun.val`, we invite our users to think of data as a stream of new information.

Respond real-time as data streams in.
```javascript
Gun().get('data').on(function (data) {
	// react to new data
})
```

Just give me the first thing you find, don't notify me about updates.
```javascript
Gun().get('data').val(function (data) {
	// just fired on the first response
})
```

> **Warning:** it can be tempting to use `.val`. Just keep in mind that one of your servers may not contain the full dataset.

### Data Migration
Gun can take almost any data that fits in the JSON format, with the main exception of arrays. For strongly ordered data, linked lists work well. Currently, we don't have any software for data migration. If you're looking to import existing data, let us know in [the gitter channel](https://gitter.im/amark/gun) and we'll help you get it imported.

--------------------------------------------------------

> Our team had little exposure to Parse before the shutdown notice. If we've glossed over important details, or misunderstood API details, please let us know!
