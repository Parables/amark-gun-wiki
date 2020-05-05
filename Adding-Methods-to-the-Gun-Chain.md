# Abstraction layers

Extending gun's API

All of gun's methods are exposed through it's prototype, or as we call it, the gun *chain*. You can define your own methods by attaching them to that chain. It behaves exactly like an object prototype, so your extension will be accessible to every gun instance. Here's how you extend `Gun.chain`:

```javascript
Gun.chain.yourExtension = function ([args]) {
  // your code
  
  return this;
}

new Gun().get('data').yourExtension()...
```

> Note the capital "G": We're talking about the Gun constructor function.

Returning `this` at the end of your function ensures that other methods can be chained after your extension.

You can access gun data from inside of your extension buy harnessing the `this` value. It is a reference to your gun context, as well as all of it's core (or extended) functionality. You can read, write and change gun using `this`. Here's what that looks like in practice:

```javascript
Gun.chain.log = function () {
  // our gun instance
  var gun = this;
  
  // we have full access to gun and it's functionality
  gun.map().val();
  
  // return gun so we can chain other methods off of it
  return gun;
}
```

So to finish out the tutorial, let's make a simple extension called `replace` that nulls out data in our context and replaces it with something else.

```javascript
Gun.chain.replace = function (replacement) {

  // grab a reference to gun
  var gun = this;
  
  // null out the data
  gun.put(null, function () {
    
    // replace it with something else
    gun.put(replacement);
  });
  
  return gun;
};


// now we can use it in our app
var user = new Gun(serverURL).get('profile')

// replace our nickname and log it to the console
users.path('nickname').replace('UberLlama').val()
```

That should get you started with extensions. If you have any questions, post an issue to this repository or our [gitter channel](https://gitter.im/amark/gun) and we'll do our best to answer them! We'd love to hear from you!