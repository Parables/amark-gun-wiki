> v0.1.0 has been deprecated.
---

Setting Data
==============

It's easiest to initially think about gun as a field/value store, where JavaScript objects can be stored and retrieved based on a particular key that identifies it. This key can be literally any string; they don't have to follow any particular pattern, although that may be useful for your own organization.

In this example, we'll look at the following objects:

```json

// Cecil (Human)
{
    "name": "Cecil",
    "species": "human",
    "location": "Night Vale"
}

// Khoshekh (Cat)
{
    "name": "Khoshekh",
    "species": "cat"
}
```

**Note:** This tutorial assumes that you've already set up a connection to gun, and that that connection is represented by the variable `gun`. You can read more about getting started [[here|Wire-Protocols-(v0.1.x)]].

To start off, we'll save Cecil to the key `person/cecil`

```javascript
gun.set({
    "name": "Cecil",
    "species": "human",
    "location": "Night Vale"
}).key('person/cecil');
```

Thanks to the [[chaining|Chaining-(v0.2.x)]] that is built into GUN, we can really easily assign multiple keys to the same object, too. For example, if we wanted to access Cecil at both `person/cecil` and `human/1`, we could do that as follows

```javascript
gun.set({
    "name": "Cecil",
    "species": "human",
    "location": "Night Vale"
}).key('person/cecil').key('human/1');
```

Now, both `person/cecil` and `human/1` can be used to look up the same object.

Next, we want to establish that Khoshekh belongs to Cecil. We can do that by using `set` again to set a particular value of Cecil, which we will refer to as `cat`.

```javascript
gun.load('person/cecil').path('cat').set({
    "name": "Khoshekh",
    "species": "cat"
});
```

In the above example, we tell gun to load the object at the `person/cecil` key (which we just set up earlier), then step into the object to the `cat` field, which we set to Khoshekh's object. We can also rely on gun's chaining once again to set a separate key that Khoshekh can be accessed at as well.

```javascript
gun.load('person/cecil').path('cat').set({
    "name": "Khoshekh",
    "species": "cat"
}).key('cat/khoshekh');
```

For more information on getting data from GUN, check out [[Getting Data|https://github.com/amark/gun/wiki/0.1.x-Table-of-Contents#Getting-Data]] on the 0.1.x Table of Contents.
