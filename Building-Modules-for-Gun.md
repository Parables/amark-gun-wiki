# EXAMPLE OF v0.8+ STORAGE ADAPTER

```javascript
Gun.on('opt', function(ctx){
	this.to.next(ctx);
	var opt = ctx.opt, no;
	if(ctx.once){ return }

	ctx.on('put', function(msg){
		this.to.next(msg);
		var err;
		Gun.graph.is(msg.put, function(node, soul){
			err = err || STORAGEwithPATCH.put(soul, node);
		});
		if(!msg['@']){ // only reply if this update isn't an ACK also.
			ctx.on('in', {'@': msg['#'], err: err, ok: err? no : 1}); // reply with ACK to msg id.
		}
	});

	ctx.on('get', function(msg){
		this.to.next(msg);
		var node, err;
		try{ node = READfromSTORAGE(msg.get['#']);
		catch(e){ err = e }
		if(node && msg.get['.']){
			node = Gun.state.to(node, msg.get['.']); // filter node down to the 1 property
		}
		ctx.on('in', {'@': msg['#'], put: Gun.graph.node(node), err: err}); // ACK the GET. 
	});
});
```

## THE REST OF THE DOCUMENTATION MAY BE OUTDATED

> ###**note:** this guide is written for gun v0.2.x and will be outdated with gun v0.3.x, currently being tested on #develop.

# Gun Plugins: a how-to guide

This guide explains how to build drivers and extensions for gun. Basic knowledge about [gun's API](API-(v0.2.x)) is recommended, but not required to follow along.

First, let's get an idea for how gun can be extended and the sorts of hooks it provides to developers. Most plugins for gun will fit into 3 main categories:

- [abstraction layers](#abstraction)
- [persistence layers](#persistence)
- [communication layers](#communication)

### Abstraction

Likely the easiest of the three, gun's API is fully extensible, allowing you to define your own methods and abstraction layers or pull in existing ones. If this interests you or if you just want to jump in and start coding, read [Adding Methods to the GUN Chain](Adding-Methods-to-the-Gun-Chain).

### Persistence

Since gun's main goal is to synchronize data between clients and servers, it doesn't take a strong stance on how your data is saved. Client side, it defaults to localStorage, while server side it exports to a file named `data.json`, although it's designed with plugins in mind so that your favorite storage engine, whether it be levelDB, Hadoop, IndexedDB, AWS, etc., should painlessly integrate with your project. Building a plugin for gun's persistence layer is fairly straightforward, and is outlined in [Adding Storage Services](Adding-Storage-Services).

### Communication

Gun uses websockets to synchronize information, falling back on jsonp if sockets aren't supported in your environment. However *both* transport layers could be exchanged with WebRTC, DDP or any other method of sending and receiving data. The process is essentially the same as writing a persistence driver, where the peer you're messaging is treated as your persistence layer.


## How To:
 - **[Add Methods to the Gun Chain](Adding-Methods-to-the-Gun-Chain)**  
   All of gun's methods are exposed through it's prototype, or as we call it, the gun chain. You can define your own methods by attaching them to that chain. It behaves exactly like an object prototype, so your extension will be accessible to every gun instance within your application.
 - **[Add Storage Services](Adding-Storage-Services)**  
   GUN is a synchronization engine and can be used with a variety of storage services.  Additional storage services can be  added.