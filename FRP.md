If you are unfamiliar with **reactive** programming, it is a code structure that emphasizes vertical readability by avoiding nested loops and callbacks. Instead of doing...

```javascript
// ugly
for(var i = 0; i < PDFs.length; i += 1){
	var fileName = PDFs[i];
	readFromDisk(filename, function(file){
		var pages = splitIntoPages(file);
		for(var j = 0; j < pages.length; j += 1){
			var page = pages[j];
			saveToFolder('page' + j, page, function(err, done){
				console.log("Done! If no," err);
			});
		}
	});
}
```

...you can simplify it to something more clear and reusable, such as:

```javascript
var PDF = {};
PDF.read = function(fileName){
	readFromDisk(filename, PDF.split);
}
PDF.split = function(file){
	splitIntoPages(file).forEach(PDF.save);
}
PDF.save = function(page, number){
	saveToFolder('page' + number, page, PDF.done);
}
PDF.done = function(err, done){
	console.log("Done! If no", err);
}
PDFs.forEach(PDF.read);
```

And that is functional reactive programming!

### Next up?

 - You should also be familiar with how GUN improves upon this with [chaining](Chain).
 - You might be curious how the internal [javascript implementation](javascript) was built.