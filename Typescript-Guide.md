## Install
> yarn add @types/gun

or

> npm install @types/gun

## Define your app state interface

Now let's say we have a gun app that syncs an input text between nodes.

```typescript
interface AppState {
    sync_input: { text1: string; text2: string; }
}
const gun = new Gun<AppState>('https://example.com/gun')
```

Now `gun` has type `GunInstance<AppState>` and it is definitely typed!

```typescript
gun.get('sync_input') // now it has type GunInstance<AppState["sync_input"]>
.put() // this field only accept { text1: string, text2: string; }
```
For more example: https://github.com/DefinitelyTyped/DefinitelyTyped/pull/33100/files#diff-a04b085b151b79955666308959e91eaa

Most of the APIs in Gun are typed like examples above. Just use them as normal. But here are some exceptions.

## Exceptions

### Sets
Set is special. If you want to use it, type it like this.
```typescript
interface State {
    users: string[]
}
```
Use Array to represent a set. Then you can `.set("x")`

For any type that not an Array, `.set()` will be typed as `.set(value: never)` to prevent you call it accidentally.

### `back(level: number)`
Apparently, this cannot be typed. It will return a `GunInstance<?>`. Avoid using it in Typescript. Always get gun node from `root` so you can get typed.

### `path?(path: string | string[])`
Same as `back`. Can't be typed. Avoid using it.

### All the addons
Like `promise()`, `then()` etc. Typed as `?`. You should make sure you have already imported the file need yourself.

When you use addons, use it like `then!()`. Add a `!` after the method name to let typescript know you want to ignore the `?`.. Add a `!` after the method name to let typescript know you want to ignore the `?`.

## Gun.SEA

Encourage you to use `async & await` so most of APIs does not include the signature for callback.