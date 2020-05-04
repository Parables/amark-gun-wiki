Let's create a simple todo application with GUN.

>This page contains live code editors with live preview. You can change the code if you like and see the effect straight away.

## The basic html

First let's create the html:

```html
::: {codepen: 'link', tab1: 'codemirror'} :::
::: {startblock: '1'} :::
<html>
  <body>
    <h1>Todos</h1>

    <ul></ul>
    
    <form><input><button>Add</button></form>

::: {endblock: '1'} :::
::: {startblock: '2'} :::
    <style>
      ul { padding: 0; }
      li { display: flex; }
      li span { width: 150px; word-break: break-all; }
      img { height: 20px; margin-left: 8px; cursor: pointer; }
      input { width: 150px; margin-right: 8px; }
      input[type='checkbox'] { width: auto; }
    </style>
  </body>
</html>
::: {endblock: '2'} :::
```

Here, the `<ul></ul>` will be used by the JavaScript code will will soon create.

The `<form>` will be used to add new todos.

You can ignore the `<style>` tag. This just contains some simple styling of (future) things on the page.

## Add GUN

Next let's add GUN and make the form operational.

At line 8 (after the form) let's insert this code:

```html
::: {codepen: 'link', tab1: 'codemirror'} :::
::: {startblock: '3'} :::
::: {hide: 'start'} :::
::: {insertblock: '1'} :::
::: {hide: 'end'} :::
    <!-- Load GUN itself. -->
    <script src="https://cdn.jsdelivr.net/npm/gun/gun.js"></script>

    <!-- Load jQuery to help make things a bit easier. -->
    <script src="https://code.jquery.com/jquery-1.12.4.min.js"></script>

    <script>
      // Initialize GUN and tell it we will be storing all data under the key 'todos'.
      var todos = Gun().get('todos')
      
      // Get the form element.
      var form = $('form')
      // Listen for submits of the form.
      form.on('submit', function (event) {
        // Get the input element.
        var input = form.find('input')
        // Tell GUN to store an object,
        // with as title the value of the input element.
        todos.set({title: input.val()})
        // Clear the input element, so the user is free to enter more todos.
        input.val('')
        // Prevent default form submit handling.
        event.preventDefault()
      })

::: {endblock: '3'} :::
::: {startblock: '4'} :::
    </script>
::: {hide: 'start'} :::
    
    <!-- Just some minimal styling. -->
::: {insertblock: '2'} :::
::: {hide: 'end'} :::
::: {endblock: '4'} :::
```

Here, we first load GUN itself.

Then we initialize GUN and tell it we will place all data under the key `todos`.

Finally we handle what happens when the user clicks the `Add` button. The key line is `todos.set({title: input.val()})`. Here we tell GUN to store our object. In GUN we use `set()` to store values in a list (set, array). So this object is added by GUN to the list of todos.

>When you run the code, type something in the input and click the `Add` button, it is already stored by GUN. But we will not yet see it appear. We will fix that in the next step.

## Show the todos

Now we will show the list of all existing todos on the screen.

At line 33 (before `</script>`) insert this code:

```html
::: {codepen: 'link', tab1: 'codemirror'} :::
::: {startblock: '5'} :::
::: {hide: 'start'} :::
::: {insertblock: '3'} :::
::: {hide: 'end'} :::
      // Listen to any changes made to the GUN todos list.
      // This will be triggered each time the list changes.
      // And because of how GUN works, sometimes even multiple times per change.
      todos.map().on(function (todo, id) {
        // Check if the todo element already exists.
        // This can happen because GUN sometimes sends mulitple change events for the same item.
        var li = $('#' + id)
        // Does is not yet exist?
        if (!li.get(0)) {
          // Create it.
          // Set the id to the GUN id of the item.
          // GUN automatically creates id's for all items.
          // Finally set the new todo element to the end of the list.
          li = $('<li>').attr('id', id).appendTo('ul')
        }
        // Does the GUN item contain any data?
        // (It sends null if it was removed from GUN.)
        if (todo) {
::: {endblock: '5'} :::
          // Create an element with the title of the GUN item in it.
          var html = todo.title
::: {startblock: '6'} :::
          // Set it to the element.
          li.html(html)
::: {endblock: '6'} :::
::: {startblock: '6a'} :::
        }
      })
::: {endblock: '6a'} :::
::: {hide: 'start'} :::
::: {insertblock: '4'} :::
::: {hide: 'end'} :::
```

