---
layout: post
title: "Creating a rich text editor - Part 2 - Text Styling"
cover: "http://i1051.photobucket.com/albums/s432/brijeshb42/ghost-blog/2abfadcb-d409-41d8-8a73-d9c07f07141d.png"
date: 2017-04-13 04:55:00 PM
tags: [react, draftjs, rte, tutorial]
---

In the first part of the [tutorial](http://bitwiser.in/2017/04/11/creating-rte-barebones-editor.html), we built the simplest form of the text editor where we can just type some text. In this part, we will be adding features to add basic text styling to the characters.

We will start by adding a functionality where one can add **bold**, *italic* and underline styles to selected text by pressing the stand keyboard shortcuts for those, i.e

| Mac                         | Windows/Linux                 | Action     |
|:---------------------------:|:-----------------------------:|:-----------|
| <kbd>CMD</kbd>+<kbd>B</kbd> | <kbd>CTRL</kbd>+<kbd>B</kbd>  | Bold       |
| <kbd>CMD</kbd>+<kbd>I</kbd> | <kbd>CTRL</kbd>+<kbd>I</kbd>  | Italic     |
| <kbd>CMD</kbd>+<kbd>U</kbd> | <kbd>CTRL</kbd>+<kbd>U</kbd>  | Uunderline |

We are using `draft-js-plugins` editor which allows us to build various plugins and pass them to the `Editor` component. We can choose to either create a single plugin with all the functionalities in it or build multiple plugins with single functionality in each. For this tutorial, we will be following the latter approach.

There are various handlers and properties that the `Editor` component can be passed for different kind of events. You can read the [documentation](https://draftjs.org/docs/api-reference-editor.html#content) here. `draft-js-editor` helps us in using these handlers in each of the plugins that we create.

The way `Draft-js` works is that instead of directly relying on browser's contenteditable API, it treats each keypress as an input. Then it analyzes the key combination and based on that it generates an output and renders it in the `contenteditable`. Internally, on each keypress, a string signal is generated which is then analyzed and output is rendered.

Now that we have understood how draft-js works, let's start by creating a plugin that handles keyboard shortcuts mentioned above. `Draft-js` has a utility method that detects common key combinations and emits desired signal strings. For example, when <kbd>CMD</kbd>+<kbd>B</kbd> is pressed on Mac, it emits string - `bold`. So we will be using this utility method to add basic styling to text using keyboard.

Simplest `draft-plugin-plugin` is just on object with all or some of the handlers that you want to use. We will be using the `keyBindingFn` handler of `Editor` to add B, I, U styling. Let's start by creating a `plugins` directory inside `src`. Now create a file called `basicTextStylePlugin.js` inside `plugins` and add the following code –

{% highlight javascript linenos %}
import {
  getDefaultKeyBinding,
  RichUtils,
} from 'draft-js';

const basicTextStylePlugin = {
  keyBindingFn(event) {
    return getDefaultKeyBinding(event);
  },

  handleKeyCommand(command, { getEditorState, setEditorState }) {
    const editorState = getEditorState();
    const newEditorState = RichUtils.handleKeyCommand(
      editorState, command
    );
    if (newEditorState) {
      setEditorState(newEditorState);
      return 'handled';
    }
    return 'not-handled';
  }
};

export default basicTextStylePlugin;
{% endhighlight %}

Above code creates the simplest form of `draft-js-plugins` where it is an object with the handlers to use, here, the handlers are `keyBindingFn` and `handleKeyCommand`. In each of the method of `draft-js-plugins`, besides the default argument passed by `draft-js`s `Editor` component, a second argument is also passed which is an object that has the `getEditorState` and `setEditorState` methods which we are using in `handleKeyCommand` to modify the content.

1. `keyBindingFn` takes the keyboard input `event` object and passes it to `getDefaultKeyBinding` utility function of `draft-js`, and returns the resultant string. In this case, when one presses <kbd>CMD</kbd>+<kbd>B</kbd>, it returns `bold`.
2. `handleKeyCommand` is called whenever a string signal is emitted. In this case, when we return `bold` string from `keyBindingFn`, it is passed onto `handleKeyCommand`. In this method, we can analyse the command string and perform actions based on that. Since we are just using internal command of `draft-js`, we won't be doing any extra checks right now. We will use `RichUtils` object of `draft-js` which takes as input, the `editorState` and the string `command` and returns the modified `editorState`. We use the `setEditorState` method to update our content after modification. We return a `handled` string if we have modified the contents on our own and don't want `draft-js` to apply the default behaviour. Otherwise we return `not-handled` so that other plugins, if any, can do their modifications.

Now, let us use this `basicTextStylePlugin` in our `MyEditor` component of `src/App.js`. Before proceeding, lets change the name of our files to reflect the names of the components they have. Change the name of `src/App.js` to `src/MyEditor.js`, `src/App.css` to `src/MyEditor.css`. Let's also updated the file refernces. In `src/MyEditor.js` update the import of `App.css` to `MyEditor.css` and update `index.js` to –

{% highlight javascript linenos %}
import React from 'react';
import ReactDOM from 'react-dom';
import MyEditor from './MyEditor';
import './index.css';

ReactDOM.render(
  <MyEditor />,
  document.getElementById('root')
);
{% endhighlight %}

At this point, your directory structure will look like this –

![Directory structure](https://res.cloudinary.com/beetoo/image/upload/v1492080966/rte/part2.png)

Now, let's start using the newly created plugin. Open `src/MyEditor.js` and modify the contents (modified lines are just below the inline comments) –

{% highlight javascript linenos %}
import 'draft-js/dist/Draft.css';
import './MyEditor.css';

import React from 'react';
import { EditorState } from 'draft-js';
import Editor from 'draft-js-plugins-editor';

/* Import the `basicTextStylePlugin` */
import basicTextStylePlugin from './plugins/basicTextStylePlugin';

class MyEditor extends React.Component {
  constructor(props) {
    super(props);

    this.state = {
      editorState: EditorState.createEmpty(),
    };

    /* Create an array of plugins to be passed to `Editor` */
    this.plugins = [
      basicTextStylePlugin,
    ];
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
          plugins={this.plugins} // Pass the plugins to the Editor
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

Now, you can open your dev server @ `http://localhost:3000` where you will be able to select some typed text and press <kbd>CMD</kbd>+<kbd>B</kbd> to make the selected text **bold**.

![Finished editor part 2](https://res.cloudinary.com/beetoo/image/upload/v1492082450/rte/part2-editor.gif)

At this point or even at the end of previous tutorial, you may have gotten a warning message in the console like this `Warning: Accessing PropTypes via the main React package is deprecated. Use the prop-types package from npm instead.`. This is because we are using the latest version of `React` from which `PropTypes` have been removed to another package called `prop-types` and other libraries (`draft-js` and `draft-js-plugins`) are using the old `PropTypes`. This is nothing to be worried about.

This completes the 2nd part of the tutorial. In next part, we will learn about `entities` and implement adding a link to the selected text.

You can get the source code from this [GitHub repo](https://github.com/brijeshb42/draft-text-editor-tutorial) . The code for this tutorial can be run from the `part2` tag. After cloning the repo, you can do

```bash
git checkout part2
```

------

List of tutorials in this series –

1. Part 1 - [Barebones Editor](http://bitwiser.in/2017/04/11/creating-rte-barebones-editor.html)
2. Part 2 - Text Styling
3. Part 3 - [Entities and decorators](http://bitwiser.in/2017/05/11/creating-rte-part-3-entities-and-decorators.html)
