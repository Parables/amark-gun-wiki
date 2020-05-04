This article is being used by Mark as an outline for how the chaining system should work.

`var gun = Gun()`

this creates a root level chain with a root level event emitter.

`var ref = gun.get('data')`

this returns a new chain with its own event emitter.

`ref.on(function(val){})`

this returns the same chain as before, and it subscribes to that chain's event emitter. As those events get called it uses the information to subscribe to the root level emitter's events on particular pieces of data, which it then calls the callback.

`var sub = ref.path('field')`

this returns a new chain and emitter, but listen's to its parent's emitter. It uses the event data from the parent to then call the child emitter.

`sub.on(function(val, field){})`

this returns the same chain as before, and uses its emitter to listen for information that it then uses to subscribe to the root level emitter's events on particular pieces of data, which it then calls the callback.

Path should be treated as `soul.field` always, even if field is pointing to a sub node. Path will reference the sub node for you, but it should not become attached to the sub node. For instance, if the field was on a subnode, we should get all updates that happen on it, but if the field changes to a value, we should cease to get updates on the subnode.