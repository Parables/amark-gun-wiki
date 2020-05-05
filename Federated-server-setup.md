# Introduction

While GUN is primarily targeted at fully distributed systems with servers only acting as a known entry point into the network, some applications may require all data to be private.

One way of achieving this is to encrypt all connections used by GUN, resulting in corrupt messages if a peer does not know the decryption key. Encrypting all connections prevents unauthorized GUN instances of talking to peers it doesn't know the encryption key for.

# Theory

Let's assume we have websocket library X, functional in both the browser and within Node.JS, which encrypts all connections using a symmetric encryption algorithm with a single key, yet otherwise works the same as for example [ws](https://npmjs.com/package/ws)

During the initialization of your GUN instance, you would indicate to use library X as it's websocket library instead of the default one. This would limit your instance to talking to only instances using the same setup, rendering your data & connections private.

On top of your instances, you now control all data flow in- and out of your data storage.

# Example

To proof this theory, [gun-private](https://github.com/finwo/gun-private) was created. It's using [RC4](https://en.wikipedia.org/wiki/RC4) as the encryption algorithm and encrypts each websocket separately through a small wrapper.