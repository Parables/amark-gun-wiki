## Radisk Storage Engine
The **Radisk Storage Engine (RAD)** is an _in-memory_, as well as _on-disk_ [radix tree](https://en.wikipedia.org/wiki/Radix_tree) that saves the GUN database graph for fast and performant look-ups.

Radix trees have a constant lookup-time.

### Flow of Data

GUN is a modular system, which allows adapters to 'hook' into the GUN instance through subscriptions to general events (such as `gun.on('in')`) or data events specifically subscribed to via `data.on(callback)`.

When data is put via `gun.get(key).put({object})`, GUN adds the {object} into it's internal graph (in-memory) and then hands the data to the next subscribed adapter. (This may be RAD, localStorage or AWS S3 etc.)

RAD is called from gun/lib/store.js, which should be the template for anyone to start building their own storage adapter for GUN.

### Store.js

```javascript
var storage = Object(null)
gun._.opt.store = {};
gun._.opt.store.put = function(file, data, cb){
	console.log(`put with ${file}, data ${data}, callback`);
	storage[file] = data
	cb(undefined, 1)
}
gun._.opt.store.get = function(file, cb){
	console.log(`get with ${file}, callback`);
	var temp = storage[file] || undefined
	console.log(`Found ${file}: ${temp}`);
	cb(temp)
}
gun._.opt.store.list = function(cb){
	console.log(`list with callback`);
	var arr= [];
	console.log(`Listed ${Object.entries(storage)[0]}`);
	arr = Object.entries(storage)[0];
	if(arr) {
		var i = 0;
		var l = arr.length;
		for(i;i<l;i++){
			if(cb(arr[i])){
				break;
			}
		}
	}
	cb()
}
```

### Radisk.js


```javascript
var fs = require('fs');
var Gun = require('../gun');
var Radix = require('./radix');

function Radisk(opt){

	opt = opt || {};
	opt.file = String(opt.file || 'radata');
	opt.until = opt.until || opt.wait || 1000; // default for HDDs
	opt.batch = opt.batch || 10 * 1000;
	opt.chunk = opt.chunk || (1024 * 1024 * 10); // 10MB
	opt.code = opt.code || {};
	opt.code.from = opt.code.from || '!';
```

_opt_ comes from the GUN instance.

We make sure opt.file is a string or defaults to 'radata'.

opt.until - the delay before writing to disk

opt.batch - the limit of 'entries' per batch

opt.chunk - file size limit before splitting to a new file

opt.code - TODO

opt.code.from - TODO

```javascript

	function ename(t){ return encodeURIComponent(t).replace(/\*/g, '%2A') }
```

`ename()` makes sure files are not named with symbols that would create errors on OS's (such as / - etc)

```javascript

	if(!opt.store){
		return Gun.log("ERROR: Radisk needs `opt.store` interface with `{get: fn, put: fn, list: fn}`!");
	}
	if(!opt.store.put){
		return Gun.log("ERROR: Radisk needs `store.put` interface with `(file, data, cb)`!");
	}
	if(!opt.store.get){
		return Gun.log("ERROR: Radisk needs `store.get` interface with `(file, cb)`!");
	}
	if(!opt.store.list){
		return Gun.log("ERROR: Radisk needs a streaming `store.list` interface with `(cb)`!");
	}
```

Check if store interface has been created and if the 3 API's are available to us.

```javascript
	/*
		Any and all storage adapters should...
		1. Because writing to disk takes time, we should batch data to disk. This improves performance, and reduces potential disk corruption.
		2. If a batch exceeds a certain number of writes, we should immediately write to disk when physically possible. This caps total performance, but reduces potential loss.
	*/
	var r = function(key, val, cb){
		key = ''+key;
		if(val instanceof Function){
			cb = val;
			val = r.batch(key);
			if(u !== val){
				return cb(u, val);
			}
			if(r.thrash.at){
				val = r.thrash.at(key);
				if(u !== val){
					return cb(u, val);
				}
			}
			//console.log("READ FROM DISK");
			return r.read(key, cb);
		}
		r.batch(key, val);
		if(cb){ r.batch.acks.push(cb) }
		if(++r.batch.ed >= opt.batch){ return r.thrash() } // (2)
		clearTimeout(r.batch.to); // (1)
		r.batch.to = setTimeout(r.thrash, opt.until || 1);
	}
```
Function _r_ can be called 2 ways:
GET (key, cb) PUT (key, val, cb)

GET Case

_key_ stringify input

_r.batch(key)_ reads back in-memory batch to check if key is waiting to be written to disk

If found, return it to caller

If not found, check the batch about to be written (staged for thrashing/flushing)

If found, return to caller

If not found, read from disk

PUT Case

_key_ stringify input

_r.batch(key,val)_ write key/val pair to batch

If a callback was attached we attach it in turn to acks in the batch. (acks are acknowledgments, sent out after the batch is written to disk)

Check if the batch count limit is reached and if so, flush to disk.

Also increase the counter _r.batch.ed_

_opt.until_ or as default 1ms is he idle time between put calls, before a flush occurs