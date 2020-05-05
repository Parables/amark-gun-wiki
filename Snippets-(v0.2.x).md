This page documents simple code snippets which augment or modify basic gun functionality.

## Strip metadata from returned nodes

### `.live()` (to replace `.on()`)

Add the following prior to instantiating Gun:
```javascript
Gun.chain.live = function(cb, opt){ 
  return this.on(function(val, field){ 
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

## `Gun.create()` to instantiate without `new`

Linters will complain if a gun instance is created without the `new` keyword.
You could either use `new`, or you could use `Gun.create([args])`.

```javascript
Gun.create = function () {
  return Gun.apply(this, arguments);
};
```

Referenced and suggested in [issue #6](https://github.com/amark/gun/issues/6)

## Using gun for localStorage and peer storage

```javascript
var me = Gun();                                   // LocalStorage
var gun = Gun('https://gunjs.herokuapp.com/gun'); // peer storage

me.put({"I'm a":'rockstar'}).key('myself');
gun.put({"We're a":'rockband'}).key('myself');

me.get('myself').val();  // Object { _: Object, I'm a: "rockstar" } undefined
gun.get('myself').val(); // Object { _: Object, We're a: "rockband" } undefined
```

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