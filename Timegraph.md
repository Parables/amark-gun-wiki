# Timegraph .time()
> Warning: The Timegraph extension isn't required by default, you would need to include at "gun/lib/time.js"

A Timegraph allows you to store and retrieve data that is stored at a relative time.


## .time( function(data, key, time) )

Subscribes to all future events that occur on the Timegraph
* `data` : The data in the node retrieved
* `key`  : The key name of the node retrieved
* `time` : The time that the node was uploaded

## .time( function(data, key, time), num)
Subscribes to all future events that occur on the Timegraph and retrieve a specified number of old events
* `data` : The data in the node retrieved
* `key`  : The key name of the node retrieved
* `time` : The time that the node was uploaded
* `num`  : The number of past nodes to retrieve

## .time( data )
Pushes data to a Timegraph with it's time set to `Gun.state()`'s time

## Example

You will need a reference to a node to store a timegraph in.
```javascript
var node = Gun.get("<Your Node Here>");
```
> Warning: As of now a Timegraph can only be stored inside one of the top level nodes. Eg. You can't do 'gun.get().get()'

Once You have your node you can use any of the **.time()**  methods on it.

# Example Chat
```javascript
let chatroom = gun.get('chatroom');

chatroom.time((data, key, time)=>{//listen setup
   gun.get(data['#']).once((d,id)=>{
      console.log(d.message);
   });
},20);//number display when loaded and time is trigger here if push.
//..
$('#enterchat').on("keyup",function(e){
   e = e || window.event;
   if(e.keyCode == 13){
      ...
      chatroom.time({alias:'alias',message:'text'}); //push here to time update listen set from top.
   }
})
//..
```
Example Input time feeds. http://jsbin.com/vazazelinu/edit?js,console,output

