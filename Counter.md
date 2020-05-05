Implementing other CRDTs on top of GUN's [CRDT](https://gun.eco/distributed/matters.html) can be done in just 12 lines of code, and automatically synchronizes in realtime across a P2P mesh network:

```javascript
Gun.chain.count = function (num) {
  if (typeof num === 'number') {
    this.set(num);
  }
  if (typeof num === 'function') {
    var sum = 0;
    this.map().once(function (val) {
      num(sum += val);
    });
  }
  return this;
};
```

Example usage:

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