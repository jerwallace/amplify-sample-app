# Amplify GraphQL/React App
An Amplify / GraphQL Sample App

For full description, see here:
https://medium.com/open-graphql/create-a-multiuser-graphql-crud-l-app-in-10-minutes-with-the-new-aws-amplify-cli-and-in-a-few-73aef3d49545

This is the streamlined instructions!

First, install amplify CLI.
```
npm install -g @aws-amplify/cli
npm install -g create-react-app
```

Next, create the new app with amplify notes. 
```
create-react-app amplifynotes
cd amplifynotes
yarn add aws-amplify aws-amplify-react bootstrap
```

If you do not have an AWS profile set up, configure one.
```
$ amplify init
Note: It is recommended to run this command from the root of your app directory
? Choose your default editor: Visual Studio Code
? Choose the type of app that you're building javascript
Please tell us about your project
? What javascript framework are you using react
? Source Directory Path:  src
? Distribution Directory Path: build
? Build Command:  npm run-script build
? Start Command: npm run-script start
Using default provider awscloudformation

For more information on AWS Profiles, see:
https://docs.aws.amazon.com/cli/latest/userguide/cli-multiple-profiles.html

? Do you want to use an AWS profile? Yes
? Please choose the profile you want to use amplify
⠦ Initializing project in the cloud...
```

Next, add a GraphQL endpoint.
```
$ amplify add api
? Please select from one of the below mentioned services GraphQL
? Provide API name: amplifynotes
? Choose an authorization type for the API  Amazon Cognito User Pool
Using service: Cognito, provided by: awscloudformation
 The current configured provider is Amazon Cognito.
? Do you want to use the default authentication and security configuration? Yes, use the default configuration.
Successfully added auth resource
? Do you have an annotated GraphQL schema? No
? Do you want a guided schema creation? true
? What best describes your project: Objects with fine-grained access control (e.g., a project management app with owner-based authorization)
? Do you want to edit the schema now? Yes
Please edit the file in your editor: /<path>/amplify/backend/api/amplifynotes/schema.graphql
? Press enter to continue
```

Use the following schema (delete what is in the schema.graphql file):
```
type Note @model @auth(rules: [{allow: owner}]){
  id: ID!
  note: String!
}
```

Push your changes to AWS.
```
amplify push
```

