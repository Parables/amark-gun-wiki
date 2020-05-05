A hash is a short yet unique "fingerprint" of your data. It can be used as a name that is self-referencing to itself, frozen at some point in time. For instance, if the data were to change, then it would also get a new name.

This is useful, because it lets you verify that data has not been changed even if it is in a public place. And using [SEA](./SEA) with GUN, it prevent peers from changing the data if some name/key combo of `'#'+hash` is used.

 > Note: This does not guarantee peers will keep the data saved though, they might expire it over time.

Here is a quick example of how to get started:

```html
<script src="https://cdn.jsdelivr.net/npm/gun/gun.js"></script>
<script src="https://cdn.jsdelivr.net/npm/gun/sea.js"></script>
<script>
var gun = Gun();

var data = "hello world";
var hash = await SEA.work(data, null, null, {name: "SHA-256"});

gun.get('#').get(hash).put(data);

gun.get('#').get(hash).once(console.log);
</script>
```

Later, try to do something like:

```
gun.get('#').get(hash).put("hi");
```

And you'll see an error that the hashes are not the same.

You should be able to mix & match this, for instance, creating an immutable inbox:

```
gun.get('notfy@Alice/2020/1/1#').get(notificationHash)
```

This would let you send notifications to Alice that she could read that others cannot edit. If you do not want others to be able to read these notifications, you can encrypt them with a [key pair](SEA#quickstart).

 > Note: If you want it to be anonymous, you can generate a throw-away key pair.

Just remember, taking the hash is always the last operation. If you encrypt the message, you must take the hash of the encrypted message, not of the original message.

Need help? Discuss this further on [the chat](http://chat.gun.eco/) and/or please update this wiki with new insights!