By doing `todos.map().on(function (todo, id)` we tell GUN we want to listen to any and all changes at the todo list. GUN will call this function for each todo that is already in the list. And it will also call this function whenever a todo is added (by `set`).

Please be aware that this function can (and often will) be called more than once for each item, even if the item was not changed. This is not a bug, but a consequence of how GUN works internally. This means inside the function you need to filter (based on `id`) whether you already have the item (so you can overwrite) or not (so you can add).

So in the code here we add a `li` element for each todo, with the `id` attribute of the element set to the `id` of the todo that is provided (and was automatically generated) by GUN.

Then each time a 'new' todo is received we can check if we already have a `li` element with the same `id`. If we do not, we can insert a new one. And next we set the content (html) of the element to the todo's title.

We can now test our changes. Type something in the `input` and click `Add`. The new todo should then be shown on the screen.

## Edit todos

Now we will make each todo editable.

Change line 53 (`var text = todo.title`).

And add 2 new functions before line 59 (`</script>`):

```html
::: {codepen: 'link', tab1: 'codemirror'} :::
::: {startblock: '7'} :::
::: {hide: 'start'} :::
::: {insertblock: '5'} :::
::: {hide: 'end'} :::
          // Create an element with the title of the GUN item in it.
          var html = '<span onclick="clickTitle(this)">' + todo.title + '</span>'
::: {endblock: '7'} :::
::: {hide: 'start'} :::
::: {insertblock: '6'} :::
::: {insertblock: '6a'} :::
::: {startblock: '8'} :::

::: {hide: 'end'} :::
      // What to do when a todo's text is clicked.
      function clickTitle (element) {
        // Get the (jQuery) element of the text.
        element = $(element)
        // Check if the element does not yet contain an input field.
        // So we will only add one input field when clicked multiple times.
        if (!element.find('input').get(0)) {
          // Turn the elements text into an input.
          element.html('<input value="' + element.html() + '" onkeyup="keypressTitle(this)">')
        }
      }
      
      // What to do when Enter is pressed while editing a todo.
      function keypressTitle (element) {
        // Is Enter pressed?
        if (event.keyCode === 13) {
          // Get the GUN item with the id that we store in the element.
          // And tell GUN to update the title of the todo item.
          todos.get($(element).parent().parent().attr('id')).put({title: $(element).val()})
        }
      }
::: {endblock: '8'} :::
::: {hide: 'start'} :::
::: {insertblock: '4'} :::
::: {hide: 'end'} :::
```

When a todo item is clicked, we turn it into an `input` so the user can change the text.

Then we wait for the user to press the `Enter` key. When he/she/it :-) does, we will tell GUN store the new text. This happens here: `todos.get($(element).parent().parent().attr('id')).put({title: $(element).val()})`.

`$(element).parent().parent().attr('id')` simply gets the `id` that we stored in the `li` element.

`todos.get(....id)` tells GUN we want to use the todo with that specific `id`.

Finally `put({title: $(element).val()})` will change the title of the todo.

Notice that for inserting a todo in the todos list we used `set`, whereas now we use `put`. The difference is that `set` will add a todo to the todo list (similar to `push` for arrays, though not the same; see the documentation for `set`), but `put` is done for one specific item (node); not on a list.

We can now try editing a todo by clicking it, changing the text and pressing the `Enter` key.

## Add a done state

Now we will add a checkbox in front of each todo, so the user can indicate whether the todo is done or not.

After line 53 (`var html = ....`) add a few lines for the checkbox.

And add a new function before line 80 (`</script>`):

