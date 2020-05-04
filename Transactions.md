** Needs Words **

one way to accomplish transactions... or action-reaction using a graph database to track the transaction.

## Method 1) 

This is a transaction to login an account.

create a record like `{ name : 'user', password:'password'}` and put it in the graph at a place the server has mapped.


```
// on a client.... 

var db = gun.get( 'dbRoot' );

db.get( "signup" ).get( "ack" ). map( serverAckSignup )
function serverAckSignup( ack, field ) {
   // field will = signup...
   if( ack.user_id ) ...  // do soemthing to save my credientials for later proof?
}


db.get( 'signup').put( { name : 'user', password:'password' } ):
```

The server will receive the signup record from the client, and then be able to process that, and add a `user_id` to the transaction, which will be sent to the client, which should have the transaction they started mapped.

```
// on a server
var db = gun.get( 'dbRoot' );
db.get( "signup" ).map( handleSignupEvent )
function handleSignupEvent( signup, field ) {
   // field will = signup...
   this.get( "ack" ).put( { user_id : "some ID", ... } );
}
```


when the signup is put, the server gets it, puts an ack(user_id), that then the client gets.
viola - transaction.


Previously completed transactions can be checked for completion and steps ignored....

```
if( !( "user_id" in signup ) ) {
   // then do the work, otherwise the job was already done.
}
```


## Method 2)

Mark shared an example a long time ago that was more nested... like

```
{ data : 123 }

{ result : { data : 123 } }

{ data : 123,
  result : { data : 123 
      , result : { data : 123 }
  }
}

which works  more like 
gun.db
   .map().on( (val)=>{
   } ).map().on( (val)=>{
   } ).map().on( (val)=>{
   } ).map().on( (val)=>{
   } );

```
