> v0.2.x is deprecated in favor of [v0.3.x](API-(v0.3.x))
---

---

***Please post documentation comments, questions, and suggestions to [[issue #70|https://github.com/amark/gun/issues/70]].***
  
--- 

### **Gun** 
#### `var gun = Gun(options)`

  - Instantiates a gun database (specifying in the options parameters the connections and persistence endpoints).
  - NOTE: This may also be called with new (e.g. `var gun = new Gun(options)`), but `new` is *not* necessary.

  - `options` as

     - `undefined` creates a local datastore using the default persistence layer, either localStorage or a file.

     - `'string'` URL creates a datastore that attempts to sync with the specified peer.

     - `['array']` of URL strings creates a datastore that attempts to sync with multiple peers.

     - `{obj:'ect'}` - All the previous options are aggregated into an actual options object, with

        - `options['module']` if you are using a GUN module, you can pass it options via its name.

        - `options.peers` is an object with the URLs as the field and a value of an object.

        - `options.hooks` is an object with functions on `put`, `key`, `get`, or `all` hook fields.

        - `options.uuid` is a function to override the default 24 random alphanumeric soul generator.

  - Examples

    - `var gun = Gun()`

    - `var gun = Gun("http://localhost:8080/gun")`

    - `var gun = Gun(["http://localhost:8080/gun", "http://gunjs.herokuapp.com/gun"])`

    - `var gun = Gun({file: 'data.json'})` to change the name of the file that gets dumped to. **Warning!** The file module is the server's default persistence, and should only be used for local development testing only!

    - `var gun = Gun({s3: {key: '', secret: '', bucket: ''}})` on the server dumps to AWS S3, this is the preferred persistence layer.

### **put**
#### `gun.put(object, callback, options)`

 - Saves the object.

 - `object` is a javascript `{obj:'ect'}`, it can be deeply nested, be a [[partial|Partials and Circular References (v0.2.x)]], or have [[circular references|Partials and Circular References (v0.2.x)#circular-references]] in it, but it cannot have the following values inside of it: `undefined`, `Infinity`, `NaN`, `[]`.

 - `callback` is a `function(){}` which gets called as `callback(err, ok)` after the highest level of persistence has happened per each connected peer. As long as there is no `err` then things should be ok, `ok` may not be defined. Acknowledgement of persistence is slow, but the write propagates across the network as fast as the pipes connecting them.

   - *no* - If you are connected to no peers, it gets called back once from your local persistence hook.

   - *one* - If you are connected to one peer, it gets called back once after that peer's persistence hook has finished; in a traditional client-server setup where the server is backing up to S3, you will not get an acknowledgement until S3 has either succeeded or failed to persist your data.

   - *many* - If you are connected to many peers, it gets called back multiple times for each individual peer. That peer will respond as if it was the only peer in the above case.

 - `options` is an `{obj:'ect'}`, no options are currently available except maybe hook specific.

 - Examples

   - `gun.put({hello: "world"})` blindly fire and forget.

   - `gun.put({hello: "world"}, function(err, ok){ })` to handle errs or know that your data actually got persisted.

   - `gun.put({field: "value", multiple: "pairs!"})`

   - `gun.put({some: {deeply: {nested: 'object'}}})` for document based data.

   - Graph based data
     
     ```javascript
     var doctor = {species: "Time Lord", home: "Gallifrey"};
     var TARDIS = {name: "Time And Relative Dimension In Space", cloak: "Police Box"};
     TARDIS.pilot = doctor;
     doctor.ship = TARDIS;
     gun.put(doctor);
     ```
     both objects get saved into a graph.

### **key**
#### `gun.put(object).key(key, callback, options)`

  - Without a key, we cannot get our data back unless we scan over the entire graph. To get fast and easy access to the data, we give it a unique key. A key is a lot like keys in real life, once made they allow you to open up doors so you can enter into your data in many different ways.

  - `key` is a `'string'`, the name you want to give your data to reference it later. Common practice is to namespace your keys into routes corresponding to resources.

  - `callback` is a `function(){}` which gets called as `callback(err, ok)` in the same way **put** does.

  - `options` as a `'string'` is a soul that you forcibly want to associate the `key` with, ignoring the current context that key is chained on.

  - Examples

    - `gun.put({hello: "world"}).key('message/from/thedoctor')` blindly fire and forget.

    - `gun.put({hello: "world"}).key('message/from/thedoctor', function(err, ok){})` to handle errs or know that your key actually got persisted.

    - `gun.put({title: "The Doctor", phone: '770-090-0461'}).key('user/thedoctor').key('phone/07700900461')` you can chain keys together to create multiple unique references to the same node.

### **get**
#### `gun.get(key, callback, options)`

  - Opens up a gun reference to the root object you had saved to that key. It will load the node so you can further manipulate it.

  - `key` is a `'string'` name that exactly matches a key you have made earlier.

  - `callback` is a `function(){}` which gets called as `callback(err, graph)` used for err handling and the raw graph. **Note** that you do not want to use this callback for every day development, as it gets called repeatedly to comply the wire protocol. Use **on** or **val** instead, they are convenience methods, this `callback` is for hooks and extensions.

  - `options` is a `{obj:'ect'}` with

    - `options.force` as `true` to pull from the local persistence layer or a peer rather than memory.

  - Examples

    - `gun.get('user/thedoctor')` returns a gun reference to The Doctor.

    - `gun.get('user/thedoctor', function(err, graph){})` to handle errs or inspect the graph from the wire.

    - `gun.get('user/thedoctor', function(){}, {force: true})` for a slower response, skipping memory.

### **on**
#### `gun.get(key).on(callback, options)`

  - Retrieve the raw javascript data associated with the `key`, and subscribe to all subsequent realtime changes. You want to use this method to actually synchronously react to your data, rather than being stuck in async land.

  - `callback` is a `function(){}` which gets called as `callback(data, field)` and every time the node changes. Where

    - `data` is a raw javascript object or primitive.

    - `field` is the field name it came from, if any.

  - `options` as `true` aggregates into an `{obj:'ect'}` with

    - `options.change` as `true` makes `data` only have the delta difference of the change that happened, rather than the full node again and again.

  - Examples

    - ... 
      ```javascript
      gun.get('user/thedoctor').on(function(who, field){
        console.log('The Doctor', who, field);
        // {title: "The Doctor", phone: '770-090-0461'}
      })
      ```

    - While this looks no different from above, the `delta` will be on subsequent events. 
      ```javascript
      gun.get('user/thedoctor').on(function(delta, field){
        console.log('What changes happened to The Doctor?', delta, field);
        // {title: "The Doctor", phone: '770-090-0461'}
      }, true)
      ```

### **val**
#### `gun.get(key).val(callback, options)`

  - Is the same as **on** except only gets called once after the first peer, including the local peer, replies. It is slower than **on** and should be avoided, but is sometimes necessary. If you are using **val** it probably means your code has become spaghetti and very procedural, **on** is much cleaner and more functional.

  - `callback` is a `function(){}` which gets called as `callback(data, field)` once after the node has been loaded, and has no realtime features.

  - `options` none currently available.

  - Examples

    - ... 
      ```javascript
      gun.get('user/thedoctor').val(function(who, field){
        console.log('The Doctor', who, field);
        // {title: "The Doctor", phone: '770-090-0461'}
      })
      ```

    - `gun.get('user/thedoctor').val()` will automatically `console.log`, for easy debugging purposes.

### **path**
#### `gun.get(key).path(path, callback, options)`

  - Traverses into the fields on the path, which allow you to explore the nested objects, relations, and values from the perspective of the root node that was given a key.

  - `path` is a `'string'` of dot notation separated fields.

  - `callback` is a `function(){}` which is called as `callback(err, data, field)` used for err handling and the raw data. **Note** like **get**, you do not want to use this callback for every day development, use **on** or **val** instead.

  - `options` currently none available.

  - Examples

    - Gives us just the phone number and whenever it changes.
      ```javascript
      gun.get('user/thedoctor').path('phone').on(function(value, field){
        console.log("The Doctor's number", value, field);
        // '770-090-0461', 'phone'
      })
      ```

    - Gives us just the title just once.
      ```javascript
      gun.get('user/thedoctor').path('title').val(function(value, field){
        console.log("The Doctor's title", value, field);
        // 'The Doctor', 'title'
      })
      ```
  - **Potentially unexpected behavior**

	- `Gun().get('example').path(0.123).set('data')`
	- `Gun().get('user').path('first.last').set('data')`
  

	 **Behavior**:
	 In these cases, if there is not an existing node at "0" or "first", these sets will silently fail.
  
		**Background**:
		In the above cases, the values passed to `path` are not a number or string, rather they are the path values.  (Similar to nested object keys, e.g. the code `var user = {}; user.first.last = 'Smith';` would fail with 'TypeError: obj.first is undefined'.)  As a result, the above are actually shorted versions of:

		`Gun().get('example').path(0).path(123).set('data')`
		`Gun().get('user').path('first').path('last').set('data')`
  

		In the first case, the number '0.123' is first stringified.

		In both cases, if the first path's node does not exist *and* there is no `set()` on the first path's node, then the second `path` and its `set('data')` are never reached.

		**Resolutions**

		- *Using just gun*

			Because path isn't taking an actual string, if parent nodes may not exist, then they should be pathed into and set after pathing into each node.  For example:

			`Gun().get('example').path(0).set().path(123).set('data')`
			`Gun().get('user').path('first').set().path('last').set('data')`
  

		- *Alternatives*
		
			Modules *can* be written which will allow application-specific behavior.  Any such modules will likely perform slower than the core implementation.  At the time of this document, no modules have been written.

### **map**
#### `gun.get(key).map(callback, options)`

  - Rather than having to know each field in advance and have to path into it separately, we can iterate over each field dynamically.

  - `callback` is a `function(){}` which gets called as `callback(data, field)`, you can either use it as is or in conjunction with **on** or **val**. In the future this `callback` will be used to filter the data or do arbitrary queries.

  - `options` as `true` aggregates into an `{obj:'ect'}` with

    - `options.node` as `true` makes map only iterate over nested objects, skipping primitive values.

  - Examples

    - ...
      ```javascript
      gun.get('user/thedoctor').map(function(value, field){
        console.log("For each field", value, field);
        // '770-090-0461', 'phone'
        // 'The Doctor', 'title'
      })
      ```

    - ...
      ```javascript
      gun.get('user/thedoctor').map().on(function(value, field){
        console.log("For each field, and on every change", value, field);
        // '770-090-0461', 'phone'
        // 'The Doctor', 'title'
      })
      ```

    - ...
      ```javascript
      gun.get('user/thedoctor').map().val(function(value, field){
        console.log("For each field, just once", value, field);
        // '770-090-0461', 'phone'
        // 'The Doctor', 'title'
      })
      ```

### **back**
#### `gun.back`

  - This is not a method function, it is the previous gun reference in the chain. Allowing you to navigate back from where you have **path**ed or **map**ped.

### **set**
#### `gun.get(key).set(data, callback, options)`

  - Is a convenience function that does a **put** to a random field name, adding the data to the set. Sets are unordered lists, similar to tables or collections in other databases. This is helpful when we are working with sets of data where we do not care what the field name is because we will be mapping over the data. **Warning** that this approach will always be slower than accessing things directly with a key, but sometimes it is exactly what we want.

  - `data` can be an `{obj:'ect'}` like with **put**, or a value like a `'string'`, `42` number, `true` boolean, or `null`. **Note** that set returns a gun reference to the nested object if you pass it an object, if you pass it a value it returns the current gun reference.

  - `callback` is a `function(){}` with the same specification as **put** because set relies upon it.

  - `options` none currently available.

  - Examples

    - `gun.get('enemies/of/thedoctor').set("dalek")`

    - `gun.get('enemies/of/thedoctor').set("cybermen").set("weeping angels")` we can only chain sets if they are primitives.

    - `gun.get('enemies/of/thedoctor').set({title: "The Master", species: "Time Lord"}).back.set("silence")` if we do set an object, we can go **back** up to the previous gun reference which corresponds to the key that we **get**. Then we can continue with adding to our set.

    - Now if we
      ```javascript
      gun.get('enemies/of/thedoctor').map().val(function(value, field){
        console.log("For each item in the set, once", value);
        // "dalek"
        // "cybermen"
        // "weeping angels"
        // {title: "The Master", species: "Time Lord"}
        // "silence"
      })
      ```
      order is not guaranteed.

    - But doing
      ```javascript
      gun.get('enemies/of/thedoctor').map(function(value, field){
        console.log("For each node in the set", value);
        // {title: "The Master", species: "Time Lord"}
      }, true)
      ```
      filters out values, leaving us with just the objects in the set.

### **not**
#### `gun.get(key).not(callback, options)`

  - Sometimes you might want to check if data on a key has not yet been defined. **Warning** there is no guarantee that the data does not actually exist somewhere on some peer that got disconnected. But, if you are using gun in a fairly traditional client-server setup this should mostly work but we still advise to avoid it.

  - `callback` is a `function(){}` which gets called as `callback(key, kick)` only if the `key` could not be found, where

    - `key` is the key name that could not be found.

    - `kick` is a `function(){}` which you call as `kick(chain)` where `chain` is some gun reference. Generally this is not needed, so you can simply `return chain` instead.

  - `options` none currently available.

  - Examples

    - ...
      ```javascript
      gun.get('somewhere').not(function(key, kick){
        return gun.put({hello: "world"}).key(key);
      }).val()
      ```
      if `'somewhere'` is not found, we put something on it and set its key. The chain that we `return` becomes the context for the `.val()` which then `console.log`s out `{hello: "world"}`!

    - Be careful as `.not` is not an idempotent operation! If a false positive does happen, gun will attempt to do a pseudo merge on **get** across nodes sharing the same key.