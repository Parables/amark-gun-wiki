Goals:

 - Write code to work in the Browser first.
 - No build step, no toolchain.
 - This code also works in NodeJS.
 - Optionally, then `unbuild` the single well-organized browser file into a bunch of sub-files that are NodeJS `require` dependent on each other.

This works well with other tools because of the `browser` field in package.json:

https://github.com/amark/gun/blob/master/package.json#L6

Basically, for other popular projects webpack/etc. would "build" a file but then people had problems with webpack wanting to build the built file.

So all systems adopted a skip/check that you could specify what that built bundle for the browser is in your package.json - in my case, that is my main/original `gun.js` file.

Slowly over time, these tools have started ignoring this and started regexp (and later, more complicated) for `require` and `module.exports`.

Initially we got around this by aliasing `var USE = require` and in some sad circumstances hiding `module.export` in some weird ( ?forget what?) `if(typeof __webpack__module__exports__ !== 'undefined` check.

Now the tools seem to be **evaluating** the code (? or regexp-ing for variables versus strings passed to `require`, thus the stupid 'request as dependency').

As-of-current (tho I might have to change this again), here is roughly what my system looks like:

```
;(function(){ // create a protective closure that variables can't escape from
  // unbuild setup ...
  function USE(path, force){ // wrapper that works in both browser & nodeJS
    return force? require(path) : ...
  }
  // ... end unbuild setup

  ;USE(function(module){
   // this would be its own NodeJS module file!
   module.exports = 42;
  })(USE, './path');

  ;USE(function(module){
   var path = USE('path'); // works in browser, and during NodeJS unbuild `USE` gets find&replaced with `require`
   module.exports = path + 2;
  })(USE, './add');

}());
```

And so on.

One last detail is, if you run the file as-is in NodeJS, you have to deal with the "master" module.export (not the per-USE polyfill), I've done this by successfully by defining a `common = module` in the outer scope during unbuild setup. Then in my "main" module I `common.export = foo` which NodeJS correctly handles.