# Want to use GUN with Svelte?

 > Check out [Svelte GUN chat](https://github.com/vnglst/svelte-gundb-chat) for a good starting example!

Svelte has an [approach to reactivity](https://svelte.dev/blog/svelte-3-rethinking-reactivity) that couples very well with GUN, since you are able to use gun directly as a store within Svelte components. As a result, you can easily bind GUN data to Svelte in a way that feels like vanilla JavaScript, and is similar to the vanilla examples used in most of GUNs documentation.

Check out the examples below or try your hand in at a simple [Todos example with the Svelte REPL](https://svelte.dev/repl/634ff063c3424bb9a203bd38cbaa2d73?version=3.16.4)

We have two recommended approaches to bind data to GUN. One for use directly in Svelte components and one for writing stores that can be reused across components.

#### Using GUN in a single component
The recommended approach to bind GUN data directly to a Svelte component is this:
```javascript
<script>
import { gun } from "./gunSetup.js" // Only initialize GUN once

// Initialize a variable to store data from GUN
let store = {}

// Set up a GUN listener to update the value
gun.get("cats").map().on(function(data, key) {
    if (data) {
        store[key] = data
    } else {
        // gun.map() can return null (deleted) values for keys
        // if so, this else clause will update your local variable
        delete store[key]
        store = store
    }
})

// Reshape data to convenient variables for the view
$: cats = Object.entries(store)
</script>

{#each cats as [key, cat]}
<div>{cat.name}</div>
{/each}
```

Svelte will watch for updates to the `store` variable, and since we use the label `$` with the `cats` variable it will update the UI when both variables changes.

This approach only works in compiled `.svelte` files, since Svelte wraps the variables with the `invalidate()` function which will tell the components to update once the variables change.

#### Adding a Svelte store for GUN data
Luckily, we can add our own subscriber system in regular `.js` files by writing our own custom store. This approach is easy to work with, since you can easily add your own custom methods to write GUN data or even adding more `gun.get()` calls for nested data.

The store below is a simplified version of the official Svelte `writable` store. You can also check it out in the [interactive demo](https://svelte.dev/repl/53ecf1dc52c245faba204f3ca94d5cde?version=3.16.5)

```javascript
// stores.js
import Gun from "gun/gun"
const gun = Gun()

function gunStore(ref, methods = {}) {
  const store = {}
  const subscribers = []

  // Add a listener to GUN data
  ref.on(function(data, key) {
      /* If the ref._get matches the data key it means we are getting
       * data from a call to gun.get(), and so we don't need the store
       * to be an object with nested data. Otherwise we are getting data
       * from a call to map() and should nest the data in an object
       */
      if (ref._.get === key) {
        store = data
      } else if (!data) {
        /* This clause will not work as intended on null values / false / 0
         * if you use such data consider subscribing to a parent node instead
         * eg. gun.get("temperature") instead of gun.get("temperature").get("value").
         * Or you can pass a validate() function (TODO: add example)
         */
        delete store[key]  
      } else {
         store[key] = data
      }
      // Tell each subscriber that data has been updated
      for (let i = 0; i < subscribers.length; i += 1) {
        subscribers[i](store)
      }
  })
  

  function subscribe(subscriber) {
    subscribers.push(subscriber)
    
    // Publish initial value
    subscriber(store)

    // return cleanup function to be called on component dismount
    return () => {
      const index = subscribers.indexOf(subscriber)
      if (index !== -1) {
        subscribers.splice(index, 1)
      }
      if (!subscribers.length) {
        ref.off()
      }
    }
  }

  return { ...methods, subscribe }
}

const ref = gun.get("messages")
export const messages = customStore(ref.map(), {
  add: text => ref.set({ text, sender: "moi", icon: "😺" }),
  delete: key => ref.get(key).put(null)
})
```

This pattern makes for very clean markup in the component itself

```html
<script>
import { messages } from "./stores.js"
</script>

<input placeholder="write message" on:change={e => messages.add(e.target.value) && (e.target.value = "")} />

{#each Object.entries($messages) as [key, {sender, text, icon}] (key)}
<div style="padding: .4rem">
  {icon} {sender}: {text}
  <a href="#" on:click|preventDefault={() => messages.delete(key)}>delete</a>
</div>
{/each}
```

## Tutorial
In this tutorial we will build a very basic todo application.

Before we begin you should be familiar with Svelte and its APIs, as we will only focus on the data binding to GUN.

### Step 1 - Set up GUN

Let's create a new file called `initGun.js`. It's better to do this in a separate file since you only want to initialize gun once. We'll import the gun object into the Svelte components from this file.

```javascript
// initGun.js
import Gun from "gun/gun"
export const gun = Gun()
```

```html
// App.svelte
<script>
import { gun } from "./initGun.js"
</script>
```

### Step 2 - Create an input to add Todos
```html
// App.svelte
<script>
// ...

let input = ""
const create = () => gun.get("todos").key(input).put({ title: input, done: false })
</script>

<input placeholder="Add todo" bind:value={input} />
<button on:click={e => create() && (e.target.value = "")}>Add</button>
```

### Step 3 - List Todos from GUN
```html
// App.svelte
<script>
// ...

let store = {}
gun.get("todos").map().on(function(data, key) {
    if (data) {
        store[key] = data
    } else {
        // gun.map() can return null (deleted) values for keys
        // if so, this else clause will update your local variable
        delete store[key]
        store = store
    }
})

// This syntax should familiar to you if you completed the official Svelte tutorial
// We'll convert the store to a sorted list that can be iterated in the view
$: todos = Object.entries(store).sort((a, b) => a[1].done)
</script>

{#each todos as [key, todo]}
<div id={key}>
    {todo.title} {todo.done ? "😺" : "😾"}
</div>
{/each}
```

### Step 4 - Add update and delete actions
```html
<script>
// ...
const update = (key, value) => gun.get(key).get("done").put(value)
const remove = key => gun.get(key).put(null)
</script>

{#each todos as [key, todo]}
<div id={key}>
    <input type="checkbox" checked={todo.done} on:change={() => update(key, !todo.done)} />
    {todo.title} <a href="/" on:click|preventDefault={() => remove(key)}>remove</a> {todo.done ? "😺" : "😾"}
</div>
{/each}
```

And now we're finished! [Check it out in the Svelte REPL](https://svelte.dev/repl/84cccb9fff64494bbec852922940ea23?version=3.16.5)

## Using Svelte's writable/readable stores
You can also merge GUN with Svelte's own stores, but they are harder to work with compared to using the custom store in the (recommended) example at the top of this page.

### Simple example with writable

```javascript
// stores.js
import { gun } from "./initGun.js"
import { writable } from "svelte/store"

function createStore(ref, initialValue) {
    const { set, subscribe } = writable(initialValue)
    ref.on(set)
    return {
        subscribe
    }
}

const todosRef = gun.get("todos")
export const todos = createStore(todosRef, {})
```

### Merged with svelte's writable store
```javascript
// stores.js
import { gun } from "./initGun.js"
import { writable } from "svelte/store"

function createMapStore(ref) {
    const { update, subscribe } = writable({})
    ref.on(function(data, key) {
        if (data) {
            update(store => ({...store, [key]: data}))
        } else {
            update(store => {
                delete store[key]
                return store
            })
        }
    })        

    return {
        subscribe,
        update: (key, value) => ref.get(key).put(value)
    }
}

const todosRef = gun.get("todos").map()
export const todos = createMapStore(todosRef, {})
```

---
Anything to add? [Add more to this article](https://github.com/amark/gun/wiki/Svelte/_edit)! 