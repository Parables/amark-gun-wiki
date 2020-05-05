# Basics
> Gun.js have some core APIs. It's essential for you to understand those basics before you getting started.

* gun.get
* gun.put
___

* gun.once
* gun.on
___

* gun.set
* gun.map
* gun.unset

## Step 0, know how to install and start a gun instance
Installation
```
npm install gun
```

Start Server
```
cd node_modules/gun && npm start
```

Import it
```javascript
import Gun from 'gun/gun'
require('gun/sea') // matters if you want to use User System
require('gun/lib/unset.js')

var gun = Gun(['http://localhost:8765/gun'])
let user = gun.user()
```

## Step 1, know what is gun
`gun` is an object. It's kind of like `a class` in python.

What can you do with a class? call functions from it! Just add a dot before your function name, that should do it. For example: `gun.get()`

## Step 2, know how to set a variable
Just call 
```javascript
gun.get("my_name").put("yingshaoxo")
```

It is equate to python version of
```python
gun['my_name'] = 'yingshaoxo'
``` 

## Step 3, know how to read a variable
Just call
```javascript
gun.get("my_name").once((value) => {
    window.console.log(value)
    // it'll print out "yingshaoxo"
})
```

It is equate to python version of
```python
value = gun['my_name']
print(value)
# it'll print out "yingshaoxo"
``` 

## Step 4, know how to get a variable when it's getting changed
Just call
```javascript
gun.get("my_name").put("yingshaoxo")
gun.get("my_name").on((value) => {
    window.console.log(value)
    // it'll print out "yingshaoxo"
})

gun.get("my_name").put("was great!")
// it'll print out "was great" even if you didn't call once() function again
```

## Step 5, how to use `set()` function
`set` is a mathematical term. Every element or object in a set should be different than each other in that set. (If you knew python, you should know what I mean.)

Say if we want to save two objects into gun database, what we do is:
```javascript
object_a = {
    'username': "yingshaoxo",
    'password': 'i_love_you'
}

object_b = {
    'username': "dear_reader",
    'password': 'i_love_you_too'
}

gun.get("users").set(object_a)
gun.get("users").set(object_b)

// Now you should have object_a and object_b under the property of "users"
// Gun set is very special, he can only save object, no list, no map, no variable, just object, remember that!
```

## Step 6, how to read data from our `set`?
```javascript
gun.get("users").map().once((object) => {
    let username = object.username
    let password = object.password
    window.console.log(username, password)
})
```

## Step 7, how to remove an object from our `set`?
```javascript
require('gun/lib/unset.js')

object_a = {
    'username': "yingshaoxo",
    'password': 'i_love_you'
}

gun.get("users").unset(object_a)
// Now only the object_b is left
```

# User System
> For the previous section, we've discussed the basics. If you smart enough, you'll notice by using the global `gun` object to save and read data is not secure. Everyone who is able to use `gun.js` can change the data. So we need to introduce a secure system for you: the User System.

* gun.user.create
* gun.user.auth
* gun.user.is
* gun.user.leave

## First, we create a user
```javascript
let username = "yingshaoxo"
let password = "i_love_you"

user.create(username, password, (feedback) => {
    if ("ok" in feedback) {
        window.console.log("You have created a new user!")
    } else if ("err" in feedback) {
        window.console.log(feedback.err)
    }
})
```

## Then, we log into our account
```javascript
let username = "yingshaoxo"
let password = "i_love_you"

user.auth(username, password, (feedback) => {
    if ("gun" in feedback) {
        window.console.log("You have logged into your account!")
    } else if ("err" in feedback) {
        window.console.log(feedback.err)
    }
})
```

## How to know if a user is currently logged in?
```javascript
if (user.is) {
    window.console.log("Welcome! You are my god.")
} else {
    window.console.log("Go away! You are nobody.")
}
```

## Can we logout?
```javascript
user.leave()
```

## Save data under a specific user
```javascript
let username = "yingshaoxo"
let password = "i_love_you"

let gun_of_this_user = null

user.auth(username, password, (feedback) => {
    if ("gun" in feedback) {
        gun_of_this_user = feedback.gun
    }
})

gun_of_this_user.get("my_data").put({
    love: "beautiful woman",
    love_to_do: "speak English and coding"
})
```

> The author of gun.js, `Mark Nadal` think that everyone in this world should have a gun to protect their own freedom. I agree with that. And I think the whole US people will also stand on this.

This data will only be saved under the user `yingshaoxo`. You will not be able to get it from the `global gun objct`.


## Find a specific user, and read his data
### Find a user by name
```javascript
let username = "yingshaoxo"

gun.get('~@' + username).once((data, _) => {
    let obj = data["my_data"]

    let a_thing = obj.love
    let an_action = obj.love_to_do

    window.console.log("this user love " + a_thing + " and love to " + an_action)
});
```

### Find a user by public key
> Sometimes username is not enough for identifying a unique person, so we use `public_key` instead.

```javascript
let username = "yingshaoxo"
let password = "i_love_you"

let public_key = null

/*
user.create(username, password, (feedback) => {
    if ("ok" in feedback) {
        public_key = feedback.pub
    }
})
*/

user.auth(username, password, (feedback) => {
    if ("gun" in feedback) {
        public_key = feedback.get.slice(1,)
    }
})

gun.user(public_key).once((data) => {
    let obj = data["my_data"]

    let a_thing = obj.love
    let an_action = obj.love_to_do

    window.console.log("this user love " + a_thing + " and love to " + an_action)
})
```

# Conclusion
That's all I could teach you.

Are you ready to die for the freedom of humanity? 

Let's Go!