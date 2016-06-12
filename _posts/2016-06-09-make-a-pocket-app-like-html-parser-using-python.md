---
layout: post
title: "Make a Pocket app like html parser in Python"
cover: "http://i1051.photobucket.com/albums/s432/brijeshb42/ghost-blog/pocket-python.png"
date: 2016-06-09 12:30:00
tags: [python, pocket]
---

This tutorial will show how to extract only the relevant html from any article or blog post by their URL in **Python**.

Most of us have used [Pocket](https://getpocket.com/) app on our phones or browsers to save links and read them later. It is kind of like a bookmark app but which also saves the link's contents. After adding the link to Pocket, you can see that it extracts only the main content of the article and discards other things like the websites's footer, menu, sidebar (if any), user comments, etc. We will not be getting into the algorithm related to identifying the html tag with the most amount of text content. You can read the discussion here on [stackoverflow](http://stackoverflow.com/questions/3652657/what-algorithm-does-readability-use-for-extracting-text-from-urls) about the algorithms.


### Newspaper

Newspaper is a Python module that deals with extracting text/html from URLs. Besides this, it can also extract article's title, author, publish time, images, videos etc. And if used in conjunction with `nltk` it can also extract article's keywords and summary.

In this tutorial, we will be using [newspaper](https://github.com/codelucas/newspaper) to extract the html tag containing the relevant contents of the URL, and then we will expose this functionality on web using [flask](http://flask.pocoo.org/).


## Get started

We will be working in the global Python environment for simplicity of the tutorial. But you should do all the process described below in a virtual environment.

1. Install `newspaper` and `flask` using `pip`.
    * For `python2.x`, `pip install newspaper flask`
    * For `python3.x`, `pip install newspaper3k flask`
    * If there is some error while installing `newspaper` you can read the detailed guide specific to your platform [here](https://pypi.python.org/pypi/newspaper#get-it-now).

2. Create a file called `extractor.py` with the following code.

```python
from newspaper import Article, Config

config = Config()
config.keep_article_html = True


def extract(url):
    article = Article(url=url, config=config)
    article.download()
    article.parse()
    return dict(
        title=article.title,
        text=article.text,
        html=article.html,
        image=article.top_image,
        authors=article.authors,
    )
```

This is the only code we need for the main part thanks to `newspaper` which does all the heavy lifting. It identifies the html element containing the most relevant data, cleans it up, removes any `script`, `style` and other irrelevant tags that are not likely to make up the main article.

In the above code `Article` is `newspaper`'s way of representing the article from a URL. By default, `newspaper` does not save the article content's html to save some extra processing. That is why we are importing `Config` from `newspaper` and creating a custom configuration telling `newspaper` to keep the article html using `config.keep_article_html = True`.

* In the `extract` function that accepts a `url`, we first create an `Article` instance passing in the `url` and custom `config`.
* Then we download the full article html with `article.download()`. `newspaper` still hasn't processed the full html yet.
* Now we call `article.parse()`. After this, all the necessary data related to the article in `url` will be generated. The data includes article's title, full text, author, image etc.
* Then we return the data that we need in a `dict`.

### Exposing as an API

Now that we have created the functionality to extract articles, we will be making this available on the web so that we can test it out in our browsers. We will be using `flask` to make an API. Here is the code.

```python
from flask import (
    Flask,
    jsonify,
    request
)

from extractor import extract

app = Flask(__name__)


@app.route('/')
def index():
    return """
    <form action="/extract">
        <input type="text" name="url" placeholder="Enter a URL" />
        <button type="submit">Submit</button>
    </form>"""


@app.route('/extract')
def extract_url():
    url = request.args.get('url', '')
    if not url:
        return jsonify(
            type='error', result='Provide a URL'), 406
    return jsonify(type='success', result=extract(url))


 if __name__ == '__main__':
     app.run(debug=True, port=5000)
```

1. We first create a `Flask` app.
2. In its `index` route we return a simple form with text input where users will paste the url and then submit the form to the `/extract` route specified in `action`.
3. In the `extract_url` function, we get the `url` from `request.args` and check if it is empty. If empty, we return an error. Otherwise will pass the `url` to our `extract` function that we created and then return the result using `jsonify`.
4. Now you can simply run `python app.py` and head over to [http://localhost:5000](http://localhost:5000) in your browser it test the implemetation.

<a href="https://github.com/brijeshb42/extractor" class="btn btn-large">Full Code</a>

#### Few things to remember

* Instead of checking the url for just empty string, we can also use regex to verify that the url, is in fact a url.
* We should also check that the `url` is not for our own domain as it will lead to an infinite loop calling the same `extract_url` function again and again.
* `newspaper` will not always be able to extract the most relevant html. Its functionality completely depends on how the content is organized in the source `url`'s website. Sometimes, it may give one or two extra paragraphs or sometimes less. But for most of the standard news websites and blogs, it will always return the most relevant html.


#### Things to do next

The above demonstration is a very simple application that takes in a URL, returns the html and then forgets about it. But to make this more useful, you can take it a step further by:

1. Adding a database
2. Save the url and its extracted contents so that you can return the result from DB if the same URL is provided again.
3. You can add some more advanced APIs like returning a list of most recent queried/saved URLs and their title and other contents.
4. Then you can use the API service to create a web or android/ios app similar in features to what [Pocket](https://getpocket.com/) is.

#### Read More
* [newspaper API](http://newspaper.readthedocs.io/en/latest/).
