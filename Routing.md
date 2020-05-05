Taken from a conversation in the chat room:

Base layer is broadcast deduplication, it is robust but NOT efficient O(C*2)+1 where C is connections, the +1 is cause message gets sent back to self.

Fortunately we can improve this with [DAM](./DAM) (Daisy-chain Ad-hoc Mesh-network) which is included by default in all GUN peers.

it is provably just as robust as broadcast deduplication, but is efficient O(P) where P is the number of peers in the network. C > P if you think about it.

now, O(P) is not efficient if only some people are subscribed to some data, and others are not.
like Bob tunneling traffic that he doesn't need/want

but I want to be clear, for any robust network, DAM is pretty good, and huge margins over broadcast deduplication.
example of "robust" networks, you have a celebrity like Beyonce that wants all her followers to get a message.
that means we want all peers in the network to get the message. So it turns out that

DAM, as a decentralized networking layer, is just as good as a centralized star topology!!!

but now for the kicker: global scale, across the pond, communications

you can't have every peer broadcasting everything, robustness gets too chatty at scale.

that is where [AXE](./AXE) comes in, its 1 of many modules that can be included & layered on top. This is already built and working and scaling to tens of millions, the next up (have some code & some tests passing) is the DHT.

what AXE does is it "evolves" the network into a quasi federated structure by cutting off unnecessary connections or data transmission.

it does this by creating subscription tables (so yes, this causes more memory utilization, tradeoff against wasting bandwidth)

imagine A <-> B <-> C <-> (D & E).

D get "foo" C gets this, knows D subscribed to "foo" but not E. C knows the directionality of the network graph (because D called C first, C does not know how to call D back, tho some peers know how to call each other like...) C sends this message "up" to Bob, get "foo"

Bob now adds C as subscribed (you can see how this builds in onion routing!)

now imagine A put "foo":"bar", B gets that, knows C is subscribed (tho doesn't know that C actually does not care about the data, they're just a route)

now C gets the data, and only sends it down to D but not E.

at any point, any peer (say C) can become a leecher

C is not required to forward traffic, maybe that is too expensive, so they can drop doing that. Obviously if a critical link goes down network splits happen (which GUN is offline-first so that is "ok" but obviously sad), but peers should NOT be trusting 1 peer

they should have at least 3+ and rotate between them, that is where the upcoming DHT will automate that.
but the last comments here about AXE is that
it is already working at huge scale, but it does not sacrifice decentralization! Why?
because (and this can be a bad thing lol)

it does selective round robin-ing

if data responding to a GET request has mismatching hashes

it will start to "devolve" into a mesh network rather than a directed network

it'll first keep forwarding the GETs to peers subscribed to that data until ?at least 3+ ACKs?(within a certain time frame) all have same hashes.

but if it runs out of matching subscribed peers, it'll start asking other peers, aka going into DAM mode, mesh. Those peers might not have the data, or know about subscriptions (they might not be running AXE), but they'll still daisy-chain relay it to peers they know about, who MIGHT have the data, and then reply back.

plenty of other things AXE does, like :( :( :( allows peers to charge peers for bandwidth/data/traffic :( :( :( but I hope enough people run volunteer peers over time.

so AXE scales to like the 10s of millions already on $0 infrastructure costs, but I'm having trouble with past that unless I spend money (NEVER!!!)

so this is where the last piece of the DHT comes in, I won't explain it in full, I did a podcast on it the other year happy to link if you want, the gist is

all peers limit the number of immediate neighbor peers they have

if they go over a certain limit, they make recommendations to those peers what other peers they can connect to instead

then all peers share a partial routing table in a radix tree
and either AXE data based on that, relay, or reply to neighbor peer with the table they can have temporarily, probably all routes will be tried, and then optimize based off latency versus churn.

the performance should be O(D) :/ which isn't saying much, where D is "depth of network"

as in, it SHOULD be constant time, but given how slow networks can be :P

even a couple network hops "deep" can rack up significant latency, but whatever, it is the best algorithm for this, when the network hits its limits/saturates, people will just have incentive to build out physical infrastructure to reduce those network hops.