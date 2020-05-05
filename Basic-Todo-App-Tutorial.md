Let's create a simple todo application with GUN.

>This page contains code editors with live preview. You can change the code and see the result straight away.

First let's create the html:

```html
::: {codepen: 'link', tab1: 'codemirror'} :::
::: {editor: 'main'} :::
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

::: {nextstepcompare: 'none'} :::
::: {step: 'Add GUN'} :::

Next let's add GUN and make the form operational.

```html
::: {codepen: 'link', tab1: 'codemirror'} :::
::: {editor: 'main'} :::
::: {codepen: 'link', tab1: 'codemirror'} :::
::: {insertblock: '1'} :::
<!-- Insert here -->

::: {insertblock: '2'} :::
```

Replace the `insert here` line with this code:

```html
::: {startblock: 'a'} :::
    <script src="https://cdn.jsdelivr.net/npm/gun/gun.js"></script>
    <script src="https://code.jquery.com/jquery-1.12.4.min.js"></script>
    <script>
      var todos = Gun().get('todos')

      $('form').on('submit', function (event) {
        var input = $('form').find('input')
        todos.set({title: input.val()})
        input.val('')
        event.preventDefault()
      })

::: {endblock: 'a'} :::
::: {startblock: 'b'} :::
    </script>
::: {endblock: 'b'} :::
```

::: {nextstepcompare: 'start'} :::
```
::: {startblock: '3'} :::
::: {insertblock: '1'} :::
::: {insertblock: 'a'} :::
::: {endblock: '3'} :::
::: {startblock: '4'} :::
::: {insertblock: 'b'} :::
    
::: {insertblock: '2'} :::
::: {endblock: '4'} :::
```
::: {nextstepcompare: 'end'} :::

Here, we first load GUN itself.

Then we initialize GUN and tell it we will place all data under the key `todos`.

Finally we handle what happens when the user clicks the `Add` button. The key line is `todos.set({title: input.val()})`. Here we tell GUN to store our object. In GUN we use `set()` to store values in a list (set, array). So this object is added by GUN to the list of todos.

>When you run the code, type something in the input and click the `Add` button, it is already stored by GUN. But we will not yet see it appear. We will fix that in the next step.

::: {step: 'Show the todos'} :::

Now we will show the list of all existing todos on the screen.

```html
::: {codepen: 'link', tab1: 'codemirror'} :::
::: {editor: 'main'} :::
::: {codepen: 'link', tab1: 'codemirror'} :::
::: {insertblock: '3'} :::
// Insert here
::: {insertblock: '4'} :::
```

Replace the `Insert here` line with this code:

```javascript
::: {startblock: 'c'} :::
      todos.map().on(function (todo, id) {
        var li = $('#' + id)
        if (!li.get(0)) {
          li = $('<li>').attr('id', id).appendTo('ul')
        }
        if (todo) {
::: {endblock: 'c'} :::
::: {startblock: 'd'} :::
          var html = todo.title
::: {endblock: 'd'} :::
::: {startblock: '6'} :::
          li.html(html)
::: {endblock: '6'} :::
::: {startblock: '6a'} :::
        }
      })