Add the following code to replace your **src/App.js** file
```javascript
import React, { Component } from 'react';
import logo from './logo.svg';
import './App.css';
import 'bootstrap/dist/css/bootstrap.min.css';
import Amplify, {API,graphqlOperation} from 'aws-amplify';
import { withAuthenticator} from 'aws-amplify-react'; 
import aws_exports from './aws-exports'; // specify the location of aws-exports.js file on your project
Amplify.configure(aws_exports);

const createNote = `mutation createNote($note: String!){
  createNote(input:{
    note: $note
  }){
    __typename
    id
    note
  }
}`;

const readNote = `query listNotes{
  listNotes{
    items{
      __typename
      id
      note
    }
  }
}`;

const updateNote = `mutation updateNote($id: ID!,$note: String){
  updateNote(input:{
    id: $id
    note: $note
  }){
    __typename
    id
    note
  }
}`;

const deleteNote = `mutation deleteNote($id: ID!){
  deleteNote(input:{
    id: $id
  }){
    __typename
    id
    note
  }
}`;

class App extends Component {
  constructor(props){
    super(props);
    this.state={
      id:"",
      notes:[],
      value:"",
      displayAdd:true,
      displayUpdate:false
    };
    this.handleChange = this.handleChange.bind(this);
    this.handleSubmit = this.handleSubmit.bind(this);
    this.handleUpdate = this.handleUpdate.bind(this);
  }

  async componentDidMount(){
    const notes = await API.graphql(graphqlOperation(readNote));
    this.setState({notes:notes.data.listNotes.items});
  }

  handleChange(event) {
    this.setState({value:event.target.value});
  }
  async handleSubmit(event) {
    event.preventDefault();
    event.stopPropagation();
    const note = {"note":this.state.value}
    await API.graphql(graphqlOperation(createNote, note));
    this.listNotes();
    this.setState({value:""});
  }
  async handleDelete(id) {
    const noteId = {"id":id};
    await API.graphql(graphqlOperation(deleteNote, noteId));
    this.listNotes();
  }
  async handleUpdate(event) {
    event.preventDefault();
    event.stopPropagation();
    const note = {"id":this.state.id,"note":this.state.value};
    await API.graphql(graphqlOperation(updateNote, note));
    this.listNotes();
    this.setState({displayAdd:true,displayUpdate:false,value:""});
  }
  selectNote(note){
    this.setState({id:note.id,value:note.note,displayAdd:false,displayUpdate:true});
  }
  async listNotes(){
    const notes = await API.graphql(graphqlOperation(readNote));
    this.setState({notes:notes.data.listNotes.items});
  }
  
  render() {
    const data = [].concat(this.state.notes)
      .map((item,i)=> 
      <div className="alert alert-primary alert-dismissible show" role="alert">
        <span key={item.i} onClick={this.selectNote.bind(this, item)}>{item.note}</span>
        <button key={item.i} type="button" className="close" data-dismiss="alert" aria-label="Close" onClick={this.handleDelete.bind(this, item.id)}>
          <span aria-hidden="true">&times;</span>
        </button>
      </div>
      )
    return (
      <div className="App">
        <header className="App-header">
          <img src={logo} className="App-logo" alt="logo" />
          <h1 className="App-title">Notes App</h1>
        </header>
        <br/>
        <div className="container">
          {this.state.displayAdd ?
            <form onSubmit={this.handleSubmit}>
              <div className="input-group mb-3">
                <input type="text" className="form-control form-control-lg" placeholder="New Note" aria-label="Note" aria-describedby="basic-addon2" value={this.state.value} onChange={this.handleChange}/>
                <div className="input-group-append">
                  <button className="btn btn-primary" type="submit">Add Note</button>
                </div>
              </div>
            </form>
          : null }
          {this.state.displayUpdate ?
            <form onSubmit={this.handleUpdate}>
              <div className="input-group mb-3">
                <input type="text" className="form-control form-control-lg" placeholder="Update Note" aria-label="Note" aria-describedby="basic-addon2" value={this.state.value} onChange={this.handleChange}/>
                <div className="input-group-append">
                  <button className="btn btn-primary" type="submit">Update Note</button>
                </div>
              </div>
            </form>
          : null }
        </div>
        <br/>
        <div className="container">
          {data}
        </div>
      </div>
    );
  }
}
export default withAuthenticator(App, { includeGreetings: true });
```

Check out the website!
```
amplify serve
```

Next, let's put this on S3:
```
amplify add hosting
amplify publish
```

Now for search, edit: schema.js
```
type Note @model @auth(rules: [{allow: owner}]) @searchable{
   id: ID!
   note: String!
}
```

Run amplify push.
```
amplify push
```

