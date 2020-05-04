![](https://gun.eco/see/ad-hoc-mesh-network.gif)

From the [gitter](https://gitter.im/amark/gun?at=573ca3741794136a7d09e7d9):

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