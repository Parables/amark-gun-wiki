# All about souls

And a summary of gun's graph architecture

There are a lot of competing database structures out there, like document, tree, tabular, relational... it just seems to go on. But there is one structure that rules them all -- the graph. With graphs, you can make any other data structure, and it's at the heart of gun's paradigm. Here's how it works:

You start with the root object
```javascript
{

}
```
then you fill that object with other named objects
```javascript
{
  bob: {},
  joe: {},
  dave: {}
}
```
but the structure never goes any deeper than that. If you want to add an object inside of another object, you name that object and simply reference it
```javascript
{
  bob: {
    friend: { name: joe }
  },
  joe: {
    friend: { name: sam }
  },
  sam: {
    friend: { name: 'pizza' }
  }
}
```
you'll never actually see names like these in gun, because they're human generated and horribly difficult to come up with. These are computers we're talking about. Instead, we let the computers come up with super long, randomly generated names. That way, we never have to worry about naming conflicts and all of our objects are uniquely identifiable

<!-- TODO: cover relations and what they do -->
```javascript
{
  'ckxAzmBrErR1': {
    friend: { name: 'qx8QuUBturo8' }
  },
  'qx8QuUBturo8': {
    friend: { name: 'u12wN9znw50y' }
  },
  'u12wN9znw50y': {
    friend: { name: 'pizza' }
  }
}
```
Now we can structure our data in whatever pattern we want without hitting complicated nesting issues, because we're just keeping a *reference* to that object, not the object itself. In gun, we refer to these programmatically generated names as "souls", and they drive the entire system. The reason we wrap them inside objects is so that we can tell them apart from ordinary strings. We call those objects "relations", because they "relate" or point to other objects. We don't actually call the property `"name"`, in gun we use a special symbol: the hash sign. Relations look something like this: `{ '#': 'aOCLhos5ADx3' }`. So whenever you see that weird-looking object, it really just means "link to this other object".

So, to recap:
- in a graph, objects never contain other objects, just *references*
- gun calls those references "souls"
- souls wrapped in objects with the hash symbol are called "relations"
- graphs can form any data structure
- and are crazy cool

That should give you an idea of how gun's graph structure works and what exactly a soul is.