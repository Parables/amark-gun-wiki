The javascript implementation of GUN is built on layers of functional utility libraries.

## Type

The first and most important one is a Type library that normalizes javascript's behavior across different browsers and environments. This is unfortunately necessary due to javascript's language quirks. GUN's constructor directly inherits from this library.

## Event

The second is an Event library that lets multiple listeners attach themselves to some event emitter. Because GUN uses a [functional reactive programming](./Functional-Reactive-Programming) coding style, almost everything is built on top of this. It lets the API be an observable on streams of data.

## State

Every stream of data is run through a [conflict resolution algorithm](Conflict-Resolution-with-Guns). This lets GUN correctly synchronize state regardless of concurrent updates to the data. So while the Event library lets us observe changes to data over time, the State system lets us always resolve data to its most recent value. Kind of like a promise, since some data may be loaded asynchronously.

## Graphs

Not all data can be expressed as a document or as a table. But graphs can express all data, which is why GUN uses them. This is particularly important for handling conflicts on sub objects. Every object in GUN has its own universally unique identifier, requiring no dependency upon its parent object. These nodes can only have properties with simple values or a pointer to another node. So there is a utility library for handling values, states, nodes, and graphs.

## Deduplication

Because events might be sent multiple times, it is useful to deduplicate them. This means every event needs to have an unique identifier to deduplicate against. If each event has an identifier, we can also use that to create request/response acknowledgements, like asking for data and getting a reply or sending data and getting a confirmation of receipt.

# Implementation

The hardest part about GUN so far has been the chaining javascript API - the actual database implementation was easy. Below is an incomplete review of how it was built, to aide in further development of the system.

## Root

In order to do anything with GUN, you need to create an instance of the database with `var gun = new Gun()`. This is a terrible idea though because it makes building adapters for GUN significantly harder. Why? Instances are in a private scope that may not be available to the adapters' scope (like a websocket server). As a result, you have to tightly couple your interface code and/or you have to add extra closures (which effects performance) to pass database instances around or adapter contexts. This is all done for the sake of being able to have multiple database instances (which I don't see the point of for a single thread), but assuming that is a good thing, let us continue.

### Cache

Every database instance has its own root context. This root context holds the most important pieces that manage the input and output of data. The most obvious piece is the graph itself, an in-memory cache of the data. For a single instance, the root graph acts as the source of truth, and therefore must be pure - always valid, wire spec compliant, serializable, have no annotations, and so on. As a result, a developer using the database should not modify the graph directly, which is why an API is needed.

### IO

GUN's API is a wrapper around its wire spec, so the root instance listens for `gun.on('in', emit)` and `gun.on('out', emit)` events. Both listeners are handled by the same logic, which is:

1. If an event does not have an identifier, add one to it.
2. Check to see if we've already seen this identifier, if we have then stop.
3. If we haven't, then add the identifier as now having been seen.
4. If it is a GET request, emit a get event with it and the root instance.
5. If it is a PUT request, emit a put event with it and the root instance.
6. Emit an out event with it and the root instance.

### PUT

Let us assume no data currently exists, and we are the only ones to create it. We generate an event and emit it out to the root:

```
gun._.on('out', {
	put: {
		ASDF: {_:{'#':'ASDF','>':{...}}
			hello: "world!"
		}
	}
})
```

It runs through the root and is emitted as a put event. The default listener should do the following:

1. Make sure the data is a valid graph.
2. Calculate the diff from the current graph using the conflict resolution rule.
3. Merge the diff into the graph for whatever data this peer is tracking.
4. Emit the diff down the chain.
5. Resume the put event.

### GET

Now we can GET the data we just saved by generating an event that follows the wire spec:

```
gun._.on('out', {
	get: {'#':'ASDF'}
	'#': gun._.ask(function(msg){

	})
})
```

`ask` creates a message ID and uses the event system to subscribe to it. This way, any messages that acknowledge that ID behave as a response to the request. The default `get` listener should do the following:

