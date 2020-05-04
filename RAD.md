RAD is a storage adapter for GUN that stores data at disk using a radix tree, it looks like this:

![](https://gun.eco/see/radix.gif)

As you see, we are able to store 3 records (Alex, Alexandria, Andrew) in only 2 rows because "Alex" and "Alexandria" share a common prefix.

Because writing storage adapters for GUN has a bunch of nuances and performance tradeoffs, we've designed RAD to handle these nuances for you and chunk GUN's graph into traditional files that can be dumped to disk. It handles correctly merging updates into each batch, managing memory allocation on heavy load, reads across ranges of chunks, and more with a generalizable performance strategy.

RAD does not have an opinion on what storage engine should actually be used, and requires you pass it a storage interface - whether that be [`fs`](https://github.com/amark/gun/blob/master/lib/rfs.js), [S3](https://github.com/amark/gun/blob/master/lib/rs3.js), [indexedDB](https://github.com/amark/gun/blob/master/lib/rindexed.js), localStorage, BitTorrent, or others.

 ## Install

RAD is now the default with GUN in NodeJS. **RAD is used automatically in NodeJS, just `require('gun')`!**. If you want to use by itself, without GUN, skip this section.

To use RAD with GUN in the **browser**, include:

```html
<script src="https://cdn.jsdelivr.net/npm/gun/gun.js"></script>
<script src="https://cdn.jsdelivr.net/npm/gun/lib/radix.js"></script>
<script src="https://cdn.jsdelivr.net/npm/gun/lib/radisk.js"></script>
<script src="https://cdn.jsdelivr.net/npm/gun/lib/store.js"></script>
<script src="https://cdn.jsdelivr.net/npm/gun/lib/rindexed.js"></script>
<script>
var gun = Gun();
// ...
</script>
```

This will automatically tell GUN to use RAD with IndexedDB.

 > Note: Pass `Gun({localStorage: false})` to disable localStorage.

**NodeJS** by default uses disk (`fs`). If you have [AWS credentials](Using-Amazon-S3-for-Storage) set in your process environment variables, then S3 will be used instead of disk. (You may need to add `aws-sdk` to your `package.json` though.)

 ## Lex

RAD supports GUN's lexical wire spec, which lets you do some cool things in GUN.

For example, you might save a chat message to `gun.get('chat').get('2019/06/20:10:10:10.30').put(message)`, and then you `gun.get('chat').map().once(cb)` to get the full chat. But as the chat grows, you probably do not want to display old messages. Lexical lookups let you fix this!

You have probably noticed GUN's soul `{'#': 'uuid'}`, this is how GUN does `get` commands between peers. Likewise, if you do `gun.get('chat').get('2019/06/20:10:10:10.30')` that gets expressed as `{'#': 'chat', '.': '2019/06/20:10:10:10.30'}`, this finds an exact lexical match. But you can also do:

```javascript
gun.get('chat').get({'.': {'*': '2019/06/'}}).map().once(cb)
```

This will grab keys with `'2019/06/'` matching prefix (grab all chats in July!) *up to a limited number of bytes*.

 > Note: To prevent flooding the network, any response to a lexical lookup that does not have an exact match will be limited. You are responsible for asking for more data if you run over the byte limit.

You can continue getting more data by using a lexical range, and passing the last key of each response you get as the starting property of the next lookup, until you reach an ending key or run out of data.

For example, what if you want to paginate through your friend list?

```javascript
gun.get('friends').get({'.': {'>': 'alice', '<': 'fred'}, '%': 50000}).once().map().once(cb)
```

This says get keys starting with Alice and try to limit the range to less than 50KB. Say you got back Alice, Bob, Carl, and Dave, before hitting a byte limit. You might want to continue, on scroll or when your user clicks a "next" page button:

```javascript
gun.get('friends').get({'.': {'>': 'dave', '<': 'fred'}, '%': 50000}).once().map().once(cb)
```

 > Note: `%` is the limit on how many bytes a peer might parse from disk, not necessarily how many bytes you'll get back. These constraints are to make the other peer's work easier, not requirements that they have to fulfill.

You can pass `{'<': 'zach', '-': 1}` to have the byte limit go in reverse lexical direction.

Lexical gets are matched based in order of cascading specificness:

 1. `=` exact match. If `{'=': 'key'}` is specified, (1 >) will not match.
 2. `*` prefix match or (2 <) match. If `{'*': 'key'}` is specified, (2 >) will not match.
 3. `>` and `<` match or (3 <) match. If `{'>': 'start', '<': 'end'}` is specified, (3 >) not matched.
 4. `>` or `<` match or (4 <) match. As `{'>': 'start'}` or `{'<': 'end'}`.

A `>` match will already include everything a `*` matches, this is true and obvious if you think about it in detail. What may not be obvious though is:

 > Note: `>` will also match for an `=` exact match even if `=` is not listed (if it is, it will overrule `>`). So `{'>': 'alice'}` will match `'alice'` also, same for `<`. So think of it as a hierarchy `> || (> && <) || * || =` or `< || (< && >) || * || =`.

There is no guarantee a peer will comply with lexical constraints. Just like in real life, if you ask somebody a question, they might answer with additional information. Every peer should enforce the constraint.

 ## Without

If you want to use RAD without GUN? Just do:

```javascript
// var Rad = require('gun/lib/radisk'); // in NodeJS
var Rad = window.Radisk; // in Browser, still needs the above script tags.
var rad = Rad(opt);
```

What is `opt`? Check the *API*!

 ## API

RAD takes 1 parameter, an option object, with at least 1 property of an `opt.store` object having:

 - `put` **function** (*key, data, cb*) for saving chunks.
 - `get` **function** (*key, cb*) for reading chunks.

It is easier to understand with some examples, here is a localStorage plugin for RAD:

```javascript
var opt = {store: {}};
opt.store.put = function(key, data, cb){
  localStorage[''+key] = data;
  cb(null, 1);
}
opt.store.get = function(key, cb){
  cb(null, localStorage[''+key])
}
```

That is all! It should be easy to implement or integrate any storage engine, or use any of the several already included.

 > Note: localStorage uses a synchronous API, most storage engines will have an asynchronous API which may make your code look ugly. But the actual integration with RAD is as simple as a file `put` and `get` command.

There are several other options you can configure:

 - `opt.file` **text** *name* of the folder data will be filed under. (Default: `'radata'`) 
 - `opt.chunk` **number** of *bytes*, the size at which a file will split into chunks, unless there is only 1 item in the chunk (like an image). (Default: **10MB** NodeJS, **1MB** IndexedDB) Adjusting this property significantly affects performance due to RAD or JSON parse time for each chunk on smaller machines.
 - `opt.until` **number** of *milliseconds* to wait `until` flushing a batch to disk. (Default: `250`)
 - `opt.batch` maximum **number** of items saved before forcing a flush to disk, regardless of `until`. (Default: **10K**)
 - `opt.pack` **number** of *bytes*, what the maximum string size can be, in order to prevent running out of memory. (Default: `1399000000 * 0.3`)

Now onto actually using RAD to save data!

 ## Write

Now that we have our `rad = Rad(opt)`, we can save data to it! Again, this is assuming non-GUN data:

```javascript
rad('alex', 27, function(err, ok){})
rad('alexandria', 'library', function(err, ok){})
rad('andrew', true, function(err, ok){})
```

 > Note: RAD only accepts `null`, `true`/`false`, *numbers*, and *text* **types**, plus a *soul* link.

 ## Read

Using our `rad = Rad(opt)`, we can read non-GUN data:


```javascript
rad(key, function(err, data, info){})
```

 - `key` **text** like `'al'`, `'alex'`, `'alexandria'`, and so on.
 - `err` **any**, whatever the lower level storage engine emits.
 - `data` (**value, tree, none**) if a value exists at that exact key, you will get the value. A radix **tree** will be passed back if sub keys exists beneath that key, this could be used to create new in-memory `Radix` instance.
 - `info` **object** for building databases (like GUN!) on top, with how many bytes `parsed`, `chunks` processed, if `some` data has been found yet, what the `next` file will be, and the `limit` to bytes parsed.

 > Note: [`Radix`](https://github.com/amark/gun/blob/master/lib/radix.js) is an in-memory Radix tree that RAD depends upon. In browser `window.Radix`, and `require('gun/lib/radix')` in NodeJS.


```javascript
rad('alex', function(err, data){ console.log(data) }) // 27
rad('alexandria', function(err, data){ console.log(data) }) // 'library'
rad('andrew', function(err, data){ console.log(data) }) // true

rad('al', function(err, data){ console.log(data) }) // sub tree
rad('al', 'hi');
rad('al', function(err, data){ console.log(data) }) // 'hi'
rad('alex', function(err, data){ console.log(data) }) // 27
```

 > Note: Radix trees do not include their parent prefix, only the sub keys.

RAD also supports **range queries**! Just pass an options object: (Not available in `<= v0.2019.422`)

```javascript
rad(prefix, function(err, data, info){}, opt)
```

 - `opt.start` the first key to start at.
 - `opt.end` the last key to include.
 - `opt.reverse` look through keys in reverse direction.

You will get back a tree that you can "forEach" over with `Radix.map(tree, function(value, key){})`.

```javascript
var opt = {start: 'ale', end: 'amy'};
rad('', function(err, tree){
  Radix.map(tree, function(value, key){
    console.log(key, value); // ('alex', 27), ('alexandria', 'library')
  }, opt);
}, opt) 
```

Have any questions? Ask on [StackOverflow](https://stackoverflow.com/questions/tagged/gun) tagged `gun`. Need help? Jump on the [chat](https://gitter.im/amark/gun)!