# Daisy-chain Ad-hoc Mesh-network

DAM is GUN's default transport layer abstraction and P2P networking algorithm. Decentralized communications can be rather chatty, so it helps "hold back" an excess flood of messages by improving upon the base brute force method.

This page focuses on both its architectural and API. DAM is the transport counterpart to RAD, the storage engine.

**DAM has not yet been cartoonized**, we apologize, if you aren't deeply technical, please come back later (the below gif is NOT an illustration of DAM, but its less efficient predecessor).

## API

DAM's API allows for swapping in different transport layers (websocket, WebRTC, etc.) in the same way RAD lets you swap out storage layers (disk, S3, IPFS, etc.).

Most of the API deals with communicating one-off messages to only neighbor peers or even individual neighbor peers in order to coordinate their immediate efficiency. If direct peer connections are optimized you should get the emergent result of the greater mesh and daisy-chaining being more efficient.

For example, peers Alice and Bob might both support a binary transport, even if other peers do not. So DAM can be used to create a shared state machine between the Alice-Bob neighbor connection that can dynamically upgrade or downgrade their transport system.

 ### Adding Peers

If you are connected to another GUN peer, you can ask them to connect to another peer, for example:

```javascript
var gun = Gun('http://localhost:8765/gun'); // Connect to a local peer
var mesh = gun.back('opt.mesh'); // DAM;
// Ask local peer to connect to another peer.
mesh.say({dam: 'opt', opt: {peers: 'https://guntest.herokuapp.com/gun'}});
```

Note, just because you ask for the peer to connect, does not mean they have to. To prevent abuse, the asked peer may only connect up to 6 other peers on a first-come first-serve basis, and may quickly drop peers if they go offline for longer than a few minutes and stop retrying connections until asked again.

 ### Neighbor Messages

From chat: yeah, DAM has a feature for this... you'll find it on var dam = gun.back('opt.mesh'); then you can send messages via gun's internal event emitter if you know how, or mesh.say({dam: 'myModule', ...}) for all (directly) connected peers or optionally ...}, peer) to specific peer. NOW IT IS IMPORTANT, to make sure message stays within those neighbors (not rebroadcast) you add a dam.hear.myModule = function(msg, peer){} which will only get called for dammed 'myModule' messages (dam will restrict messages from escaping). I use this system to create a shared state machine between connected peers, so they can send short lived messages/protocol/handshakes that may change/influence how the rest of their communications is processed. For instance, if anybody watches the gun socket network tab, you'll see the ~first 2 messages are the peers using DAM to handshake a peer ID, this is because the GUN layer does not assume network, network topology, peer, or require peers to be identifiable (the sync algorithm could work with a bunch of blindfolded people in a room who stay anonymous), however IF we are willing to identify ourselves, DAM can create significant optimizations compared to the brute force approach.

