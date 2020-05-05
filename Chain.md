If you are unfamiliar with **chaining**, it is an API style that allows you to zoom into a context without having to create new variables. So rather than doing...

```javascript
// ugly
var page = document.getElementById("page");
var child = page.firstElementChild;
child.style.background = "blue";
child.style.color = "green";
child.addEventListener('click', function(event){
  console.log("hello world!");
}, true);
```

...you can just do

```javascript
$('#page')
	.children()
	.first()
	.css("background", "blue")
	.css("color", "green")
	.on("click", function(event){
		console.log("hello world");
	});
```

As you can tell, it was popularized by jQuery.

### Next up?

 - You should also be familiar with how GUN improves upon this with [reactive programming](FRP).
 - You might be curious how the internal [javascript implementation](javascript) was built.