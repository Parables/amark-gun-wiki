## Advanced eXchange Equation

<div style="position: relative; padding-bottom: 56.25%;"><iframe src="https://www.youtube.com/embed/EHZyaupYjYo" frameborder="0" allowfullscreen style="border: 0px; position: absolute; width: 100%; height: 100%;"></iframe></div>

This is the very first public document about AXE and is intended as a prototype draft for developers to provide feedback on the architecture. This page is not ready for general readership though, and will be entirely rewritten in the future.

## Cartoon Whitepaper

Yes, we're notorious for our comic strip explainers for everything - our [conflict resolution algorithm](https://gun.eco/distributed/matters.html), [how cryptography works](https://gun.eco/explainers/data/security.html), and more!

And yes, this is **mandatory reading** before you read the actual whitepaper. Check it out here:

[Trusting Strangers with Axes](Trusting-Strangers).

## Lightning Talk

Here is a 5 minute lightning talk that I had the honor of doing before Bram Cohen, Inventor of BitTorrent, at Berkeley.

It focuses on providing a brief explanation of how the **Proof of Propagation** algorithm works.

It is highly recommended you watch it before reading the whitepaper. 

https://www.facebook.com/BlockchainatBerkeley/videos/2006069823011271/ (starts at 12:26)

## Whitepaper

The actual whitepaper is hosted at Stanford, here:

https://www.stanford.edu/~nadal/A-Decentralized-Data-Synchronization-Protocol.pdf

However, we will be moving it to our new website when we launch it and officially hosting it there for future reference.

It is split into 2 papers, the first on AXE itself, the second on how GUN works. Why? Because what we claim to do in AXE is impossible unless you know how GUN's CRDT algorithm works. These two things unfortunately cannot be separated, especially since many people will hear about AXE without knowing about what GUN is.

The whitepaper is in a review draft, and may be inaccurate or incomplete in areas. We would greatly appreciate **pull requests** on the paper, especially little things like filling in the citation links, and stuff like that.

I have answered a ton of follow up questions already in person and over email, they mostly repeat a lot of information that is already in the paper, however they also provide a lot of valuable nuanced details that the paper does not go into. When I have time, I hope to dump those questions and answers into this wiki page, and then it would be useful for the community to help merge them into the official whitepaper so everything is in one place.

## Contributing

Pull requests would be greatly appreciated (see comments above), however the whitepaper itself is in a google document form. So here is how to contribute a pull request:

1. Open a PR or issue with "AXE:" as the title.
2. Describe which Section & Sub-section you are editing. (Not page number!)
3. Provide your changes as a whole paragraph or paragraphs. (Clearly marked start/finish)
4. It would also be nice to provide what the previous version was (clearly marked start/finish) to prevent any conflicting simultaneous PR/changes with others.

Your contributions will be recognized in the paper in an acknowledgements section.

In the future we will probably (if needed) migrate the paper PR/issues into its own repo.

## Roadmap

AXE will be developed in three (3) stages:

1. (✓) Assume main relay peer, optimize for best bandwidth based off of browser subscriptions and "cutting off" relaying unnecessary data. Implemented: see @rogowski 's `axe` branch PANIC [tests](https://github.com/amark/gun/pull/684#issuecomment-453315017)!
2. Optimize for peer/relay-peer failover or too many peers, form stable radix DHT across all peers.
3. Turn on end-to-end encryption and contracts on bandwidth/relay, tokenize bandwidth by metering throughput, etc.

## Code

[SEA](https://hackernoon.com/so-you-want-to-build-a-p2p-twitter-with-e2e-encryption-f90505b2ff8) is a necessary primitive to AXE, and it is already in alpha.

We have working demonstrations of AXE's identity/wallet system running on a fully P2P social networking dApp and automated load tests (using [PANIC](https://github.com/gundb/panic-server)) for it across a physical test network here:

https://youtu.be/C3akdQJs55E

Obviously, there is a lot more work to be done though. So I apologize in advance that this section is somewhat sparse other than showing that demonstrations of AXE already exist and work (this is important, since most cryptocurrencies are just vaporeware), but we will update docs with more details later.

Additionally, it is fun to note that we're recognized by GitHub as [#2 in Blockchain](https://github.com/topics/blockchain)!

Please join the [chatroom](https://gitter.im/amark/gun) to discuss things further! Cheers.

---

# FAQ

 - **Will latency increase when trying to establish connections?**

Not noticeable by humans.

Especially during Stage 1 and Stage 2, where most of it is proxied via existing TCP/websocket/WebRTC connections anyways, the setup cost is virtually the same.

 - **How can tokens be traded on an exchange if they all have different value?**

Using cartoon analogies, this is how tokens can be converted on exchanges:

_1)_ Imagine creating a new physical vault, each one must have the same size of space inside.

 > Analog: This makes a static block, like with Bitcoin, that can be ERC20/other compatible and traded on exchanges.

_2)_ Now, dump a ton of your coins into the vault, such that they fill it entirely.

 > Each one of your coins might have a different value. (a USD dime, penny, quarter, nickel, etc.)

_3)_ You add those coins up and promise to sell the vault as a whole at a guaranteed price for the next hour.

 > Analog: This would be through a smart contract, timing can be adjusted.

_4)_ Now other people can choose to buy your vault in a marketplace.

 > They can add the coins up themselves to verify the price.

 > The coins' worth might go up, down, or stay the same.

_5)_ If they buy, the vault unlocks and all the coins spill out into their ownership.

_6)_ If nobody buys, the time vault's smart contract expires, and the vault opens up and you get your coins back.

 > If you still want to sell, repeat this process (potentially adjusting the price where needed).