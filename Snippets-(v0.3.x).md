# Simple plugins to augment, extend, or modify gun

## WARNING: THIS DOCUMENTATION IS LARGELY OUT OF DATE (v0.3.0 not v0.6+), TO SEE MORE UPDATED ONES, CHECK OUT: https://github.com/kristianmandrup/chain-gun

**Table of Contents**
 - [Tables](Tables) :arrow_upper_right:
 - [Strip metadata from returned nodes](#strip-metadata-from-returned-nodes)
   - [`.live()` (to replace `.on()`)](#live-to-replace-on)
   - [`.value()` (to replace `.val()`)](#value-to-replace-val)
 - [Saving/getting images in gun](#savinggetting-images-in-gun)
 - [`Gun.create()` to instantiate without `new`](#guncreate-to-instantiate-without-new)
 - [Using gun for localStorage and peer storage](#using-gun-for-localstorage-and-peer-storage)
 - [Preventing data synchronization](#preventing-data-synchronization)
 - [gun.each](#guneach)
 - [gun.date](#date)
 - [Anonymous Put](https://gist.github.com/metasean/d039054506c1ab6bafc6)  :arrow_upper_right:
   - The anonymous_put method `.put()`s a value onto a parent object without the need for a pre-defined key.
 - [crdt counter](#counter)
 - [recursion](#recursion)

---

## [Tables](Tables) :arrow_upper_right:
  
---
  
## Strip metadata from returned nodes

### `.live()` (to replace `.on()`)

Add the following prior to instantiating Gun:
```javascript
Gun.chain.live = function(cb, opt){ 
  return this.on(function(val, field){ 
    val = Gun.obj.copy(val);
    delete val._; 
    cb.call(this, val, field); 
  }, opt); 
}
```

Instead of using `.on()`, use `.live()`.  
For example,

```javascript
// subscribe to changes to my player bucket
playerNamesDB.on(function(data){
```
would be:
```javascript
// subscribe to changes to my player bucket
playerNamesDB.live(function(data){
```
For more information, see the thread starting at : https://gitter.im/amark/gun?at=566de558187e75ea0e48771e



### `.value()` (to replace `.val()`)

Add the following prior to instantiating Gun:
```javascript
Gun.chain.value = function(cb, opt){
  return this.val(function(val, field){
    val = Gun.obj.copy(val);
    delete val._;
    cb.call(this, val, field);
  }, opt);
}
```

Then, instead of using `.val()`, use `.value()`.  
For example,

```javascript
// subscribe to changes to my player bucket
playerNamesDB.val(function(data){
```
would be:
```javascript
// subscribe to changes to my player bucket
playerNamesDB.value(function(data){
```
For more information, see the thread starting at : https://gitter.im/amark/gun?at=566de558187e75ea0e48771e

## Saving/getting images in gun
*Gun cannot save dom nodes.* That said, you can save a lot of other things, strings included, and image data can be expressed as a string.

```javascript
Gun.chain.image = function (img) {
  if (!img.src) {
    return this.val(function (src) {
      img.src = src
    });
  }
  var canvas = document.createElement('canvas');
  var ctx = canvas.getContext('2d');
  canvas.width = img.width;
  canvas.height = img.height;
  ctx.drawImage(img, 0, 0, img.width, img.height);
  var data = canvas.toDataURL();
  return this.put(data);
}
```

Since there is no standard for extracting raw image data, [the best way](http://stackoverflow.com/questions/934012/get-image-data-in-javascript#answer-934925) is to render it to a canvas, then read the raw canvas data.

Now you can save your image. Use `db.path('example image').image(yourImage)`. To read it back out again, pass in an image *without* a `.src` and the raw data will be loaded onto the empty image.

```javascript
// writing
var img = find('existingImage')
db.get('images').path('your image').image(img)

// reading
var img = document.createElement('img')
// img.src === undefined
db.get('images').path('your image').image(img)
// img.src === 'data:image/png;base64,iVBO...'
```
  
---
  
## `Gun.create()` to instantiate without `new`

Linters will complain if a gun instance is created without the `new` keyword.
You could either use `new`, or you could use `Gun.create([args])`.

```javascript
Gun.create = function () {
  return Gun.apply(this, arguments);
};
```

Referenced and suggested in [issue #6](https://github.com/amark/gun/issues/6)
  
---
  
## Using gun for localStorage and peer storage

```javascript
var me = Gun();                                   // LocalStorage
var gun = Gun('https://gunjs.herokuapp.com/gun'); // peer storage

me.put({"I'm a":'rockstar'}).key('myself');
gun.put({"We're a":'rockband'}).key('myself');

me.get('myself').val();  // Object { _: Object, I'm a: "rockstar" } undefined
gun.get('myself').val(); // Object { _: Object, We're a: "rockband" } undefined
```
   
---
  
## Preventing data synchronization

Supposing your application has user specific data that you don't want to synchronize
(for example, a game where you don't want your opponent to know your internal state),
you could use this extension to save to localStorage *without* synchronizing.

```javascript
Gun.chain.local = function (data, cb, opt) {
  opt = opt || { };
  opt.peers = { };
  return this.put(data, cb, opt)
}

var gun = new Gun().get('example')
gun.path('data').local(private)
gun.path('data').put(synchronized)
```

> **note:** this should work, but it is still under development.

## gun.each
`gun.map` streams pieces of each node in. To only get the full object, you can use this snippet.

```javascript
Gun.chain.each = function () {
  var each = this.map();
  return this.val.apply(each, arguments)
}
```

**Usage**
```javascript
gun.get('examples').each(function (example) {
  console.log(example)
})
```

---

## <a name="date"></a> Storing Dates

Simply store them as the millisecond value:

```javascript
Gun.chain.date = function (data) {
  if (Gun.fns.is(data)) {
    return this.val(function (val) {
      data.call(this, new Date(val));
    }
  }
  return this.put(data.getTime());
};
```

Now you can easily save and read dates from gun. Examples:

```javascript
var date = new Date('Jan 1 2020');
gun.path('now').date(date) // saves the date
gun.path('now').date(function (date) {
  console.log(date instanceof Date) // true
})
```

---

## [Anonymous Put](https://gist.github.com/metasean/d039054506c1ab6bafc6) :arrow_upper_right:

The anonymous_put method `.put()`s a value onto a parent object without the need for a pre-defined key.
  
---

## <a name="counter"></a> CRDT Counter

A simple counter that can run in distributed systems without risking duplication. This is more an example of how gun can be used in distributed settings.

```javascript
Gun.chain.count = function (num) {
  if (typeof num === 'number') {
    this.path(Gun.text.random()).put(num);
  }
  if (typeof num === 'function') {
    var sum = 0;
    this.map().val(function (val) {
      num(sum += val);
    });
  }
  return this;
};
```

**Example**
```javascript
var db = gun.get('count')
db.count(+5)
db.count(-8)
db.count(function (value) {
  console.log(value) // 5, -3
})
// prints: 5
// prints: -3
db.count(+10)
// prints: 7
```

---

## <a name='recursion'></a> Recursively iterate over a tree
You can use this method to recursively map over a document structure.

> If you recurse over a huge dataset, you might end up loading the entire thing into memory. Use this method wisely.

```javascript
Gun.chain.recurse = function (cb, filter) {
  if (!(filter instanceof Object)) {
    filter = {};
  }
  this.val(cb);
  this.map().val(function (data) {
    if (!(data instanceof Object)) {
      return;
    }
    var soul = Gun.is.node.soul(data);
    if (filter[soul]) {
      return;
    }
    filter[soul] = true;
    this.recurse(cb, filter);
  });
  return this;
};
```
---

## <a name="no"></a> No

Similar to `.not` but blocks the rest of the chain from running if there is no data. Useful for **non-idempotent** operations, which causes us to have a big warning: If you are doing updates based off of the conditionality of whether data exists or not, you need to decide what level of certainty is happening!

For instance, if this is just running in the browser which is connected to a single server - then you are probably good. However if you are running this as a federated server that only occasionally syncs with other servers, you might get false positives (where your local server doesn't have the data yet, so it triggers the `no` condition immediately, but then the other peers "later" discover the data does exist). Just use a different extension if this might be a problem.

```javascript
// requires gun v0.5.9+
Gun.chain.no = function(cb){
    var gun = this, chain = gun.chain(), flag;
    gun.not(function(a, b, c){
        flag = true;
        if(!cb){ return }
        cb.call(this, a, b, c)
    });
    gun.get(function(at, ev){
        if(flag){ return ev.off() }
        chain._.on('in', at);
    });
    return chain;
}
```

**Example**

```
gun.get('alice').no(function(){
  // if I get called, then nothing below will ever get called.
  // if I do not get called, the others will get called.
}).val(cb).path('pet').on(cb);
```