Haven't read the [**crash course**](graphs) yet? Check it out first, then come back here.

**Table of Contents**
 - [Introduction](#introduction)
   - [Offline-First](#offline-first)
   - [Peer-to-Peer](#distributed)
 - [Hello World - Browser](#hello-world---browser)
 - [Hello World - Node.js](#hello-world---nodejs)
 - [Syncing Peers in Real-Time](#real-time-sync)
 - [Using Graphs](#graph-structures)
 - [Further Reading](#how-to)

## Introduction
GUN is a small, distributed data sync and storage solution that runs everywhere JavaScript does. GUN lets you focus on the data you need to store, retrieve and share without worrying about merge conflicts, network partitions, or synchronizing offline edits.

#### Offline-First
When a browser peer sends a request, it'll merge the responses with its own model using our [conflict resolution](https://github.com/amark/gun/wiki/Conflict-Resolution-with-Guns), then cache the result. Since it's cached on the client, there are a few interesting side effects:

 - The next time the client sends that request, the response is instantaneous, even when offline.
 - Data is replicated on each client that requests it.
 - If your server catastrophically fails, you can still recover your data from the clients.

This makes the loss of important information nearly impossible, as all copies of the data must be destroyed for it to be unrecoverable.

Servers act pretty much the same, but aren't as picky about what they cache.

#### Distributed
GUN is peer-to-peer (multi-master replicated), meaning updates don't need a centralized controller. You save data on one machine, and you can sync it with other peers without needing complex consensus systems. It just works.

However, you don't need peers or servers to use GUN, they're completely additive. Here's an example of GUN running in a browser without connecting to any peers...

### Hello World - Browser

> Follow along! Paste this code into an `index.html` file.

```html
<!DOCTYPE html>
<html>
	<head>

		<!-- Boilerplate HTML -->
		<meta charset="utf-8">
		<title>Hello World - GUN</title>
	</head>
	<body>

		<!-- Loads the GUN library -->
		<script src="https://cdn.rawgit.com/amark/gun/master/gun.js"></script>

		<script>

		// We're not connecting to any peers
		// just yet...
		var peers = [];
		var gun = Gun(peers);

		// Create an interface for the `greetings`
		// key, storing it in a variable.
		var greetings = gun.get('greetings');

		// Update the value on `greetings`.
		greetings.put({ hello: 'world' });

		// Read the value and listen for
		// any changes.
		greetings.on(function (data) {
			console.log('Update!', data)
		})
		</script>
	</body>
</html>
```

### Hello World - Node.js

> **Note:** If you don't have [node](http://nodejs.org/) or [npm](https://www.npmjs.com/) installed, [read this](https://docs.npmjs.com/getting-started/installing-node).

Before writing any node code, install the gun library:
```sh
$ npm install gun
```

> Plug this into a `hello.js` file.

```javascript
// Import the gun library
var Gun = require('gun');

// Create a new gun instance
var gun = Gun();

// Read `greetings`, saving it to a variable.
var greetings = gun.get('greetings');

// Update the value on `greetings`.
greetings.put({
	hello: 'world',
})

// Print the value!
greetings.on(function (update) {
	console.log('Update:', update)
})
```

Now we run the program. It should have the same result as the browser (albeit a bit more verbose):

```sh
# Tell node to run `hello.js`.
$ node ./hello.js
```

Now we've got two hello-world examples, one on Node and one on a browser. Why not sync them in real-time?

#### Real-Time Sync
Here's where the fun begins! We're gonna take the Node.js demo and sync it with the browser.

When you call `require('gun')`, some extra server-side plugins are loaded too, and one of those is `wsp.js`, a websocket transport plugin that falls back to ajax polling. Since it's already included, we can call `gun.wsp(server)`, passing it an `http.Server` instance.

Let's put this in our `hello.js` file, right at the top.
```javascript
// `http` is a standard library
// built into node.
var http = require('http');

// Create a new server instance.
var server = http.createServer();

// Our GUN setup from the last example.
var Gun = require('gun');
var gun = Gun({web: server});

// Start the server on port 8080.
server.listen(8080, function () {
  console.log('Server listening on http://localhost:8080/gun')
})

// ... The rest of `hello.js` ...
```

Sweet! Now gun is accepting websocket connections through `http://localhost:8080/gun`. Now we can connect to it from our browser and sync stuff.

Back in `index.html`, lets add that url to our list of peers, the array we passed to `Gun` constructor...

```javascript
// ... html setup above ...

// This points to the server we
// just started.
var peers = [
  'http://localhost:8080/gun',
]

// Set up gun, same as before.
var gun = Gun(peers);

// ... the rest of index.html ...
```

Make sure the server is running (`$ node ./hello.js`), then open up `index.html` in a browser window. Try this in your browser console:

```javascript
greetings.put({
  browser: 'Hello, Node!',
})
```

Check back to your server - since we had it `console.log` updates, you should see the data. But this isn't limited to just node... open up `index.html` in another browser. Since they're both syncing with the server, you'll see real-time updates on both browser windows.

#### Graph Structures
Throughout our examples, we've only used simple objects. But what happens when you need to represent richer data, like page hierarchy, social networks, or a machine learning model?

Graphs can represent any structure, no matter how complex or interconnected. In fact, that's how JavaScript objects work under the hood, which is why you can do weird infinite self-references without breaking your computer:

```javascript
var object = {};
object.self = object;

object.self.self.self.self; //.....
```

GUN uses graphs internally, so anything representable as a JavaScript object can also be saved into gun. Well, not quite everything. If you want more detail, you can [read more about graphs here](https://github.com/amark/gun/wiki/Graphs).

Let's try something a bit more fun. You'll need your browser open to `index.html`...

```javascript
// Update to a nested object.
greetings.put({
  hello: {
    browser: 'Hey everyone!'
  }
})
```

Now you should see something different this time. Instead of the usual object with `hello: 'world'`, it'll point to a weird-looking object like this:

```json
{ "#": "98VORPhZQhhUeNJaDdneuiGE" }
```

You don't have to worry about what that is, gun uses it to figure out how objects are connected. This is where we introduce a new method called `.path`, which is used to navigate an object's properties.

We'll use the [`path`](https://github.com/amark/gun/wiki/API-(v0.3.x)#path) method to read the value on `hello`.

[Warning: Not included by default! You must include it yourself via require('gun/lib/path.js') or <script src="/gun/lib/path.js"></script>!](https://gun.eco/docs/path.html)

```javascript
// Get the property "hello" on greetings.
var hello = greetings.path('hello');

// Listen for data on `greetings.hello`
hello.on(function (value) {
  console.log('Update:', value)
})
```

> If you don't want to listen for updates, you can use `.val()` instead, which works the same as `.on()` but is only called once.

Let's go crazy and put a circular reference in there, just for fun. We'll link the `hello` object to itself.

```javascript
// Write the `hello` reference to the
// property "self".
hello.path('self').put(hello);
```

To drive the point home, let's print out the circular reference:

```javascript
var self = hello.path('self.self.self.self');

// Print the value on `self`
self.on(function (update) {
  console.log('Self:', update)
})
```

It should log the `hello` object.

Now let's think about why you'd ever want to do that. Self references aren't something you generally want... right? Not so, let's give some concrete examples!

##### Social Features
Bob is a new user on MyFace&trade;, and his profile looks a bit like this:

```javascript
var bob = {
  name: 'Bob Carlson IV',
  friends: {},
}
```

He's just sent a friend request to Alice, also a new user.

```javascript
var alice = {
  name: 'Alice Davison',
  friends: {},
}
```

Once Alice accepts, they add each other to their friends list, and you've got a circular reference going, since they both reference each other. Check this out:

```javascript
// This could go on forever.....
var alice = bob.friends.alice.friends.bob.friends.alice;
```

###### In GUN

GUN can do this easily. You can try it out in your browser console!

```javascript
// Get a reference to Bob's profile.
var bob = gun.get('bob');

// Update Bob's information
bob.put({
  name: 'Bob Carlson IV',
  friends: {},
})

// Same thing for Alice
var alice = gun.get('alice');
alice.put({
  name: 'Alice Davison',
  friends: {},
})

// New method `.set()`: Adds the item to
// a list. Similar to a mathematical "set",
// where each member is unique.
alice.path('friends').set(bob);

// Same thing for bob.
bob.path('friends').set(alice);

// Print out Bob's name. Very inefficiently.
bob
  .path('friends.alice.friends.bob.name')
  .on(function (name) {
    console.log("Bob's name:", name)
  })
```

##### Author/Books
You're an online bookstore keeping records of authors and the books they write. A book in the database might look like this:

```javascript
var book = {
  title: 'The Martian',
  published: 1391238000000,
  author: { '#': 'authors/Andy_Weir' }
}
```

And each author might look something like this...

```javascript
var weir = {
  name: 'Andy Weir',
  books: {
    'The Martian': { '#': 'books/The_Martian' }
  }
}
```

Here you are again, circular reference.

```javascript
var weir = book.author.books.The_Martian.author
```

###### In GUN
Here's how you'd do it with gun:

```javascript
// Get a reference to the book.
var book = gun.get('books/The_Martian');

// Set it's information.
book.put({
  title: 'The Martian',
  published: 1391238000000
})

// Get a reference to the author.
var weir = gun.get('authors/Andy_Weir');

// Add some author information.
weir.put({
  name: 'Andy Weir',
  books: {}
})

// Set the author to point to Andy Weir.
book.path('author').put(weir);

// Add the book as a unique member of
// Weir's "books" list.
weir.path('books').set(book);

// Print the author's name!
book.path('author.name').on(function (name) {
  console.log('Author:', name)
})
```

-----

Wrapping things up, graphs can do some pretty awesome stuff, and are powerful for beginners and masters alike.

Hopefully that gives you a good place to start, and an idea of how gun works. Next, you might want to check these out:

 - [Our API reference sheet](https://github.com/amark/gun/wiki/API-%28v0.3.x%29)
 - [Our Gitter channel (say hi!!)](https://gitter.im/amark/gun/)
 - [The gun starter app](https://github.com/gundb/gun-starter-app)

`TODO: Page needs to be redone. Should be newbie simple, this is too elaborate.`