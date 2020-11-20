---
layout: post
title:      "React and Props Explained"
date:       2020-11-20 05:43:53 +0000
permalink:  react_and_props_explained
---


Recently I was contemplating on finding a way understand what a component is how how to setup the parent/child relationship between them. Component, from my the point of comparison in my idea, are the models on your frontend realm that incorporated to React, about what you see as a user on the browser, are the building blocks that are defined by components at some point. 

Spin up a new React app by running: 

```
npx create-react-app my-app
```

instead of my-app, you can name anything you want: banana-app, my-dog-loves-superheros, etc.. it is endless!

It will generate all the gears you need to spin up a new React app and with all the file structures necessary to render a bare minimum React page. Once all installation complete, cd to that directory and type:

```
npm start
```

It will open a new port for you. If you have any local port already open, go ahead and create a new one but also don't forget about to turn it off as you making more projects. Something to think about, that I haven't touched yet but it should be fairly simple!


Index.js is the mother of all components that is the first to be rendered into browser. Look below: 

``import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';
import App from './App';
import reportWebVitals from './reportWebVitals';
// import BlogContent from './BlogContent';

ReactDOM.render(
  <React.StrictMode>
    <App />
    
  </React.StrictMode>,
  document.getElementById('root')
);

// If you want to start measuring performance in your app, pass a function
// to log results (for example: reportWebVitals(console.log))
// or send to an analytics endpoint. Learn more: https://bit.ly/CRA-vitals
reportWebVitals();


Second to that is the <App />. Wait?! What is the "< />? Well, this is why JSX, an abomination between the JavaScript and HTML comes to play, where you can actually use functions on your HTML element-like format, makes it more easier to see how an element functions. Well, this one is just simply runs App. The App is below: 

```
import logo from './logo.svg';
import './App.css';
import BlogPost from './BlogPost'

function App() {
  return (
    <div className="App">
      <header className="App-header">
        <img src={logo} className="App-logo" alt="logo" />
      
        <a
          className="App-link"
          href="https://reactjs.org"
          target="_blank"
          rel="noopener noreferrer"
        >
          This is rendered from App.js
        </a>
       <BlogPost />
      </header>
    </div>
  );
}

export default App;



```

A little blow, you see another <BlogPost />, so on and so on. As you see, whenever you see this structure know that this is a child componet to be called on. It says "Hey BlogPost! give me whatever you suppose to render". The BlogPost says "Yes, I will", but then you wonder how these components are communicating and render things between each other. The only way is to explicitly use syntax imports on top of the file and exports on the buttom. Depending on your needs, you define what should be imported or exported between each components, But ultimately all the child components export what they should return:


if there is entrance door, there should be an exit door! :D Just make sure whoever enters do or bring what they should suppose to do or bring respectively. 



So lets look at <BlogPost /> Component: 

```
import React from 'react'

import BlogContent from './BlogContent'


class BlogPost extends React.Component {
    render() {
      return (
        <div>
            <BlogContent articleText={"This line rendered from BlogContent"}/> 
        </div>
      )
    }
}

export default BlogPost
```


Ok! sounds cool! We are now expecting BlogContent return something to us, a text. You know what, let's take a look at BlogContent component: 

```
import React from 'react'
import Comment from './Comment'

class BlogContent extends React.Component {
    render() {
      return (
        <div>
          {this.props.articleText}
          <Comment commentText={"This is THE FIRST COMMENT rendered from Comment Class"}/>
          <Comment commentText={"This is THE SECOND COMMENT rendered from Comment Class "} />
        </div>  
      )
    }
  }

  export default BlogContent
```

Inside the dive, the first thing is  {this.props.articleText}. "articleText" inside the BlogContent? ah! So you are passing that text from BlogPost(parent class) as props to "BlogContent" so that you can connect these two inseperable family. the props are endless. you can pass text, numbers, boolean, objects, even functions!!! crazy huh? Next, look at the "<Comment />(also "Capitalize" comment with C! It is a norm :D) Another props with a text is passing called "commentText". Let's look at Comment Component: 

```
import React from 'react'
import SubComments from './SubComments'


class Comments extends React.Component {
    render() {
      return (
        <div>
            <ul>
            {this.props.commentText}
            </ul>
            <ul><SubComments subCommentText={"These are coming from subcomments"} /></ul>
            <ul><SubComments subCommentText={"These are coming from subcomments"} /></ul>
            <ul><SubComments subCommentText={"These are coming from subcomments"} /></ul>
            <ul><SubComments subCommentText={"These are coming from subcomments"} /></ul>
            <ul><SubComments subCommentText={"These are coming from subcomments"} /></ul>
        </div>
      )
    }
  }

  export default Comments
```

I think by now, you can see the pattern, next one is <SubComments /> with subCOmmentText prop: 

```
import React from 'react'

class SubComments extends React.Component {
    render(){
            return (
                <div>
                    <li>{this.props.subCommentText}</li>
                </div>
            )   
            
        }
    }


export default SubComments



Now take a look at the browser and see if you can identify what each of those lines come from which components . 

Happy Coding! 
