"DHT" stands for Distributed Hash Table which is also a not-very-helpful academic phrase. In short, imagine if there was a single contacts list for everybody in the world (a phone book), a DHT is how we split up that phone book amongst everybody so that way we do not have to rely upon a single person to hold it. The benefit of this is that things should be faster since we will not have to "wait in line" if everybody were to ask only 1 person.

The DHT is inspired by [RAD](./RAD) and managed by [AXE](./AXE).

Auto "sharding" DHT logic uses a state machine between 2 peers, not a state machine on a single machine! Such that
(peer A <-> peer B) becomes a state machine between each neighbor peer connection (peer A <-state_machine-> peer B <-state_machine-> peer C).

This produces an emergent property: As long as the state machines between individual neighbor peers works correctly the SUM TOTAL of the entire network of machines should come to an optimal configuration, without any 1 machine knowing the state of the whole network.

The panic test we already have in place for the DHT so far is, if 1 peer gets too many connections it emits a "I'm mobbed with people" state back to new peers trying to connect to it. This state suggests the next states those peers can go into, as in, other peers they can connect to instead. Those peers can then choose to connect to some of the next peers, but if those next peers are also mobbed you'll move thru the network of peers until you find ones that are not mobbed.

Thus producing an equilibrium of all peers being well-balanced, resulting in a happy stable network!

Here are some more details from the chatroom, a discussion how it could work:

https://gitter.im/amark/gun?at=5c1c26a700ab9f27111e3d27

15:32
@amark new PR #668. Solve the problem of peer.id undefined first gun.get in subscription.
Now we can use Radix to create the DHT.
I'm thinking how store this in Radix. Maybe:
var d = Radix()
d('SOUL-PEERID', axedata)
/// and to retrieve all peer's subscriptions
Radix.map(d('SOUL-'))

Mark Nadal @amark 15:34
@/all w00h000   @JamieRez 's PR has been pulled,   super awesome example of OSS contribution!!   We have heroes in the community!!!

rogowski @rogowski 15:35
I see your comment in the PR.

James Rezendes @JamieRez 15:35
:DDDDD

Mark Nadal @amark 15:35
@rogowski   also pulled your PR to axe branch!!!   what awesome exemplars contributing to the whole community!!!     !!!!

James Rezendes @JamieRez 15:35
All in a days work

Mark Nadal @amark 15:36
everrrrrryyyboooody, it is time to celeeeebraaate!
@rogowski Radix DHT I was thinking as being radixKey = soul+key, value = {peerURLs/peerIDs}

James Rezendes @JamieRez 15:37
https://i.imgur.com/xazaFcK.gif

Mark Nadal @amark 15:38
@rogowski so hopefully each radix key represents a "shard" that certain peers are storing (IDK whether they are subscribed or not)
it would be cool if we could combine "current subscribers" and the Radix DHT
but I'm thinking current subscirbers = lots of high churn peers
@JamieRez ROFL
@rogowski and realistically the radix DHT is primarily about indexing relay peers that are sharding whole radix subsets
(so honestly... it wouldn't even be soul+key, it would just be... 'a', 'b', 'c', up to certain radix depth)
@rogowski one of my other thoughts was storing that list of peers on the value
as a unique set that is stringified into GUN rather than on its own
so that way we avoid the whole "deleting data in GUN keeps tombstone" around issue
now it might be possible two relay peers come online at same time, and both add themselves to the same radix prefix
and since they are stringifying the result, the concurrency would result in only 1 of the 2 being listed
but I feel like that can be easily fixed
by these peers have a heartbeat anyways
if they go offline etc. they need to be removed by other peers as no longer current
so current peer can just upon a heartbeat (or simpler, just subscribe to the radix DHT listing of peers associated with the radix key they are also hosting)
and just frequently check stringPeerList.indexOf('myPeerId')
if it isn't there, and if they are still online & hosting the data, they just re-update the list.

Mark Nadal @amark 15:43
now... what if there are like... millions of peers all on 1 radix prefix?
hmm, I think that indicates something wrong
there should be some sort of generic replication factor
like 3
or 9
or whatever
and if peers see that there are at least X number of peers on that radix prefix THEN
they should look for another part of the radix tree to contribute hosting/replication/backup for
this especially true as total data set gets bigger and bigger
that means tree depth is going to be deeper and deeper
and those "millions of peers" now need to be dispersed on lower levels of the tree, not the top
so over time the radix DHT... the very top root of the tree is just ?64? entires (alphanumeric) with maybe up to 9 ipv6 addresses as the value
every peer in the world should store that top entry cause that creates initial global routing table yet super small
so 64 * (9 * 40) bytes
say ipv6 addresses are roughly 40 bytes long
only 23kb for global radix DHT lookup tree! :O
top level

Mark Nadal @amark 15:48
(well... oh shoot, I guess gun souls can have any UTF8 character not just alphanumeric, shoot.... bleh, I guess we could always hash encode it... like base64 version of those utf8 keys, whatever you get the point)
so literally any peer you connect to in the world (censorship resistance!) should have 23kb of 576 other peers you might need to know about
we hope that some of those 576 peers are still up
those initial 576 peers when dataset is small, probably can store all data under 'a' properly themselves
but as dataset grows to exabytes
they can't, so what they do is
peers on 'a' for example
will absolutely store the next level or rDHT peers for all 64 characters beneath 'a'
now when I want to lookup "alice" from some bootstrapping peer
I see that the soul/key 'alice' my GET request should be sent to the 9 peers on rDHT 'a'
those rDHT peers will (1) send me back the sub-rDHT peer listings of 'l' for me to cache (2) DAM (daisy chain, forward) my GET request to those 'l' peers
now if I don't receive a reply for my GET request fast enough from my assumed (2) forwarded message
I can now always send my GET again to peers on 'al' for my 'alice' lookup
since now I can connect to them directly, and I might drop my connections to top level peers
now this is where some of the fun science gets involved

Mark Nadal @amark 15:53
Dr. Cazzell's social psychology field, there is a sub-field on degrees of separation now popularized as "Six Degrees of Kevin Bacon"
turns out that was for physical mail forwarding
on FB the global degree of separation for billions of people in the world
is down to 3.5
what this means is we should be able to optimize the number of P2P connections we have to maintain to some small subset
because even if we are not connected to all peers in the world
this radix DHT should know how to forward our data to the right peer
hopefully within some 3.5 * F ratio of daisy-chain hops, where F is some type of failure rate.
like... I think my intuition is that within 9 degrees of DAM hops we'll find peer that does have the data (if they are online, of course)
that should be worst case
this will still be able to be super fast
yet allow for a totally global decentralized/P2P infrastructure to run with the optimizations AXE makes to prevent bandwidth waste/duplication.