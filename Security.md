> This page needs contributions! These were notes taken during a webcast, and need to be expanded into documentation. Only the last 20 min (of 2 hour 30min) of the webcast was recorded: https://drive.google.com/open?id=19ABZE6rKwIWJCNuGqKrO_pzsUxHGGQJ8 (4GB)

To learn how to use GUN's security system, read the ["How To"](Auth) guide on authorization or the [SEA](SEA) documentation. This article is intended to explain the **architecture** for decentralized security.

 - Privacy: Cypher
 - Authenticity: Signatures

To load gun and sea.
```javascript
var Gun = require('gun');
require('gun/sea');
var gun = Gun();
var user = gun.user()
```

## Individual 1-1 security model (Diffie-Hellman)
Working [sample](https://gist.github.com/amark/7dceae874a20878fdb9e2a8eed109bb5).

When creating and authenticating an user, will be created an user public key.

```javascript
user.create('me', 'secret')
user.auth('me', 'secret')
console.log(user.is.pub); /// here we see the user pubkey
```

## N-N Group security models

### The two core models

| WRITE   | READ    | Description
|---------|---------|----------------
| Private | Public  | Admin group writes new data for public consumption
| Private | Private | Read/write only for authorized group members

#### Other two models that not so important to security:

| WRITE   | READ    | Description
|---------|---------|----------------
| Public  | Public  | Doesn’t need encryption
| Public  | Private | Not generally useful (inbox mode?)

Messaging models:
 - Aggregator: determines *read/write* permissions for group members
 - Writer: can *write* data to system
 - Reader: can *read* data from system

Fine grained access possible with group level keys shared at different levels of the hierarchy.

Avoid models where keys are completely public as this is not secure.

Specifics of implementation may depend on data model being used for application.

Assuming group of good actors with shared key.
When user leaves:

1) (most secure) change decryption key and re-encrypt ALL data

2) (centralized/federated/p2p) user still has access to any offline data from before leaving
 -  a) Change decryption key and use for new data
 -  b) Must manage multiple *dec* keys (old key and new key)

3) (small groups) hope people abandon old keys and don’t worry about it

Aggregator model example:

 - (A) Aggregator: Alice, Bob
 - (C) Customers: Carl, Dave, Ed
 - (O) Others: Zach, Xavian

Must create a gun user for each user.
Example: 
```javascript
var user = gun.user();
user.create('alice', 'password', function(ack) {
  console.log('User created: ', ack.pub);
});
```

Public keys are searchable by username.

**Never** use username as user id (non-secure). 

Usernames are not guaranteed to be unique, so doing this increases possibility of usin incorrect keys for encryption.

Logged in users have access to personal private key (`gun.user()`).

If you need access to a user who isn’t logged in, you can iterate through gun users in local storage.

Preferred for each user to have a list of trusted users with public keys (friends list). This can be combined with app level routing to find public keys and save to trust table as needed.

Users can be searched by public key with convenience method (`gun.user(<pubkey>)`).

Alice and Bob need a reciprocal trust relationship


Google “diffie hellman colors” -> Explains how both parties can get to the same shared secret.