::: {endblock: '6a'} :::
```

::: {nextstepcompare: 'start'} :::
```
::: {startblock: '5'} :::
::: {insertblock: '3'} :::
::: {insertblock: 'c'} :::
::: {endblock: '5'} :::
::: {insertblock: 'd'} :::
::: {insertblock: '6'} :::
::: {insertblock: '6a'} :::
::: {insertblock: '4'} :::
```
::: {nextstepcompare: 'end'} :::

By doing `todos.map().on(function (todo, id)` we tell GUN we want to listen to any and all changes at the todo list. GUN will call this function for each todo that is already in the list. And it will also call this function whenever a todo is added (by `set`).

Please be aware that this function can (and often will) be called more than once for each item, even if the item was not changed. This is not a bug, but a consequence of how GUN works internally. This means inside the function you need to filter (based on `id`) whether you already have the item (so you can overwrite) or not (so you can add).

So in the code here we add a `li` element for each todo, with the `id` attribute of the element set to the `id` of the todo that is provided (and was automatically generated) by GUN.

Then each time a 'new' todo is received we can check if we already have a `li` element with the same `id`. If we do not, we can insert a new one. And next we set the content (html) of the element to the todo's title.

We can now test our changes. Type something in the `input` and click `Add`. The new todo should then be shown on the screen.

::: {step: 'Edit todos'} :::

Now we will make each todo editable.

```html
::: {codepen: 'link', tab1: 'codemirror'} :::
::: {editor: 'main'} :::
::: {insertblock: '5'} :::
::: {insertblock: 'd'} :::
::: {insertblock: '6'} :::
::: {insertblock: '6a'} :::

// Insert here
::: {insertblock: '4'} :::
```

Change line the line `var html = todo.title` to:

```javascript
::: {startblock: 'e'} :::
          var html = '<span onclick="clickTitle(this)">' + todo.title + '</span>'
::: {endblock: 'e'} :::
```

And replace the `Insert here` line with these two functions:

```javascript
::: {startblock: 'f'} :::
      function clickTitle (element) {
        element = $(element)
        if (!element.find('input').get(0)) {
          element.html('<input value="' + element.html() + '" onkeyup="keypressTitle(this)">')
        }
      }

      function keypressTitle (element) {
        if (event.keyCode === 13) {
          todos.get($(element).parent().parent().attr('id')).put({title: $(element).val()})
        }
      }
::: {endblock: 'f'} :::
```

::: {nextstepcompare: 'start'} :::
```html
::: {startblock: '7'} :::
::: {insertblock: '5'} :::
::: {insertblock: 'e'} :::
::: {endblock: '7'} :::
::: {insertblock: '6'} :::
::: {insertblock: '6a'} :::
::: {startblock: '8'} :::
::: {insertblock: 'f'} :::
::: {endblock: '8'} :::
::: {insertblock: '4'} :::
```
::: {nextstepcompare: 'end'} :::

When a todo item is clicked, we turn it into an `input` so the user can change the text.

Then we wait for the user to press the `Enter` key. When he/she/it :-) does, we will tell GUN store the new text. This happens here: `todos.get($(element).parent().parent().attr('id')).put({title: $(element).val()})`.

`$(element).parent().parent().attr('id')` simply gets the `id` that we stored in the `li` element.

`todos.get(....id)` tells GUN we want to use the todo with that specific `id`.

Finally `put({title: $(element).val()})` will change the title of the todo.

Notice that for inserting a todo in the todos list we used `set`, whereas now we use `put`. The difference is that `set` will add a todo to the todo list (similar to `push` for arrays, though not the same; see the documentation for `set`), but `put` is done for one specific item (node); not on a list.

We can now try editing a todo by clicking it, changing the text and pressing the `Enter` key.

::: {step: 'Add a done state'} :::

Now we will add a checkbox in front of each todo, so the user can indicate whether the todo is done or not.

```html
::: {codepen: 'link', tab1: 'codemirror'} :::
::: {editor: 'main'} :::
::: {insertblock: '7'} :::
// Insert here first
::: {insertblock: '6'} :::
::: {insertblock: '6a'} :::
::: {insertblock: '8'} :::

// Insert here also
::: {insertblock: '4'} :::
```

Replace the `Insert here first` line with this line:

```javascript
::: {startblock: 'g'} :::
          html = '<input type="checkbox" onclick="clickCheck(this)" ' + (todo.done ? 'checked' : '') + '>' + html
::: {endblock: 'g'} :::
```

And replace the `Insert here also` line with this functions:

```javascript
::: {startblock: 'h'} :::
      function clickCheck (element) {
        todos.get($(element).parent().attr('id')).put({done: $(element).prop('checked')})
      }
::: {endblock: 'h'} :::
```

::: {nextstepcompare: 'start'} :::
```
::: {startblock: '9'} :::
::: {insertblock: '7'} :::
::: {insertblock: 'g'} :::
::: {endblock: '9'} :::
::: {insertblock: '6'} :::
::: {insertblock: '6a'} :::
::: {startblock: '10'} :::
::: {insertblock: '8'} :::
      
::: {insertblock: 'h'} :::
::: {endblock: '10'} :::
::: {insertblock: '4'} :::
```
::: {nextstepcompare: 'end'} :::

We add a `<input type="checkbox">` in front of the title and give it the done state of the todo item we got from GUN. Of course no existing todo had this todo property just yet, but that is fine.

When the user clicks the checkbox, again we get the todo item (node) from GUN via it's `id` and tell GUN to store the new todo state. If the todo property what not yet stored in the todo item, it will be added. If it already existed, it will be overwritten.

Notice that we did not provide the title in the object. But that does not mean GUN will remove the 'old' title. GUN does not replace objects set by `put`, but rather merges the new object into the old object.

::: {step: 'Delete todos'} :::

Finally let's add some functionality for delete todo's.

```html
::: {codepen: 'link', tab1: 'codemirror'} :::
::: {editor: 'main'} :::
::: {insertblock: '9'} :::
// Insert here first
::: {insertblock: '6'} :::
// Insert here next
::: {insertblock: '6a'} :::
::: {insertblock: '10'} :::

// Insert here also
::: {insertblock: '4'} :::
```

Replace the `Insert here first` line with this line:

```javascript
::: {startblock: 'i'} :::
          html += '<img onclick="clickDelete(this)" src="https://cdnjs.cloudflare.com/ajax/libs/foundicons/3.0.0/svgs/fi-x.svg"/>'
::: {endblock: 'i'} :::
```

Then replace the `Insert here next` line with this line:

```javascript
::: {startblock: 'j'} :::
        } else {
          li.remove()
::: {endblock: 'j'} :::
```

And replace the `Insert here also` line with this functions:

```javascript
::: {startblock: 'k'} :::
      function clickDelete (element) {
        todos.get($(element).parent().attr('id')).put(null)
      }
::: {endblock: 'k'} :::
```

::: {nextstepcompare: 'start'} :::
```
::: {startblock: '11'} :::
::: {insertblock: '9'} :::
::: {insertblock: 'i'} :::
::: {insertblock: '6'} :::
::: {insertblock: 'j'} :::
::: {insertblock: '6a'} :::
::: {insertblock: '10'} :::

::: {insertblock: 'k'} :::
::: {insertblock: '4'} :::
::: {endblock: '11'} :::
```
::: {nextstepcompare: 'end'} :::

In a distributed graph database, like GUN, data can not literally be deleted. Hence GUN does not provide a `delete` function. What you need to do is overwrite the old value with `null`.

So here with `todos.get($(element).parent().attr('id')).put(null)`, whenever the user clicks the trashcan we overwrite the old todo item with `null`.

This will trigger the function in `todos.map().on(function (todo, id)`, but instead of a new or changed todo object, `todo` will now be `null`. So we know we must now delete the `li` instead of change it's content.

::: {step: 'Well done'} :::

We now have a fully working todo application with create, update and delete (CRUD). All data is stored in GUN.

NOW REFRESH THE BROWSER! Yes, really. No worries. Just do it.

Notice how all todo data you entered still is there?

That is because GUN stores (caches) all it's data in the browser's `localStorage`. Also, if you would connect the app to a GUN server and also open the app in different computers (connecting to that same server), all data would be continuously synced.

So you have now created an offline-first real-time multi-user todo app with GUN. Congratulations! You rock!

As a final review, here is the full code of the GUN todo app we just created:

```html
::: {codepen: 'link', tab1: 'codemirror'} :::
::: {insertblock: '11'} :::
```