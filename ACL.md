Why the community has had a hard time creating consensus around a [`User()`](User) conveniences methods that abstract [`SEA`](SEA) ACL & permissions: (for the 4th time to be explained - please update this wiki!)



@Davay ah, yes, [many of the concepts in the cartoons](https://gun.eco/docs/Cartoon-Cryptography) are exposed as the SEA library, then "combining them together" is what you need to do manually for now (user() API will have this built out over time, but currently should not be used, because community has not come to consensus on what those defaults should be).

we've had serious discussions on the "various options" that can be implemented as the defaults at least 3+ times over the last 1.2 years. However, people seem unwilling to commit past knowing what options exist, to which ones we should choose.

so right now, the community consensus has been "leave it to individual projects to experiment on their own, and see who wins, or wait until Mark has finishes X/Y/Z priority and has to do it anyways" :stuck_out_tongue: which the first half of that is good but 2nd half is bad :stuck_out_tongue:

Davay

@marknadal can I use SEA.sign to create a read-only node in public gun?

marknadal

at least 3+ projects seem to have done this already, but then very little buy in from community to contribute to modular-izing it & turning it into a self-sustaining framework.

yes.

oh

hmm, it could still be changed by somebody else

but the signature won't validate of course

the new content addressable thing might work tho no docs on it yet

it is something like gun.get('#').get(await SEA.work(content, null, null, {name: 'SHA-256'}).put(content)` code probably has typo

this is alpha feature tho, only enforced on 2020 peers, and also represents an attack vector for peers so assume not finalized either.

Davay

It's all to complex for a frontender... I can't even understand, what's going on here :slight_smile:

marknadal

@Davay true, agreed.

it is also complex for backender :stuck_out_tongue:

which seems to be why we haven't come to a consensus yet

marknadal

a good case study tho is to also look at Firebase's permissions system, because it has also been heavily criticized (and praised).

so I certainly think there is something about the ACL/permissions themselves being a fundamentally difficult subject

prior-art that is universally agreed as being "great"/"easy" would be fantastic to hear from devs. I'm kinda out of the loop :stuck_out_tongue:

myself on what people use these days. Back when I wasn't working on GUN, I was wrote a MongoDB NodeJS driver, so the way I handled ACL/permissions was custom MEAN logic on CRUD REST APIs. It was also a pain & nightmare

would have to query a _acl or _write or _read from mongo, iterate over user IDs, match, etc. then reject/accept response (before promises).

Firebase I do think was certainly an improvement, from the little I've seen of it

but is a beast on its own, a JSON object you upload, with executable code strings that have magic this & other variables, that you do then equality checks against.

if we figure out how to "do this right" ourselves, it'll certainly shake up the industry.

but other systems being complex/hard (or easy) of course is not a justifiable excuse for GUN to be doubleX lacking on that front.

Davay

I think it's about a certain schema of key storage by the users... As we can see in user.grant() an so on

marknadal

yes. That is where I was going

but schemas can be expressed in I think 3 different ways based off the existing layer of assumptions I wa already building on:

based on path hierarchy (inheritability of rules/permission)

or based on unique soul (allows for recursion)

(?) I forget other.

at that point I stopped building, cause either route I went with that "abstraction" would limit somebody from building something, as in, neither routes were composable.

in one of our community podcast calls

we went over each permutation of options available

and I think we found there are 3 strategies, each with 2 (or 3) tradeoffs of sub-choices.

which for a very simple tree, the combintoric on that explodes quickly

building an abstraction at User+SEA level that is not flexible, would quickly become trash & bloat

so I didn't want to commit API space to it, when GUN/User already has a very powerful modular extension feature

safer to defer it to frameworks built in user-land that others can plugin, and therefore knowingly consent/opt-in to a whatever assumption/business-logic they need.

Davay

yes, it sounds great! But we need more examples of building those extensions...

I see examples of SEA signing and encryption and GUN getting and putting in the docs, but never together

marknadal

right, will need to call upon @JamesRez @mmalmi @Dletta @jabis & others

can help please? add more docs on this based on your experience? Try to answer direct questions from @Davay but publish them to wiki so we can return to them.

fire away now, and I'll try to answer what I can, if not too lengthy (I would otherwise, but testing 10K+ table performance)

Davay

For now I've understood only one method of dealing with rights – create some node in user() space and then link it to public space

so it has the extended key and is write protected

but what if I want to open some of it's fields to public? Is it possible?

Example: user creates a chat room, publishes it in a directory and waits for any other user to join

marknadal

so far others have accomplished this thru "threading" (like email), the chat room owner adds the pubkeys of the users they've invited to a "write" property on the room itself.

then your app logic you iterate over each of those pubkeys, and load data from a common path they & the app all share

then you take these data points & thread them together in the UI.

at some point, this technique will have an abstraction that does this automatically, but ultimately is still using same method underneath

so first person to build that abstraction, wins!

@Dletta I think already built some utility tools to ease this too.

but as you can guess/predict

the naming/schema of where those pubkeys are stored, etc.

suddenly becomes massively critical

as any change in it could break all apps using the abstraction/framework, or could make them incompatible.

and data is more brittle than code

code can change, data... must always have old code for it (migration, etc.)

Davay

yeah! It's like too many critical decisions to be made in the beginning

marknadal

yupe!