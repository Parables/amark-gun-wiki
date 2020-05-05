This is referencing gun `v0.3.x` just to be clear. This script is a hold over until we get GUN `v0.4.x` out the door, which will not have performance problems.

https://rawgit.com/amark/41d31de8911e6d5ebe13127180cf9588/raw/bb658b2460b14001e891f982073bc5a7e4e35186/gun3import.js

or

`<script src='https://rawgit.com/amark/41d31de8911e6d5ebe13127180cf9588/raw/bb658b2460b14001e891f982073bc5a7e4e35186/gun3import.js'></script>`

Here is an example of how to use (after including the code):

Get some sample data to import, like https://api.github.com/repos/octocat/Hello-World/issues . Save this array to a variable called `githubReply`;

Now we can import the data:

```javascript
githubReply.forEach(function(data){
  importGUN3(data);
});
```

and check to see if one of the records got saved:

```javascript
var record = githubReply[0];

var gun = Gun();
gun.get(record.id).on(function(data){
  console.log("The record", data);
}).path('user').val(function(data){
  console.log("The user", data);
});
```

It works!

NOTE: Caveats! The import code imports directly to localStorage, so right now it **can only be used in the browser** (but upon request, we can make it plug into file.js/data.json [which are for local testing purposes only] for NodeJS also). This also means that the importing alone **will not** actually sync the data or bring it into memory for your app to use. **You still must** use the GUN api to then do the rest.