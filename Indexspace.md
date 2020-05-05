**docs coming soon**

Please see https://github.com/amark/gun/blob/master/examples/simple/space.html as an example.

> NOTE: Space only returns the next *latest* match in the tree, not all sub-items!

> NOTE: `lib/space` is in ALPHA! API may change (please use it and provide feedback) and complain about bugs!

Or copy & paste this into JSbin or a HTML file:

```
<script src="https://cdn.jsdelivr.net/npm/gun/gun.js"></script>
<script src="https://cdn.jsdelivr.net/npm/gun/lib/space.js"></script>
<script>
var gun = Gun();

gun.get('index').space('marknadal', 'ME!'); // index me
gun.get('index').space('marttimalmi', 'HIM!');

gun.get('index').space('mar', res => console.log(res));
/*
{
	tree: {'marknadal': 'ME!', 'marttimalmi: 'HIM!'}
	key: 'mar'
	data: undefined
}
*/
</script>
```