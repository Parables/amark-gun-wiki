<div style="position: relative; padding-bottom: 56.25%;"><iframe src="https://www.youtube.com/embed/5fCPRY-9hkc" frameborder="0" allowfullscreen style="border: 0px; position: absolute; width: 100%; height: 100%;"></iframe></div>

 > Code from the above tech talk is [available here](https://github.com/gundb/port).

 - @ChronosX88's Java 
 - @chernousov-m's C++
 - [@xmonader's Python](https://github.com/xmonader/pygundb)
 - @LaPingvino's Go
 - [@go1dfish's JS](https://github.com/notabugio/gun-receiver)
 - @kellytk's Rust
 - @masterex1000 's (?)

We'll walk you through creating a reduced GUN server in your language of choice. It will not be useful as a library to other developers, but it will help you understand what is going on and communicate with GUN peers. You should be able to then expand upon it, adding a nice language specific API so it is usable by other developers.

There are three main categories we'll need to implement for our minimal GUN server.

1. Running a server that speaks GUN's wire spec.
2. Understanding and implementing GUN's graph structure.
3. Processing data through GUN's conflict resolution algorithm.

That's it! Our target implementation will be in-memory, WebSockets, JSON, and HAM.

## Running a Server

Your first objective is to get a WebSocket server up in your language which sends "hello world!" once every second to a browser which connects to it. Might as well append the count to the message as well. This tests to see if your language can do asynchronous and event driven logic. If it can't, then it will be pretty difficult to implement GUN. Hopefully your language of choice also has a WebSocket server module that somebody has written for you - if not, you can get your hands dirty and build it for everybody else or emulate WebSockets with JSONP over HTTP which we won't cover here.

Here is the solution implemented in NodeJS:

> Note: Code samples throughout this entire article should be considered as more pseudo code than anything else. There is no guarantee that any of them are perfectly working.

```javascript
var WebSocketServer = require('ws').Server
  , wss = new WebSocketServer({ port: 8080 });

wss.on('connection', function connection(ws) {
  ws.on('message', function incoming(message) {
    console.log('received:', message);
  });
  var count = 0;
  setInterval(function(){
    count += 1;
    ws.send('hello world ' + count);
  }, 1000);
});
```

And here is the code that you can paste into the browser to see if it and your WebSocket server works.

```javascript
// paste this into your browser console to test your WebSocket server!
var ws = new WebSocket('ws://localhost:8080'); // change this address to your server, if different!
ws.onopen = function(o){ console.log('open', o) };
ws.onclose = function(c){ console.log('close', c) };
ws.onmessage = function(m){ console.log('message', m.data) };
ws.onerror = function(e){ console.log('error', e) };
```

You might need to paste it into a browser tab that isn't on HTTPS or blocking cross origin. If you have problems, just wrap it into an HTML file that you save to your computer and then open it in the browser.

Before moving on and upgrading our WebSocket server into a GUN server, we are going to take a detour into explaining graphs with JSON.

## Graph

The fundamental problem is that JSON does not allow for circular references, yet a graph might be full of them.

```javascript
// paste this into your browser console to see JSON explode.
var object = {};
object.self = object;
JSON.stringify(object); // Error!
```

It turns out that JSON is actually a really bad choice for GUN, however it does have really good compatibility support across languages which makes it a convenient interchange format. Therefore we'll roll with it for these examples and it should work just fine.

So how do we get around JSON's problems? Well, quite simply we wrap everything into a root level JSON object, then give every object at every level a UUID, pull them out of their nested hierarchy into the root JSON object under their UUID, and then backfill the hierarchy with UUID pointers. Was that too much text? Here is a data example:

```javascript
var mark = {
  name: "Mark Nadal"
}
var cat = {
  name: "Fluffy",
  species: "kitty" // for science!
}
cat.slave = mark;
mark.boss = cat;
```

As you see, we have a fairly simple JavaScript object but it can't be serialized to JSON. If we normalize it into a graph then it won't be a problem:

```javascript
var graph = {
  mark: {
    name: "Mark Nadal",
    boss: {'#': 'cat'}
  },
  cat: {
    name: "Fluffy",
    species: "kitty",
    slave: {'#': 'mark'}
  }
}
JSON.stringify(graph);
```

Here I have decided to use `{'#': "UUID"}` as the JSON format for a pointer. However, `'mark'` and `'cat'` are hardly *universally unique* - so let's fix that. If we are going to do this though, it would be wise for us to actually contain the UUID on the object itself so that way it can know its own UUID in case it gets isolated. Let's update the graph to the following:

> Note: For human clarity, we are going to fake the UUID with something readable. In practice, you should use a real UUID.

```javascript
var graph = {
  ASDF: {_: {'#': 'ASDF'},
    name: "Mark Nadal",
    boss: {'#': 'FDSA'}
  },
  FDSA: {_: {'#': 'FDSA'},
    name: "Fluffy",
    species: "kitty",
    slave: {'#': 'ASDF'}
  }
}
JSON.stringify(graph);
```

> Note: This creates a lovely structure that is always a UUID, field, and value! This winds up being very important for later sections.

There we go, now we have a nice graph. We added the `_` property to each object to contain metadata, like its own UUID. It is JSON safe, which means we can serialize it between JavaScript and your WebSocket server! Let's go back to that.

 > 

## GUN Wire Spec

 > Please read [the wire spec](./Wire). Below is an old summary.

Now that you know how we are going to transfer data between languages, we can upgrade the WebSocket server to speak GUN. There is only two wire commands that GUN supports - write and read. That's it, let's go over those in a little bit more detail:

 - `PUT` is any write to the data. It is always pushed over the network as a graph, although different serializations than JSON might be used. There is no CRUD! All CUD are treated as the same, an update. The graph that is sent only contains the part

 - `GET` is any read on the data. It is always pushed over the network as a lexical cursor, this is intended to allow for even low memory devices to process large datasets. A lexical cursor is broken down into these parts, and could be serialized differently than JSON:
   - `#` UUID lexical match. For purposes here we will only handle an exact lexical match, we will not be going over the other matching conditions.
   - `.` Field lexical match. We will not be covering this at all in this article.
   - `=` Value lexical match. Same as above.
   - `>` State lexical match. What are states? You'll find out soon enough.

> Note: As of the recent release of GUN v0.3.3, `GET` wire commands are not formatted properly. The reason why, as alluded above, is because only exact UUID matches are implemented. Thus the only thing that gets sent is the UUID whether it be a lexical cursor or not. We are fixing this.

Both `PUT` and `GET` expect acknowledgements, as you might guess. But this is where things get interesting. GUN is fully peer-to-peer, meaning regardless of whether you are running a centralized GUN server or not you have to comply by the decentralized architecture. What this means is that every wire command you receive, you also forward it along to everybody else as well.

... TO BE CONTINUED ...

## Conflict Resolution - THIS SECTION IS NOT FINISHED AND WILL PROBABLY BE ENTIRELY REWRITTEN

Any GUN library in any language must first start with the Hypothetical Amnesia Machine, which is is the conflict resolution algorithm. Also check out the wiki page on [Conflict Resolution with Guns](Conflict-Resolution-with-Guns).

Parameters:

 * `machineState`: The current time of the local machine.
 * `incomingState`: The time of the update from the remote machine.
 * `currentState`: The time of the last update on the local machine.
 * `incomingValue`: The data coming from the remote machine.
 * `currentValue`: The data stored in the local machine.

[Check out the super simple source code in 50 lines of code](https://github.com/amark/gun/blob/master/src/HAM.js)

If you can implement this in your language of choice, then congratulations you now have a GUN driver. Everything else about GUN is pretty much just an API surface that wraps around this, providing convenience functions for the end developer. So first thing first, write this in your language of choice.

To recap, all data in GUN gets boiled down to these 5 parameters such that they can be compared and converged on, given the HAM results. It is an incredibly simple algorithm but it is universally detailed in how to handle data synchronization.

There are several edge cases for different types of business use cases, so please read the [Conflict Resolution](https://gun.eco/distributed/matters.html) cartoon.

 ## Work in Progress

... more coming soon ...