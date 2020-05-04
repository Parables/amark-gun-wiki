### FAQ <a name="top"></a>
[Is GUN a distributed database?](#is-gun-a-distributed-database)

[Is there a single graph across all users in the network?](#is-there-a-single-graph-across-all-users-in-the-network)

[How do I delete data?](#delete)

[How are conflicts handled?](#how-are-conflicts-handled)

[How is GUN distributed / replicated across peers?](#how-is-gun-distributed--replicated-across-peers)

[What is the difference between Relay Peer and other Peers?](#what-is-the-difference-between-super-peer-and-other-peers)

[What is a soul? What does a node look like?](#what-is-a-soul-what-does-a-node-look-like)

[Can you use SQL like queries? What about pagination and aggregation?](#can-you-use-sql-like-queries-what-about-pagination-and-aggregation)

[How does GUN work underneath?](#how-does-gun-work-underneath)

[How do subscriptions work?](#how-do-subscriptions-work)

[How can I change a user password? (SEA)](#how-can-i-change-a-user-password)

[Does GUN have some form of ACL (access rights)?](#acl)

[Can a lost password be recovered?](#lost_password)

[Can I use GUN timestamps for ordering lists of nodes in UI?](#timestamps)

[Can i use .on with event.off as an alternative for .once?](#event.off)

[Collaborative document or text editing?](#text)

***

#### Is GUN a distributed database?<a name="is-gun-a-distributed-database"></a>

Yes. Data is kept and shared between peers, across the network. 
To give an example: 
- We assume a network configuration of 3 peers with Alice - Bob - Charlie.
- Alice and Charlie do not share a direct connection. 
- When Charlie asks for a record that only Alice has, Alice will return the record to Charlie, because they are connected via Bob.

<a href='FAQ#top'>Back to Top</a>
***

#### Is there a single graph across all users in the network?<a name="is-there-a-single-graph-across-all-users-in-the-network"></a>

Each peer initiates an instance of gun. Depending on application logic, the peers will sync their data into their instance of the graph, with a goal to be consistent across the application that they are connected to. Depending on data structure you may have multiple roots available which can act as a division of data in the instance of the peer. That way, all graphs can start with one root node or multiple root nodes to divide app logic.

<a href='FAQ#top'>Back to Top</a>
***

#### How do I delete data?<a name="delete"></a>

Tombstoning problem in distributed system:
To delete, you need to null data with gun.get('data').put(null), so anytime someone comes online, they know it's to be deleted. If you do not 'tombstone' a peer may overwrite your data, as it thinks your data doesn't exist.

Many a solution has been proposed to deleting in distributed systems, including adding 'expiration dates' on data, but the base issue remains, that if you delete something, whilst not all peers are available at the same time to also do the delete. (e.g. a user is offline) When another peer comes, from his standpoint, his 'old' data is new data to be sent to other clients. And you would receive back the data you have deleted as new data.
When you put null and a peer comes online, he in turn will receive an update to null his 'old' data, making it eventually consistent across the world (and the universe).

<a href='FAQ#top'>Back to Top</a>
***

#### How are conflicts handled?<a name="how-are-conflicts-handled"></a>

Conflicts are handled by the conflict resolution algorithm. see [Hypothetical Amnesia Machine](https://gun.eco/docs/Hypothetical-Amnesia-Machine)

<a href='FAQ#top'>Back to Top</a>
***

#### How is GUN distributed / replicated across peers?<a name="how-is-gun-distributed--replicated-across-peers"></a>

Gun uses 'websockets' underneath. In the future Gun will have a system called AXE that will do smart routing and encrypted transports. (Work is in progress)
Whenever a client 'puts' something in a graph, a message is sent to the [network](https://gun.eco/docs/DAM) and other clients that have subscribed to that data, or relay peers, will pick it up and update their internal state. The guarantee is, that it will be real-time for those who are online at the same time and that even those who are offline, will eventually receive those updates. (this is called eventual consistency [CAP](https://gun.eco/docs/CAP-Theorem))

<a href='FAQ#top'>Back to Top</a>
***

#### What is the difference between Relay Peer and other Peers?<a name="what-is-the-difference-between-super-peer-and-other-peers"></a>

Gun saves data in two ways. Browser - localStorage, Node - RAD (Radix Storage Engine to files).
Due to Browser Limitations, not all data is stored on all clients, persistence at this time is what is left in your localStorage (5mb limit) and what the 'relay peer' (node.js out of necessity right now) saves to files on hard disk. Clients subscribe to the data they need to stay informed on and the relay peer will dispatch data. Once the data is on the client, the client may serve data to other clients as well.

<a href='FAQ#top'>Back to Top</a>
***

#### What is a soul? What does a node look like?<a name="what-is-a-soul-what-does-a-node-look-like"></a>

When using gun to add data to a graph, or to retrieve data from a graph, gun creates metadata, such as a unique identifier for each node that is generated. This is called a soul in 'gun lingo'.
The data structure of a node is as follows, assuming a node Alice with a relation of friend Bob.
```javascript
{
'_':{
    '#': 'eJgh6QkdhlFs8Ed', // <= Soul, UUID of Alice
    '>': 12533221562251  // <= State Vector of this node, used for Conflict Resolution
    },
'friend': {
          'Bob' : 'edDKJj4Fsliojfdf' // reference to the Soul, UUID of the connection
          }
}
```
(This may vary on your data)

<a href='FAQ#top'>Back to Top</a>
***

#### Can you use SQL like queries? What about pagination and aggregation?<a name="can-you-use-sql-like-queries-what-about-pagination-and-aggregation"></a>

##### Pagination and Aggregation

Working with graphs data is quite different from relational databases. Things like pagination, aggregation etc work completely different and require a different data structure design.

##### Querying

Querying is not provided out of the box, but path traversal and map functions can do a lot of the heavy lifting. gun.get(‘index’).map().on(callback) or .once(callback) will execute / traverse all children of that index and given a callback that can check properties against a query will return the data as a query result.

To keep queries performant your data model should keep data accessible in small networks. This can be done in using specialized indices to keep the set of data small. For example, querying for a person with the name 'Alice' in a huge database by following the 'allNodesIndex' will perform poorly, as the gun.get('allNodes').map().once(callback) will execute the callback on each node in the graph, while a 'person' index only iterates over the person nodes in the graph. And if your data includes gender, you may have a 'female' index, reducing the search space further.

If you have arbitrary values, such as ages or numbers or multi-node data points, you can build a binary tree structure for indexing.

Modeling your data for your use case and for your most common queries can greatly increase the performance of your queries. Often the best approach is to start studying graphs and knowledge representation in depth instead of trying to use something like SQL queries in an attempt to keep using old data approaches.

<a href='FAQ#top'>Back to Top</a>
***

#### How does GUN work underneath?<a name="how-does-gun-work-underneath"></a>

GUN is a modular library. [Internal Workings](https://gun.eco/docs/javascript)

The main functional 'layers' of the system are:
1. Conflict Resolution Algorithm ([Hypothetical Amnesia Machine](https://gun.eco/docs/Hypothetical-Amnesia-Machine))
2. Graph Data Structure (In-Memory Graph)
3. Signing, Encryption, Authentication ([SEA](https://gun.eco/docs/SEA), [Users](https://gun.eco/docs/User),[Security](https://gun.eco/docs/Auth))
4. Data Storage (We call [RAD](https://gun.eco/docs/Radisk), supports Browser/Files/S3/IPFS)
5. Network ([DAM](https://gun.eco/docs/DAM), uses websockets at this point)
6. _WIP_ Routing and Sharding ([AXE](https://gun.eco/docs/AXE))
7. _WIP_ App Integration for global distribution (includes decentralized OpenAuth service)

Each component builds on top of the others and can be used based on the needs of the developer.

<a href='FAQ#top'>Back to Top</a>
***

#### How do subscriptions work?<a name="how-do-subscriptions-work"></a>

Subscriptions are like event hooks that each instance of GUN can add at specific points. 
When data is subscribed to by using gun.get('myKey').on(callback, change-only-flag) [ON](https://gun.eco/docs/API#on), peers indicate to their Storage Adapter, that the data of 'myKey' should be stored when seen, and if seen, to then fire the callback provided. 

To subscribe or not subscribe from data, does not mean, that the network will reject or stop propagating data requested by other peers via your peer instance. The network layer operates independent from the storage layer and all peers participate in data relay regardless of subscriptions.

<a href='FAQ#top'>Back to Top</a>
***

#### How can I change a user password? (SEA)<a name="how-can-i-change-a-user-password"></a>

```javascript
user.auth(alias, passphrase, callback, { change: 'new-pass-value' })
```

<a href='FAQ#top'>Back to Top</a>
***

#### Does GUN have some form of ACL (access rights)?<a name="acl"></a>

Not yet. But you can add ACL yourself.
If you want an example of how to implement ACL, check out this short demo video:
[https://youtu.be/ZiELAFqNSLQ](https://youtu.be/ZiELAFqNSLQ)
And peek at this ~150 lines of HTML/CSS/JS that implements it:
[https://gist.github.com/amark/44b8110a3c848917d6c738f9c3a36e24](https://gist.github.com/amark/44b8110a3c848917d6c738f9c3a36e24)

To restrict a group of data: (decentralized method)

One thing you could do (to keep it decentralized / P2P) is to create a user (see tutorial above), make that user be an organization (that maybe only you have the login to), and then the organization can specify OTHER users (actual user profiles) to be admins.
Now your app's users all "trust" the public key of the organization, and then run fully decentralized logic where they reject (which can happen automatically in SEA) data from anybody that isn't an admin, even as admins change/add/remove over time from the organization.

To add a bit more detail:
You would save an ACL table on the organization account. In that ACL table have the path (or soul) of the data that will have others write permission on it. Store on that path/soul in the ACL table, a table of pubkeys allowed to write to that data. Now write wrapper extension around a read function (or do this at a wire adapter level) that checks the ACL table for that record (actually, on 2nd thought, maybe easier to just have the organization reference the ACL table on the record itself, rather than as a separate record), then perform a read from those other pubkeys on the matching path (+org name, depending on your schema) and merge (ideally with HAM) the results before passing back the data.
Achieve this in 2 ways either wire adapter that sniffs for ACL schema structures and lets matching writes pass through the firewall or chain extension that upon read of this item, just checks/does an extra lookup and merges results through Ham before returning.


To restrict a group of data: (centralized method)

You are able to write middleware hooks to restrict data - although note, they're usually speaking "the wire spec" but it isn't that bad, for example, check this sample out:

https://github.com/zrrrzzt/bullet-catcher

https://github.com/zrrrzzt/gun-restrict-examples

Again, this is more "centralized" logic which is perfectly possible in GUN, the community just mostly focuses around building tooling for P2P/decentralized logic.


<a href='FAQ#top'>Back to Top</a>
***

#### Can a lost password be recovered?<a name="lost_password"></a>

Because of the p2p design of GUN, there is no server storing the password, so a lost password can not be recovered. That is indeed an issue, as users tend to lose passwords.

**One** way to solve it is to have a separate server for password recovery (bad design in p2p).

**Or** 3-Friend-Factor Authentication. The user may choose 3 friends. Given the public keys of the friends generate 3 different AES keys with SEA.secret. Encrypt a recovery key with PBKDF2 extended lexically concatenated 3 keys.
Then using ECDH share each AES key with a friend. When you want to recover the account generate a throw-away ECDH key and ask 3 friends to visit a recovery page. (Out of Band, increase of security)
They approve, ECDH encrypt their original AES piece, which gets sent back to the user. Using their information you decrypt your recovery key via PBKDF2 / lexical concatenate.
There may be variations to this approach that are more secure.

**Or** upon account creation show something like a QR code, containing the password, on the screen and have the user store it in a safe place (vulnerable because user might take a picture, etc).

<a href='FAQ#top'>Back to Top</a>
***
#### Can I use GUN timestamps for ordering lists of nodes in UI? <a name="timestamps"></a>

GUN already stores vector/timestamp for every item, you can do Gun.state.is(node, 'key') to get it (node needs to be data node), if you want to generate a timestamp (using time sync, if gun/nts.js is enabled) do Gun.state().

<a href='FAQ#top'>Back to Top</a>


***
#### Can i use .on with event.off as an alternative for .once? <a name="event.off"></a>

Yes! But .on() may give partials. So maybe consider adding a schema check on the object, that way you only event.off() after you know you have the full object. Here's a small extension for it:
```
Gun.chain.onceComment = function(cb){
  return this.on(function(data, key, msg, event){
    if(!isValidComment(data)){ return }
    event.off();
    cb(data, key);
  });
}
```
Now gun.get('comment').onceComment(cb) only fires once with valid comment 

<a href='FAQ#top'>Back to Top</a>

-----------------

A response to ["A Local-first Database"](https://jaredforsyth.com/posts/local-first-database-gun-js/) while it was in draft. Below sections cover a lot of subject areas and worth being added here until that article finalizes or this information is merged into the FAQ.

This was taken from the chat room:

Correctness:

- "last write wins" is a nice summary, tho no such thing exists in a distributed system so may confuse some people or (what I'm concerned about) mislead them to assume incorrect classification.

- Yes, consistency verification built in, our famous "Holy Grail" test is pretty cool at demonstrating this... wow 5 years ago: 1 of the first and most important things GUN does https://youtu.be/nG3yz1FsAxc?t=1800 

- Preserve intent: The "CRDT" answer here should actually be moved up & replace the "last write win". Preserving intent is viewed as a good thing, and you say that GUN does do it, but at this layer GUN does not (lol, most of my comments will be "you said GUN does not do this, but it does", this is 1 of the few where I'm saying opposite, "you oversold GUN!"), but GUN is designed to allow other CRDTs run on top that DO preserve intent for specific use-cases, for example see a counter CRDT implemented in 12 lines of code (this should be mind boggling for anyone familiar with CRDTs) https://gun.eco/docs/Counter and remember, you additionally get for free all the networking mechanics built into GUN which for most CRDT implementations ignore.

Cost:
 - I'm not sure if I understand section. My comment would be just that "GUN peers can store only the data they want, or all data." (ie: Blockchain nodes must store all data vs. Master-Replica databases store only portions. GUN can do either)
Network Transfer: Is the Big O(N) referencing total data or just the diff of data?
- :open_mouth: what 12 tests failing?? Travis shows me 126 passing, 10 pending, 0 failures. Where did 130 come from!?
- Committers: true core is pretty quiet & small, lots of modules collaborated on by many tho, they're just not in core.

Flexibility:
 - Up for mentioning `.set(` in the "no Arrays"? Then + `.open(`  loads nested data for you. ( https://gun.eco/docs/API#open )
 - Used heavily with other _storage engines_ (indexedDB, file systems, blob/object stores, LevelDB, etc.) but yeah, dumping into MySQL/Mongo/etc. pretty pointless, either lose GUN perks or lose other DB's perks.
 - :P nothing stops you from running GUN peer inside Dropbox/etc., haha, I actually do this. Dropbox then syncs the persisted GUN data to other machine. Tho yes, they'll conflict if both run at same time.
 - Encryption is supported, but not automatic! 1 of the few "you oversold GUN" in my list. Data under your user graph is _signed_ but not encrypted unless you (or app dev) remembers to.
 - Multiuser collab, yes majorly supported, please see the list of apps I mentioned in my first tag to you (Iris, old DTube PMs, Linked In example, Cale's video tutorial, etc.)
 - Hmm, `LWW-Map` actually not too bad, I know I was just complaining about that above, but this seemed more accurate in thsi context

#### Collaborative document or text editing?<a name="text"></a>

 - Collaborative text been built on top a bunch of times :D check out (A) our cartoon explainer on this stuff https://gun.eco/explainers/school/class.html , (B) @nmaro's CRDT library on top of GUN for this https://github.com/nmaro/crdt-continuous-sequence (C) demo of it in a wiki editor here https://gun-preview.nmaro.now.sh/?id=collabtext (tho just noticed terribly slow currently, was fast few months ago) and (D) even early prototypes I demoed in 2016 ( https://www.youtube.com/watch?v=rci89p0o2wQ ) and (E) been working on a project in examples/docs.html that takes a slightly different approach that is more scalable but "cheats" a little, see my explanation at https://twitter.com/marknadal/status/1216976371617947649

Production Ready:
 - "no sync without central server" would highly confuse people GUN is centralized after couple paragraph before saying it is p2p. I think what should clarify is (A) browser-to-browser sync works if you have the WebRTC module added (B) without the WebRTC module you'd need to be connected to at least 1 of many _relay peers_ (again: not necessary if you are WebRTC connected, but WebRTC fails a lot so relay peers proxy *without* centralizing) to get live updates. For example of this, see https://www.youtube.com/watch?v=-FN_J3etdvY (wow, yet another 4~5 year old video!).
 - https://github.com/amark/gun/wiki/FAQ#lost_password . And if possible, clarify that private graphs are allowed. Custom auth systems being built now, like Iris, and the new Houses system Cale/etc. working on. So we're very much designing for this. :) 

Other Notes:
 - valid point that all gun peers treat all protocol operations as allowed. I feel you present this in a negative light "oh you'll get spammed", if all peers enforce app-specific rules the whole network will be broken, walled gardened, incompatible, and siloed off, not useful to anyone. Now, that said (and my strong words for it!), many of the most popular apps are already doing this anyways, notabug (p2p reddit, quite popular, beware of content!) rejects any update not matching its schema (go1dfish even wrote custom typescript ports of GUN), and https://github.com/zrrrzzt/bullet-catcher frequently referenced by community on how to add custom rules.
 - I think as others mentioned, IndexedDB very much supported https://gun.eco/docs/RAD#install