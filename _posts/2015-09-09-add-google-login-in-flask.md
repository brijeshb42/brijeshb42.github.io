---
layout: post
title: "Add Google Oauth2 login in your flask web app"
color: light-blue
cover: "https://res.cloudinary.com/beetoo/image/upload/v1538725439/Artboard_1_baajan.png"
date: 2015-09-09 14:40:00
categories: 
tags: [google, oauth2, flask]
---

This post explains how to add [Google Oauth2](https://developers.google.com/identity/protocols/OAuth2?hl=en) login in a [Flask](http://flask.pocoo.org/) web app using the [requests-oauthlib](https://github.com/requests/requests-oauthlib) package for OAuth 2.0 and [flask-sqlalchemy](https://pythonhosted.org/Flask-SQLAlchemy/).

To get started, first we have to create a project in Google Developers Console to get client key and secret.

### Creating a Google project

1. First go to [Google Developers Console](https://console.developers.google.com/). Sign in using your Google credentials if you haven't already. There will be a list of projects(if you have previously created any).

2. Click on **Create Project** to create a new project.
![Project Screen](https://res.cloudinary.com/beetoo/image/upload/v1538725935/googflask/0_LbjTtoiOKxJ7Goja.png)

3. Provide a project name in the dialog box and press enter. For explanation purposes, lets say the project name is **test-project-123xyz**. `test-project-123xyz` will appear in the list of projects after creation.
![Create New projects](https://res.cloudinary.com/beetoo/image/upload/v1538725985/googflask/0_68HKckuDRB7g3lA5.png)

4. Now go to the project page. Click `APIs and Auth -> Credentials` in the sidebar. Then goto the `OAuth Consent Screen`. Provide the `Product Name`(you can also provide other details but they are optional). `Product Name` is what users see when they are logging into your application using Google.
![Product Name](https://res.cloudinary.com/beetoo/image/upload/v1538725966/googflask/0_khYIE00Mqx5tambK.png)

5. Now click on the `Credentials` part of the same page. Then click on `Add Credentials` and then select `OAuth 2.0 client ID`.
![OAuth Credentials](https://res.cloudinary.com/beetoo/image/upload/v1538726086/googflask/0_W2vKPKBaayFVCNNE.png)

6. Select `Application Type` as **Web Application**, Provide a `Name`, `Authorized Javascript origins` and `Authorized redirect URIs` and click on `Create`. During development, we will use `localhost` as our URL. Later, for production, we can add our original URL. The `redirect URIs` is important here as this is the URL the users will be redirected to after Google Login. Make sure that all the urls use `https` protocol as `OAuth2` supports only `https`.
![Create credentials](https://res.cloudinary.com/beetoo/image/upload/v1538726127/googflask/0_WVpwB_8QXYdQNRbn.png)

7. After the above step, you will be presented with a dialog box having your `client ID` and `client secret`. Copy both the strings and save in a text file as we will be needing these later.
![Copy credentials](https://res.cloudinary.com/beetoo/image/upload/v1538726158/googflask/0_YD7pfoE9XGuV4mGE.png)


### Creating a User table in Database

We will be using [flask-sqlalchemy](https://pythonhosted.org/Flask-SQLAlchemy/) to handle DB operations.
This is what our `User` table looks like.

{% highlight python linenos %}
class User(db.Model):
    __tablename__ = "users"
    id = db.Column(db.Integer, primary_key=True)
    email = db.Column(db.String(100), unique=True, nullable=False)
    name = db.Column(db.String(100), nullable=True)
    avatar = db.Column(db.String(200))
    active = db.Column(db.Boolean, default=False)
    tokens = db.Column(db.Text)
    created_at = db.Column(db.DateTime, default=datetime.datetime.utcnow())
{% endhighlight %}

The `tokens` column stores the access and refresh tokens JSON, dumped as string.


### Creating configuration for our app.

If using `flask-login` to manage user sessions, we can check whether a user is logged in or not. If not logged in, we redirect the user to a login page that contains the link to Google login.
Lets create a `config.py` that has our Google OAuth credentials and our app configuration.

{% highlight python linenos %}
import os

basedir = os.path.abspath(os.path.dirname(__file__))


class Auth:
    CLIENT_ID = ('688061596571-3c13n0uho6qe34hjqj2apincmqk86ddj'
                 '.apps.googleusercontent.com')
    CLIENT_SECRET = 'JXf7Ic_jfCam1S7lBJalDyPZ'
    REDIRECT_URI = 'https://localhost:5000/gCallback'
    AUTH_URI = 'https://accounts.google.com/o/oauth2/auth'
    TOKEN_URI = 'https://accounts.google.com/o/oauth2/token'
    USER_INFO = 'https://www.googleapis.com/userinfo/v2/me'


class Config:
    APP_NAME = "Test Google Login"
    SECRET_KEY = os.environ.get("SECRET_KEY") or "somethingsecret"


class DevConfig(Config):
    DEBUG = True
    SQLALCHEMY_DATABASE_URI = 'sqlite:///' + os.path.join(basedir, "test.db")


class ProdConfig(Config):
    DEBUG = True
    SQLALCHEMY_DATABASE_URI = 'sqlite:///' + os.path.join(basedir, "prod.db")


config = {
    "dev": DevConfig,
    "prod": ProdConfig,
    "default": DevConfig
}
{% endhighlight %}

Here,

* `REDIRECT_URI` is what we set in Google Developers Console,
* `AUTH_URI` is where the user is taken to for Google login,
* `TOKEN_URI` is used to exchange a temporary token for an `access_token` and
* `USER_INFO` is the URL used for retrieving user information like *name*, *email*, etc after successful authentication.
* `SCOPE` is the types of user information that we will be accessing after the user authenticates our app. [Google OAuth2 Playground](https://developers.google.com/oauthplayground/) has a list of scopes that can be added.

### Implementing the URL routes for login and callback

After the configuration is done, we have to create a `Flask` app, load configurations and finally define our routes.

{% highlight python linenos %}
app = Flask(__name__)
app.config.from_object(config['dev'])
db = SQLAlchemy(app)
login_manager = LoginManager(app)
login_manager.login_view = "login"
login_manager.session_protection = "strong"
{% endhighlight %}

### `requests_oauthlib.OAuth2Session` helper:
We create a helper function `get_google_auth` that we will use to create `OAuth2Session` object based on the arguments provided.

{% highlight python linenos %}
def get_google_auth(state=None, token=None):
    if token:
        return OAuth2Session(Auth.CLIENT_ID, token=token)
    if state:
        return OAuth2Session(
            Auth.CLIENT_ID,
            state=state,
            redirect_uri=Auth.REDIRECT_URI)
    oauth = OAuth2Session(
        Auth.CLIENT_ID,
        redirect_uri=Auth.REDIRECT_URI,
        scope=Auth.SCOPE)
    return oauth
{% endhighlight %}

* When none of the parameters are provided, e.g. `google = get_google_auth()`, it creates a new `OAuth2Session` with a new state.
* If `state` is provided, that means we have to get a `token`.
* If `token` is provided, that means we only have to get an `access_token` and this is the final step.

### Root URL:
{% highlight python linenos %}
@app.route('/')
@login_required
def index():
    return render_template('index.html')
{% endhighlight %}

This route is only served to logged in user. If a user is not logged in, they are redirected to `login` route as set previously using `login_manager.login_view = "login"`.

### Login URL:
{% highlight python linenos %}
@app.route('/login')
def login():
    if current_user.is_authenticated:
        return redirect(url_for('index'))
    google = get_google_auth()
    auth_url, state = google.authorization_url(
        Auth.AUTH_URI, access_type='offline')
    session['oauth_state'] = state
    return render_template('login.html', auth_url=auth_url)
{% endhighlight %}

Here we save the value of `state` in cookie using `session['oauth_state'] = state` to be used later.

### Callback URL:
Here, the route `gCallback` must be the same as we mentioned in our project page in Google Developers Console.

{% highlight python linenos %}
@app.route('/gCallback')
def callback():
    # Redirect user to home page if already logged in.
    if current_user is not None and current_user.is_authenticated:
        return redirect(url_for('index'))
    if 'error' in request.args:
        if request.args.get('error') == 'access_denied':
            return 'You denied access.'
        return 'Error encountered.'
    if 'code' not in request.args and 'state' not in request.args:
        return redirect(url_for('login'))
    else:
        # Execution reaches here when user has
        # successfully authenticated our app.
        google = get_google_auth(state=session['oauth_state'])
        try:
            token = google.fetch_token(
                Auth.TOKEN_URI,
                client_secret=Auth.CLIENT_SECRET,
                authorization_response=request.url)
        except HTTPError:
            return 'HTTPError occurred.'
        google = get_google_auth(token=token)
        resp = google.get(Auth.USER_INFO)
        if resp.status_code == 200:
            user_data = resp.json()
            email = user_data['email']
            user = User.query.filter_by(email=email).first()
            if user is None:
                user = User()
                user.email = email
            user.name = user_data['name']
            print(token)
            user.tokens = json.dumps(token)
            user.avatar = user_data['picture']
            db.session.add(user)
            db.session.commit()
            login_user(user)
            return redirect(url_for('index'))
        return 'Could not fetch your information.'
{% endhighlight %}

In the above code,

* We check if a user is already logged in. If yes, we then redirect them to the home page.
* Then we check if the url has an `error` query parameter. This check is done to handle cases where a user after going to the Google login page, denies access. We then return an appropriate message to the user.
* We then check if the url contains `code` and `state` parameters or not. If these are not in the URL, this means that someone tried to access the URL directly. So we redirect them to the login page.
* After handling all the side cases, we finally handle the case where the user has successfully authenticated our app.
    * In this case, we create a new `OAuth2Session` object by passing the `state` parameter.
    * Then we try to get an `access_token` from Google using
{% highlight python %}
token = google.fetch_token(
    Auth.TOKEN_URI,
    client_secret=Auth.CLIENT_SECRET,
    authorization_response=request.url)
{% endhighlight %}

If error occurs, we return appropriate message to user.

* After getting the `token` successfully, we again create a new `OAuth2Session` by setting the `token` parameter.
* Finally we try to access the user information using

{% highlight python %}
google = get_google_auth(token=token)
resp = google.get(Auth.USER_INFO)
{% endhighlight %}

The user information is a JSON of the form:
{% highlight json linenos %}
{
  "family_name": "Doe", 
  "name": "John Doe", 
  "picture": "https://lh3.googleusercontent.com/-asdasdas/asdasdad/asdasd/daadsas/photo.jpg", 
  "locale": "en", 
  "gender": "male", 
  "email": "john@gmail.com", 
  "link": "https://plus.google.com/+JohnDoe", 
  "given_name": "John", 
  "id": "1109367330250025112153346", 
  "verified_email": true
}
{% endhighlight %}

After getting the user information, its upto us to how to handle the information. In the callback code, we handle the information by:

* First we check if a user with the retrieved `email` is already in the DB or not. If user is not found, we create a new `user` and assign the `email` to it.
* Then we set the other attributes like `avatar`, `tokens` and then finally commit the changes to DB.
* After commiting the changes, we login the user using `login_user(user)` and then redirect the user to home page.

### [Full Code - app.py](https://gist.github.com/brijeshb42/f4dcac5c8f4d4ab4a73a)

To run the above code, first create the DB by opening the python console and executing:
{% highlight python %}
from app import db
db.create_all()
{% endhighlight %}

Then create a test ssl certificate using `werkzeug`:
{% highlight python %}
from werkzeug.serving import make_ssl_devcert
make_ssl_devcert('./ssl', host='localhost')
{% endhighlight %}

Finally create a file `run.py` in the same directory as `app.py` and add the following code and finally run `python run.py`:

{% highlight python %}
from app import app
app.run(debug=True, ssl_context=('./ssl.crt', './ssl.key'))
{% endhighlight %}

#### Edit

From the comments, I have come to know that many of you have been running into problems with successfully running the flask server. It was a mistake from my side which I have realized in last few days.

In Step 6 above, while adding a redirect uri in the **Google Project Console**, I attached a screenshot in which, the `redirect uri` was `http://localhost:5000/gCallback` instead of it starting with `https`. So I have updated the screenshot in which I have added both `http` and `https`. You should add both `http` and `https` URLs as redirect uri. Also add both `http://localhost:5000` and `https://localhost:5000` in the **Authorized Javascript Origins**.

Also if you want to simply run the flask server on http instead of https, add the following 2 lines of code at the top of your `app.py`

{% highlight python %}
import os
os.environ['OAUTHLIB_INSECURE_TRANSPORT'] = '1'
{% endhighlight %}
