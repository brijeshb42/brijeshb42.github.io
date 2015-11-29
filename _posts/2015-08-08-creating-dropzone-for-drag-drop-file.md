---
layout: post
title: "Creating a dropzone to enable drag and drop file upload"
color: purple
cover: "http://i1051.photobucket.com/albums/s432/brijeshb42/ghost-blog/dragndrop.png"
date: 2015-08-08 22:40:00
categories: 
tags: [javascript, dragndrop, fileupload]
---

This tutorial will show how to enable a particular `div` (or any other tag) on a webpage to accept files that are dropped on them.

<a href="http://bitwiser.in/tutorial-demos/2015/08/08/drag-n-drop-demo.html" class="mui-btn mui-btn-primary mui-btn-lg" target="_blank">Demo</a>
<a href="https://raw.githubusercontent.com/brijeshb42/tutorial-demos/gh-pages/_posts/2015-08-08-drag-n-drop-demo.md" class="mui-btn mui-btn-accent mui-btn-lg" target="_blank">Code</a>

We all have encountered websites that allow us to upload files (mostly images) to their servers by directly dragging the file from some application (mainly file explorer of the OS) to a desginated area on the web page. This a replacement of the good old `<input type="file" />` elements inside forms. **Drag and drop** is an HTML5 standard.

### Browser support

* All of the modern browsers fully support drag and drop HTML5 API except IE (partially supported).

### Getting started
As Drag n Drop is a new feature, we will also implement a fallback method to add support to older browsers so that users can upload files regardless of drag and drop is supported by their browsers or not.

We will create a function that will accept a DOM node and a callback function that is passed the list of files.

* The signature of the function will be `makeDroppable(element, callback)`.

* Inside the `makeDroppable` function,
  * We will first create an 'input' element of type `file`.
  * Set its display to **none** so that it is not visible on the page.
  * Add a listener to its `change` event that will call the function `triggerCallback` discussed further
  * Append it to the `element` provided.
  * This is to implement a fallback method in browsers that do not support drag n drop. This will also provide an additional functionality that the users can click the `element` to add files if they want, instead of drag n drop.

{% highlight javascript %}
var input = document.createElement('input');
input.setAttribute('type', 'file');
input.setAttribute('multiple', true);
input.style.display = 'none';

input.addEventListener('change', triggerCallback);
element.appendChild(input);
{% endhighlight %}

#### Now, we will add listeners to the `element`.

* The `dragover` event:
  * This event is called when files are being dragged over the `element` and are yet to be dropped.
  * We call the event's `preventDefault` method to prevent the browser from triggering its default behavior, i.e, it will try to open the file directly.
  * We call `stopPropagation` method to prevent the event from bubbling up the DOM tree (this will prevent the triggering of `dragover` event of any of the parent of `element`).
  * To add an effect to notify users that the `element` can accept files, we will add the class `dragover` to it that will have a slight different styling (like background color).

{% highlight javascript %}
element.addEventListener('dragover', function(e) {
  e.preventDefault();
  e.stopPropagation();
  element.classList.add('dragover');
});
{% endhighlight %}

* The `dragleave` event:
  * This event is fired when something was being dragged on the `element` but has been taken out instead of being dropped.
  * We call both the `preventDefault` and `stopPropagation` method as in `dragover` for the same reasons.
  * To notify that the files being dragged has been removed, we remove the class that we added in the `dragover` event so that `element` can return to its initial styling. 

{% highlight javascript %}
element.addEventListener('dragleave', function(e) {
  e.preventDefault();
  e.stopPropagation();
  element.classList.remove('dragover');
});
{% endhighlight %}

* The `drop` event:
  * This event is fired when the files being dragged are finally dropped on the `element`.
  * We call both the `preventDefault` and `stopPropagation` method as above.
  * We will then remove the `dragover` class to return the `element` to its inital style.
  * Now we call the `triggerCallback` function to handle the files.

{% highlight javascript %}
element.addEventListener('drop', function(e) {
  e.preventDefault();
  e.stopPropagation();
  element.classList.remove('dragover');
  triggerCallback(e);
});
{% endhighlight %}

