---

> ### NOTE: Examples work with gun version 0.2.x.  Will be upgrading soon for 0.3.x

---

So you want Tabular data with GUN? Huh? You can do that too! 

```javascript
localStorage.clear(); // refresh the storage for this example.

var gun = Gun();
var table = gun.get('my/table').set();

function Person(name, age){
	this.name = name;
	this.age = age;
}

table.set(new Person('Alice', 24));

table.set(new Person('Bob', 25));

table.set(new Person('Carl', 24));

table.map(function(person){
	alert(person.name + " of age " + person.age + " is in the table!");
});
```

(try "run with js" on http://jsbin.com/fijajojala/edit?html,js,output)

# What is happening?

First we need to instantiate GUN. Then we need to select which table we want to load `FROM` (actual SQL support coming soon!) by doing `gun.get('my/table')`. You also notice the `.set()`? Tables (or collections or lists or 'arrays') are called 'sets' in GUN, akin to a [mathematical set](https://en.wikipedia.org/wiki/Set_(mathematics)). Calling `.set()` instantiates an *empty set* (or the *null set*) in case the table doesn't already exist.

Next up is the schema for the table! GUN does not require a schema, but using schemas is recommended - if you are interested in a schema extension for GUN check out [RangerMauve's gun-schema](https://github.com/gundb/gun-schema). For this example we're going to simply use a class instead, thus the `function Person(){}`.

Now we want to insert records into our table! In GUN lingo, all we now have to do is add an *item* into the set. Thus `table.set({name: 'Alice', age: 24});`, but since we want a schema we call `new Person('Alice', 24)` instead. Let's repeat this a couple times adding records into our set.

Finally we want to read each record in the table. All we have to do is call `table.map(function(record){})`! However **WARNING** this also subscribes to all realtime updates too! That means **an individual record might be called multiple times** so do not be alarmed if you get multiple repeats. GUN by default is *functional* and *reactive* in design.

How do you check for uniqueness then? GUN has a unique primary ID called a 'soul' on every record in the database, however you can add your own ID as well and use that to check. But this leads to a bigger question...

# How do Tables work with Graphs?

You can actually think of a graph as table with every record in it. Any other 'table' you create then has a record that just points to primary ID in the graph table. Pretty simple, huh? But it is not just tables that work with graphs, documents do too! With a graph you don't actually store the entire document as a single record, instead every sub object in the document is stored in the graph table and the document is reconstructed with pointers. Woohoo!

# What is the whole Tuple thing?

People often throw around the word "tuple" in graph databases, however tuples are not necessary for graph databases. For instance, Neo4J uses a property graph model - and GUN is similar. That doesn't mean you cannot do tuples though, you can. But that is for a different section.