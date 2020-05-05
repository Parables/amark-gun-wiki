[THE CRASH COURSE PAGE HAS CHANGED LOCATIONS, CLICK HERE!](Crash-Course)

(This page will be used for explaining data structures in the future).

This article will go over how it is easy to create interconnected data with GUN's graph features, combining key/value, relational, and document based data together. It will also be a great introductory guide on how to use pretty much every one of GUN's API methods.

> Note: This article requires gun v0.8.x or above.

First let's instantiate the database.

```javascript
var gun = Gun();
```

Then we'll add some people to our database using a simple key/value approach.

```javascript
var alice = gun.get('person/alice').put({name: 'alice', age: 22});
var bob = gun.get('person/bob').put({name: 'bob', age: 24});
var carl = gun.get('person/carl').put({name: 'carl', age: 16});
var dave = gun.get('person/dave').put({name: 'dave', age: 42});
```

> Note: If no data is found on the key ('person/alice', etc.) when we [`.get`](API#get) it, gun will implicitly create and update it upon a [`.put`](API#put). This is useful and convenient, but can be problematic for some types of apps. If you want to check if the data does not exist, use [`.not`](API#not) first.

What if we want to get their data? We can either chain off of the reference directly or get it again:

```javascript
alice.on(function(node){
  console.log('Subscribed to Alice!', node);
});

gun.get('person/bob').val(function(node){
  console.log('Bob!', node);
});
```

> Note: GUN is a functional reactive database for streaming event driven data, gotta hate buzzwords - right? This means that [`.on`](API#on) subscribes to realtime updates, and may get called many times. Meanwhile [`.val`](API#val) grabs the data once, which is useful for procedural operations. 

Now lets add all the people into a [set](https://en.wikipedia.org/wiki/Set_(mathematics)), you can think of this as a table in relational databases or a collection in NoSQL databases.

```javascript
var people = gun.get('people');
people.set(alice);
people.set(bob);
people.set(carl);
people.set(dave);
```

> Note: [`.get`](API#get) and [`.put`](API#put) are the core API of gun, everything else is just a convenient utility that wraps around them for specific uses - like [`.set`](API#set) for inserting records.

It is now easy to iterate through our list of people.

```javascript
people.map().val(function(person){
  console.log("The person is", person);
});
```

> Note: If [`.map`](API#map) is given no callback, it simply iterates over each item in the list "as is" - thus acting like a for each loop in javascript. Also, everything is continuously evaluating in GUN, including [`.map`](API#map), so it will get called when new items are added as well as when an item is updated. It does not iterate through the whole list again every time, just the changes. This is great for realtime applications.

Next we want to add a startup that some of these people work at. Document oriented data does a perfect job at capturing the hierarchy of a company. Let's do that with the following:

```javascript
var company = gun.get('startup/hype').put({
  name: "hype",
  profitable: false,
  address: {
    street: "123 Hipster Lane",
    city: "San Francisco",
    state: "CA",
    country: "USA"
  }
});
```

Now let's read it out!

```javascript
company.val(function(startup){
  console.log("The startup:", startup);
});
```

> Note: The data given in the callback is only 1 layer deep to keep things fast. What you'll see logged out on `startup.address` is not the address itself, but a pointer to the address. Because documents can be of any depth, GUN only streams out what you need by default, thus optimizing bandwidth.

So what if you want to actually access the city property on the company's address then? [`.get`](API#path) also lets you traverse into the key/value pairs on sub-objects. Take this for example:

```javascript
company.get('address').get('city').val(function(value, key){
  console.log("What is the city?", value);
});
```

Good news! We just found out the company got funding and moved to a new office! Let's go ahead and update it.

```javascript
gun.get('startup/hype').put({ // or you could do `company.put({` instead.
  funded: true,
  address: {
    street: "999 Expensive Boulevard"
  }
});
```

> Note: GUN saves everything as a partial update, so you do not have to re-save the entire object every time (in fact, this should be avoided)! It automatically merges the updates for you by doing conflict resolution on the data. This lets us update only the pieces we want without worrying about overwriting the whole document.

However documents in isolation are not very useful. Let's connect things and turn everything into a graph!

```javascript
var employees = company.get('employees');
employees.set(dave);
employees.set(alice);
employees.set(bob);

alice.get('spouse').put(bob);
bob.get('spouse').put(alice);

alice.get('spouse').get('employer').put(company);
alice.get('employer').put(company);

dave.get('kids').set(carl);
carl.get('dad').put(dave);

carl.get('friends').set(alice);
carl.get('friends').set(bob);
```

> Note: We can have 1-1, 1-N, N-N relationships. By default every relationship is a "directed" graph (it only goes in one direction), so if you want bi-directional relationships you must explicitly save the data as being so (like with Dave and his kid, Carl). If you want to have meta information about the relationship, simply create an "edge" node that both properties point to instead. Many graph databases do this by default, but because not all data structures require it, gun leaves it to you to specify.

Finally, let's read some data out. Starting with getting a key/value, then navigating into a document, then mapping over a table, then traversing into one of the columns and printing out all the values!

```javascript
gun.get('person/alice').get('spouse').get('employer').get('employees').map().get('name').val(function(data, key){
  console.log("The employee's", key, data);
});
```

Awesome, now run it all together: http://jsbin.com/webikepoxa/edit?js,console (Hit "Run", all logs except for the last one have been commented out).

GUN is that easy! And it all syncs in realtime across devices! Imagine what you can build?

 - Be it [distributed machine learning with gun](https://github.com/cstefanache/cstefanache.github.io/blob/master/_posts/2016-08-02-gun-db-artificial-knowledge-sharing.md#gundb)!

 - Or IoT apps with timeseries analysis and [live temperature data visualizations](https://github.com/Stefdv/gun-ui-lcd#syncing).

 - How about multiplayer VR experiences, or [React apps](https://github.com/PsychoLlama/connect-four) like [online games](https://github.com/PsychoLlama/Trace)? Or [realtime GPS tracking](https://youtu.be/7ALHtbC9aOM) for autonomous drones that deliver burritos or Uber/Lyft like apps. 

 - Maybe you just want to create a social networking app, [getting started with a basic server](https://github.com/gundb/gun-starter-app) and using Angular or Webcomponents/Polymer instead. Better learn about P2P logins, security, and authentication with our [1 minute video explainers crash course on encryption](https://gun.eco/explainers/data/security.html)!

Whatever it is ([except for banking](CAP-Theorem)), we hope you are **excited and tackle it with gun**! Make sure you join the [chat room](https://gitter.im/amark/gun) (everybody is nice and helpful there) and ask questions or complain about bugs/problems. Or if you think your company might be interested in using gun, we have some great [partnership plans](https://www.patreon.com/gunDB) (as well as some donation options, if you want to personally contribute some yummy meals to our tummy)!
