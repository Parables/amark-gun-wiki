Mesh vs Hierarchical network topologies:

 > Note: This article has been pulled from the [chatroom](https://gitter.im/amark/gun?at=5ce5b3fcad024978c60fc29b) and is formatted as such.

It seems like we can boil the two categories down into 2 base use cases:

 - Self driving car broadcasting swarm mesh
 - Radio tower mesh routing

This model takes what you would want from a hierarchical topology (like Facebook or Google or Reddit) and puts it in the inevitable physics extremes.
FB/G/R not a swarm, could run there, but more useful is for connecting people on opposite corners of the world.
so you'd instead want optimized routing through cell tower / satellite meshes.

As you go daisy-chaining across towers/satellites, you do not want to "broadcast down" unless a user there is subscribed to the FB/G/R thread.
this saves flooding every user, but the unfortunate problem is... FB/G/R-like apps, could have 1 user in every single cell/satellite site around the whole world.
this is back to the Beyonce problem (her update has to broadcast out to all her fans!)
so it is very possible, in worse case conditions
that the mesh tower/satellites need to spread to all other tower/satellite neighbors and to all of their neighbors and onwards
and DAM already optimizes in those types of "robust" networks by using a wave-propagation like algorithm for deduplication.
this gets ultra serious because in this example case, you literally are simulating a radio wave broadcasting out from the initial user (say Beyonce) somewhere in the world.
that radio wave naturally spreads like a wave, and effectively in this case (worse case, if 1 user in every tower/satellite cell) the satellite/towers are going to keep that wave strong as it goes around the whole world

When the wave encounters itself on the other side :) it'll interfere with itself and dedup, per DAM!!! (well, assuming the TTL is tuned right)
so ultimately the question is... making sure these mesh tower/satellites have enough capacity to handle total user packet load. This of course is where HAM and caching comes in, why something like BitTorrent is more scalable than streaming from 1 host, just reuse neighbor bytes. However live-updating data becomes stale quickly :/ so there is only so much you can do here, realistically.

Any system, at peak capacity in worse-case-Beyonce(s)-condition, would have to have good model for multitasking & tiering time.
Somebody told me that live TV broadcast is actually usually 1min30s delayed anyways.
Sorry this will just have to be the limit of physics and capacity, if overflooded, peers watching their Beyonce on opposite side of the world are just gonna have to accept (may be even unknown) there gonna be X time lagging. Our hope/goal is just that it isn't choppy/interrupted.
in which case, you can actually use the wavepropagation as a load-reducing strategy.
towers/satellites in increasingly larger radiuses of Beyonce can prioritize the in-hot-demand (by GET subscription size) live updates in larger cached batch intervals.
so your wavefront acts as your CDN edge network.
for that X time delay factor.
alright, now this comes to other situations

If we aren't in worse-case condition, but still have FB/G/R like apps, it seems like towers/satellites can discover shortest weighted path between users (say there are only 3 users on opposite sides of the world from each other, all on the same app/key/subscription/whatever).
after daisy chaining towers go through initial waveprop, they can form a triangulated link over other towers, now rebroadcast through those lines.
any new joining user would emit a GET, say some worse-case waveprop, until that wave intersected with the line.
then it could be redirected through the line, and a new line drawn (possibly at the intersection point?) to satellite/tower of new user.
(so imagine a a triangle with a stick coming out one of its edges. ? possibly also form triangulation to that dangling point ?)

Actually yes, hypotenuse is always shorter, so that'd form a quad made of 2 triangles for the 4 users.
ok
this now gets back to the difference between self-driving car use case (traditional mesh) versus tower/satellite design.
self-driving car doesn't need to know/care who is trying to establish a connection with it, because its survival depends upon (A) making its existence well known by broadcasting out (visually, sound, radio, laser, etc. all) and (B) autonomously reacting to visual/sound/radio/laser observations.
unfortunately it is selfish, and the safest system would be have this way
that means it is not going to bother rebroadcasting received messages to other cars.

It can
but you wouldn't want the system dependent it (ahem, you wouldn't want your self driving car dependent upon an us-east-1 AWS server, or even a cell tower).
this is very different than the Beyonce condition, even tho they are both broadcast.
and why I think this is the key differentiating factor, not whether something is mesh vs. hierarchical.

So what this means, again, is that it is not optimal for users/cars to form directed connections between each other (they can, but not optimal) UNLESS they are the 2 end recipients (think like the triangulation/quad thing)
it is OK for them to attempt broadcast out into the dark (like with cars)
but not necessarily daisy-chain or rebroadcast.
so this means there are only directed "up" peers, broadcast "down" peers, and broadcast "shout" blind.
an earbud might be directed "up" to a phone, and a phone directed "up" to a tower. This is hierarchical but can be represented instead as a directed mesh to unify mesh algorithms and routing algorithms.
modeling things after the physical topology
a user will usually be connected "directed up" to 3 towers. So I think AXE should by default have browsers connected to 3 relay peers.

But what defines something as a tower or not, I'm hoping doesn't need to be some special option
but simply as to whether other peers have "reached out" to connect to me.
if peers can reach out and connect to me, that means I'm discoverable. However they may not be discoverable (simple example, NAT traversal)
so even if they are connected to multiple peers, as long as those are mesh directed "up", then me-peer needs to treat myself as if I'm a tower/satellite.
even if I am not subscribed to some data, the peer mesh directed "up" to me needs to be routed the message.
and boom, that is where we have the broadcast "down" peers.
when I receive a message, I should not broadcast to all my "down" peers (peers who connected to me, directed) only ones that match subscription.
but when I receive a message, I should rebroadcast to my neighbor towers/satellites to do waveprop (like in case of Beyonce worse condition), or via along constructed "lines" like in the case of the triangulation/quad.

So peers who form directed meshes up should probably max out at 3 directed connections.
Peers who have directed connections to them, should probably max out at 6. Why 6? If a cell phone is connected to 3 towers, and those towers themselves form a triangle, and if you continue that in all directions (waveprop radius), you get 6 neighbor towers that surround you in a circle. Physically, these towers could be linked to each other using line-of-sight highspeed links, not waveprop, and act as the backhaul.

The 6 neighbor towers will also likely have directed connections up to it, meaning that when we connect to our neighbor towers, we can detect they aren't a "down" peer.
tho I guess this poses a first-mover question
which tower is gonna "connect" first to the other? Could that effect the directedness?
Hmm, maybe it goes 2 ways, I connect out to it cause it is discoverable, and it connects out back to me because I'm discoverable.
consolidate the link.
so a bidirectional connection means not a "down" peer.

Right, cause in both our cases, if both peers connect "up" to the neighbor peer, they're both in their "up" paths for routing anyways.
Maybe I should stop here. Still lots to explain further on this and remaining edge cases I haven't talked about, but I feel like this finishes the "mesh" section about how even hierarchical networks can be modeled as a subset of mesh topology by just tracking the directed edges in the network.