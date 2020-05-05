# Design Examples

## Blog
Here’s an example on how to structure a blog’s data structure using gunDB. We will start with 4 object types:
* User
* Post
* Tag
* Comment

Here’s how they are all connected:
* User to Post — One To Many   
A user can have many posts but a post belongs to only one user.
* User to Comment — One To Many   
A user can have many comments but a comment belongs to only one user.
* Post to Tag — Many To Many   
A post can have many tags and a tag can be applied to many posts.
* Post to Comment — One To Many   
A post can have many comments but a comment can only apply to one post.

### Creating our users
```javascript
var markInfo = {
 name: "Mark",
 username: "@amark"
};
var jesseInfo = {
 name: "Jesse",
 username: "@PsychoLlama"
};
var mark = gun.get('user/' + markInfo.username).put(markInfo);
var jesse = gun.get('user/' + jesseInfo.username).put(jesseInfo);
```

### Creating a post
The Post object:
```javascript
var lipsum = {
 title: "Lorem ipsum dolor",
 slug: "lorem-ipsum-dolor",
 content: "Lorem Ipsum is simply dummy text of the printing and typesetting industry. Lorem Ipsum has been the industry’s standard dummy text ever since the 1500s, when an unknown printer took a galley of type and scrambled it to make a type specimen book. It has survived not only five centuries, but also the leap into electronic typesetting, remaining essentially unchanged. It was popularised in the 1960s with the release of Letraset sheets containing Lorem Ipsum passages, and more recently with desktop publishing software like Aldus PageMaker including versions of Lorem Ipsum."
};
```
Some tags
```javascript
var lorem = {
 name: "lorem",
 slug: "lorem"
};
var ipsum = {
 name: "ipsum",
 slug: "ipsum"
};
var dolor = {
 name: "dolor",
 slug: "dolor"
};
```
Lets store them!
```javascript
var lipsumPost = gun.get('post/' + lipsum.slug).put(lipsum);
var loremTag = gun.get('tag/' + lorem.slug).put(lorem);
var ipsumTag = gun.get('tag/' + ipsum.slug).put(ipsum);
var dolorTag = gun.get('tag/' + dolor.slug).put(dolor);
```
Let’s connect them! 
First, we connect the post to its author, then we connect all the tags:
```javascript
lipsumPost.path('author').put(mark).path('posts').set(lipsumPost)
 .path('tags').set(loremTag).path('posts').set(lipsumPost)
 .path('tags').set(ipsumTag).path('posts').set(lipsumPost)
 .path('tags').set(dolorTag).path('posts').set(lipsumPost);
```
### Adding a comment
The comment & its connections
```javascript
var sit = {
 id: 1,
 text: "Sit amet, consectetur adipiscing elit."
};
var sitComment = gun.get('comment/' + sit.id).put(sit);
sitComment.path('author').put(jesse).path('comments').set(sitComment)
     .path('post').put(lipsumPost).path('comments').set(sitComment);
```
### Checking it all works
If you want to check that everything worked fine, you can log some of the references’ values to check.
```javascript
mark.val(function(v){console.log(v);});
mark.path('posts').map().val(function(v){console.log(v);});
jesse.val(function(v){console.log(v);});
jesse.path('comments').map().val(function(v){console.log(v);});
lipsumPost.path('comments').map().val(function(v){console.log(v);});
lipsumPost.path('author').val(function(v){console.log(v);});
lipsumPost.path('tags').map().val(function(v){console.log(v);});
loremTag.path('posts').map().val(function(v){console.log(v);});
sitComment.path('author').val(function(v){console.log(v);});
sitComment.path('post').val(function(v){console.log(v);});
```

Having some fun
```javascript
// Circular references + chaining = fun
mark.path('posts').map() 
    .path('tags').map()
    .path('posts').map()
    .path('comments').map()      
    .path('author'); 
// All the authors of the comments on the posts with the tags of the // posts that mark has posted. (Say that 5 times real fast)
// Same thing, explained
mark.path('posts').map()     // All of mark's posts
    .path('tags').map()      // All of mark's posts' tags
    .path('posts').map()     // All of the tags' posts
    .path('comments').map()  // All of the posts' comments
    .path('author');         // All of the comments' authors
// Based on our inserted dataset this would return jesse
```
If you want to test this out yourself and maybe change a thing or two, [check out the code here.](http://jsbin.com/labubaf/edit?js,console)
