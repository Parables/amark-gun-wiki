Not all adapters are created equal, and you will need to choose an adapter to best suit your application's needs.

By default, Gun dumps its graph into localStorage on the browser and into a flat file in Node environments.

The flat file is _not_ meant for production and the primary reason is performance at scale. Once your graph becomes larger, it will not (and is not intended) to keep up with production IO requirements.

# Key Considerations

* Are updates or creates more important?
* How large are your nodes? Keep in mind that anything created by a `set` is just building up a node over time.

# How Benchmarks Work

Benchmarks include four operations on nodes of 3 sizes:

**Operations:**
* Create `n` nodes
* Read `n` nodes
* Full re-write `n` nodes
* Update single property on `n` nodes

**Node sizes:**

* Small nodes: 10 properties each
* Medium nodes: 1,000 properties each
* Large nodes: 10,000 properties each

These performance tests do not completely reflect a production environment since each operation waits for the preceding to complete before continuing. In production, you should probably anticipate some level of concurrency.

## Gun-DB

Storage in local filesystem via Sqlite + VFS

[npm](https://www.npmjs.com/package/gun-db)/[github](https://github.com/d3x0r/gun-db) by [d3x0r](https://github.com/d3x0r)

### sqlite, native filesystem (windows)
**__ Small Nodes: 10 Properties Each __**
* Write 10000 nodes: : 19841ms; 19.841s; 1.984 ms/node; errors: 0.
* Read 10000 nodes: : 7690ms; 7.69s; 0.769 ms/node; errors: 0.
* Update 10000 nodes: : 22273ms; 22.273s; 2.227 ms/node; errors: 0.
* Update single field on 10000 nodes: : 3215ms; 3.215s; 0.321 ms/node; errors: 0.

**__ Medium Nodes: 1000 Properties Each __**
* Write 100 nodes: : 18155ms; 18.155s; 179.752 ms/node; errors: 0.
* Read 100 nodes: : 15554ms; 15.554s; 154.000 ms/node; errors: 0.
* Update 100 nodes: : 21044ms; 21.044s; 208.356 ms/node; errors: 0.
* Update single field on 100 nodes: : 245ms; 0.245s; 2.426 ms/node; errors: 0.

**__ Large Nodes: 10000 Properties Each __**
* Write 10 nodes: : 20773ms; 20.773s; 1888.455 ms/node; errors: 0.
* Read 10 nodes: : 17342ms; 17.342s; 1576.545 ms/node; errors: 0.
* Update 10 nodes: : 22796ms; 22.796s; 2072.364 ms/node; errors: 0.
* Update single field on 10 nodes: : 1919ms; 1.919s; 174.455 ms/node; errors: 0.

#### sqlite, vfs
**__ Small Nodes: 10 Properties Each __**
* Write 10000 nodes: : 11667ms; 11.667s; 1.167 ms/node; errors: 0.
* Read 10000 nodes: : 7086ms; 7.086s; 0.709 ms/node; errors: 0.
* Update 10000 nodes: : 13439ms; 13.439s; 1.344 ms/node; errors: 0.
* Update single field on 10000 nodes: : 2530ms; 2.53s; 0.253 ms/node; errors: 0.

**__ Medium Nodes: 1000 Properties Each __**
* Write 100 nodes: : 12935ms; 12.935s; 128.069 ms/node; errors: 0.
* Read 100 nodes: : 5792ms; 5.792s; 57.347 ms/node; errors: 0.
* Update 100 nodes: : 16061ms; 16.061s; 159.020 ms/node; errors: 0.
* Update single field on 100 nodes: : 200ms; 0.2s; 1.980 ms/node; errors: 0.

**__ Large Nodes: 10000 Properties Each __**
* Write 10 nodes: : 16134ms; 16.134s; 1466.727 ms/node; errors: 0.
* Read 10 nodes: : 7653ms; 7.653s; 695.727 ms/node; errors: 0.
* Update 10 nodes: : 21076ms; 21.076s; 1916.000 ms/node; errors: 0.
* Update single field on 10 nodes: : 1962ms; 1.962s; 178.364 ms/node; errors: 0.


#### sqlite, vfs (with encryption)
__ Small Nodes: 10 Properties Each __
* Write 10000 nodes: : 14899ms; 14.899s; 1.490 ms/node; errors: 0.
* Read 10000 nodes: : 7301ms; 7.301s; 0.730 ms/node; errors: 0.
* Update 10000 nodes: : 18248ms; 18.248s; 1.825 ms/node; errors: 0.
* Update single field on 10000 nodes: : 3224ms; 3.224s; 0.322 ms/node; errors: 0.

__ Medium Nodes: 1000 Properties Each __
* Write 100 nodes: : 20645ms; 20.645s; 204.406 ms/node; errors: 0.
* Read 100 nodes: : 5952ms; 5.952s; 58.931 ms/node; errors: 0.
* Update 100 nodes: : 27732ms; 27.732s; 274.574 ms/node; errors: 0.
*Update single field on 100 nodes: : 350ms; 0.35s; 3.465 ms/node; errors: 0.

__ Large Nodes: 10000 Properties Each __
* Write 10 nodes: : 28288ms; 28.288s; 2571.636 ms/node; errors: 0.
* Read 10 nodes: : 9496ms; 9.496s; 863.273 ms/node; errors: 0.
* Update 10 nodes: : 37709ms; 37.709s; 3428.091 ms/node; errors: 0.
* Update single field on 10 nodes: : 3587ms; 3.587s; 326.091 ms/node; errors: 0.

## Gun-Mongo-key

Key:value Adapter for MongoDB

[npm](https://www.npmjs.com/package/gun-mongo-key)/[github](https://github.com/sjones6/gun-mongo-key) by [sjones6](https://github.com/sjones6)

Tests run on a 2012 Macbook Pro, 2.5 GHz Intel Core i5, 16 GB RAM.

__ Small Nodes: 10 Properties Each __
* Write 10000 nodes: 25558ms; 25.558s; 2.556 ms/node; errors: 0.
* Read 10000 nodes: 10365ms; 10.365s; 1.036 ms/node; errors: 0.
* Full Update 10000 nodes: 22895ms; 22.895s; 2.289 ms/node; errors: 0.
* Update single field on 10000 nodes: 9299ms; 9.299s; 0.930 ms/node; errors: 0.

__ Small Nodes: 1000 Properties Each __
* Write 1000 nodes: 133160ms; 133.16s; 133.027 ms/node; errors: 0.
* Read 1000 nodes: 21993ms; 21.993s; 21.971 ms/node; errors: 0.
* Full Update 1000 nodes: 143452ms; 143.452s; 143.309 ms/node; errors: 0.
* Update single field on 1000 nodes: : 1298ms; 1.298s; 1.297 ms/node; errors: 0.

__ Large Nodes: 10000 Properties Each __
* Write 50 nodes: 54347ms; 54.347s; 1065.627 ms/node; errors: 0.
* Read 50 nodes: 12230ms; 12.23s; 239.804 ms/node; errors: 0.
* Full Update 50 nodes: 57784ms; 57.784s; 1133.020 ms/node; errors: 0.
* Update single field on 50 nodes: 185ms; 0.185s; 3.627 ms/node; errors: 0.

## Gun-Mongo

Node Adapter for MongoDB

[npm](https://www.npmjs.com/package/gun-mongo)/[github](https://github.com/sjones6/gun-mongo) by [sjones6](https://github.com/sjones6)

Tests run on a 2012 Macbook Pro, 2.5 GHz Intel Core i5, 16 GB RAM.

__ Small Nodes: 10 Properties Each __
**On an empty collection:**
* Write 10000 nodes: : 7188ms; 7.188s; 0.7188 ms/node.
* Read 10000 nodes: : 4383ms; 4.383s; 0.4383 ms/node.
* Update 10000 nodes: : 8077ms; 8.077s; 0.8077 ms/node.
* Update single field on 10000 nodes: : 7661ms; 7.661s; 0.7661 ms/node.

**On a collection with ~1.4 million documents:**
* Write 10000 nodes: : 9764ms; 9.764s; 0.9764 ms/node.
* Read 10000 nodes: : 4579ms; 4.579s; 0.4579 ms/node.
* Update 10000 nodes: : 8599ms; 8.599s; 0.8599 ms/node.

## Gun-File

A local filesystem adapter.

[npm](https://www.npmjs.com/package/gun-file)/[github](https://github.com/d3x0r/gun-file) by [d3x0r](https://github.com/d3x0r)

**__ Small Nodes: 10 Properties Each __**
* Write 10000 nodes: : 2164ms; 2.164s; 0.216 ms/node; errors: 0.
* Read 10000 nodes: : 3005ms; 3.005s; 0.300 ms/node; errors: 0.
* Update 10000 nodes: : 2404ms; 2.404s; 0.240 ms/node; errors: 0.
* Update single field on 10000 nodes: : 2593ms; 2.593s; 0.259 ms/node; errors: 0.

**__ Medium Nodes: 1000 Properties Each __**
* Write 100 nodes: : 2302ms; 2.302s; 22.792 ms/node; errors: 0.
* Read 100 nodes: : 2411ms; 2.411s; 23.871 ms/node; errors: 0.
* Update 100 nodes: : 1636ms; 1.636s; 16.198 ms/node; errors: 0.
* Update single field on 100 nodes: : 1667ms; 1.667s; 16.505 ms/node; errors: 0.

**__ Large Nodes: 10000 Properties Each __**
* Write 10 nodes: : 2838ms; 2.838s; 258.000 ms/node; errors: 0.
* Read 10 nodes: : 3444ms; 3.444s; 313.091 ms/node; errors: 0.
* Update 10 nodes: : 2097ms; 2.097s; 190.636 ms/node; errors: 0.
* Update single field on 10 nodes: : 2177ms; 2.177s; 197.909 ms/node; errors: 0.

## Gun-Level

An adapter that can be plugged in to LevelDB, and any LevelDown implementations.

[npm](https://www.npmjs.com/package/gun-level)/[github](https://github.com/PsychoLlama/gun-level) by [PsychoLlama](https://github.com/PsychoLlama)

# Running Benchmarks

Currently, please reference this repo: [github](https://github.com/sjones6/gun-adapter-perf).

In the near future, a CLI tool will be released and documented to make this easy.

`gun-flint` is bundled with such a tool already and adapters built on flint can take advantage of this. See flint's [documentation](https://github.com/sjones6/gun-flint/blob/master/docs/PERFORMANCE_TESTING.MD).

# Future work

These benchmarks are by and large put together by the good folks creating the storage adapters. Their benchmarks reflect the system that they ran the tests on. In the future, we plan on being able to run these on a system with the same specs.

We need benchmarks for:

* standard file adapter
* gun-level
* gun-file
* complete suite for gun-mongo