Update src/App.js
```javascript
import React, { Component } from 'react';
import logo from './logo.svg';
import './App.css';
import 'bootstrap/dist/css/bootstrap.min.css';
import Amplify, {API,graphqlOperation} from 'aws-amplify';
import { withAuthenticator} from 'aws-amplify-react'; 
import aws_exports from './aws-exports'; // specify the location of aws-exports.js file on your project
Amplify.configure(aws_exports);

const createNote = `mutation createNote($note: String!){
  createNote(input:{
    note: $note
  }){
    __typename
    id
    note
  }
}`;

const readNote = `query listNotes{
  listNotes{
    items{
      __typename
      id
      note
    }
  }
}`;

const updateNote = `mutation updateNote($id: ID!,$note: String){
  updateNote(input:{
    id: $id
    note: $note
  }){
    __typename
    id
    note
  }
}`;

const deleteNote = `mutation deleteNote($id: ID!){
  deleteNote(input:{
    id: $id
  }){
    __typename
    id
    note
  }
}`;

const searchNote = `query searchNotes($search: String){
  searchNotes(filter:{note:{match:$search}}){
    items{
      id
      note
    }
  }
}`

class App extends Component {
  constructor(props){
    super(props);
    this.state={
      id:"",
      notes:[],
      searchResults:[],
      value:"",
      displayAdd:true,
      displayUpdate:false,
      displaySearch:false
    };
    this.handleChange = this.handleChange.bind(this);
    this.handleSubmit = this.handleSubmit.bind(this);
    this.handleUpdate = this.handleUpdate.bind(this);
    this.handleSearch = this.handleSearch.bind(this);
  }

  async componentDidMount(){
    const notes = await API.graphql(graphqlOperation(readNote));
    this.setState({notes:notes.data.listNotes.items});
  }

  handleChange(event) {
    this.setState({value:event.target.value});
  }
  async handleSubmit(event) {
    event.preventDefault();
    event.stopPropagation();
    const note = {"note":this.state.value}
    await API.graphql(graphqlOperation(createNote, note));
    this.listNotes();
    this.setState({value:""});
  }
  async handleDelete(id) {
    const noteId = {"id":id};
    await API.graphql(graphqlOperation(deleteNote, noteId));
    this.listNotes();
  }
  async handleUpdate(event) {
    event.preventDefault();
    event.stopPropagation();
    const note = {"id":this.state.id,"note":this.state.value};
    await API.graphql(graphqlOperation(updateNote, note));
    this.listNotes();
    this.setState({displayAdd:true,displayUpdate:false,value:""});
  }
  async handleSearch(event) {
    event.preventDefault();
    event.stopPropagation();
    const search = {"search":this.state.value};
    const result = await API.graphql(graphqlOperation(searchNote, search));
    this.setState({searchResults:result.data.searchNotes.items,notes:[],displaySearch:true,value:""});   
    if(JSON.stringify(result.data.searchNotes.items) === '[]'){
      this.setState({searchResults:[{note:"No Match: Clear the search to go back to your Notes"}]});
    };
  }
  selectNote(note){
    this.setState({id:note.id,value:note.note,displayAdd:false,displayUpdate:true});
  }
  async listNotes(){
    const notes = await API.graphql(graphqlOperation(readNote));
    this.setState({notes:notes.data.listNotes.items,searchResults:[],displaySearch:false});
  }
  
  render() {
    const data = [].concat(this.state.notes)
      .map((item,i)=> 
      <div className="alert alert-primary show" role="alert">
        <span key={item.i} onClick={this.selectNote.bind(this, item)}>{item.note}</span>
        <button key={item.i} type="button" className="close" data-dismiss="alert" aria-label="Close" onClick={this.handleDelete.bind(this, item.id)}>
          <span aria-hidden="true">&times;</span>
        </button>
      </div>
      )
    const searchResults = [].concat(this.state.searchResults)
      .map((item,i)=> 
        <div className="alert alert-success show" role="alert">
          <span key={item.i}>{item.note}</span>
        </div>
      )
    return (
      <div className="App">
        <header className="App-header">
          <img src={logo} className="App-logo" alt="logo" />
          <h1 className="App-title">Notes App</h1>
        </header>
        <br/>
        <div className="container">
          {this.state.displayAdd ?
            <form>
              <div className="input-group mb-3">
                <input type="text" className="form-control form-control-lg" placeholder="New Note" aria-label="Note" aria-describedby="basic-addon2" value={this.state.value} onChange={this.handleChange}/>
                <div className="input-group-append">
                  <button className="btn btn-primary border border-light" type="button" onClick={this.handleSubmit}>Add Note</button>
                  <button className="btn btn-primary border border-light" type="button" onClick={this.handleSearch}>Search</button>
                </div>
              </div>
            </form>
          : null }
          {this.state.displayUpdate ?
            <form onSubmit={this.handleUpdate}>
              <div className="input-group mb-3">
                <input type="text" className="form-control form-control-lg" placeholder="Update Note" aria-label="Note" aria-describedby="basic-addon2" value={this.state.value} onChange={this.handleChange}/>
                <div className="input-group-append">
                  <button className="btn btn-primary" type="submit">Update Note</button>
                </div>
              </div>
            </form>
          : null }
        </div>
        <br/>
        <div className="container">
          {searchResults}
          {this.state.displaySearch ?
            <button className="button btn-success float-right" onClick={this.listNotes.bind(this)}>
              <span aria-hidden="true">Clear Search</span>
            </button>
          : null }
          {data}
        </div>
      </div>
    );
  }
}
export default withAuthenticator(App, { includeGreetings: true });
```

Serve the new site!
```
amplify serve
```

When ready to kill:
```
amplify delete
```
