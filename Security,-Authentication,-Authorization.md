This page will be filled in at some point. But for now you can read the discussion and explanation on security starting here:

https://gitter.im/amark/gun?at=5744bec963e41bd84bf024be

To upvote prioritizing this wiki please join the gitter channel and tell us.

## Introduction

We had a discussion on gitter on how to handle security in GUN, a peer-to-peer database. Before we start, let's present some figures we are going to work with. We have:

* **Frodo** a nice person who totally trusts Gandalf and shares things with Samwise.
* **Gandalf** trusted friend of Frodo.
* **Samwise** trusted by Frodo but not for everything.
* **Sauron** not trusted by anyone, has eyes everywhere.
* **Gollum** doing what he can but often making a mess around him.

The security question contains multiple parts:

**How do we avoid someone masquerading as someone else ?**

The chat example is easy to understand: we do not want messages that seem to originated from Gandalf to actually be from Sauron. A game is another issue where player 1 (Sauron) could send position updates to player 2 (Frodo) and win by cheating.

**How do we protect sensitive data from being read by unauthorised people ?**

The plan of action discussed by Frodo and Gandalf should not be seen by Sauron, right ? Real-world issues are countless but we can list a few: medical information, a book in the making, notes on your sex life, etc.

**How do we share sensitive data with a group ?**

Gandalf and Frodo started exchanging messages and then Samwise joins the group and they want him to be able to read all their past and present exchanges.

**How do we avoid app spamming ?**

If the database is not linked to a central (filtering and authenticating) authority, how do we avoid someone filling it up with junk data such as: "My Precious, My Precious, My Precious, My Precious, My Precious, My Precious, My Precious, My Precious, My Precious, My Precious, My Precious" ? Or simply someone writing `null` to every part of the database until the peer-to-peer sync erases all the data in everyone's copy.

In a gaming scenario, how do we avoid DOS (Denial Of Service) attacks like this ?

## Some key concepts

**Asymmetric or public-key cryptography**

A public key allows anybody to encrypt something but only the private key can be used to decrypt. How and why this works is like a fairy tail with lots of strange animals ([elliptic curve](https://en.wikipedia.org/wiki/Elliptic_curve_cryptography) is one of them).

    data + public key ==> unreadable junk
    unreadable junk + private key ==> data

**Digital signature**

This is the process by which a user can verify that a message really originated from a given user. Say Frodo receives a letter from Gandalf, he wants to be sure Gandalf actually wrote every single word of the letter. Digital signatures work like this:

    message + private key ==> signature (short junk like piece of text)
    message + signature + public key ==> true / false

## Answers !

Now that we have our playground setup, let's try to answer some of our initial questions.

### How to avoid someone masquerading as someone else ?

Gandalf and Frodo want to post news in the Shire about their adventures and they want to make sure the readers can check if these messages are legit.

Each of them hides in their home, sits down and throws stuff around randomly while shaking the key pair generator. They must shake and to random stuff otherwise someone might find a way to generate the same keys from the pair generator. Now each of them has his own public and private key pair:

    Gandalf: gandalf.pub, gandalf.priv
    Frodo: frodo.pub, frodo.priv

Before leaving the Shire, Gandalf and Frodo give their public keys (gandalf.pub, frodo.pub) to everyone. They also publish these keys in the big book of known keys. And to make sure everyone recognises the link between these specific public keys and themselves, they create some selfies (showing the keys as QR codes on their T-Shirts). Feeling joyful, they even make a song using the first ten letters of their public keys.

During their perilous adventures, they send back news and jokes to the Shire by signing the message (see Digital Signature) with their private keys. This is what happens when Gandalf signs a message and someone in the Shire checks it for authenticity (remember, here the message is not encrypted):

    Gandalf: (message, gandalf.priv) ==> (message, signature) ==> Shire
    Shire:   (message, signature)    ==> check (message, signature, gandalf.pub) ==> true / false

### How do we protect sensitive data from being read by unauthorised people ?

If the scenario is simply Gandalf sending a private message to Frodo, this is what he does:

    Gandalf: (message, frodo.pub) ==> unreadable junk
    Frodo:   (unreadable junk, frodo.priv) ==> Gandalf's message

This is the basic principle of asymmetric cryptography. This is what https is all about. But there is a catch: Gandalf must be certain that he is using Frodo's public key. In this case, it's ok since they met and exchanged the keys. But when Samwise joins the batch, Gandalf does not have his public key. How can he get it, making sure Sauron does not intercept messages and exchanges keys (man in the middle attack):

    Gandalf: "Samwise, send me your public key, I have a message for you"
    Samwise: "Sure"
    (samwise.pub) ==> Sauron hooked on the network ==> (sauron.pub) ==> Gandalf
    Gandalf: "Thanks, here is the message"
    (message, sauron.pub) ==> junk ==> Sauron (junk, sauron.priv) ==> message (Sauron reads the message)
                                              (message, samwise.pub) ==> junk ==> Samwise

As we see, Sauron is intercepting all communications and decrypting, reencrypting all messages. One way to solve this is to use a "known authority" to sign the public key. Frodo could do it by signing a message containing Samwise's public key:

    Frodo: ("Hi Gandalf, this is Samwise's public key: LKJASFASLFIO...", frodo.priv) ==> signed message

Another solution is to use social media to post the public key (since it would be hard for an attacker to control all of a users social network accounts, post the fake key there and not having the user notice).

### How do we share sensitive data with a group ?

This is where the RING comes into play ! If we want to encrypt communication within a possibly large group of people, we cannot encrypt each and every message with every user's public key. So what we do is we generate a public/private key pair, just for this project. Let's call this project.pub and project.priv.

When writing messages to the group, authorised users encrypt the messages with the project.pub key. When they need to read, they use the project.priv key.

In order to grant access to the project to a newcomer (Samwise), Frodo or Gandalf simply add the project keys to Samwise's keyring. How do they do this ?

    Gandalf: (project.priv, samwise.pub) ==> project-samwise.junk ==> public storage of Samwise keyring
    Samwise: (project-samwise.junk, samwise.priv) ==> project.priv

To revoke access, a new key pair must be created and content has to be reencrypted with the new project keys.

Notice that we do not need to encrypt the project.pub key. And even though the data is now encrypted, it should still be signed (to know who wrote what in the group).

### How do we avoid app spamming ?

TODO...