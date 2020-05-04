Forget, omit, do not sync, etc. this is a new GUN + SEA enforced rule in `>=0.2020.421`:

 > Note: Extremely alpha. Candidate protocol extension proposal, may be dropped or not included by default in SEA peers if it fails as an experiment.

Try it yourself:

1. Run local peer, open 2 browsers.
2. `gun.get('room-name<?60').set("Mark Nadal")`
3. `gun.get('room-name<?60').map().once(console.log)`

Experiment with:

 - This tells SEA to hardcode 60 seconds (1 minute) expiry of records. (Set the limit you want, but it cannot be changed later unless new object, new soul with different limit).
 - Hyper mode: Combine it with [content hashed tables](./Content-Addressing)! `gun.get('room#name<?86400').get(hash).put(data)`, nobody can edit, can only add, index forgets after 1 day.
 - Note: **data may still be stored on disk**, but it does stop them from syncing to other peers, we can deal with disk later.

Now after 1min+ try reloading, and you'll see map().once not trigger old values!
Have fun playing with it, EXTREMELY ALPHA, please report bugs!

Hopefully this can super charge more advanced/complex collaborative features:
 - Online-only user inboxes
 - Meta-data coordination for threading convos/groups
 - Spawning 1-off WebRTC connections or other fallbacks, etc.
 - Faster indexing, have always recent results like Google search, AirBnB listings, etc.
 - Countless more mind blowing possibilities.

Q&A

Does the expiration only work at the top level? Or can we use it in Iris to create special expiring messages? Or even use it as a channel config like “all messages expire after 7 days”? Might be good as a security stopgap until we have ratchet keys.

 - Everything in GUN is at top level. Just link it as a sub-object if you want it there.
 - This is wire level, hopefully somebody can wrap it with a nice API on top so devs don't have to type protocol symbols.
 - You specify any expiry in 1 second units, but for a soul, you cannot change it, tho a wrapper on top could "switch" by generating new tables as needed and managing context.
 - Expiry is relative to HAM, no peer is forced to agree on what time it is, altho gun/nts.js will try to sync peer's time and find consensus (interesting easter eggs on this in the white paper), so don't assume expiry = security or true deletes. As it stands, I'm guessing RAD will store within-expiry records, but then has no way (yet) to garbage collect them later, data is probably still on disk & could certainly always be on other peers.