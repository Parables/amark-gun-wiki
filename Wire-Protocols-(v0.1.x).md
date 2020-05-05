## Peer

First, we have the base URL a GUN peer is running on. Take these as examples:

- [https://gunjs.herokuapp.com/gun](https://gunjs.herokuapp.com/gun)
- [http://localhost:8080/gun](http://localhost:8080/gun)
- [wss://gunjs.herokuapp.com/gun](wss://gunjs.herokuapp.com/gun)
- [ws://localhost:8080/gun](ws://localhost:8080/gun)

Currently only HTTP, JSONP, and WebSockets are implemented. It is not necessary to host a GUN server on '/gun', it could be on any path you want - including the root domain. The only important thing is that your users load your website and that developers know where to connect GUN peers.

## Security

Obviously use https or wss if you care about preventing man in the middle attacks or snooping. Beyond this, GUN does not know your security requirements and makes no assumption on how they might be structured. If you want to enforce permissions, you should pass HTTP style Authorization Headers to an endpoint you control and check for security before relaying it to GUN. Basically, do not expose GUN directly unless you want your data to be publicly editable.

## Stateless

By default GUN's wire protocol is stateless, emulating HTTP. However, all realtime push notifications of changes happen over a stateful connection. This is achieved by passing an HTTP style `gun-sid` header with some unique ID. This hybrid model gives us the low bandwidth high throughput when available, but the safety and convenience of full transfers when needed.

## Load

Assuming you have the base URL of your peer, just append the key as the `pathname`. 
 - **HTTP** GET `peer` [/my/key/here](/my/key/here), _ex. [https://gunjs.herokuapp.com/gun/example/angular/data](https://gunjs.herokuapp.com/gun/example/angular/data)_
```json
{
  "_": {
    "#": "vsTA2AUwkkE56kbLRdekWmN6",
    ">": {
      "awesome": 1422485319074,
      "cool": 1422166957059
    }
  },
  "awesome": "sauce",
  "cool": "beans"
}
```
 - **WS** `peer` SEND `'{"url": {"pathname": "/my/key/here"}, "headers": {"ws-rid": "random"} }'`, _ex._
```javascript
// paste this into your browser console
var ws = new WebSocket('wss://gunjs.herokuapp.com/gun');
ws.onopen = function(o){ 
  console.log('open', o);
  ws.send(JSON.stringify({"url": {"pathname": "/example/angular/data"}}));
};
ws.onclose = function(c){ console.log('close', c) };
ws.onmessage = function(m){ console.log('message', m) };
ws.onerror = function(e){ console.log('error', e) };
```
```json
{
  "headers": {"Content-Type": "application/json", "ws-rid": "random"},
  "body": {
    "_": {
      "#": "vsTA2AUwkkE56kbLRdekWmN6",
      ">": {
        "awesome": 1422485319074,
        "cool": 1422166957059
      }
    },
    "awesome": "sauce",
    "cool": "beans"
  }
}
```
 - JSONP, same as the HTTP.

If the peer does not have a node associated with that key, it will reply with a `null` body. This is never an error. Additionally, this does not mean that the data does not exist, just simply that this peer has no recollection of it - it might exist on another peer.

If you do not know the key or want to load a node by its soul, change the value of the pathname to a URI encoded component of '[?#=soul](?#=soul)', _ex. [https://gunjs.herokuapp.com/gun?%23=vsTA2AUwkkE56kbLRdekWmN6](https://gunjs.herokuapp.com/gun?%23=vsTA2AUwkkE56kbLRdekWmN6)_.

## Set

To set data over the wire, you must send the delta update encapsulated within a graph which allows for batch updates. The soul in the following examples were shortened for sake of clarity, where we are updating a single field of "yay" to the value of "lol" on a single node "pfNh":

 - **HTTP** POST `peer` `{"pfNh":{"_":{"#":"pfNh",">":{"yay":1426023776027}},"yay":"lol"}}`, _ex. [https://gunjs.herokuapp.com/gun](https://gunjs.herokuapp.com)_ `{"pfNh":{"_":{"#":"pfNh",">":{"yay":1426023776027}},"yay":"lol"}}`
 - **WS**, `peer` SEND `{"headers": {"ws-rid": "random"}, "body": {"SOUL": {"_":{"#":"SOUL",">":{"field": STATE }}, "field": "value"} } }`, _ex._
```javascript
// paste this into your browser console
var ws = new WebSocket('wss://gunjs.herokuapp.com/gun');
ws.onopen = function(o){ 
  console.log('open', o);
  ws.send(JSON.stringify({"body":{"pfNh":{"_":{"#":"pfNh",">":{"yay":1426024314477}},"yay":"lol"}}}));
};
ws.onclose = function(c){ console.log('close', c) };
ws.onmessage = function(m){ console.log('message', m) };
ws.onerror = function(e){ console.log('error', e) };
```
 - JSONP, **...**

## Key

You can tell other peers to remember a key and the soul it references, but there is no guarantee that other peers will accept that request. There are no conflict resolution guarantees on keys either, meaning you could overwrite a key. If you want to mitigate these situations, you need handle [security and permissions](#security) yourself. Other than that, here is the way you _would_ request peers to remember a key's association with a node:
 - **HTTP** POST `peer` [/my/key/here](/my/key/here) `{"#": "SOUL"}`, _ex. [https://gunjs.herokuapp.com/gun/example/angular/data](https://gunjs.herokuapp.com/gun/example/angular/data) `{"#": "vsTA2AUwkkE56kbLRdekWmN6"}`_
 - **WS** `peer` SEND `'{"url": {"pathname": "/my/key/here"}, "headers": {"ws-rid": "random"}, "body": {"#": "SOUL"} }'`, _ex._
```javascript
// paste this into your browser console
var ws = new WebSocket('wss://gunjs.herokuapp.com/gun');
ws.onopen = function(o){ 
  console.log('open', o);com
  ws.send(JSON.stringify({"url": {"pathname": "/example/test/make/key"}, "body": {"#": "vsTA2AUwkkE56kbLRdekWmN6"} }));
};
ws.onclose = function(c){ console.log('close', c) };
ws.onmessage = function(m){ console.log('message', m) };
ws.onerror = function(e){ console.log('error', e) };
```
 - JSONP, **...**

And then view [http://gunjs.herokuapp.com/gun/example/test/make/key](http://gunjs.herokuapp.com/gun/example/test/make/key) in your browser, and you'll see the association was made (although other people have probably already done this before).