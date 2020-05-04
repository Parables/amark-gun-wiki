The static file server module included with GUN had a **serious vulnerability**:

 - Using `curl --path-as-is` allowed reads on any parent directory or files.

This did not work via the browser or via curl without as-is option.

 ### Fixed

This has been fixed since version `0.2019.416` and higher.

 ### Who Was Effected?

Most NodeJS users who use the default setup, such as:

 - `npm start`
 - `node examples/http.js`
 - `Heroku` 1-click-deploy
 - `Docker`
 - `Now`

If you have a custom NodeJS code then you are probably safe *unless* you have something like `require('http').createServer(Gun.serve(__dirname))` in it.

If you have not upgraded, it is **mandatory** or else it is highly likely your environment variables and AWS (or other) keys could be leaked.

 ### Credit

It was reported and fixed by [JK0N](https://github.com/amark/gun/pull/527), but I did not understand the `--path-as-is` condition.

Joonas Loppi from [function61](http://function61.com) rediscovered it and explained the urgency to me to fix it.