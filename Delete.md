Gun takes a different approach to deletion than most other centralized services. Since other (potentially offline) peers might still have a copy of the node, removing your own copy is only temporary. As soon as that peer comes back online, the data will resynchronize and the node is back again. While this is pretty cool from the perspective of data recovery, it might be annoying if you're trying to get rid of something. This can be solved by replacing the references to the node with a different value, like a primitive or a pointer to different node.

```javascript
var gun = Gun().get('data').put({
  object1: object2,
  object2: object2
})

// set 'object3' to a new node
gun.path('object3').put({
  field: 'value'
})

// replace the value at 'object3' with `null`
gun.path('object3').put(null)

// instead of pointing to an object, now it shows 'null'
```

As soon as the value is set, in this case `null`, the update propagates to each peer, breaking the reference to the old node. If storage space is a concern, a garbage collection module could be built on top of gun that removes data when it's sat on the shelf too long. At that point you might want to consider migrating to a hosted solution such as AWS, which has ridiculously cheap rates and oodles of storage space.

Another way of thinking about deletion is that you're not removing the data, but rather it's *discoverability*. It would be like Google removing a result from it's search. You can still enter the URL, but the website is effectively invisible.

If you have any more questions about removing data from gun, ping us on [Gitter](https://gitter.im/amark/gun) and we'll do our best to help out.

## Options

Check out `lib/memdisk`, `lib/erase`, `lib/forget` (**Call for Help**: We need documentation for these modules!) are all adapters for deleting/discarding old or unused data. Depending on what your application needs are, one of them should work.

## Why

Deleting data in a distributed system is actually very hard. If there 9 peers, and 5 peers online send null to each other and each of those 5 peers then delete the information, when the remaining 4 peers come online, the 5 peers either must send null (which requires they keep the null to indicate the delete) or those 5 peers actually deleted the data (and therefore have no knowledge of it) and when the 4 peers come back online they'll re-sync the data, and the 5 peers will have the data again.

The best solution is to either keep the null (which is what GUN does by default) or:

Use a case-specific delete behavior, like the modules mentioned above.

Or use another CRDT on top of GUN, that deletes all keys in a node below a certain vector. (**Call for Help**: We're looking for somebody to contribute this module!)

For further explanation, a great conversation on why this is the case is talked about here:

https://gitter.im/amark/gun?at=5ac4f513c574b1aa3e67f6ea

We recommend you read it, as you'll rediscover why deletes must use a tombstone to work properly in a distributed system.