About Query `as` HTML...

Starting with this sample code, from https://github.com/amark/gun/blob/master/examples/contact/index.html:
```
       <div id="people" class="hue2 page">
            <ul name="users" class="pad">
                <h3>Here is a list of people:</h3>
                <li name="#">
                    <ul>
                        <a href="person" name="#" class="whitet act">
                            <li>
                                <span name="who">
                                    <b name="name"></b>
                                </span>
                                <i>~</i><i name="alias" class="sort"></i>
                                <input name="pub">
                            </li>
                        </a>
                    </ul>
                </li>
            </ul>
        </div id="people">
```
Note that `as` looks for nested HTML name attributes and generates gun chains, in this case, as follows:
```
gun.get('users').map().map().get('who').get('name').on(bindToB);
gun.get('users').map().map().get('alias').on(bindToI);
gun.get('users').map().map().get('pub').on(bindToInput);
```
(NOTE: "#" equals "map" in `as`.)

However, since gun caches and reuses chains, this becomes:
```
var nest = gun.get('users').map().map();
nest.get('who').get('name').on(bindToB);
nest.get('alias').on(bindToI);
nest.get('pub').on(bindToInput);
```
So `as` logically maps the HTML directly to gun chains, and # represents "item in list", telling `as` to reuse the HTML model multiple times in a list, not just as a single item.

You might be asking why there are two maps as well...?
It actually has nothing to do with `as` or `map` but with how the P2P contact app (from which this code was taken) models its data.

NORMALLY you'd think the data is modeled as `gun.get('user').get('alice').get('profile').get('name')` ...
but it isn't because in a P2P app, alice cannot be globally unique (unless you were to run a blockchain that enforced that).

Multiple users might register the name alice (the app TRIES to prevent this, but it is not guaranteed) and,
as a result, the table of users doesn't have users as the records.

The table of users has username records that themselves are ANOTHER table.
That other table is a table of public keys that are associated with that username.
So if we want to display all the users, we need to display every user associated with each username, for all usernames.

THUS the query `as` HTML (note that sentence, it is where the name `as` comes from) is
`users.#.#.who.name`as nested HTML elements.

That query maps to the JS gun query as:
`gun.get('users').map().map().get('who').get('name')`
(and of course the other ones, for the alias, public key, etc. that I wanted to display in the app).

**So the cool thing is you can build 90%+ of your app SIMPLY by writing HTML with named attributes**
**and `as` automatically does 2 way binding to it!**

**PLUS since we added a SEA tag we also get automatic security/permissions on those 2 way bindings without writing any extra logic!**

As of today `as` is very alpha and supports only primitive behavior ... and isn't nearly as powerful as React/Vue/Riot, etc. My hope is either:

A) somebody builds gun bindings for react/vue/riot so that way people don't use `as` because those bindings are just as powerful. (personally, I prefer HTML based approach like with `as` and riot, not JS based approached as in react and vue.)

B) if no other UI Framework evolves the HTML framework, I'll probably continuing adding features to `as` to be more expressive/powerful, but not a priority for me.