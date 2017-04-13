---
layout: post
title: "Creating a rich text editor - Part 1 - Barebones Editor"
cover: "http://i1051.photobucket.com/albums/s432/brijeshb42/ghost-blog/2abfadcb-d409-41d8-8a73-d9c07f07141d.png"
date: 2017-04-11 11:30:00 PM
tags: [react, draftjs, rte]
---

In this tutorial series, we will be creating a Rich Text Editor (RTE) using [Draft-js](https://draftjs.org/), [draft-js-plugins](https://www.draft-js-plugins.com) and [React](https://facebook.github.io/react/).

**Draft-js** is a framework that you can use to build rich text editors with customised functionalities. It stores the editor data in a json structure that you can use to have full control over the rendered HTML.

**draft-js-plugins** is a wrapper over *Draft-js* and provides easy way to create and use plugins with specific functionalities.

#### The editor will have the following features -

* Inline styling (like bold, italic, link).
* Block styling (like heading, quote, lists and nested lists).
* Basic keyboard shortcuts to minimise mouse usage.
* Ability to re-arrange blocks by dragging within the editor.
* Drag and drop images from your hard disk onto the editor.
* Images with captions.
* External embed support (using embedly).

#### Prerequisite Knowledge –

* You should have built at least one project in `React` (assuming that you already know JS) in ES6.
* You should also be familiar with `git` if you want to checkout the source code at every stage of the tutorial.

#### What you need ?

* You should have installed [`nodejs`](https://nodejs.org/) alongwith `npm`.

### Getting started

We will use the awesome [create-react-app](https://github.com/facebookincubator/create-react-app) library to bootstrap a basic React project. Then we will build our editor on top of it. First we will install `create-react-app` as a global node library. Fire up your terminal and run this command -

```bash
npm install -g create-react-app
```

After installation, a `create-react-app` command will be available to you to create your project.

Let's create our project. Run this command in a directory of your liking —

```bash
create-react-app my-text-editor
```

This will create a basic project in `my-text-editor` directory and install all the necessary initial dependencies. We will install our own dependencies as we require them.

Now, you can move into the directory and start the dev server using —

```bash
cd my-text-editor
npm start
```

This will open http://localhost:3000 in your default browser and at this point, your application will look something like this,

![](https://res.cloudinary.com/beetoo/image/upload/v1491933270/create-react-app_d8fjjq.png)

and the directory structure of the project will be like this —

![](https://res.cloudinary.com/beetoo/image/upload/v1491933266/directory-structure_sacemz.png)

Now, let us install our dependencies that will help us in implementing our editor. As already mentioned, we will install `draft-js` and `draft-js-plugins-editor`—

```bash
npm install draft-js draft-js-plugins-editor --save
```

This will install both the libraries and update the `package.json` file.
Now, let us implement the initial barebones editor where you can place your cursor and start writing some text. It won’t have any other features yet.

Open `src/App.js` file and remove all the initial code and add the following code —

{% highlight javascript linenos %}
import 'draft-js/dist/Draft.css';

import './App.css';

import React from 'react';
import { EditorState } from 'draft-js';
import Editor from 'draft-js-plugins-editor';

class MyEditor extends React.Component {
  constructor(props) {
    super(props);

    this.state = {
      editorState: EditorState.createEmpty(),
    };
  }

  componentDidMount() {
    this.focus();
  }

  onChange = (editorState) => {
    this.setState({
      editorState,
    });
  }

  focus = () => {
    this.editor.focus();
  }

  render() {
    const { editorState } = this.state;
    return (
      <div className="editor" onClick={this.focus}>
        <Editor
          editorState={editorState}
          onChange={this.onChange}
          ref={(element) => { this.editor = element; }}
          placeholder="Tell your story"
          spellCheck
        />
      </div>
    );
  }
}

export default MyEditor;
{% endhighlight %}

Lets see what the above code is doing —

1. We are importing the basic css from draft-js which will apply styling to the editor and the placeholder text. We are also importing our own `App.css` to add our own styles.
2. `EditorState` is used by draft-js to represent the contents, selection, undo/redo history stack to be used in the Editor.
3. `Editor` is the React component provided by `draft-js-plugins-editor` that renders the data created using `EditorState`. This component also accepts `placeholder` text, `spellCheck` boolean, a list of editor plugins and many other data which we will learn about as we need them.
4. We create our own `MyEditor` React component that is rendered to the page in `index.js`.
5. In the constructor, we are creating an empty `editorState` object in the state to store the editor data.
6. Then we have implemented the `onChange` and `focus` methods.
    1. `onChange` updates the `editorState` to the new contents as we type into the editor.
    2. `focus` places the cursor into the editor. It is called in the `componentDidMount` lifecycle method so that you can start typing as soon as the component is rendered.
7. In the `render` method, we render the `Editor` component wrapped in a `div`. `editorState` and `onChange` props are mandatory to be passed to this component. We are also passing the `placeholder` prop to show some initial text when the editor is empty and `spellCheck` so that spelling mistakes are highlighted.
8. In `ref`, we are saving a reference to the `Editor` component so that we can call methods of that component easily, like the `focus` method.

Lets also add some styling to make the editor look slightly better. It is pretty trivial styling and does not need any explanation.

Replace the contents of `src/index.css` with this —

{% highlight css linenos %}
body {
  margin: 0;
  padding: 10px;
  font-family: 'Georgia', serif;
}
{% endhighlight %}

and that of  `src/App.css` with this —

{% highlight css linenos %}
.editor {
  width: 600px;
  margin: 0 auto;
}
{% endhighlight %}

You can also remove the `src/logo.svg` file as it is not needed. And since unit testing is a tutorial in itself, I won’t be getting into that. You can also remove the `src/App.test.js` file.

At this point, your page should automatically reload and you should be able to see this —

![](https://res.cloudinary.com/beetoo/image/upload/v1491933269/part1-editor_cbcdfl.gif)

You will be able to type text into the editor, perform undo/redo using keyboard but you won’t be able to add style to the text like bold, italic etc.

You can get the source code from this [GitHub repo](https://github.com/brijeshb42/draft-text-editor-tutorial) . The code for this tutorial can be run from the `part1` tag. After cloning the repo, you can do

```bash
git checkout part1
```

and you will be able to see the code upto this point.

In the 2nd part of the tutorial, we will be adding features to add some basic styling to the text using keyboard.

---

List of tutorials in this series –

1. Part 1 - Barebones Editor
2. Part 2 - [Text Styling](http://bitwiser.in/2017/04/13/creating-rte-part-2-text-styling.html)
