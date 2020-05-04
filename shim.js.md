GUN has a unique and powerful API. We recommend you learn it, but we have started `shim.js` to provide some abstraction/wrappers that look similar to Firebase, Parse, or other tools. This **is not a complete shim** to those APIs, nor will it ever be. It is just a gateway drug to playing with gun, so use responsibly.

Contributions would be highly appreciated, consider this the place to add common abstractions/wrappers from other systems that other developers would be familiar with. It will be a community driven shim.

# Use

## NodeJS
```javascript
var Gun = require('gun');
require('gun/lib/shim');
```

## Browser
Without a build tool, you'll need to include each extension that the shim depends upon - as shown below.

With a build tool, you should know how to bundle them into 1 file based on the tags below, then use that.
```
<script src="/gun.js"></script>
<script src="/gun/lib/open.js"></script>
<script src="/gun/lib/bye.js"></script>
<script src="/gun/lib/shim.js"></script>
```

# Firebase

## `.on('value', cb)`

This will loosely approximate Firebase's behavior. Here is an example:

```javascript
var gun = Gun('http://localhost:8080/gun'), ID = 'asdf';

gun.get('interview/' + ID).on('value', function(tree){
  console.log(tree); // no need for calling snapshot
});
```

This will give you the full document every time it changes. It works for documents with circular references too!

```javascript
var interview = gun.get('interview/' + ID);

var data = {
  id: ID,
  position: {
    role: "engineer",
    experience: "ninja"
  }
};
data.position.parent = data;

interview.put(data);
```

When the data is saved with `put` it will trigger `.on('value', cb)` to be called with the full interview document.

[Try this shim out now, with this live demo on jsbin](http://jsbin.com/xodoxolawa/edit?html,js,console)!

This shim wraps [gun.open](https://github.com/amark/gun/wiki/API#open) in the extended API.

## `.onDisconnect()`

This loosely approximates Firebase's behavior. Remember though, GUN uses `put` instead of `set` - just as a tip. Here is how it works:

```javascript
var gun = Gun('http://localhost:8080/gun'), sessionID = 'asdf';

var tab = gun.get('marknadal').get('tabs').get(sessionID).put('online');

tab.onDisconnect().put('offline');
```

We can use this to create a "presence" system, such that another user can track whether their friend is online or not based on how many tabs they have open:

```javascript
gun.get('ambercazzell').get('friends').get('marknadal').get('tabs').on('value', function(sessions){
  for(var tab in sessions){
    if('online' === sessions[tab]){
      UI.update("Your friend is online!");
      return;
    }
  }
  UI.update("Your friend is offline.");
});
```

This shim proxies [gun.bye](https://github.com/amark/gun/wiki/API#bye) in the extended API.

## `.connected(cb)`

A cleaner implementation of Firebase's

```javascript
var connectedRef = new Firebase('http://abc.firebaseio.com/.info/connected');
connectedRef.on('value', function(snap){
  //...
});
```

Instead, just do

```javascript
var gun = Gun('http://localhost:8080/gun');

gun.connected(function(boo){
  if(boo){
    UI.notify("You are back online!");
    gun.get('status').onDisconnect().put('offline');
  } else {
    UI.notify("You are offline!");
  }
});
```

> **Note**: GUN uses lazy loading to conserve bandwidth and battery! So `connected` will only fire after the first read or a write has been performed.  This is not a bug.

Please report if `connected().on('value', cb)` would be preferred.