```html
::: {codepen: 'link', tab1: 'codemirror'} :::
::: {startblock: '9'} :::
::: {hide: 'start'} :::
::: {insertblock: '7'} :::
::: {hide: 'end'} :::
          // Add a checkbox in front and check it if the GUN item has a done state.
          html = '<input type="checkbox" onclick="clickCheck(this)" ' + (todo.done ? 'checked' : '') + '>' + html
::: {endblock: '9'} :::
::: {hide: 'start'} :::
::: {insertblock: '6'} :::
::: {insertblock: '6a'} :::
::: {startblock: '10'} :::
::: {insertblock: '8'} :::
::: {hide: 'end'} :::
      
      // What to do when a checkbox is clicked.
      function clickCheck (element) {
        // Set the done state of the GUN todo item.
        // Notice that we do not need to put the full object (including it's title state).
        // GUN will only change the done property of the item and leaves the other properties (like title) intact.
        todos.get($(element).parent().attr('id')).put({done: $(element).prop('checked')})
      }
::: {endblock: '10'} :::
::: {hide: 'start'} :::
::: {insertblock: '4'} :::
::: {hide: 'end'} :::
```

We add a `<input type="checkbox">` in front of the title and give it the done state of the todo item we got from GUN. Of course no existing todo had this todo property just yet, but that is fine.

When the user clicks the checkbox, again we get the todo item (node) from GUN via it's `id` and tell GUN to store the new todo state. If the todo property what not yet stored in the todo item, it will be added. If it already existed, it will be overwritten.

Notice that we did not provide the title in the object. But that does not mean GUN will remove the 'old' title. GUN does not replace objects set by `put`, but rather merges the new object into the old object.

## Delete todos

Finally let's add some functionality for delete todo's.

After line 55 (`html = ....`) add a few lines for the trashcan.

Also after line 57 (`li.html(html)`) add a few lines for removing the todo's `li`.

And add a new function before line 90 (`</script>`):

```html
::: {codepen: 'link', tab1: 'codemirror'} :::
::: {startblock: '11'} :::
::: {hide: 'start'} :::
::: {insertblock: '9'} :::
::: {hide: 'end'} :::
          // Add a trashcan icon and make it clickable.
          html += '<img onclick="clickDelete(this)" src="https://cdnjs.cloudflare.com/ajax/libs/foundicons/3.0.0/svgs/fi-x.svg"/>'
::: {hide: 'start'} :::
::: {insertblock: '6'} :::
::: {hide: 'end'} :::
        } else {
          // The item was removed from GUN, because we got null.
          // Delete it from the screen.
          li.remove()
::: {hide: 'start'} :::
::: {insertblock: '6a'} :::
::: {insertblock: '10'} :::
::: {hide: 'end'} :::

      // What to do when a trashcan is clicked.
      function clickDelete (element) {
        // In GUN the way to delete an item, is to set it's value to null.
        // This is because of how graph databases, like GUN, work internally.
        todos.get($(element).parent().attr('id')).put(null)
      }
::: {hide: 'start'} :::
::: {insertblock: '4'} :::
::: {hide: 'end'} :::
::: {endblock: '11'} :::
```

In a distributed graph database, like GUN, data can not literally be deleted. Hence GUN does not provide a `delete` function. What you need to do is overwrite the old value with `null`.

So here with `todos.get($(element).parent().attr('id')).put(null)`, whenever the user clicks the trashcan we overwrite the old todo item with `null`.

This will trigger the function in `todos.map().on(function (todo, id)`, but instead of a new or changed todo object, `todo` will now be `null`. So we know we must now delete the `li` instead of change it's content.

## Now refresh the browser

We now have a fully working todo application with create, update and delete (CRUD). All data is stored in GUN.

NOW REFRESH THE BROWSER! Yes, really. No worries. Just do it.

Notice how all todo data you entered still is there?

That is because GUN stores (caches) all it's data in the browser's `localStorage`. Also, if you would connect the app to a GUN server and also open the app in different computers (connecting to that same server), all data would be continuously synced.

So you have now created an offline-first real-time multi-user todo app with GUN. Congratulations! You rock!

## The full code

As a final review, here is the full code of the GUN todo app we just created:

```html
::: {codepen: 'link', tab1: 'codemirror'} :::
::: {insertblock: '11'} :::
```