1. Check that it is a valid lexical cursor, if not then resume the get event. (That way other listeners can handle non-standard requests.)
2. If there is no lexical match in the graph, resume the event. (So other listeners, like storage adapters, can process it.)
3. If there is a match, acknowledge the `get` request with a `put` message that has a graph of only the matching data.
4. If this peer is not tracking that data, or if this is the first time this peer has seen matching data, or if there could be more matching data, then resume the event.

> A note for storage adapters, same thing applies there as well. Since storage is usually an asynchronous operation, it is smart to cache writes as they happen. But until a read has merged against the graph, there is no guarantee that the pending write is the correct read.

The responses we get may be eventually consistent, duplicates across different peers, partial, or out of order.

## Chain

Speaking the wire spec directly to the root is not a usable API. Instead, it would be more useful to have an API that supports these features:

1. Key/value lookup from the graph to a node.
2. Document traversal. This is the same as a key/value lookup but lets us access nested values and nodes. The simplest API for this would be to chain key/value lookups.
3. Table scan. Not every property on a node may be known, so it would be useful to dynamically get each sub property easily. Technically this is no differen from (1), because (1) should load the full node which would include all of its properties. The only difference is that the chaining API has to be expressive enough to handle asynchronous lookups on data.
4. Conditionally load items from a table. This should be possible using regular old javascript statements as the conditional expressions. If the conditions were not javascript, then this would no longer be the javascript implementation.

Therefore the chaining API needs to be powerful enough to support at least all of these features. We will focus on the most difficult aspects.

### Performance

We learned this lesson when people built data visualization tools and games on top of gun. People simply will not use gun if it has any overhead, either in lag or difficulty. This means we have to learn how to push the limits of javascript, so if any lag does happen it is the fault of physical or hardware resource limits, not gun.

### Asynchronous

However, achieving performance while having a potentially asynchronous API is non-trivial. I really need to stress this, nothing about the database side of the equation was hard or difficult. Creating an easy API in javascript that is performant and asynchronous has been hard. So here we go:

 - Because any API use might be asynchronous, everything needs to be asynchronous by default.
 - However, to keep things fast, all results should be cached to return immediately and synchronously when possible.
 - If we do not have cached results yet, multiple chains should reuse the same asynchronous action by queuing.
 - At any point the data may change, which may cancel or cause everything to dynamically restructure.

If it was not already self evident, each one of these requirements contradicts one or more of the other requirements. Which is why building the system has been so difficult.

### Plural

One of the easiest edge cases to note is that a chain might just be a chain of many other chains. So you have to be able to differentiate between a chain of chains and just a chain, dynamically without any advanced knowledge and subject to change.

One might conjecture that we force everything to be a plural chain, that way there is no edge cases. However, when I tried to do this nothing would actually resolve to any 1 thing becuase it kept on thinking was resolving to many things. Either this is my own stupidity, or at some level the computer has to be told to do this stupidly simple thing.

What this means:

1. When nothing has been resolved yet, we do not know whether this is a plural chain or not. Therefore, as things are resolved, check and if so, resolve each singleton.
2. If some things have already been resolved, immediately use the cached singletons.

However, it is impossible to differentiate between these two things:

1. We have an existing singleton, and we receive a different singleton. Therefore, change and replace that singleton.
2. We have an existing singleton, and we receive a different singleton. Therefore, this is now a plural chain, therefore add it along side the current singleton.

As a result, there is a couple things we can do.

1. Specify which chains are a singleton.
2. Have proxy chains which reference singletons.

Here are a couple of examples:

A) We know that a key/value lookup from the root must be a singleton, like `gun.get('mark')`.
B) We know that a document traversal from the root is a singleton, whatever depth it is at it must proxy some root node (or a property on a node).

Therefore, anything that is not A and not B, might be a plural chain. But anything that is A or B cannot be a plural chain.