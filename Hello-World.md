Let's create a simple Hello World app in 1 small file that will contain both our HTML and JS code:

> This editors has a live preview. You can change the code and see the result right away.

```html
::: {codepen: 'link', tab1: 'codemirror'} :::
<html>
  <body>
    <h1>Hello</h1>
    <p>What is your name?</p>
    <form><input><button>Submit</button></form>

    <script src="https://cdn.jsdelivr.net/npm/gun/examples/jquery.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/gun/gun.js"></script>
    <script>
      var gun = Gun()

      $('form').on('submit', function (event) {
        event.preventDefault()
        var input = $('form').find('input')
        gun.get('hello').put({ name: input.val() });
        input.val('')
      })

      gun.get('hello').on(function(data, key) {
        var h1 = $('h1')
        h1.text('Hello ' + data.name)
      })
    </script>
  </body>
</html>
```

Try it out but entering your name above. The hello message will show your name.

Now refresh the whole page. After the refresh your name will still be there as GUN will load it for you.

### What's going on?

First we load GUN with `<script src="https://cdn.jsdelivr.net/npm/gun/gun.js"></script>` and initialize it with `var gun = Gun()`.

Then each time the user submits a new name `$('form').on` we update the GUN node with an object `{ name: input.val() }` that we `put` on the key `'hello'` with `gun.get('hello').put(...)`.

Finally, by using `gun.get('hello').on(...)` we get realtime updates each time the `'hello'` node changes, so we can display it in the `h1`.

Notice that when we refresh the page `gun.get('hello').on(...)` is called automatically because GUN knows there is data already existing on the `'hello'` node.

### Next up?

 - Figure out how to best use this documentation system in [the next step](Next-Steps).
 - Watch the 2 minute [Crash Course](Crash-Course)?.
 - Hands-on learn answers to all your questions in [THE INTERACTIVE TUTORIAL](Todo-Dapp)!