 ## Grants

[ERA](http://era.eco) supports Open Source research into computer science and interactive art experiments that benefit humanity. For research that does not exclusively benefit ERA, such as publications that may be used by the public, other corporations or competing institutions, or governments, ERA offers Grant Fellowships. ERA is devoted to the advancements of applications that empower individual's data privacy and ownership, web-based creative editing and authoring tools that enhance artist's productivity, and improvement to underlying protocols and data structures in distributed systems.

If you are an individual doing research in this area, please apply to our Grant Fellowship:

 > **[APPLY](https://era.eco/2019/grant.pdf)**

## $2K Reward for QA Testing

 > **Note** `1 of 8` bounty tests completed, first by @rogowski!

In 2018 GUN got deployed into production on some large sites like Internet Archive (global top 300 site), D.Tube (1M monthly users), notabug.io (has hit 1TB/day P2P traffic with GUN), etc. Using it by itself, with no extensions or adapters like SEA, RAD, etc., it has gotten "good enough" that Mark would be comfortable declaring a v1.0 release. However, important and commonly used components of the stack, like SEA and other features, don't have as reliable of performance or behavior or are as easy to "just work" out of the box. This is likely due to the fact that GUN is thoroughly unit tested, PTSD tested, and PANIC tested, but other components of the stack are lacking rigorous tests.

To change this, in 2019 we are starting an open ended bounty for Quality Assurance to add PANIC tests that ruthlessly specific edge cases and real world use cases for SEA, DAM, AXE, RAD, and more.

 - **~$250 per test**, with prior design of test approved by Mark.
 - We're looking to start with about 8 diverse tests, for the first $2K batch.
 - Possibly more to come later.

The primary goal of these QA tests, other than increasing ecosystem reliability, is to shoot for creating a big GUN bundle that has all the bells and whistles as a complement to GUN's usual modular underpinnings. It should include everything needed to create Facebook/Reddit/Twitter and Google/Wikipedia clones:

 - Automatic scalable no-DevOps decentralized DBaaS infrastructure (AXE, DHT, etc.)
 - Full fledged user account management systems (SEA, ACL, etc.)
 - Highly performant O(1) search and indexing features (Timegraph, Indexspace, etc.)
 - Media storage and seeding (IndexedDB, WebRTC, etc.)
 - Declarative UI binding and pluggable rendering and gaming logic (as, NTS, etc.)

Core competency will be making sure our flagship community projects & organizations, like D.Tube, Internet Archive, notabug.io, have continued reliability assurance.

Interested in the $2K? Please read these requirements:

 - Thoroughly understand how to write & run PANIC tests, start with the [tutorial here](https://github.com/amark/gun/blob/master/test/panic/load.js).
 - Be familiar with how to configure more sophisticated tests, [HTTPS](https://github.com/amark/gun/tree/master/test/https) in localhost, running across [many physical machines](https://youtu.be/C3akdQJs55E), and asymmetric [correctness](https://github.com/amark/gun/blob/master/test/panic/holy-grail.js) tests (inspired by Aphyr's [Jepsen](https://jepsen.io/talks)).
 - Have good test ideas, ability to maintain tests, and modify tests for debugging purposes.
 - Some tests should deterministically simulate user interactions with [basic versions of fully fledged apps](https://github.com/amark/gun/blob/master/examples/basic/user.html), including testing infinity scroll, memory leaks, etc.
 - Bonus points if some tests are modeled after and could even be live tested in production (if and only with approval) against NAB.

Contact Mark with your test proposal.

 ## OLD

Below is old/expired bounties no longer available.

## $5K Reward for Timegraph

 > **Note** THIS BOUNTY IS NO LONGER OPEN.

`lib/time` is currently in alpha, and Mark is seeking a maintainer for the [timegraph](Timegraph).

1. Current features need to be stable and [bugs fixed](https://github.com/amark/gun/issues/638).
2. Being able to "scroll" through the timegraph, with X number of items in "view" or buffer/sliding window.
3. Load items from "last seen" date.
4. Performance is key/critical, especially making sure no memory leaks and can handle large lists.
5. A way to pause "subscribe" while "scrolling" or "from last seen", by queuing up & notifying "X more new items have arrived".
6. Flexible enough for future extension/feature-improvement/hooks/addons/expansion, play well with SEA and other systems.

Maintainer must have rock solid data structure and tree traversal skill set. Mark will decide how the API, naming, options, & parameters look/feel.

Requirements: OSS as MIT/ZLIB/Apache2, ES5 or lower, no build tools needed working in browser out of the box, work well with any UI framework, must pass demo tests with mock data.

Please contact Mark for more info. Be careful of conflicting claims.

## $10K Reward for HTML

 > **Note** THIS BOUNTY HAS BEEN CLAIMED.

Mark is writing a deterministic HTML normalizer/sanitizer and has issued a $10K bounty for it to be finished.

The original source code is from [5 years ago](https://github.com/amark/normalize) and already has some modifications not yet published.

The goal, and difference from other tools, is to have a strict limited subset of valid HTML tags that are allowed, and that the library always outputs identical HTML for any visually similar rendered code.

### Example
`<b><i>hello</i></b>` and `<i><b>hello</b></i>` are visually the same, but different in code. This should have a canonical form that can be deterministically transformed regardless of the input. See the above link for another example.

### Why
In P2P dApps there is no server to clean/sanitize user input, therefore all peers must enforce sanitization. To create a good experience for end users though, we want users to have a Medium-like editor (there are plenty of OSS options for this, but those are not the point or tool we're looking to have written), but we also want a good experience for developers (that they don't have to include 3 extra external frameworks), and the way that is done is by forcing protocol level (in this case, plain HTML!) correctness/transformation of HTML that *any* peer can run it. This is why it is separate from the editor itself, and may be fed into other tools like an editor.

### Requirements

 - OSS as MIT/Zlib/Apache2.
 - ES5 or lower.
 - Work out of the box in the browser, no build/transform tools necessary.
 - Target size less than [2KB minified gzip](https://closure-compiler.appspot.com/home).
 - jQuery 1.x dependency OK.
 - Fast enough to run on every `keyup`.
 - Safe, not cause XSS or other security leaks.
 - HTML text input, HTML text output. Bonus for optional DOM in or out.
 - Modular settings/configurations so it can be extended with other transforms/features.
 - Must pass unit tests proving that it should work for any/all permutations/edge-cases.
 - Insurance policy, some maintenance/fixes/adjustments may be required after "completion".

Please contact us for more info.

## History

Old bounty hunting attempts that we have tested:

> Update: The results of this study have been poor. No significant change in behavior was observed.

> Old bounty program is currently not in place, but can be re-activated for individuals who want to help - contact us.

We are introducing a bounty program to incentivize the common tasks that nobody wants to do, especially ones in Open Source communities that people feel like are a time suck but everybody knows is important.

From our experience, we have seen that the way other communities solve this is through "that one person" that volunteers to sacrifice themselves to do the menial tasks. However, they quickly become a central bottleneck to progress in the community.

So we are going to attempt an experimental bounty incentivization mechanism to see if it works. During the process, we will be **studying any potential negative affects**, such as:

<a href="https://youtu.be/u6XAPnuFjJc" title="What Motivates Us"><img src="http://img.youtube.com/vi/u6XAPnuFjJc/0.jpg" width="425px"></a>

 - Whether introducing monetary incentives will damage Open Source morale.

 - Whether it will actually make the bottlenecking problem worse.

### Documentation

Our first goal is to reward improvements to the documentation, so more people can know and understand how GUN works. This is our philosophy:

 - Mark is [BDFL](https://en.wikipedia.org/wiki/Benevolent_dictator_for_life), and uses a socratic discipleship approach to educating people.
 - This means that people get personalized, detailed help to their questions.
 - The goal is that this equips people with nuance and comprehension, not robotic copy&paste behavior.

However, it also locks up that information in the [chatroom](https://gitter.im/amark/gun) in ways that are harder for new people to access.

To date, we have had a 10% or less success rate in getting the person who asked the question to copy it and the answer to StackOverflow or the documentation. Our conclusion from this failure is somewhat obvious:

**Observation**: _Content curation is a menial task that nobody wants to do_.

Open Source is great, and inevitably all software will become Open Source (that is, software wants to be free) at some point due to economic pressure over time. However, it is clear that because menial tasks do not require creativity and give contributors autonomy, it may be better to reward through monetary means.

 > **Note** While this may be counterintuitive to "common sense", that we are offering monetary compensation for menial curation but not for feature improvements, scientific research backs this up. Check out the RSA animated explainer talk linked up top.

How it will work:

 - A budget of **$50/day** for curating Mark's socratism into wiki form.
 - Expected **1~4 hours** worth of work to claim daily bounty.
 - Upon review and approval, bounty will be allocated.
 - Only **1 person/day** can receive allocation, so you must declare your work to prevent conflicts.
 - Depending upon country and payment laws, you will need to have **Paypal** or Venmo account.
 - You cannot exceed **$600/year** (2018) unless consult with us first about a US 1099 Form.
 - We will try to be as fast as possible to make payments, but they could take a month.
 - When possible, payments will be combined, to reduce transaction fees.

Open Source contribution are welcome and appreciated, but if you are doing it for free, please tell us.

 > **Note** Any edits to the terms on this page that are not from authorized representatives are not valid, it is your responsibility to check the [git history and verify it](https://github.com/amark/gun/wiki/Bounty/_history) and to contact us directly to confirm the rules match.

### Bug Fixes

We are considering opening up a $1000 USD bounty for killing off bugs. However, as it is not a menial task, and has very strict requirements, and requires extensive knowledge and programming skills, we are still determining rules around it. We may not offer it at all, as it may not be possible to do without ruining Open Source morale.

**It is very important** that "why didn't I get paid for my contribution?" does not become the prevalent mindset, as it will breed all sorts of misunderstandings, resent, greed, and unhealthy comparisons within the community.

So to answer the question very clearly, as it is a "fair" and "just" question to ask: If it is true that we are entitled to getting paid for Open Source contribution, **then Mark would have no other option but to change GUN's truly Open Source license** (MIT/ZLIB/Apache2) to a proprietary license so that he and the company can forcibly charge everyone for using GUN.

We do not want that to happen, and we will not let that happen, by everyone taking active steps to prevent or eradicate that type of thinking in the community, including if necessary deleting the bounty program to reduce any possible confusion or mixing of community and business.

Business related matters should never be discussed in the community, to prevent any possible exploitation of the community's time and resources. Mark, as BDFL, may occasionally bring up company related matters so that the community has exclusive early access to exciting new business related announcements that they deserve to hear before others do.

But as much as possible, we ask that everyone discourage any person who comes into the community that wants to talk shop. Remind them that this is a community effort, largely by Open Source volunteers. Understand that you are free to help anyone, but to protect yourself from being exploited, you may want to excitedly (and positively) ask if their project is Open Source or personal - and encourage them to share it if so.

If they have vague responses, or indicate that there is a team or boss or client, or talk about how they have some secret awesome thing they want to talk to you privately about, be wary of they may be trying to trick you into free work or exploit your Open Source help. Instead, very politely recommend they should seek consulting and refer them to somebody at the company - also, if you are interested, indicate to us if you would want to do paid consulting for them through the company, as this would be a win for everyone.

**For these reasons and concerns**, we would like to get further feedback on bug bounty (and other) rewards. For now, we may do limited tagging of GitHub issues as bounty-able to experiment, but we are highly open and encourage you to privately ask us about bounty opportunities. Bounty hunting comes with pretty strict requirements though.

### Caution

In conclusion, it is an important reminder that communities require proactive and positive behavior reinforcement, or else the complexity and exhaustion of managing numerous relationships can deplete you. If having a single roommate takes a lot of energy, be aware that although a community might be fun, thoughtful, and engaging, it does require conscious effort to prevent drama.

Leadership is about having the bravery and initiative to do what is right even during times that are extraordinarily tense, awkward, or uncomfortable. Our goal as a community is to flourish through creating for ourselves and encouraging others to have autonomy and mastery of the matters that matter. This is done through always honoring and lifting people up, by speaking with confidence but listening with humility and care, correcting our ways when proved wrong and forgiving others, and assuming the best in people especially during conflict by consciously checking for misunderstanding.

That is why we take the socratic approach of discipleship, we want people not robots - copy&pasting documentation is helpful, but never assume it covers the edge case that the person is asking about, or that they even know how to word the problem they are facing.

Treat everyone as you would want to be treated yourself. [Arrogance](https://xkcd.com/1053/) is not the trait or charter of a programmer. Hiding behind complexity is often the pretense of elitism; simplicity is often the most performant code, yet can only be conceived of by the outsider with fresh perspectives.