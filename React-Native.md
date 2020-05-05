Gun works out of the box with React Native.

We will show you how to build a mobile app version of the Hello World [example](https://gun.eco/docs/Hello-World) with live synchronization between the mobile and the web app.

![Hello World GUN React Native](https://raw.githubusercontent.com/bogdant/bogdant/master/HelloGunReactNative.jpg)

# Prerequisites
## Web Hello World
Follow the Hello World steps [here](https://gun.eco/docs/Hello-World) and create a simple GUN app. Modify the app to connect to a remote GUN server, if you don't have one you can use our demo server:

```javascript
var gun = new Gun('http://gunjs.herokuapp.com/gun')
```

## React Native Dev Environment
We are assuming you are familiar with React Native and have the environment set up. If not, follow the instructions in the "Building Projects with Native Code" tab [here](https://facebook.github.io/react-native/docs/getting-started.html). Continue below when you were able to successfully launch and edit a React Native app on an emulator or real device. 

# Hello World GUN in React Native
Create a new React Native app:
```javascript
react-native init HelloWorldGUN
```
This generates a HelloWorldGUN folder prepopulated with a barebones app. 

Go to the HelloWorldGUN folder and add GUN to the project:
```javascript
cd HelloWorldGUN
npm install gun
```
Now let's modify the App.js code file. First, import and define the GUN variable:
```javascript
import Gun from 'gun/gun.js' // or use the minified version 'gun/gun.min.js'
const gun = new Gun('http://gunjs.herokuapp.com/gun') // or use your own GUN server
Component.prototype.$gun = gun
```
We will need a couple of UI controls, so add them to the import statement:
```javascript
import {
//...
TextInput,
Button
} from 'react-native' 
```
Add a constructor to the App class where we initialize the state and set up the Gun sync object:
```javascript
export default class App extends Component<Props> {
  constructor(props){
    super(props)
    this.state = {
      text: 'What is your name?',
      name: ''
    }
    this.$hello = this.$gun.get('hello')
    let _this = this
    this.$hello.on(function(data, key) {
      let name=data.name
      _this.setState({name:name})
    })
  }
```
Finally, replace the content of the render function's  View component with the code below: 
```javascript
<Text style={styles.welcome}>
  Hello {this.state.name}
</Text>
<TextInput value={this.state.text}
  onChangeText={(text) => this.setState({text})} 
/>
<Button title='Update' 
  onPress={()=>{
    this.$hello.put({name:this.state.text})
    this.setState({text:''})}}
/>
```
Now try launching the app:
```javascript
react-native run-android 
   or
react-native run-ios
```
You should be able to see the Hello line update on both app and web page with a new name when you press Submit button on either side.
# Development Tips
## Long Term Storage

In the browser, Gun will dump the graph into LocalStorage to achieve longer-term storage. On a Node server, it can dump into a JSON file or into another storage system via an adapter.

In React Native, if you want to retain application data for either offline use or for long-term storage on the device itself, you can plug Gun into a few community-built adapters:

* [AsyncStorage](https://github.com/staltz/gun-asyncstorage) by [staltz](https://github.com/staltz)
* [SQLite](https://github.com/sjones6/gun-react-native-sqlite) by [sjones6](https://github.com/sjones6)
* [Realm](https://github.com/sjones6/gun-realm) by [sjones6](https://github.com/sjones6)

If you choose not to use any long-term storage option, that's totally fine. Gun will keep as much of the graph as possible in memory, which will be dumped every time the application is restarted.

## Local Development

Let's say you've got a Gun server running on a Node process on your machine; while you're developing your React Native application, you want to connect up to that server rather than some remote, production server over the internet. Instead of giving `localhost`, provide the Local-Area-Network (LAN) IP address for your machine:

```javascript
const gun = new Gun("http://192.168.0.01/gun");
```

# Example Application

[Co-Libate](https://github.com/sjones6/co-libate) by [sjones6](https://github.com/sjones6): A collaborative wine-tasting application