---
---
Mark's random typings during the webcast, need to be converted/cleaned up:
---
```
Security

Privacy - Cypher
Authenticity - Signatures

Individual
1-1 // Diffie-Hellman
Group

Talk about:
A) Group - private write, public read
	example: websites, wiki

B) Group - private write, private read
	
Group - public write, private read
public write, public read
```
---
```
GUN Developers Group

Write access

GROUPS: private write, public read 


r(w(A, B, C), D, E)

A (SERVER) B (SERVER) C (SERVER) D (SERVER) E (SERVER)

E -> "server logic" A || B || C || D || E => NO

E => yes I want (D) to have write access.
A, B, C, D are still going to say "NO".


// common shared path
gun.get('code').get('gun').get('core')

A = w(A, B, C)
B = w(A, B, C)

A, B, C, D = have the same world
```
---
```
A, B, C, D, E

GROUPS: private write, private read
Aggregator = gathers things together = a(param)

M) groups of less than 100
N) groups more than 100


Facebook followers
(you have to be my friend to read my posts)

Data schema = Append-only Log (chat room)

X) Facebook messenger (group chat)
(1-1, N-N)

Y) Facebook group
(Aggregator-N)

a(A, B)
r(A, B, C, D, E)


Shared Secret **Dec**ipher Decryption Key => A B C D E
you cannot post the key online

Assumption: A B C D E are good actors.

1 key (decipher)

m(dec)
if a user leaves
	technically if you want the system to be secure
		 moving forward into the future

		 1. you need to generate a new DEC and
		 reencrypt everything with the DEC

		 2. centralized system, a federated system, p2p system
		 if the user who left
		 	had access to old data
		 they could have an off"chain" copy of the data
		 read(old < dateTheyLeave)
		 generate a new DEC and use it
		 to encrypt everything moving forward
		 multi-DEC key management

		 3. just hope and trust they give up
		 the DEC and thow it away and do no share it.

		 (M) less than 100 ~= (3)
		 (N) option (3) is not an option

```
---
```
GROUP private write, private read AGGREGATOR MODEL

(A) ACME = Alice, Bob

(C) Customers = Carl, Dave, Ed

(O) Others = Zach, and Xavian

ACME = (A, B)

Alice gun.create('alice', ...)
Bob gun.create('bob', ...)

NEVER NEVER NEVER ever use the username as the user ID for security.

var whoItrust = await gun.user(thePublicKeyItrust);
var alias = await whoItrust.get('alias').then();
alice.get('trust').get(alias).put(whoItrust);

Alice -> trust Bob
Bob -> trust Alice


Alice: gun.get('acme').trust(bob);
Bob: gun.get('acme').trust(alice);


gun.get('acme').get('address').once(x => {
	console.log(x);	
});



Conflict Resolution Algorithm
HAM

AlicebrowserData -> HAM
BobbrowserData -> HAM
Alice + Bob => realtimeData

HAM(HAM(alice) + HAM(bob)) = secure data.

IN MEMORY (current view)

ACME address === the latest update by Bob or Alice.

Gun.chain.read = function(cb){
	var aliceACME = await gun.get('acme').get('address').then();
	var bobACME = await gun.get('acme').get('address').then();
	var merge = Gun.HAM(aliceACME, aliceACME, vector, vector);
	cb(merge);
}

gun.get('acme').get('address').read(function(merge){

})

var gunRef = getgetgetget
var gunRef2 = getgetgetget

var ACME = gun.union(gunRef, gunRef2).on(function())



Alice -> diffie hellman -> Bob
Bob -> diffie hellman -> Alice


Alice: var Adec = await Gun.SEA.secret(bob.epub, user.pair());
Bob: var Bdec = await Gun.SEA.secret(alice.epub, user.pair());
Adec === Bdec

shared decipher key

1-1 Alice & Bob

gun.get('acme').get('address').put(cipher("123 Market st.", DEC))
gun.get('acme').get('address').read(x => {
	console.log(decipher(x, DEC));
})


alice and bob
var acme = await Gun.SEA.pair();
alice.get('iAmBossOf').get('acme').put(await Gun.SEA.encrypt(acme.pair(), DEC));


private write private read ACME data
private write public read ACME data



private write GROUP read
A, B, Mark
A diffiehellman B -> entity keypairs
(A || B) diffiehellman Mark -> entity keypairs

A B, Mark 



var acme = gun.user().auth(acme);
acme.get('read').get('customer').set(carl)
acme.get('read').get('customer').set(dave)
acme.get('read').get('customer').set(ed);

acme.get('read').get('specialcustomers').set(ed);
acme.get('read').get('specialcustomers').set(dave);

acme.get('address').put("123 Market St.").grant('customer')
{
	addres: '123 market st', // customer
	founded: 1993, // customer
	something: 'hello', // specialcustomer
}

THe customer group (defined by ACME entity, aggregated of Alice and Bob) has a DEC through ACME diffiehellman to each customer.





acme.get('rules').put(FirebaseRuleString);
|| acme.get('rules').put(ACL);
|| acme.get('rules').put(myOwnRule);

Customers: var rule = await acme.get('rules');
var command = parseACL(rule);
gun.on('secure', function(msg){
	var to = this.to;
	if(command(msg)){
		to.next(msg);
	}
	throw error("The Rule Command says NO!!!");
});



C group.get('acme').open(x => {
	decerypt(x, dec)
	console.log(x);
})
```
---