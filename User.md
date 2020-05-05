This page will explain [SEA](./SEA)'s `user` API as well as explain architectural decisions underneath each method for security purposes. 

Assume:

```javascript
gun = Gun();
user = gun.user(); 
```

## User.create

Creates a new user and calls callback upon completion.

### Syntax

```
user.create(alias, pass, cb, opt)
```

### Parameters

alias (string) - Username or Alias which can be used to find a user. 

pass (string) - Passphrase that will be extended with PBKDF2 to make it a secure way to login.

cb(ack) (function) - Callback that is to be called upon creation of the user.

opt (object) - Option Object containing options for creation. (In gun options are added at end of syntax. opt is rarely used, hence is added at the end.)

### Return via Callback

On successful creation of the user the callback is called with an 'ack' object. (acknowledgment)
```
//calls cb(ack) where ack is an object as below
{
    ok: 0,
    pub: 'fe4...ee3' //public key of the user that was just created
}
```

On failure you will receive an 'ack' object. (acknowledgment)
```
//calls cb(ack) where ack is an object as below
{
    err: // with one of 2 possible errors described below
}
```

If user is already being created:
"User is already being created or authenticated!"

If user already exists:
"User already created!"

### Unexpected Behavior

There is no enforcement of uniqueness for the alias. If this is required for your application, you need to enforce it with a gun.get("~@alias").once(callback) to check if user already exists.

## User.auth

Authenticates a user, previously created via User.create.

### Syntax

```
user.auth(alias, pass, cb, opt)
```

### Parameters

alias (string) - Username or Alias which can be used to find a user.

pass (string) - Passphrase for the user

cb(userReference) (function) - Callback that is to be called upon authentication of the user.

opt (object) - Option Object containing options for authentiaction. (In gun options are added at end of syntax. opt is rarely used, hence is added at the end.)

### Return via Callback

```
// on success calls callback with a reference to the gun user
// cb(at) where at is an object as below
{
    ack: 2,
    back: object, //reference to the root object (internal)
    get: "~publicKeyOfUser",
    gun: object, //gun root (internal)
    id: 6, //id of the node in graph (internal)
    on: function onto(),
    opt: object, //uuid function object (internal)
    put: object, //object containing pub, alias and epub of the user
    root: object, //gun root reference (internal)
    sea: object, //object containing keys of the user
    soul: "~publicKeyOfUser",
    tag: object //gun in and out reference (internal)
}
// on failure callback is called cb(ack) where ack is as below
{
    err: 'Wrong user or password.'
}
```

### Unexpected Behavior

Please comment on anything you may encounter.

## User.pair (Deprecated)

Returns the key pair in the form of an object as below.

**NOTE: Deprecated due to use of  MetaMask/Party/etc. private keys may not be exposed.  You will get the following message:**

```
user.pair() IS DEPRECATED AND WILL BE DELETED!!!
```

### Syntax

```
var pair = user.pair();
```

### Parameters

None

### Return as object

Returns the key pairs of the current authenticated user as an object as below

```
{
    epriv: EncryptionPrivateKey,
    epub: EncryptionPublicKey,
    priv: PrivateKey,
    pub: PublicKey
}
```
## User.is

To check if you are currently logged in...

```
if (user.is) {
   console.log('You are logged in');
} else {
   console.log('You are not logged in');
}
```
If the user is not logged in it will return "undefined".

## User.leave

Log out currently authenticated user. Parameters are unused in the current implementation.

### Syntax

```
user.leave(opt, cb)
```

### Parameters

opt (object) - unused in current implementation.

cb (function) - unused in current implementation.

### Returns

Returns a reference to the gun root chain. 

### Unexpected Behavior

There is no callback called at this time to confirm successful logging out. 
Personal recommendation of the author (@dletta) of this part of the documentation is to check if user._.sea exists after leave is called. If it no longer contains a keypair, you are successfully logged out. It should be 'undefined' and a truth check would come back false.

## User.delete

Deletes a user from the current gun instance and propagates the delete to other peers.

### Syntax

```
user.delete(alias, pass, cb)
```

### Parameters

alias (string) - Username or alias.

pass (string) - Passphrase for the user.

cb({ok: 0}) (function) - Callback that is called when the user was successfully deleted.

### Return via Callback

Callback is called with an object:
```
{
    ok:0
}
```

### Unexpected Behavior

If an error occurs, the Error is not sent via the callback, but rather logged to the console. This is unusual.

User data will be null'd out. It will most likely appear as if the user never existed.

## User.recall

Recall saves a users credentials in [sessionStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/sessionStorage) of the browser. As long as the tab of your app is not closed the user stays logged in, even through page refreshes and reloads.

### Syntax

```
user.recall(opt, cb)
```

### Parameters

opt (object) - option object 
If you want to use browser sessionStorage to allow users to stay logged in as long as the session is open, set opt.sessionStorage to true

cb(userReference) (function) - internally the callback is passed on to the user.auth function to logged the user back in. Refer to user.auth for callback documentation.

### Unexpected Behavior

Please let us know if you find anything.

## User.trust

Still in development, might be deleted or changed.
Intent of this function was to allow users to trust other users, which would allow them to read/write data from your user graph.

### Syntax

```
user.get('name').trust(user) //trust him with my name
```

### Parameters

user (user reference) - a user you trust.

## User.grant

Still in development, might be deleted or changed.
Intent of this function is to grant the right to read your encrypted data. Imagine having some data you want to grant access to, then you can do gun.user().get('address').grant(acmeCorp).

### Syntax

```
user.get('address').grant(Mark) // allow Mark to read my address
```

### Parameters

user (user reference) - a user you grant access to.

## User.secret

Still in development, might be deleted or changed.
Intent of this function is to save encrypted data to your user graph only trusted users can read.

### Syntax

```
user.get('mysecrets').secret(string, callback)
```

## Getting a user via alias

### Syntax

```
gun.get('~@alias').once((data, key)=>{});
//data is a gun node object and key is a string with ~ before alias
{
    "_": aliasObject
   "~pubKeyOfUser": pubKeyObject
}
```

## Getting a user via gun.user

If you know a users publicKey you can get his user graph and see any unencrypted data he may have stored there.

### Syntax

```
gun.user(publicKey).once(console.log)
```


## SEA

### Security

See [SEA](./SEA) for algorithm choices.

Essentially, it works like this:

```javascript
//<script src="https://cdn.jsdelivr.net/npm/gun/gun.js"></script>
//<script src="https://cdn.jsdelivr.net/npm/gun/sea.js"></script>
var sea = Gun.SEA;
var pair = await sea.pair(); // generate a new key pair
console.log(pair);
var alias = "alice"
var pass = "secret";
var salt = 1; // random
var proof = await sea.work(alias, pass); // don't do this! (pass, salt) instead!
var auth = await sea.encrypt(pair, proof);
console.log(auth); // data saved in a cryptographically linked user graph
// now on another machine...
var login = await sea.work(alias, pass);
var keys = await sea.decrypt(auth, login); // encrypted auth loaded from graph
console.log(keys); // equal to the original key pair
```