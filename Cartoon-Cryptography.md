## Security

<div style="position: relative; padding-bottom: 56.25%;"><iframe src="https://www.youtube.com/embed/ccKThyaDR30" frameborder="0" allowfullscreen style="border: 0px; position: absolute; width: 100%; height: 100%;"></iframe></div>

When building an application, there is two ways to do security. The real way, and the fake way.

Most websites you use today have fake security. When you log onto their service, your password gets sent up to their proprietary servers. There they check to see if it is correct and grant you access to *your* data.

Sure, their servers might be in a top secret location. But the problem is that they know your password. Which means any bad actor, like a rogue employee, a hacker, or a government agency can snoop on your data without you knowing.

So what does real security look like?

Rather than your password getting broadcast over the internet, you instead get sent a bullet proof vault that you try to open. If your password is correct, the vault unlocks. But if you are a hacker, it would take a 100 years to crack open - and you would have to do this for EVERY piece of data sent to you.

This makes it very hard for bad actors to spy on you because your password never leaves your computer.

So, which security would you choose?

While GUN can certainly be used in the traditional setup, we're going to show you how to build applications using real security instead.

## Cryptography

<div style="position: relative; padding-bottom: 56.25%;"><iframe src="https://www.youtube.com/embed/Zl0nk_ZZV6Q" frameborder="0" allowfullscreen style="border: 0px; position: absolute; width: 100%; height: 100%;"></iframe></div>

To build a web application, we need to... create a user, sign into that user, save private data, publish content, send private messages, and have group conversations. Phew! That is a lot.

The most important piece is first creating user accounts. Unfortunately, a username/password combo is not very secure. We need something better.

During the Cold War a new type of security was invented. It is similar to a username/password, but imagine instead having to memorize the first thousand digits of PI! Having a password that long would take a hacker one hundred years to break. This is why it is secure and it is called public/private key cryptography.

However, your users would probably get really confused with that. So how can we let them keep their own password... but still have real security? Well, we take their normal password and mix with other information. The end result is long and complicated, creating a secure account for our users to sign into.

## Work

<div style="position: relative; padding-bottom: 56.25%;"><iframe src="https://www.youtube.com/embed/jXni0KDQNsc" frameborder="0" allowfullscreen style="border: 0px; position: absolute; width: 100%; height: 100%;"></iframe></div>

I have a secret recipe for a delicious pie. How hard would it be for you to make that pie without knowing the recipe?

I will even give you a hint. One of the ingredients is salt.

It might be easy for you to guess what the other ingredients are. But you would still have to bake a pie for each guess. That is a lot of extra work.

And that is exactly why mixing our password with some salt* keeps our account secure. This recipe is called Password Based Key Derivation Function number Two or PBKDF2 for short.

If you know the password, it is easy to bake the same pie again and again. If you don't know the password, guessing is easy, but baking is hard.

This idea is called "proof of work" and is important for creating and signing into a user account.

We have one last step. With public/private key cryptography, the proof of work and the private key are different, even though they are both long and complicated. Instead, we use the proof of work to encrypt and decrypt the private key.

## Encryption

<div style="position: relative; padding-bottom: 56.25%;"><iframe src="https://www.youtube.com/embed/_RDC37hrTo8" frameborder="0" allowfullscreen style="border: 0px; position: absolute; width: 100%; height: 100%;"></iframe></div>

Let's summarize. So far, we created a user account with public/private key cryptography. This way, it is secure, unlike passwords. We then used PBKDF2 to extend the user's password into a "proof of work". That way, if a hacker tries to guess the password, they have to do hours of extra work for every guess.

Now, we use the proof of work to decrypt the private key. This private key is used to lock and unlock our private data, in the same way you lock your house or car to stop other people from getting at your stuff. But, how does a lock work?

Well, take a minute to stare at this animation and replay it again and again while listening to my epic voice. This is how a lock works.

Encryption is the same thing, it rotates each letter by a different distance, thus scrambling the message. You can only unscramble the message if you have a key that perfectly restores every letter in the message. This is called Advanced Encryption Standard, or AES for short.

But wait. If we use the private key to keep our things safe, how do we then protect the private key itself? That is where the proof of work comes in. And answers why we encrypt the private key with the proof of work. Keeping our accounts secure and our users happy.

## Privacy

<div style="position: relative; padding-bottom: 56.25%;"><iframe src="https://www.youtube.com/embed/-SiLnaSDkh4" frameborder="0" allowfullscreen style="border: 0px; position: absolute; width: 100%; height: 100%;"></iframe></div>

It is easy to confuse encryption keys and public/private keys. One is like a door knob, you need a key to both lock and unlock it. While public/private keys are like a padlock, the public key can close the lock but only the private key can open it.

This becomes useful when we want to have a private conversation. Alice and Bob exchange padlocks. Now Alice can write a message that only Bob can open. When Bob replies to Alice, he uses her padlock so only she can open his message.

Simple enough, but what if we want to have a group conversation? Alice creates 4 copies of an encryption key, which will let everybody in the group both lock and unlock. She now can invite Bob, Carl, and Dave to the group by sending them the key by using their individual padlocks.

Once everybody has the same shared key, they can now write and lock a message to the group that only the group can see. Allowing everybody to pass messages around.

## Signatures

<div style="position: relative; padding-bottom: 56.25%;"><iframe src="https://www.youtube.com/embed/tg06mdcGs5k" frameborder="0" allowfullscreen style="border: 0px; position: absolute; width: 100%; height: 100%;"></iframe></div>

We have used public/private key cryptography for everything from creating user accounts to having private conversations. But there is one final use case we have yet to explore.

Imagine that the police are doing an investigation, they can look for fingerprints in a room to prove that somebody was there.

This is how Digital Signatures work. A user can leave their fingerprint on a blog post or message, which then allows others to verify who sent it.

This is useful because the internet is full of fake users, who may try to steal your information. But once you trust somebody, you can then use their Digital Signature to prove they are the author and ignore any imposters.

## Summary

<div style="position: relative; padding-bottom: 56.25%;"><iframe src="https://www.youtube.com/embed/gBp_vA6D9PI" frameborder="0" allowfullscreen style="border: 0px; position: absolute; width: 100%; height: 100%;"></iframe></div>

Let's take what we have learned and put it into technical terms:

To create a new user account, we first generate a public/private key pair. We then add the ability to login to the account by hashing the password and a salt with PBKDF2. This produces a proof of work that is used to encrypt and decrypt the private key with AES.

Once we have the private key, we can then read and write our own personal data using the private key as an AES encryption key. We can also have private conversations with others by sending them our public key padlock.

Finally, we can prove to others we said something by adding a digital signature to our published content, which the public key can then verify.

This covers some of the most advanced security possible. And is why GUN pushes the standard of true security. Covering all of your application's needs.

### Next up?

 - To build a crypto dApps like this today, try [THE INTERACTIVE TUTORIAL](Todo-Dapp)!
 - Or look at [SEA](SEA) which implements the algorithms discussed in this series.