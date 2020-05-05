It's easiest to initially think about gun as a field/value store, where JavaScript objects can be stored and retrieved based on identifier keys. A key can be any valid string; keys don't have to follow any particular pattern, although key patterns may be useful for applications.

In this example, we'll look at the following objects:

```javascript

// Sarah Jane (Human)
{
    "name": "Sarah Jane Smith",
    "species": "human",
    "location": "London"
}

// K-9 (Canine)
{
    "name": "K-9",
    "species": "canine"
}
```

**Note:** This tutorial assumes that you've already set up a connection to gun, and that the connection is represented by the variable `gun`. You can read more about getting started on the [[wire specification|Wire-specification-and-API-(v0.2.x)]] page.

To start off, we'll save Sarah Jane to the key `person/sarahjane`

```javascript
gun.put({
    "name": "Sarah Jane Smith",
    "species": "human",
    "location": "London"
}).key('companion/sarahjane');
```

Thanks to the [chaining](Chaining-(v0.2.x)) that is built into GUN, we can really easily assign multiple keys to the same object, too. For example, if we wanted to access Cecil at both `person/cecil` and `human/1`, we could do that as follows

```javascript
gun.put({
    "name": "Sarah Jane Smith",
    "species": "human",
    "location": "London"
}).key('companion/sarahjane').key('human/1');
```

Now, both `companions/sarahjane` and `human/1` can be used to look up the same object.

Next, we want to establish that K-9 belongs to Sarah Jane. We can do that by using `set` again to set a particular value of Sarah Jane, which we will refer to as `k9`.

```javascript
gun.get('companion/sarahjane').path('canine').put({
    "name": "K-9",
    "species": "canine"
});
```

In the above example, we tell gun to load the object at the `companion/sarahjane` key (which we just set up earlier), then step into the object to the `canine` field, which we set to K-9's object. We can also rely on gun's chaining once again to set a separate key that K-9 can be accessed at as well.

```javascript
gun.get('companion/sarahjane').path('canine').put({
    "name": "K-9",
    "species": "canine"
}).key('canine/k9');
```

For more information on getting data from GUN, check out the [[Getting Data|Getting-Data-(v0.2.x)]] wiki page.