See this [dWeb Podcast](https://anchor.fm/dweb/episodes/Data-Dimensions--DAM-Networking--Kanban--Single-Sessions--In-Memory-e592v6) episode for a code walkthrough of DAM's algorithm.

 ### Self

There is some special code in core that might look funny, this explains what it is: Why do we delete the ID from dedup list after a `dam: '?'` on `'hi'`? Because if we connect to ourselves, we want our message to go through (by default it gets deduped) so we can detect that we are connected to ourselves, and disconnect. This is a special hack to deal with this edge case. Keep that line of code as is!

## Architecture

Given some constraints, we are able to prove that optimal mesh topology can be `O(P)` (where P is peers in the network) which is surprising, since for the same constraints, star (centralized) topology is also `O(P)`. Ignoring those constraints, mesh topology is about `O(C*2)+1` (where C is connections between peers), which is not good, compared to optimized federated sharded routing can be `O(S)` (where S is the number of subscriptions to a data record, regardless of P). This is why [AXE](./AXE) is built on top of DAM, as it opt-in allows for highly scalable incentivized traffic, without violating (it is still compatible with) the underlying P2P mesh topology that DAM accounts for. More details, and proof, will be in a later cartoon.

An explanation of how DAM works was ripped out from a chatroom discussion, [bounty](./Bounty) for anyone who cleans it up and documents it into wiki format: 

**Mark Nadal**: DAM is best understood with visual illustrations, but since I haven't had a time to draw cartoons for it, here it goes.

Here is an image of the base brute force method, DAM improves on top of this:

![Brute Force](https://gun.eco/see/ad-hoc-mesh-network.gif)

a GET requests #soul.key or #soul (full node) OUT to all peers it is connected to every message gets a unique ID

each peer receives IN that request, checks the message ID and deduplicates against it (if it has seen it before, it ignores it)

if it hasn't seen it before

it (A) rebroadcasts OUT the message to all of its peers

and (B) processes the message by seeing if it has the requested data that it can ACK back

DAM specifically adds a couple intelligent upgrades to these first couple of steps ^

DAM (Daisy-chain Ad-hoc Mesh-network)

also (when possible) adds the IP/url/ID of the peers the current peer is connected to to each message

this allows on hearing IN on other peers

if they rebroadcast to other peers, they will "drop" echoing to any overlapping peer.

(right now, unfortunately, browser peer IDs aren't stable, so this advantage only really happens with gateway peers)

when each peer with DAM re-broadcasts the message (to any remaining peers not listed)

it deletes that list of IP/url/IDs of peers on the message and replaces it with its own

this prevents the message from getting bogged down with a bunch of appended IPs

IF a peer has data to ACK, then in DAM that peer also (if it is fast enough, aka if it is cached) calculates a fast hash

of the data it is replying with

and adds it to the GET message it is re-broadcasting (that it is daisy chaining)

so that way if those peers receive the GET and attempt to reply, they can ALSO deduplicate their response by the hash

aka Alice asks -> Bob daisy chains -> Carl

assume Bob and Carl have same data to ACK back

because Bob added his fast hash onto the daisy chain out to Carl

when Carl attempts to reply, he stops, and doesn't

because Carl sees his reply would be the same as Bob.

okay, resuming

Alice's GET propagates out daisy chained to all peers

peers now reply by making a PUT operation with an ACK on the outbound message where the ACK is the ID of the GET message's ID

that peer then sends OUT that PUT ACK to all its peers in base system, with DAM, it only sends it back to the previous daisy chained peer that sent the GET, this means in DAM that ACKs can be directly routed (no need to propagate to everyone).

Alice is subscribed to all messages coming IN that have an ACK on the same ID as the GET she sent OUT.

this allows her to receive responses (note: may be many, in a P2P network! Stream replies)

the beauty of DAM is that it "holds back" a bunch of unnecessarily duplicate messages, because ontop of the base message ID deduplication, it also deduplicates against neighbor peer ID and fast hashes.

Finally, outbound updates are trivial

when Alice changes data on something, she sends OUT a PUT command that has the diff she made, to all of her peers, those peers who receive IN the PUT command deduplicate off the message ID, rebroadcast if needed, and process the message depending upon whether they are subscribed to that data or not. If they do save the update, they ACK back a success or failure using the same technique as the ACK to the GET, that way Alice can get notified about replication.

and that is DAM! the now default messaging that happens across any transport wire that is implemented.

To learn more about how peers might more efficiently connect to each other, check out [Service Discovery](./Service-Discovery).

 ### Brute Force

An explanation of how it works from the [gitter](https://gitter.im/amark/gun?at=573ca3741794136a7d09e7d9):

GUN's messaging algorithm is based off a full mesh network

but it works out nicely and self-mitigates against rebroadcasting the same message too many times.

I'm going to use very specific examples

let me find a table really quickly

first example is the common example:

Alice is connected to Server.

Bob is connected to Server.

via Server, Alice is connected to Bob and vice versa.

The default network is VERY meshy, because the goal is that you can (later) intelligently only connect to fingers to mitigate things. The important part is creating an emergent system that works in all cases as the base case, and then can be fine-tuned/optimized for specific cases.

so let's say Alice saves some data

she's only connected to 1 peer

so she blasts that delta (change set) out to the Server

it is in a message envelope

which contains an ID for that message.

the server receives the message

and checks to see if the ID exists in a small fixed-size in-memory list.

the ID does not exist.

therefore the Server then broadcasts it out to all the peers it is connected to. (NOTE: Intentional! However for security purposes you might not want this, so that is why you'd add an authorization filter. But we're not talking about this. We're talking about the base case).

that means Server broadcasts BACK to Alice and to Bob.

Alice now receives her own message

but she doesn't know it is her own

(well, she could, but base case is that she doesn't have to)

Alice checks to see if the message ID is in her small fixed-size in-memory list.

it is (cause she had added that ID to the list when she originally sent the message)

because it already exists in her list

she does 2 things

1) bumps the ID to the top of the list (this is a fixed-size list, so the oldest data gets flushed/removed away to save memory). Bumping the ID to the top is very important because it represents the liveliness of the data, and for lively data you want to reduce re-broadcasts.

2) she does NOT rebroadcast.

the message rebroadcasting has now terminated with Alice.

for the peers she's connected to

HOWEVER, the Server had also sent that same message to Bob

Bob receives it

checks his msg ID list

it isn't there

therefore he rebroadcasts it to the peers he's connected to.

the msg ID goes BACK up to the Server.

the Server checks its ID list and sees it already exists and does 2 things:

1) bumps the ID to the top of the list to keep it "alive" for deduplication.

2) Deduplicates by NOT rebroadcasting.

now every peer in the network has successfully received the message while also mitigate rebroadcast storms. This algorithm works quite nicely, but can very very easily be upgraded and optimized for specific use cases and fine tuning. Like.... if you want the network traffic to NOT be anonymous, you can tag the IDs of the peers in the msg envelope so that way receive peers do an exclusion from the peer list they rebroadcast to (this cuts the echo out) however can expose identity (which might be okay for some apps). You can also limit rebroadcasts to X number of peers, like have peers only ever rebroadcast to 6 peers even if they're connected to a 100. Or only have every peer have 6 connections. Etc.

However, we're not done yet!

all we've done is UDP level fire-and-forget

but Alice SAVED data

she needs to know the data got saved.

the echo of her ID is not an ACK.

hehe

so when the Server processes the message (and depending upon clock drift, the messages might be shoved into historical/operating/future states and processed at "different states")...

if the server successfully saves the data it then sends a NEW message

with a NEW ID

but with a reply-to field of the OLD ID.

this message (without intelligence/identity optimization) gets broadcasted to all the peers it is connected to

Alice and Bob

Alice receives the message and sees that it is a reply-to her original save ID (which she has stored in a separate list)

she now has an ACK from 1 peer that her data got replicated

technically the algorithm though says that she also rebroadcast this reply message to all her peers (since the deduplication/mitigation happens first, not last). If so, she rebroadcasts the message to all her peers (in this case only the Server) which the server receives and sees that the msg ID is in its already-existing msg ID list. And thus doesn't rebroadcast. There are no acks to acks.

however it would be trivial to make it slightly more intelligent that Alice knowing it is her own message (from her other list) that she could stop rebroadcasting then and there.

Meanwhile.... Bob also receives the Server ack

sees it isn't in his message list (but adds it to it) and rebroadcasts again (back to the Server) which receives it and sees that it is in the message list and halts.

now the Server's ACK has successfully been sent to all peers (NOTE: this is assuming that the network has been reliable this whole time!!! ooo-ooo-aaahh--aahhh! Fun CAP Theorem stuff. Don't worry that is covered too).

but now we have to deal with BOB's ACK to Alice's original save request which he received via the middle peer of the Server.

BOB then saves the data (well in the real world, browser peers will only save the data to their localStorage only if it is a key they are already subscribed too. But again lets stick to the base-case)

and after a successful save ACKS back a new MSG ID with a reply-to to the OLD msg ID.

the only peer he is connected to is the Server, which doesn't have it in its message list so it rebroadcasts it to everybody (including back to Bob which does have it and then bumps&halts), and to Alice... who also rebroadcasts it (back to the Server which now does have it and then bumps&halt) and Alice also sees that it is a ACK to her save requests and now knows that she's gotten 2 replications.

finally

we're done.

that is ONE scenario, with limited peers, a star configuration (peers connected to a middleman of the server), and with no network hiccups.

Fortunately this algorithm works for every configuration which is great, however it is not always the most intelligent or savvy for all setups. But thankfully again, it can be fine tuned to get better performance.... but as a base-case it mitigates message flooding and infinite-loops.

I think so! Please do, cause I should get this up on the docs.

NOW, to handle network hiccups. I'm not going to go over the full thing again, I'm just going to explain the rules.

the peer that created the data and wants it replicated is SOLEY responsible for retries if it hasn't received ACKs that it wants.

No other peer is responsible for retrying messages.

So it is very very very important for the peer trying to replicate data out that the hold onto that data as much as possible until they at least get an ACK. What this means is that it is important that the peer (no matter how "weak" it is, like a browser) does store their un-ACKed data at least, at minimum, in something like localStorage (which only has 5mb).

so it has a responsbility to prioritize persistence until it gets ACKs, and even if it does get ACKs it should still probably store a replica of the data (in case the other peers fail/corrupt).

but saying that the initiating peer has the retry responsibility simplifies things greatly.

so if it doesn't get an ack... it can retry. In which case, the retry runs through the same algorithm.

Now, network hiccups.

*sorry peers crashing/rebooting

^retries are covered with network hiccups.

what happens if we have new peers join while this is happening?

what happens if a peer crashes and restarts?

worse case is that the Server crashes

and its fixed-size msg ID list was in memory only

so when it comes back online it has forgotten which messages have already been in a broadcast storm

so two quick cases:

if the server crashes AFTER the whole process is done and comes back online. Than whoopty-doo, the message rebroadcasting has already been sent to all peers and been mitigated.

It doesn't really matter that it has lost its history, cause that message won't resurface.

HOWEVER, if the server crashes WHILE its connected peers are still running this

the connected peers then "echo back" the msg ID that it (the server) just sent to them....

the Server goes ahead and rebroadcasts it AGAIN (while adding it to its seen list)

but that is okay

because all of the peers it is connected to have already seen it so they halt it.

unless of course if 2 of the neighbor peers crash while this is happening... then one of them might not know that it has seen it and rebroadcast. But the moral of the story is that the rebroadcast storms are neighbor-walled actually.

which is a good thing.

so for instance (new example)

if I have a peer which is connected to 3 other people

and each 3 other people are connected to a DIFFERENT 2 other people.

ugggh, hmm.

how do I explain this

imagine a wave

like a literal water droplet dropping into water

it creates a wave rippling outwards

the wave doesn't reverse back in

because it interferes with itself (this is also what happens at the particle level with photons in quantum mechanics)

the interference BACK into the center of the wave... mitigates the wave from spreading where it has already been seen

but where it hasn't been seen.... it keeps rippling out

so back to the example...

Server is in the center of 3 other peers (Alice, Bob, Carl)

Server rebroadcasts the message out to Alice Bob Carl

it crashes and restarts before Alice Bob Carl echo back

so it thinks it hasn't seen the message yet

so it rebroadcasts back out to Alice Bob and Carl

but since all of Server's neighbor peers have already seen the message, they DON"T re-emit it again. So it gets halted. It didn't really matter that the server crashed and restarted. This is the importance of the priority bump on the msg ID... to indicate the liveliness of the message storm.

so let's say Alice was connected to Dave & Fred

Bob was connected to Evan & Gabe

Carl was connected to Hugh & Ian.

the FIRST time that Server rebroadcasted to Alice&Bob&Carl. Server crash&restarts before A&B&C echo back....

the echo back from A&B&C is also the rebroadcast that A&B&C did to THEIR neighbor peers. Which receive the message and echo back.

but the echo back of the 2-degrees-separated don't come back to Server, cause A&B&C mitigate it (since they've already seen it since they delivered it to D&E&F&G&H&I)

this works nicely too

because even if server (and my use of "server" here is just a name. It has not special meaning.... let's say that they are ALL servers)

because even if server crashes AND Gabe crashes

if they restart and have forgotten they've seen the ID

they'll rebroadcast

but their neighbors will mitigate

so Server's forgotten-and-rebroadcasted message won't get to Gabe again. And Gabe's forgotten-and-rebroadcasted wont' get back to Server's.

This is great. Because it literally behaves as a wave.



Ahmed Fasih @fasiha 10:57

:clap: It seems complicated but it's actually the simplest possible thing



Mark Nadal @amark 10:57

and that feature happens because of the msg ID priority bump that keeps the msg ID's lively broadcast storm from constantly being mitigated.... until it truly does die off and becomes so "old" in the fixed-size msg ID list that it gets purged. At which point it is safe to purge because there is not broadcast storm.

done

yupe, and most emergent for smarter and more intelligent behavior!

(the smarter/intelligent behavior can be added on top)