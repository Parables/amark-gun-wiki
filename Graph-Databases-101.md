This article will go over how it is easy to create interconnected data with GUN's graph features. Graphs allow for a mix of many different data structures, and we will use all of them in this overview.

> Note: This article requires gun v0.3.3 or above.

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

> Note: If no data is found on the key ('person/alice', etc.), gun will implicitly create, update, and `.key` it upon a `put`. This is useful and convenient, but can be problematic for some types of apps. To force gun to require that you explicitly initialize data first, instantiate with `Gun({init: true})` instead.

What if we want to get their data? We can either chain off of the reference directly or get it again:

```javascript
alice.once(function(node){
  console.log('Alice!', node);
});

gun.get('person/bob').once(function(node){
  console.log('Bob!', node);
});
```

> Note: GUN is a functional reactive database for event driven data, so using `.val` is discouraged because it indicates that you are doing things procedurally. Calling `.val()` (no callback) automatically asynchronously logs out the data, which is useful for debugging purposes.

Now let's add all the people into a [set]( https://en.wikipedia.org/wiki/Set_(mathematics) ), you can think of this as a table in relational databases or a collection in NoSQL databases.

```javascript
var people = gun.get('people');
people.set(alice);
people.set(bob);
people.set(carl);
people.set(dave);
```

It is now easy to iterate through our list of people.

```javascript
people.map(function(person){
  console.log("The person is", person);
});
```

> Note: In functional languages `map` is used similarly to a for each loop in javascript. Everything is continuously evaluating in GUN, including `.map`, and will get called when new items are added to the set or when an item is updated - but only for those changes, not the whole list again. This is great for realtime applications.

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
company.once(function(startup){
  console.log("The startup:", startup);
});
```

> Note: GUN returns partials to keep things fast. What you'll see logged out on `startup.address` is not the address itself, but a pointer to the address. Documents can be of any depth, as a result GUN streams only what you need by default.

So what if you want to access the city property on the company's address? You can traverse into your document using the standard dot syntax.

```javascript
company.path('address.city').once(function(value, field){
  console.log("What is the city?", value);
});
```

> Note: As a result, your fields cannot have a '.' in them! The `field` in the callback is always the last property on the path.

We just found out the company got funding and moved to a new office! Let's go ahead and update it.

```javascript
gun.get('startup/hype').put({ // or you could do `company.put({` instead.
  funded: true,
  address: {
    street: "999 Expensive Boulevard"
  }
});
```

> Note: Even writes are treated as partials in GUN. Meaning that the data will merge together with what already exists. This lets us update only the pieces that need to be without worrying about overwriting the whole document.

Documents in isolation are not very useful. Let's connect things and turn everything into a graph!

```javascript
var employees = company.path('employees');
employees.set(dave);
employees.set(alice);
employees.set(bob);

alice.path('spouse').put(bob);
bob.path('spouse').put(alice);

alice.path('spouse').path('employer').put(company);
alice.path('employer').put(company);

dave.path('kids').set(carl);
carl.path('dad').put(dave);

carl.path('friends').set(alice);
carl.path('friends').set(bob);
```

> Note: The distinction between `.put` and `.set` are important. You can have a simple directed link or you can have a one-to-many relationship. This is why we call everything a node in a graph.

Finally, let's read some data out. Starting with a key/value, pathing into fields on a document, then mapping over a table, then navigating into one of its columns and logging out its value with `.val`'s shortcut helper.

```javascript
gun.get('person/alice').path('spouse.employer.employees').map().path('name').once("The employee's");
```

Awesome, now all together - run it yourself: http://jsbin.com/webikepoxa/edit?js,console (Hit "Run", all logs except for the last one have been commented out).