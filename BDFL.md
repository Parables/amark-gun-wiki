[source](https://gitter.im/amark/gun?at=5bd779bb82893a2f3b5dd2f5)

I've recently came to the conclusion that there are a couple really interesting ideas lately that deserve my full attention and are worthwhile enough that even the community should rally and organize around. Many of you know I'm a hardcore voluntarist, so I'd never ask anybody to join, but this idea is so important to bettering people's lives it would be irresponsible if I didn't ask.
There are lots of little steps that need to be taken, code that has to be written, to get there.
Improvements to SEA, getting AXE going, etc. are obvious things, many of you also know MetaMask integration is happening as well.
but then there are odd side things, like needing to sanitize and deterministically canonize HTML end-to-end for user input.
This one, for example, is based off my old normalize library
It will let us attach MarkDown, Medium, or custom editors into dApps
yet make sure the input can be synced in GUN and show up on the other side in different browsers/devices the same way as it is edited/shown/previewed on yours.

To me, it is important to build a meta-editor
at first, it might look just like a Twitter or Facebook or SMS input
but if you start typing longer, automatically expands into a Medium-like article/blogging editor.
and can start behaving like a gDoc collaborative editor, for example, if you want others to help work on it.
one cool piece, that is inspired by @robertheessels who built our Docs system, is that I also want hyper-linkable/inlineable articles
this is possible because of the graph data structure.
for my scientific brain, I want to have small standalone arguments or explanations of an idea that I put on its own page
but then I might be writing another article, and want to include that "idea"
so I should be able to inline paste it
but whenever the standalone version gets updated
the article, since it is using GUN, would also be showing inline the updates.
You know we have a lot of collaborative, interactive educational pieces

the Cartoon Cryptography, Sorting Story, Todo Dapp, Trusting Strangers, Distributed Matters, etc.
I want to make more of these pieces, and make it easier to make them, and easier/reusable to interlink and reference.
so that is just a couple examples of such a meta-editor
but then we get into the idea of ... wouldn't it be nice to have your blog posts / tweets also in audio format?
Say, for as moral of an argument as... so it is accessible to the blind?
Or, for as greedy of an argument as... so your tweets/blogs are also standalone podcasts that drive more users to your Twitter account?
so this meta-editor also should be expandable to recording audio
bunch of interesting things here

Need help writing binary converters/adapters for GUN, for instance.
Need people who have Web Audio API experience to help/teach/code.
Beyond just being able to record audio in the meta-editor, for podcasting, what if while you are writing your tweet that has expanded into a long-form Medium-like article, that you are now gDoc sharing with your friends... what if it by default started a audio call with anybody who jumps on to help? Now you can talk with somebody on the opposite side of the world while you are collaborating on writing the article!

If these snippets also have accessible audio segments to them, then it would also be possible to "remix" podcasts in the same way earlier, with text, I suggested you could inline insert other standalone ideas/arguments into your article, and they'd also realtime update, now you get free audio in your new articles that inline old portions.
How cool would it be able to highlight some text, read it, and the system automatically know that is annotated?

And don't get me even started on my old Accelsor ideas around how the meta-editor could also help draw cartoons/images/graphics or be like Google Slides, let alone let you add a meta-editor into your meta-editor (super meta!) where in your "article" or "post" you've added the ability for people to write a reply/comment to your "article" or "post"
so in the same way you might insert an image into a blog
you could also insert a meta-editor
which again, would allow people reading your article, to write an tweet in it... aka you've now built your own commenting system!
another thing I want to use the podcast/audio/accessibility thing for
is to create dynamic music
I have an old demo of a synthesizer that generates classic/acoustic piano sounds while you write English
so you wind up playing a song by writing a tweet.

Or cooler, one of the next things I want to do.... and why I'm looking for an Web Audio API volunteer is to hum into the meta-editor podcast system, and then ask it to convert the hums in the waveform into notes, that I can then apply different synths to (orchestra, piano, guitar, drums, etc.)
once the waveform is converted to notes, those notes should be editable - maybe I made a mistake in my hum, so I can delete that, or change the note.

This has other even neater ideas:
The hum-converted-to-notes now lets you adjust the timing, you could draw a line/arc/something to adjust the notes-per-second, so it even if your hum wasn't quarter-second consistent, your synth generated version of the notes is, and or it could slowly speed up or slow down, based on the arc you draw.

Now what if we apply this hum-to-editable-notes idea, and apply it back to podcast recording? What if you recorded while selecting text, so the system knows the for-blind-people audio-accessible portion of the text... but you messed up? You said "um" or something. Couldn't the system detect the peak and valleys in the waveform, like it did with a hum, to let you go back and delete the "um" like you could delete a note? Rather than just clipping the audio, it could "blur" the transition from before and after, so the deleted word doesn't even sound like it was clipped, it plays smoothly, etc.

But, first things first
I need normalize with the HTML to guarantee the structure of the collaborative edits and rendering on different devices/browsers.
who wants to help write a deterministic HTML sanitizer? :)
I'll be more proactively using the channel to make BDFL asks, cause I think the ideas and code themselves are exciting enough, but more importantly... as I'll be using the meta-editor to then EXPLAIN the big idea that is so compelling, with cartoons, podcasts, logic-proofs, articles, etc. and more.
I feel like if I were to try and explain it all right here and now, it would be pages and pages and pages and then get lost in the chat room. A meta-editor would let it be its own dedicated page on the wiki, and stuff, and make it accessible!