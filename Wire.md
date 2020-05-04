 ## Mandatory

Read **[how to port GUN](./Porting-GUN)** first. This page is just an outline.

 ## UTF8

UTF8 is used as the default encoding.

 1. `{` or `[`, if a wire message (after decompressing) starts with one of these symbols, use `JSON` (not RAD) to parse it. `[` will be a list of messages, just iterate over them and call them individually.

RAD is not used on the wire yet.

 ## Envelope

GUN messages may have extra properties on it, especially if DAM is used, that make up the message envelope.

 ## Message

 - `#` message identifier to deduplicate against. **mandatory**
 - `@` is the identifier that this message is acknowledging a reply to.
 - `get` is a [lexical](#Lexical) lookup.
 - `put` is a [graph](./Porting-GUN#graph).
 - `err` text.

Non-standard:

 - `ok` text.

 ## WIP

... more coming soon ...