* The `click` event:
  * Since the `input` element that we previously created is hidden on the page, we will add `click` event to the `element` so that we can trigger the file chooser window to allow users to choose files by browsing their file explorer instead of dragging and dropping.
  * We set the value of `input` to null.
  * Then we call the `input`'s `click` method to open the file chooser.

{% highlight javascript %}
element.addEventListener('click', function() {
  input.value = null;
  input.click();
});
{% endhighlight %}

* The `triggerCallback` function:
  * This function handles both the dropped files and the files selected through file chooser window.
  * We first create a variable `files` that will have the list of files chosen and will be passed to the `callback` provided.
  * Then we check if the event that called this function has a `dataTransfer` object or not.
  * If it has, that means the files were selected by dropping them on `element`. We assign the `e.dataTransfer.files` to the `files` variable.
  * If there is no `dataTransfer` object, that means the files were selected through the file chooser window. So we assign `e.target.files` to the `files` variable.
  * Now we pass the `files` variable to the `callback` function provided.
  * The `files` variable passed to the `callback` is an array of `File` objects. Each `File` object has the following main properties:
    * `lastModified`: Integer timestamp
    * `name`: The name of the file.
    * `size`: Size of file in bytes.
    * `type`: Mime-type of the file. (For ex: image/png for png files)

{% highlight javascript %}
function triggerCallback(e) {
  var files;
  if(e.dataTransfer) {
    files = e.dataTransfer.files;
  } else if(e.target) {
    files = e.target.files;
  }
  callback.call(null, files);
}
{% endhighlight %}

### Full code

{% highlight javascript linenos %}
function makeDroppable(element, callback) {

  var input = document.createElement('input');
  input.setAttribute('type', 'file');
  input.setAttribute('multiple', true);
  input.style.display = 'none';

  input.addEventListener('change', triggerCallback);
  element.appendChild(input);
  
  element.addEventListener('dragover', function(e) {
    e.preventDefault();
    e.stopPropagation();
    element.classList.add('dragover');
  });

  element.addEventListener('dragleave', function(e) {
    e.preventDefault();
    e.stopPropagation();
    element.classList.remove('dragover');
  });

  element.addEventListener('drop', function(e) {
    e.preventDefault();
    e.stopPropagation();
    element.classList.remove('dragover');
    triggerCallback(e);
  });
  
  element.addEventListener('click', function() {
    input.value = null;
    input.click();
  });

  function triggerCallback(e) {
    var files;
    if(e.dataTransfer) {
      files = e.dataTransfer.files;
    } else if(e.target) {
      files = e.target.files;
    }
    callback.call(null, files);
  }
}
{% endhighlight %}

### Usage:
We use the `makeDroppable` function like this:

* First create a `div` (or any other) element:
  * `<div class="droppable">Drop your files here.</div>`
  * Add style to the `div`:
    * `.droppable {
      background: #08c;
      color: #fff;
      padding: 100px 0;
      text-align: center;
    }`
  * Create another class `dragover` that will override the background of `.droppable`.
    * `.droppable.dragover {
      background: #00CC71;
    }`
  * Call the `makeDroppable`:

{% highlight javascript %}
var element = document.querySelector('.droppable');
function callback(files) {
  // Here, we simply log the Array of files to the console.
  console.log(files);
}
makeDroppable(element, callback);
{% endhighlight %}

### Uses:
This implementation can be used for several purposes:

* Showing a preview of image files (or videos) to the users
* Upload the file list to the server through ajax call.
* Read the files on the browser itself using `FileReader` (a topic for another tutorial).

#### Uploading the files through ajax (jQuery.ajax for simplicity)
The `callback` function that we defined earlier simply logs the `FileList` to console. We can modify that function to upload the dropped/selected files to the server.

{% highlight javascript %}
var element = document.querySelector('.droppable');
function callback(files) {
  var formData = new FormData();
  formData.append("files", files);

  $.ajax({
    url: '/server_upload_url',
    method: 'post',
    data: formData,
    processData: false,
    contentType: false,
    success: function(response) {
      alert('Files uploaded successfully.');
    }
  });
}
makeDroppable(element, callback);
{% endhighlight %}
