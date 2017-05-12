---
layout: post
title: "Creating a rich text editor - Part 3 - Entities and decorators"
cover: "http://i1051.photobucket.com/albums/s432/brijeshb42/ghost-blog/2abfadcb-d409-41d8-8a73-d9c07f07141d.png"
date: 2017-05-11 09:15:00 PM
tags: [react, draftjs, rte, tutorial]
---

In the second part of the [tutorial](http://bitwiser.in/2017/04/13/creating-rte-part-2-text-styling.html), we added keyboard shortcuts to change styling of selected text to bold, italic or underline.

In this part, we will use **draft-js** `Entity` to add metadata to characters that can be used to add hyperlinks to the selected text besides the basic styling.

In `Draft`, each and every character can have a metadata, meaning any kind of extra information about a character, besides the character itself, is stored in the editor JSON data. This range of text with metadata is called an `Entity`. An `Entity` also has a `decorator`. A `decorator` in `Draft` has two keys,

1. `strategy` - A function that iterates over the characters of a block and finds continuous ranges of text having the same entity.
2. `component` - The range of text is then rendered using the `React` component that is provided.

Each `Entity` has a `mutability` status which can be read about [here](https://draftjs.org/docs/advanced-topics-entities.html#mutability).

We will use the good old browser `prompt` to prompt user to type the link they want to add to the selected text.

The flow will be -

1. User selects a range of text
2. Presses a key combination (in this case <kbd>CMD/CTRL</kbd> + <kbd>K</kbd>)
3. Browser prompt opens up and asks user to type the link
4. User types the link and presses `RETURN`
5. The input link is added to the selected text and this will be designated by an anchor tag appearing for the selected text.

Let us create the link plugin to add links to text.

1. Create a file `addLinkPlugin.js` in `src/plugins`.
2. Add the following code

{% highlight javascript linenos %}
import React from 'react';
import {
  RichUtils,
  KeyBindingUtil,
  EditorState,
} from 'draft-js';


export const linkStrategy = (contentBlock, callback, contentState) => {
  contentBlock.findEntityRanges(
    (character) => {
      const entityKey = character.getEntity();
      return (
        entityKey !== null &&
        contentState.getEntity(entityKey).getType() === 'LINK'
      );
    },
    callback
  );
};


export const Link = (props) => {
  const { contentState, entityKey } = props;
  const { url } = contentState.getEntity(entityKey).getData();
  return (
    <a
      className="link"
      href={url}
      rel="noopener noreferrer"
      target="_blank"
      aria-label={url}
    >{props.children}</a>
  );
};

const addLinkPluginPlugin = {
  keyBindingFn(event, { getEditorState }) {
    const editorState = getEditorState();
    const selection = editorState.getSelection();
    // Don't do anything if no text is selected.
    if (selection.isCollapsed()) {
      return;
    }
    if (KeyBindingUtil.hasCommandModifier(event) && event.which === 75) {
      return 'add-link'
    }
  },

  handleKeyCommand(command, editorState, { getEditorState, setEditorState}) {
    if (command !== 'add-link') {
      return 'not-handled';
    }
    let link = window.prompt('Paste the link -');
    const selection = editorState.getSelection();
    if (!link) {
      setEditorState(RichUtils.toggleLink(editorState, selection, null));
      return 'handled';
    }
    const content = editorState.getCurrentContent();
    const contentWithEntity = content.createEntity('LINK', 'MUTABLE', { url: link });
    const newEditorState = EditorState.push(editorState, contentWithEntity, 'create-entity');
    const entityKey = contentWithEntity.getLastCreatedEntityKey();
    setEditorState(RichUtils.toggleLink(newEditorState, selection, entityKey))
    return 'handled';
  },

  decorators: [{
    strategy: linkStrategy,
    component: Link,
  }],
};

export default addLinkPluginPlugin;
{% endhighlight %}

Here we are -

* First creating a `strategy` as explained earlier to find continuous characters with `entity` type of `LINK`. It has the signature as defined above.
* Then we define a `Link` component that renders an `anchor` tag with the url from the entity data.
* Then we are creating the actual plugin.

In this plugin,

* We use the `keyBindingFn` to return a `string` of `add-link` if the key combination matches <kbd>CMD/CTRL</kbd> + <kbd>K</kbd> and some text is selected. `draft-js` has a utility object `KeyBindingUtil` with method `hasCommandModifier` to check `CMD` press on OSX or `CTRL` press on other device.
* Then the `handleKeyCommand` method checks whether the command is `add-link`. If not, it does nothing. Otherwise, it opens the browser prompt, takes in the input url, and applies that url as `LINK` entity to the selected text. `Entity` is applied by first creating a new content data using `content.createEntity` with entity type of `LINK`, mutability of 'MUTABLE' and the entity data. If no url is input, it tries to remove any existing link from the selection.
* This new content with entity applied is push onto the editor stack and then, `RichUtils.toggleLink` is called which sets the entity on the selected range of text.
* Finally, there is `decorators` which should be an array of objects with each object having a `strategy` and a `component`. We set the first object to the `strategy` and `component` defined earlier.

Now, let us use this newly created plugin in our editor component. Open `src/MyEditor.js` and update the code to this -

{% highlight javascript linenos %}
import 'draft-js/dist/Draft.css';
import './MyEditor.css';

import React from 'react';
import { EditorState } from 'draft-js';
import Editor from 'draft-js-plugins-editor';

import basicTextStylePlugin from './plugins/basicTextStylePlugin';
// import the add link plugin
import addLinkPlugin from './plugins/addLinkPlugin';

class MyEditor extends React.Component {
  constructor(props) {
    super(props);

    this.state = {
      editorState: EditorState.createEmpty(),
    };
    
    // add the plugin to the array
    this.plugins = [
      addLinkPlugin,
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
          plugins={this.plugins}
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

Now, type some text, select it and press <kbd>CMD/CTRL</kbd> + <kbd>K</kbd>, type a link and press <kbd>ENTER</kbd>. The selected text should show up as a link. If it does not, read away. I have encountered this bug while using `draft-js-plugins-editor` where before rendering the `Editor` its decorator is set to `null`. If the decorator is `null`, `Draft` won't know how to render a link entity. To prevent this, you should change the `onChange` method of `MyEditor` to this -

{% highlight javascript linenos %}
onChange = (editorState) => {
  if (editorState.getDecorator() !== null) {
    this.setState({
      editorState,
    });
  }
}
{% endhighlight %}

After this change, repeat the steps to add a link. It should now show the links as links.

Let's add some CSS to amke the links more brighter. Change the contents of `src/MyEditor.css` to this -

{% highlight css linenos %}
.editor {
  width: 600px;
  margin: 0 auto;
}

.editor a {
    color: #08c;
}
{% endhighlight %}

![Finished editor part 3](http://res.cloudinary.com/beetoo/image/upload/v1494533095/rte/draft-part3.gif)

Now we have a working link addition functionality. You can get the source code from this [GitHub repo](https://github.com/brijeshb42/draft-text-editor-tutorial) . The code for this tutorial can be run from the `part3` tag. After cloning the repo, you can do

```bash
git checkout part3
```

>
#### Note

>
If you are getting a `getEditorState` is not a function error in the console, change `handleKeyCommand` method in all the plugins from

>
```js
handleKeyCommand(command, { getEditorState, setEditorState})
```
>
TO
>
```js
handleKeyCommand(command, editorState, { getEditorState, setEditorState})
```
>
and also modify the use of `editorState` inside the method.

---

List of tutorials in this series –

1. Part 1 - [Barebones Editor](http://bitwiser.in/2017/04/11/creating-rte-barebones-editor.html)
2. Part 2 - [Text Styling](http://bitwiser.in/2017/04/13/creating-rte-part-2-text-styling.html)
3. Part 3 - Entities and decorators
