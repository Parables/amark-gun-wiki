Partials are a really important concept for more than just their convenience. There is actually a lot of math and concurrency benefits to them, but we will not explore that here. Instead, we will go over how it practically impacts your life and makes things easier.

To illustrate, take the example:

```javascript
var mark = gun.set({
  username: "marknadal",
  name: "Mark Nadal",
  email: "mark@gunDB.io"
});
```

In most databases, if you were to then do:

```javascript
mark.set({
  hacker: true,
  country: "USA"
});
```

It would completely wipe out all your data previously there and replace it with that new object. This is fundamentally bad beyond just it not being the behavior you intended, it can also cause race conditions. Which is why most other databases are centralized around one master.

But we are not like that, not only because we want a bunch of your users to simultaneously interact with data, but also because that is the pattern you _expect_. No need to constantly do some `.update({find: 'foobar'}, {$set: {just: "my stinking location"}})` command.

Additionally, if you did want such behavior, it is much more explicit to declare `.update({find: 'foobar'}, null).update({find: 'foobar'}, {some: "new object"})`. As a result, gun accepts partial updates by default and preserves your data unless you explicitly `null` it out.

Partials let you work with your data without having to redownload the whole snapshot every time. Which is a common work around for dealing with cache invalidation in other databases. Even still, it is a poor one because it does not properly control for the concurrency during the latency of the expensive round trip of the data.

**In conclusion**, _partials rock_. Enjoy the freedom and flexibility of using them.

## Circular References

Similarly, you might be familiar with doing something like:

```javascript
var mark = {
  name: "Mark Nadal",
  gender: "male"
};
var amber = {
  name: "Amber Nadal",
  gender: "female",
  husband: mark
};
mark.wife = amber;
```

And then somewhere down the road you attempt to `JSON.stringify(mark)` and you get some nasty circular reference error. This is also silly, if the algorithm is smart enough to catch infinite recursions then it is trivial to serialize it properly. GUN does this for you, using the power of graphs. It is nothing fancy, it is the same thing that the web does - two webpages can link to each other.

Unfortunately, JSON was designed with a documented oriented structure in mind, otherwise known as a tree. This was also a fatal flaw of many NoSQL databases. Working with JSON is great for developers, since our minds easily grasp the behavior of documents. But storing data as a tree creates a nightmare of usability if and when the developer wants to do anything remotely more advanced.

GUN makes this easy, all you have to do is use that `mark` object you created earlier:

```javascript
gun.set(mark).key('user/marknadal');
```

This will save both `mark` and `amber`. However it does not create an index for Amber, so we should do that also:

```javascript
gun.load('user/marknadal').path('wife').key('user/ambernadal');
```

This lets us do some crazy fun inception things like:

```javascript
gun.load('user/ambernadal')
  .path('husband.wife.husband.wife.name')
  .get(function(name){
    console.log(name); // "Amber Nadal"
});
```

**In conclusion**, _graphs rock_. Enjoy the using both relational and documented structured data.