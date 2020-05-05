### Basic JSON object

The way GUN stores and communicates data is through a specific subset of denormalized JSON. This might sound confusing, but it is actually pretty simple.

For instance, take this object.

```javascript
{
  age: 23,
  hacker: true,
  name: "Mark Nadal"
}
```
### Metadata

This object is composed of **field** and **value** pairs, where a value is always either `null`, `true` or `false`, a _number_ or _decimal_, a _string_, or a _relation_. For clarity, GUN will use a consistent [[vocabulary|Glossary]], where values are primitives that get saved as a whole update on their field.

In order for GUN to deterministically converge on an update across many decentralized peers, it has to hold meta data about the fields and their values. This meta data is included in the GUN object itself, objects in GUN are called a **node**.

```javascript
{
  _: {},
  age: 23,
  hacker: true,
  name: "Mark Nadal"
}
```

That meta data is stored on the reserved `_` field. 

### Soul  

Every node also must have a universally unique ID, called a **soul**, which is also stored in the meta data under the reserved `#` field. Like so:

```javascript
{
  _: {'#':'ASDF'},
  age: 23,
  hacker: true,
  name: "Mark Nadal"
}
```

For the sake of comprehension, our example is using a very short soul. Souls should never be this short, and should be generated as a UUID or GUID or as some sufficiently long random alphanumeric string. They also should never conflict with any other node in any other app that uses GUN - more on this later. But first:

```javascript
{
  _: {
    '#':'ASDF',
    '>': {}
  },
  age: 23,
  hacker: true,
  name: "Mark Nadal"
}
```

#### <a name="HAM"></a>Hypothetical Amnesia Machine (HAM) State
Next up now is the meta data for the Hypothetical Amnesia Machine, reserved as the `>` field. This contains a copy of all the fields that have been on the node, with their corresponding convergence states. Such as:

```javascript
{
  _: {
    '#':'ASDF',
    '>': {
      age: 2,
      hacker: 2,
      name: 2
    }
  },
  age: 23,
  hacker: true,
  name: "Mark Nadal"
}
```

There you go, that is a complete GUN **node**. Since GUN can sync on partial nodes, you get a very clean and lean way to perform delta updates. You only have to send the diff of a node and not the whole node itself. Now let us look back at the original object and try to do the following:

```javascript
{
  age: 23,
  friend: {
    hacker: true,
    name: "Shreyas Raman"
  },
  hacker: true,
  name: "Mark Nadal"
}
```

### Graph

What GUN will do, is it will denormalize it into a subset of JSON which is called a **graph**. We are going to hide the HAM meta data from our view for sake of clarity.

```javascript
{
  'ASDF': {
    _: {'#':'ASDF'},
    age: 23,
    friend: {'#':'FDSA'}
    hacker: true,
    name: "Mark Nadal"
  },
  'FDSA': {
    _: {'#':'FDSA'},
    hacker: true,
    name: "Shreyas Raman"
  }
}
```

You can see that we flatten the object into a graph, which has field and node pairs. The fields on a graph correspond to the soul of the node it has. But how do we maintain who Mark's friend is? Well, we mentioned that a primitive value can also be a **relation**. A relation has a very simple structure, it is an object with just a soul field on it.

```javascript
{'#': 'FDSA'}
```

The value of that field is the soul of the node that it points to. Since souls are universally unique, they could point to data within your app or outside of it. This subset of JSON allows us to serialize a web of data, including circular references! Now, finally, there is only one thing beyond a graph:

```javascript
{
  ...
}
```

The graph of all graphs over all time, called the **universe**. However, since a single machine cannot process that much data simultaneously, we must distribute the data across many machines, each focusing on some finite graph over a particular interval of time. These graphs contain nodes which hold the actual fields and values we are interested in, as well as the meta data to guarantee convergences.

### la Fin
And there you go, ladies and gentleman. From the smallest atom all the way up to a universe of data. Expressed succinctly in simple permutations of **value**, **field**, **node**, **graph**. I hope you enjoyed this discourse in GUN's formatted subset of JSON. If you have any questions, please ask away at `mark@gunDB.io`!