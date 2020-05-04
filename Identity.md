[Mark Nadal @amark 10:49](https://gitter.im/amark/gun?at=5bb7a41164cfc273f99eca8e)
@go1dfish the way I understand what Martti has talked to me and shown the code of Identi.fi and stuff
is imagine you have 149 friends + you.
a lie has greater entropy, truth has low entropy (higher collision rate)
what do I mean about this?
thought experiment
pretend we ask you to play the middleschool game 2 truths & a lie about your friends
over some amount of time, you say 2 true things and 1 wrong thing about each 1 of your 149 friends
likewise they do the same about you.
in the network as a whole
we can probabilistically determine which thing was the lie
without ever having been told it was a lie
because "My friend, he has brown hair"
several people might state that as a fact

go1dfish @go1dfish 10:52
but the lies will tend to be more random, you're saying the truths will converge more

Mark Nadal @amark 10:53
but "My friend, he traveled to Jupiter" is random to another made up claim like "My friend, he was raised by Spartans"
exactly
in order to get lies to converge
you'd need a larger and larger group of people to collude
around the same lie.
but when we're dealing with the statements being simple factual statements like "what is their public key?"
in larger and larger network effects there is an incentive for one split-brain of the network
to actually honestly report on the public key of their friends
while the other part of the network is trying to dupe people into IDK
so public key distribution is the known greatest weak link in all cryptography
Martti's system inverts it
rather than playing a middle school game of telephone, from you to 7B+ people in the world
in school, kids will corrupt (accidentally even if you ignore malicious) the message by even a couple people out.
so to fix that problem
we invert it
have 7B+ people in the world, make attestations about who has what public key(s).
now we're able to probabilistically rank how many people attest to the same thing
but the cool part is
specifically we do it weighted through your web of trust!
web of trust gets interesting relative to Dr. Cazzell being an applied social psychologist
the "Six Degrees of Kevin Bacon" was an old social experiment done via mail forwarding
and found that for the then ~6B people in the world, they were 6 degrees separated.

Mark Nadal @amark 10:58
Now with FB & stuff, it is more like ? down to 4 degrees ? or something like that.
point being is
Identi.fi is an indexer that you (or your friends) run
that after you've added people to your contact list (aka, stated who your friends are and what their public keys are, etc.)
it'll start crawling your friends of friends
and building an index of claims your friends of friends, and their friends of friends, etc. have made
now when you or your friends want to ask any search term "Mark Nadal" "go1dfish" etc.
your index is able to reply O(1) with the pre-indexed/crawled list, sorted by probabilistic kevin bacon effect + attestations, of who the real go1dfish with what confidence score.
but even COOLER
is the system works even if YOU and the person you are searching for.... aren't in the system!!!!
as long as SOME people/entities you are remotely okay with trusting sit between you and the person you are finding
you could query N of these people/entities identity index
and since it is pre-computed, each replies O(1) itself, and then now with it being on top of GUN, you can locally do the graph intersection of those responses
that is fast since it is all in memory

Jachen Duschletta @Dletta 11:02
That sounds like a PageRank modified by trust instead of linking.

Mark Nadal @amark 11:03
so now you get trusted identity verification (probabilistically weighted) even if the you + the end person (beyonce? justin beiber? etc.) is not in the system.

Jachen Duschletta @Dletta 11:04
Could I assume that people who are more trusted by their friends also create more trust in the people they are trusting?

Mark Nadal @amark 11:04
because your N friends/their-friends-of-friends/entities have made attestations that the search key "TBL" means "Tim Berners-Lee" who has this verified domain name we trust, even if TBL isn't in our public key system.
@Dletta zinger! Yes, somebody else explained it that way, I think it is a good way of thinking about it, right (?)
@Dletta yes.

Jachen Duschletta @Dletta 11:05
Definitely, if you are trusted, you have more trust to give away

Mark Nadal @amark 11:06
but even if you are a popular person with lots of current trust, that doesn't has as much impact as you think

Jachen Duschletta @Dletta 11:06
Which is what PageRank Algo does for graphs, the more in degrees a website has the more base points it distributes to the out degrees which when run over the whole internet shows the websites that are most linked and therefore must be best to return for a related search

Mark Nadal @amark 11:07
as connections to people are combinatoric (permutations), which explodes much faster than something like exponential, etc.
@Dletta what Google USED to do :P
not really any more, PageRank has basically been dead for a long time :P
Long Live PageRank but decentralized!!!

Jachen Duschletta @Dletta 11:08
But people tricked the system by creating loops that would self distribute

Mark Nadal @amark 11:08
:D :D this part gets cool
Martti ran the crawler on the Bitcoin network
and noticed a sharp fall off around the 4th (or 5th?) degree out
even though there were tens of thousands of connections
at 5th and 6th
they oddly didn't have any higher priority connections 1 ~ 4
and he realized OH
those are bot nets...
sybils!
so you can actually use the system to detect those loops themselves, based off your decentralized trust view (which Google doesn't have since they are centralized).