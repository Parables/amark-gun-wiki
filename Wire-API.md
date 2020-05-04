# THIS PAGE IS UNFINISHED AND THEREFORE MIGHT BE WRONG

# put

 - **HTTP** POST `peer` `graph`

```bash
curl -X POST \
-H "Content-Type: application/json" \
-d '{
  "soul": {
    "_": {"#": "soul", ">": {"field": 1, "hello": 1}},
    "field": "value",
    "hello": "world!"
  }
}' \
https://gunjs.herokuapp.com/gun
```

#### ack

```
{ok: 1}
```

or

```
{err: "Persistence layer failed"}
```

 - **WS**

```javascript
// paste this into your browser console
var ws = new WebSocket('wss://gunjs.herokuapp.com/gun');
ws.onopen = function(o){ 
  console.log('open', o);
  ws.send(JSON.stringify({
    "headers": {"ws-rid": "id"},
    "body": {
      "soul": {
        "_": {"#": "soul", ">": {"field": 1, "hello": 1}},
        "field": "value",
        "hello": "world!"
      }
    }
  }));
};
ws.onclose = function(c){ console.log('close', c) };
ws.onmessage = function(m){ console.log('message', m) };
ws.onerror = function(e){ console.log('error', e) };
```

#### ack

```
{headers: {"ws-rid": "id"}, body: {ok: 1}}
```

or

```
{headers: {"ws-rid": "id"}, body: {err: "Persistence layer failed"}}
```

# get

 - **HTTP** GET `peer/key`

```bash
curl https://gunjs.herokuapp.com/gun/some/key
```