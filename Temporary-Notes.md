we have a base event system. Over time, it can receive many events.

the observable is built on top of this. It treats all events as mutations to the same thing. It then caches the most recent updates.

To make it more intelligent for gun, it treats all events by the same `gun` to be the same, and iterates over all of them as a plural chain.

When we chain after a plural, each chain caches the last known values.

gun		.get()		.map()		.path()		.on()
								|					|
								 - alice	 - path()
								|					|
								 - bob		 - path()

This makes removing a parent item in the chain hard.

The change is happening to the `.get()`. It has a responsibility to emit `{alice: newAlice}`.

When that happens, the get().path() inside of map is also called, aka Alice.

Alice has no `next` because they are all attached to the `map().path()`.

====================

var both = gun.get('alice').and('bob');
var pets = both.path('pet').on(cb);

both = [alice, bob]
pets = [asdf, bnmv]

====================

gun.get('alice') // unique
gun.get('alice').path('age') // unique
gun.get('alice').path('pet') -> gun.get('asdf') // proxy
gun.get('users').path('age') -> [alice.path('age'), bob.path('age')] // proxy
gun.get('users').path('pet') [] // proxy
gun.get('alice').and('bob') -> [gun.get('alice'), gun.get('bob')]


====================

What do we do, when we have a chain that receives souls? It is easy to incrementally add each new soul as it comes in, but what happens further down the chain based on the parents? Let us try...

(both = gun.get('alice').and('bob')).path('pet').on(cb)

We should get realtime live updates of the pets - not just any pets though, but the ones specifically determined by the parent chain. If for instance, a pet dies, that pet which was cached in the `path` chain should get nulled out.

First the parent `both` will receive two events:

`a = {get: 'alice', put: {pet: {#asdf}}, gun: alice, via: via}`
`b = {get: 'bob', put: {pet: {#bfds}}, gun: bob, via: via}`

When these are received, the `both` chain will iterate over the changes to see if any children chains need to be notified.

This will trigger `path('pet')` with the following items:

`{get: 'pet', put: {#asdf}, gun: alice.path('pet'), via: a}`
`{get: 'pet', put: {#bfds}, gun: bob.path('pet'), via: b}`

Now `alice.path('pet')` and `bob.path('pet')` are added to the observable cache. Why? Because they are fundamentally unique. However since their values are pointers it will cause them to proxy a root node. In order to prevent duplicate references being added to the cache, the unique `path`s should proxy forward their self as well as their field.


========================================

gun.get('users').map().path('pet').on(cb);

First, we add an observable to the map. Because there is no previously cached data, nothing triggers and we fire a request for data up the chain. This is emit out to storage drivers. A storage driver replies with

`g = {put: {
	users: {_:{#users}
		alice: {#asdf}
		bob: {#bfds}
	}
}}`

Which gives input to the `map()` as

`m = {get: 'users', put: {alice: {#asdf}, bob: {#bfds}}, gun: get('users'), via: g}`

To which `map` iterates over the `put` and emits on its chain

`a = {get: 'alice', put: {#asdf}, gun: users.path('alice'), via: m}`
`b = {get: 'bob', put: {#bfds}, gun: users.path('bob'), via: m}`

which gets cached in the observable.

Then `path('pet')` is read, adding an observable to the chain. Because there is no previously cached data, nothing triggers and we fire a request for data up the chain. This emits out to the parent which replies with cached data.

That cached data is `a, b` on the parent. This means we need to do a lookup on `#asdf.pet` and `#bfds.pet` which gets gets emit out to the storage drivers. They reply with:


`g = {put: {
	#asdf: {_:{#asdf}
		pet: {#pasdf}
	}
}}`

First `get('asdf')` gets called, which proxies to `users.path('alice')`.

What SHOULD be emitted to the `path('pet')` is as follows:


`{get: 'pet', put: {#pasdf}, gun: alice.path('pet'), via: a}`
`{get: 'pet', put: {#pbfds}, gun: bob.path('bob'), via: m}`

===============================


gun		.get()		.map()		.path()			.path()		.on()
								|					|
								 - alice	 - path()
								|					|
								 - bob		 - path()


===============================
