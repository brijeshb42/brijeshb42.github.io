---
layout: post
title: "Implementing todo list in Draft.js"
cover: "https://i1051.photobucket.com/albums/s432/brijeshb42/ghost-blog/2abfadcb-d409-41d8-8a73-d9c07f07141d.png"
date: 2016-08-31 05:30:00
tags: [react, draftjs]
---

In this tutorial, we will be using [Draft.js] [draft-js] to implement custom text blocks that will showcase the usage of block level metadata feature that comes with the latest [release](https://github.com/facebook/draft-js/releases/tag/v0.8.1) of Draft.js.

<p><a href="http://codepen.io/brijeshb42/pen/amojWq" class="btn btn-large" target="_blank">Full Code</a></p>

<p data-height="265" data-theme-id="dark" data-slug-hash="amojWq" data-default-tab="result" data-user="brijeshb42" data-embed-version="2" data-preview="true" class="codepen">See the Pen <a href="http://codepen.io/brijeshb42/pen/amojWq/">Draft.js with Todos</a> by Brijesh Bittu (<a href="http://codepen.io/brijeshb42">@brijeshb42</a>) on <a href="http://codepen.io">CodePen</a>.</p>
<script async src="//assets.codepen.io/assets/embed/ei.js"></script>

We will be implementing a Todo block where users will be presented with a check box besides each line of text which they can check or uncheck to mark that task as finished or unfinished respectively. The **Todo block** will look something like below :

<style type="text/css">
p.todo {
    position: relative;
    padding: 10px 20px;
    border: 1px solid #f7f7f7;
    margin-bottom: 10px;
    box-shadow: inset 0px 0px 10px 2px #f7f7f7;
    border-radius: 4px;
}

p.todo input[type=checkbox] {
    float: left;
    position: relative;
    top: 4px;
    left: -5px;
}
</style>
<p class="todo">
<input type="checkbox" checked readonly disabled> Create a rich text editor on top of Draft.js implementing advanced features.
</p>

<p class="todo">
<input type="checkbox" checked readonly disabled> Open source the <a href="http://bitwiser.in/medium-draft/">project</a>.
</p>

<p class="todo">
<input type="checkbox"> Write some tutorials explaining Draft.js and ways of achieving custom features with it.
</p>


### Assumption

You know what [Draft.js] [draft-js] is.

### Requirements

1. First we will have to allow users a way to add todo block or convert current text block into a todo block.

    To implement this, we will utilise the `handleBeforeInput` method of the Draft.js `Editor` component. `handleBeforeInput` is called just before a new character is typed into the editor. It is passed in as argument, the character that was typed.

    So whenever `[]` is typed at the start of a line, we will convert that line into a todo block.

2. Then we will create a custom `TodoBlock` which will have a checkbox to toggle that todo item.

Let us first implement a bare minimum editor. This won't even allow us to make text selection bold/italic etc. We will incrementally build upon this editor to achieve our features.

{% highlight javascript linenos %}
import React from 'react';

import {
    Editor,
    EditorState,
} from 'draft-js';

class MyTodoListEditor extends React.Component {
    
    constructor(props) {
        super(props);

        this.state = {
            editorState: EditorState.createEmpty(),
        };

        this.onChange = (editorState) => this.setState({ editorState });
    }

    render () {
        return (
            <Editor
                editorState={this.state.editorState}
                onChange={this.onChange} />
        );
    }
}

ReactDOM.render(<MyTodoListEditor />, document.getElementById('app'));
{% endhighlight %}

Now, since we will be implementing a custom block, we will need to pass a `blockRendererFn` and `blockRenderMap` as props to the `Editor` so that it knows what component to render for a particular block type. `blockRendererFn` is passed the block to be rendered. We will also need access to the `editorState` and `onChange` inside the `TodoBlock`. So we will implement a higher-order function that is passed the `editorState` and `onChange` and it will return another function to be used as `blockRendererFn`. `blockRenderMap` is an `Immutable` `Map`. We will also add a `blockStyleFn` prop to add custom classes to each block for styling purposes. Just before `MyTodoListEditor` definition add this function:

{% highlight javascript linenos %}
import { Map } from 'immutable';
import { DefaultDraftBlockRenderMap } from 'draft-js';

const TODO_BLOCK = 'todo';

/*
A higher-order function.
*/
const getBlockRendererFn = (getEditorState, onChange)
        => (block) => {
    const type = block.getType();
    switch(type) {
        case TODO_BLOCK:
            return {
                component: TodoBlock,
                props: {
                    getEditorState,
                    onChange,
                }
            };
        default:
            return null;
    }
};

/* Inside the constructor of MyTodoListEditor, add this property. */

this.blockRenderMap = Map({
    [TODO_BLOCK]: {
        element: 'div',
    },
}).merge(DefaultDraftBlockRenderMap);

/* Add this method inside MyTodoListEditor*/

blockStyleFn(block) {
    switch (block.getType()) {
      case TODO_BLOCK:
        return 'block block-todo';
      default:
        return 'block';
    }
}
{% endhighlight %}

Now the full code will look like this:

{% highlight javascript linenos %}
import React from 'react';

import { Map } from 'immutable';

import {
    Editor,
    EditorState,
    DefaultDraftBlockRenderMap,
} from 'draft-js';

/*
A higher-order function.
*/
const getBlockRendererFn = (getEditorState, onChange)
        => (block) => {
    const type = block.getType();
    switch(type) {
        case 'todo':
            return {
                component: TodoBlock,
                props: {
                    getEditorState,
                    onChange,
                },
            };
        default:
            return null;
    }
};

class MyTodoListEditor extends React.Component {
    
    constructor(props) {
        super(props);
        

        /* blockRenderMap */
        this.blockRenderMap = Map({
            [TODO_BLOCK]: {
                element: 'div',
            },
        }).merge(DefaultDraftBlockRenderMap);

        this.state = {
            editorState: EditorState.createEmpty(),
        };

        this.onChange = (editorState) => this.setState({ editorState });
        
        /*
        This function will be passed to getBlockRendererFn so that
        the component can access the latest editorState instead of an
        instance at some point.
        */
        this.getEditorState = () => this.state.editorState;
        
        // Get a blockRendererFn from the higher-order function.
        this.blockRendererFn = getBlockRendererFn(
            this.getEditorState, this.onChange);
    }

    blockStyleFn(block) {
        switch (block.getType()) {
          case TODO_BLOCK:
            return 'block block-todo';
          default:
            return 'block';
        }
    }

    render () {
        return (
            <Editor
                editorState={this.state.editorState}
                onChange={this.onChange}
                blockRenderMap={this.blockRenderMap}
                blockRendererFn={this.blockRendererFn}
                blockStyleFn={this.blockStyleFn} />
        );
    }
}

ReactDOM.render(<MyTodoListEditor />, document.getElementById('app'));
{% endhighlight %}

Above code won't run because we have yet to create the `TodoBlock` component. Let's do that:

{% highlight javascript linenos %}
import React from 'react';

import { EditorBlock } from 'draft-js';

const updateDataOfBlock = (editorState, block, newData) => {
  const contentState = editorState.getCurrentContent();
  const newBlock = block.merge({
    data: newData,
  });
  const newContentState = contentState.merge({
    blockMap: contentState.getBlockMap().set(block.getKey(), newBlock),
  });
  return EditorState.push(editorState, newContentState, 'change-block-type');
};

class TodoBlock extends React.Component {
    constructor(props) {
        super(props);
        this.updateData = this.updateData.bind(this);
    }

    updateData() {
        const { block, blockProps } = this.props;

        // This is the reason we needed a higher-order function for blockRendererFn
        const { onChange, getEditorState } = blockProps;
        const data = block.getData();
        const checked = (data.has('checked') && data.get('checked') === true);
        const newData = data.set('checked', !checked);
        onChange(updateDataOfBlock(getEditorState(), block, newData));
    }

    render() {
        const data = this.props.block.getData();
        const checked = data.get('checked') === true;
        return (
          <div className={checked ? 'block-todo-completed' : ''}>
            <input type="checkbox" checked={checked} onChange={this.updateData} />
            <EditorBlock {...this.props} />
          </div>
        );
    }
}
{% endhighlight %}

With the `TodoBlock`, we also have created a utility function `updateDataOfBlock` that will be called whenever the checkbox is clicked to toggle the `checked` value of that todo.

With all the above done, only thing remaining is a way to allow users to add todo blocks. As discussed, we will use `handleBeforeInput` to create a todo block whenever user types `[]` in an empty block. If user types `[]` in a todo block again, it will convert back to normal `unstyled` block. Let's add `handleBeforeInput` method inside the `MyTodoListEditor`:

{% highlight javascript linenos %}
/*
Returns default block-level metadata for various block type. Empty object otherwise.
*/
const getDefaultBlockData = (blockType, initialData = {}) => {
  switch (blockType) {
    case TODO_BLOCK: return { checked: false };
    default: return initialData;
  }
};

/*
Changes the block type of the current block.
*/
const resetBlockType = (editorState, newType = 'unstyled') => {
  const contentState = editorState.getCurrentContent();
  const selectionState = editorState.getSelection();
  const key = selectionState.getStartKey();
  const blockMap = contentState.getBlockMap();
  const block = blockMap.get(key);
  let newText = '';
  const text = block.getText();
  if (block.getLength() >= 2) {
    newText = text.substr(1);
  }
  const newBlock = block.merge({
    text: newText,
    type: newType,
    data: getDefaultBlockData(newType),
  });
  const newContentState = contentState.merge({
    blockMap: blockMap.set(key, newBlock),
    selectionAfter: selectionState.merge({
      anchorOffset: 0,
      focusOffset: 0,
    }),
  });
  return EditorState.push(editorState, newContentState, 'change-block-type');
};


/* Add this as a method inside MyTodoListEditor */

handleBeforeInput(str) {
    if (str !== ']') {
      return false;
    }
    const { editorState } = this.state;
    /* Get the selection */
    const selection = editorState.getSelection();

    /* Get the current block */
    const currentBlock = editorState.getCurrentContent()
      .getBlockForKey(selection.getStartKey());
    const blockType = currentBlock.getType();
    const blockLength = currentBlock.getLength();
    if (blockLength === 1 && currentBlock.getText() === '[') {
      this.onChange(resetBlockType(editorState, blockType !== TODO_BLOCK ? TODO_BLOCK : 'unstyled'));
      return true;
    }
    return false;
}
{% endhighlight %}

We have also defined `resetBlockType` and `getDefaultBlockData` utility functions to be used inside the `handleBeforeInput`

Now the Todo List Editor is implemented. But there are still some basic features remaining like allowing text to be styled as Bold/Italic/Underline. For that, we will add the following method in `MyTodoListEditor`.

{% highlight javascript linenos %}
import { RichUtils } from 'draft-js';

/*
Inside MyTodoListEditor, add this method. Also add
this as a prop to editor
*/

handleKeyCommand(command) {
    const {editorState} = this.state;
    const newState = RichUtils.handleKeyCommand(editorState, command);
    if (newState) {
      this.onChange(newState);
      return true;
    }
    return false;
}
{% endhighlight %}

Now users can press <kbd>Command/CTRL</kbd> + <kbd>B or I or U</kbd> and the style will be added to the selected text.

I have implemented above feature and many more advanced features in my project: [medium-draft](http://bitwiser.in/medium-draft/). Don't forget to check it out.

[draft-js]: https://facebook.github.io/draft-js/ "Draft.js Home Page"
