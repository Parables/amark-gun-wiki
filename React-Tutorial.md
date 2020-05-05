In addition to this article, there are a couple of React code examples in the main repo, and some other resources:

 - [Simple React Example on Glitch by @Lightnet](https://glitch.com/edit/#!/project-react-gundb)

## Intro to Decentralized Databases with GUN

_by Eddie Kollar of [AppendTo](https://appendto.com/courses/)_

### Distributed Systems and Decentralization

Whether we realize it or not, the software systems that we use in our day to day are an amalgamation of varying services. They are distributed across many computers often in different geographical locations. Most of the distributed systems rely on a design that centralizes ownership and/or authority to an organization. Lately, there has been a rise in technology, such as cryptocurrencies and the blockchain, that implements distributed systems in a decentralized manner; participation is open to those who abide by the agreed upon and implemented protocols.

Databases are an important component of software systems and are often themselves distributed for purposes of high availability. In these scenarios, the data are ultimately under a centralized model. The abstracted database architecture is the authority and results in a mechanism that maintains the current state of the data. Looking from the perspective of a decentralized system, each participant or node would own their data and share with nodes across the network as needed. This style of architecture is referred to as peer-to-peer or p2p and is used in file sharing protocols like BitTorrent. 

### Decentralized Databases with GUN

[GUN.js](https://gun.eco/) is a real-time, decentralized, offline first, graph database. Sure is a mouthful. Knowing a little bit about decentralized architecture we can understand the other features. Under the hood, GUN allows for data synchronization to happen seamlessly between all connected nodes by default. Its offline first capabilities mean that if connectivity is lost to other nodes due to a network error or no availability, the application will store all changes locally and automatically synchronize as soon there is a connection. Finally, the flexible data storage model allows for tables with relations ([MSSQL](http://www.microsoft.com/en-us/sql-server/sql-server-2016) or [MySQL](https://www.mysql.com/)), tree-structured document orientation ([MongoDB](https://www.mongodb.com/)), or a graph with circular references ([Neo4j](https://neo4j.com/)).

## A Working Example

In this article, we are going to build a simple note taking application in React using GUN that will update real time between two different clients. This will help to illustrate how the key features of GUN look in action.

### Getting Started

I have a simple [React boilerplate](https://github.com/eddiekollar/ReactSimple) project that I use for some of my projects in order to avoid recreating an empty project. I really recommend doing this for beginners to become familiar with the libraries used and the build tools. There are plenty of starting projects and command line tools used to generate React projects, but those are recommended after you’ve gained proficient understanding.

Let’s begin by adding the GUN library via the command line:

$`yarn add gun --save`

The project comes with a server.js file that creates an HTTP server using Express.js. It has been configured to run both for development and production environments. Here we will add a GUN datastore that clients will connect to and for our purposes, the data will persist in a JSON file on our server. I’ve created a directory /db and the file data.json where GUN will write to:

```
├── LICENSE.MD
├── README.md
├── db
│   └── data.json
├── images
│   └── favicon.ico
├── package.json
├── server.js
├── src
│   ├── components
│   │   ├── App.js
│   │   ├── Auth.js
│   │   ├── Home.js
│   │   └── NoteForm.js
│   ├── index.html
│   └── index.js
├── test
│   └── App.test.js
├── webpack.config.js
└── yarn.lock
```

Modifying server.js add the following:

```javascript
const Gun = require('gun');

//...

app.use(Gun.serve);

const server = app.listen(port);

Gun({ file: 'db/data', web: server });
```

Earlier I explained how GUN is a distributed database, where it can have many nodes connecting to each other, so we will add the library to the front-end. In the previous step we added the server node for all client nodes to connect to, we’ll pass the URL to it as a configuration for the client stores to synchronize with. For now, we’ll keep the database operations in src/components/App.js:

```javascript
import React, { Component } from 'react';
import Gun from 'gun';
import Home from './Home';

class App extends Component {
  constructor() {
  super();
    this.gun=Gun(location.origin+'/gun');
    window.gun = this.gun; //To have access to gun object in browser console
  }

  render() {
    return (
      <Home gun={this.gun} />
    );
  }
}

export default App;
```

To test that this works we’ll run the application. In the package.json file there is section for scripts that can be run and the start command will run the development server:

$`yarn start`

Open up a browser window and navigate to [http://localhost:8080](http://localhost:8080/) to see the home page. In that window open the developer tools. In the console we will run some commands to interact with the database to see that it works on the client and that it synchronizes with the server peer node.

&gt;`var note = {title: 'first item', text: 'from command line'};`

&gt;`gun.put(note);`

Inspecting db/data in our project we can see there is data files that start with key/ids opening them in a text editor will show something similar to this: (the exception being 1%c which is an index file and can be ignored)

```JSON
{
  "graph": {
    "EVz9V7xwmMW2MZBGHkwAntex": {
 	 "_": {
   	 "#": "EVz9V7xwmMW2MZBGHkwAntex",
   	 ">": {
     	 "title": 1498156296164.74,
     	 "text": 1498156296164.74
   	  }
 	 },
 	 "title": "first item",
 	 "text": "from command line"
    }
  }
}
```

This can be verified in the localStorage of the browser as well by finding a similar key/value pair:

```JSON
{"_":
  {"#":"EVz9V7xwmMW2MZBGHkwAntex",
   ">":
     {"title":1498156296164.74,
      "text":1498156296164.74}},
  "title":
  "first item","text":"from command line"
}
```

So what exactly happened? We’ve just successfully stored a note into the GUN database with the JSON data from the variable note and some extra data. Compared to original data we can deduce that “_” is a metadata object created by GUN. The document is assigned a unique id ‘#’ or “[soul](https://github.com/amark/gun/wiki/GUN%E2%80%99s-Data-Format-%28JSON%29#soul)” in GUN parlance and another child object ‘&gt;’ containing the timestamp when a field was last updated.

For good measure, let’s open up a new incognito window to the localhost URL to verify that we can access this data. When you inspect the localStorage you will notice that it is empty. This is because this node has not yet retrieved or subscribed to any data. Using the [.get()](https://github.com/amark/gun/wiki/API#get) call we go ahead and call it by chaining the [.on()](https://github.com/amark/gun/wiki/API#on) function:

&gt;`gun.get('g0ZMK77W4wwVEHuyXzlPdVgc').on(function(data, key){ console.log(data, key); })`

And now the same data will show up in the localStorage of this window as seen in our first instance. In the documentation, you will see that there are two methods for getting the data by chaining .on() or [.val()](https://github.com/amark/gun/wiki/API#val). With the above example, we’ve subscribed to the data, which will give us real-time updates to the object. .val() is used to only get the data at the time of the call with no future updates. What is important to note here is that if we don’t explicitly make that .get() call in the new node of the application, that node will not know of that key/value pair.

### Data Modeling

Let’s take a step back and consider how to design the data modeling for our sample application. Since we would like to work with a list of notes, we need to consider how that can be stored in GUN. Looking the documentation we can see that [.put()](https://github.com/amark/gun/wiki/API#put) only accepts objects, strings, numbers, booleans, and null. The resulting object will automatically name the object based on its soul, the unique id. A deeper read on the .get() call shows that it can be chained with .put() to set the name of the key:

```javascript
gun.get('key').put({property: 'value'})

gun.get('key').on(function(data, key){
 // {property: 'value'}, 'key'
})
```

GUN does not use arrays but uses the concept of a set from mathematics, where each element is a unique object. With this understanding, we will store each note at the top level of the database and add a reference to the document into a set called ‘notes’. This way each node only needs knowledge of the notes object in order to subscribe to the data synchronization and get access to the individual notes.

In the next step, we’ll create the component to create, list, and view notes. First, n each window clear out the data by calling localStorage.clear(). Stop the running server process and delete the data in db/data.json. Once finished you can restart the server. The server needs to be stopped because GUN maintains the data in memory as well. This feature adds more fault tolerance as the file can be deleted and will be recreated again.

![react-gunjs-notes-300x166](https://user-images.githubusercontent.com/433412/27883421-40cd082e-6185-11e7-9852-a261cefef949.png)

### User Interface

For the UI framework we will go with the React version of Bootstrap:

&gt;`yarn add react-bootstrap --save`

I've gone ahead and created another component called NoteForm to keep the Home component from being too cluttered. Though this isn't the cleanest design practice, it serves to help teach us how to use GUN with React.

NoteForm.js abstracts the UI for the form:

```javascript
import React, {Component} from 'react';
import {Panel, ButtonToolbar, Button, FormGroup, ControlLabel, FormControl} from 'react-bootstrap';

class NoteForm extends Component {
  componentWillMount() {
    this.resetState = this.resetState.bind(this);
    this.resetState();
  }

  componentWillReceiveProps(nextProps) {
    const {id, title, text} = nextProps.note;
    this.setState({id, title, text});
  }

  resetState () {
    const {id, title, text} = this.props.note;
    this.setState({id, title, text});
  }

  onInputChange (event) {
    let obj = {};
    obj[event.target.id] = event.target.value;
    this.setState(obj);
  }

  saveBtnClick () {
    this.props.onSaveClick(this.state);
  }

  render() {

    return (
      <Panel bsStyle="primary">
      <form>
        <FormGroup>
          <ControlLabel>Title</ControlLabel>
          <FormControl
            id="title" 
            type="text" 
            placeholder="Enter a title" 
            value={this.state.title} 
            onChange={this.onInputChange.bind(this)}/>
        </FormGroup>
        <FormGroup>
          <ControlLabel>Note text:</ControlLabel>
          <FormControl 
            id="text"
            componentClass="textarea" 
            placeholder="..." 
            value={this.state.text}
            onChange={this.onInputChange.bind(this)}
            />
        </FormGroup>
        <ButtonToolbar>
          <Button bsStyle="primary" onClick={this.saveBtnClick.bind(this)}>Save</Button>
          <Button onClick={this.resetState}>Cancel</Button>
        </ButtonToolbar>
      </form>
      </Panel>
    );
  }
}

export default NoteForm;
```

Home.js subscribes to the data and updates it. The list rendering is managed here as well:

```javascript
import React, {Component} from 'react';
import {Panel, Button, Col, ListGroup, ListGroupItem} from 'react-bootstrap';
import Gun from 'gun';
import _ from 'lodash';

import NoteForm from './NoteForm';

const newNote = {id: '', title: '', text:''};

class Home extends Component {
  constructor({gun}) {
    super()
    this.gun = gun;
    this.notesRef = gun.get('notes');
    
    this.state = { notes: [], currentId: ''};
  }

  componentWillMount() {
    let notes = this.state.notes;
    const self = this;
    this.gun.get('notes').on((n) => {
      var idList = _.reduce(n['_']['>'], function(result, value, key) {
        if(self.state.currentId === '') {
          self.setState({currentId: key});
        }

        let data = { id: key, date: value};
        self.gun.get(key).on((note, key) => {
          const merged = _.merge(data, _.pick(note, ['title', 'text']));
          const index = _.findIndex(notes, (o)=>{ return o.id === key});
          if(index >= 0) {
            notes[index] = merged;
          }else{
            notes.push(merged);
          }
          self.setState({notes});
        })  
      }, []);
    })
  }

  newNoteBtnClick () {
    this.setState({currentId: ''});
  }

  itemClick (event) {
    this.setState({currentId: event.target.id});
  }

  getCurrentNote() {
    const index = _.findIndex(this.state.notes, (o) => { return o.id === this.state.currentId});
    const note = this.state.notes[index] || newNote;
    return note;
  }

  getNoteItem(note) {
    return (<ListGroupItem key={note.id} id={note.id} onClick={this.itemClick.bind(this)}>
      {note.title}
    </ListGroupItem>)
  }

  onSaveClick(data) {
    const note = _.pick(data, ['title', 'text']);
    if(data.id !== '') {
      this.gun.get(data.id).put(note);  
    }else{
      this.notesRef.set(this.gun.put(note))
    }
  }

  render() {
    this.getCurrentNote = this.getCurrentNote.bind(this);
    return (
      <div>
        <Col xs={4} >
          <Panel defaultExpanded header='Notes'>
            <Button bsStyle="primary" block onClick={this.newNoteBtnClick.bind(this)}>New Note</Button>
            <ListGroup fill>
              {this.state.notes.map(this.getNoteItem.bind(this))}
            </ListGroup>
          </Panel>
        </Col>
        <Col xs={8}>
          <NoteForm note={this.getCurrentNote()} onSaveClick={this.onSaveClick.bind(this)}/>
        </Col>
      </div>
    );
  }
}

export default Home;
```

The two important functions to observe in Home.js are componentWillMount()  and onSaveClick() . When the component is mounted the .on() calls subscribes the component first to 'notes'. The code in the callback is triggered on first when initially called and then each subsequent time when a **new note is added**. As notes is a list of references to actual note objects, additions are the only changes that will happen. Inside the callback, we see _.reduce()  call that goes through each note reference and creates an individual subscription for each note. The callback inside of `self.gun.get(key).on((note, key) => { ... })` are triggered when that specific note is updated.

onSaveClick() saves the new note or changes to an existing one. When a new note is created this.gun.put(note) returns a reference to the note which is added to the exists set inside of 'notes'.

Open an incognito tab as we did, in the beginning, to see how the real-time updates show up in the UI as you add and edit new notes.

### Conclusion

We've created a very simple note taking application that synchronizes the data across all connected peers. From here it would be useful to create a user authentication component and expand on the data model to include security and ownership to lists and individual notes. GUN is an extremely powerful yet minimal database system that can be utilized in a wide variety of scenarios for web and mobile. It is highly recommended digging deeper into the [documentation](https://github.com/amark/gun/wiki/) to learn about more features and the computer science theory behind its design. GUN has very active and friendly [community on Gitter](https://gitter.im/amark/gun) as well that you can reach out to. For access to the completed project code go this [repo](https://github.com/eddiekollar/gunjs-notes-app).

---

Eddie Kollar writes for appendTo, who offers [web development training courses](https://appendto.com/courses/) for teams. Special thanks to Kyle Pennell for publishing this piece.