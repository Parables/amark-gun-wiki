There is often a difference in the architectural design of peer connectivity, and the actual implementation details of topology. So please keep in mind the following, for documentation purposes:

 - (A) Peers can either be outbound connected or inbound connected.
 - (B) The browser has an out connection to a server.
 - (C) The server has an in connection to a browser.
 - (D) Outbound connections are manual (must be given/updated via Gun(opt) or gun.opt(opt))
 - (E) Outbound connections can be retried when they fail.
 - (F) Inbound connections are automatic, and cannot be retried if they fail.
 - (G) Your "peers" count as being both inbound and outbound connections.
 - (H) Messages are sent out to your peers.
 - (I) Each one of your peers will then re-emit the message to their peers, and repeat again, etc.
 - (J) So even without being directly connected, other peers will receive the message via "daisy chaining".
 - (K) The mesh networking algorithm helps optimize and deduplicate bandwidth and prevent floods/loops.

As such, messages are discovered by all peers (J) that are without the need for service discovery.
However, service discovery may still be useful for certain apps.
If you need/want this, it should be trivial to build yourself by having peers do the following:

 1. Save their own outbound ID to a peer table synced in GUN, so that other peers can find them.
 2. Upon being synced those new peers, do some custom filter logic, and then update your peer options in GUN.
 3. GUN is very sensitive about errored/closed peers (Note: GUN is offline-first, but if peers are specified, it treats any peer error [including disconnection] as a critical database error. It you don't want that, it is your responsibility to remove that peer so you no longer get critical database errors and nasty timeout issues.)
 4. As of March 2018, GUN has not been well tested for changing/rotating peers, and there are some known bugs in the default adapters that will get fixed, we'd love help reporting more problems, testing more thoroughly, and fixes.

Note: It is not recommended to have every peer directly connected to every other peer in any system, we guess that at most 6 direct (working) connections is plenty, but are still testing this.

Note: WebRTC itself, not GUN, still requires bootstrapping signaling hub(s), and WebRTC ICE candidates are unfortunately inbound based (they do not persist past browser refresh).

[AXE](./AXE) will provide a full suite of opinionated, radix shard based service discovery and routing.

For information on how the daisy-chain ad-hoc mesh-network algorithm works, check out [DAM](./DAM).