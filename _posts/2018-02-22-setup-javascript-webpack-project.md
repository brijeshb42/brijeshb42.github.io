---
layout: post
title: "Setup a ES6 javascript project using webpack and babel"
cover: "https://res.cloudinary.com/beetoo/image/upload/v1519315520/js-webpack-babel_peqovw.png"
date: 2018-02-22 09:30:00 PM
tags: [javascript, webpack, babel, es6, tutorial]
---

Table of contents -

1. [Setup](#part-1)
2. [Styling](#part-2)
3. [Linting](#part-3)

Whenever someone learns a new programming language, there is a 99% chance that their first program is going to be a **Hello World** program. In this proverbial program, they are supposed to print `Hello World` on their screen/console. Depending on the language, it can range from 1 line program to multiline just for printing this **Hello World**.

In Javascript, in olden times (4-5 years back), one would simply create an HTML file with this content and open it up in their browsers to see **Hello World** printed in their browser windows (and also in the browser console).

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Hello World</title>
  </head>
  <body>
    <p>Hello World</p>
    <script>
      console.log('Hello World');
    </script>
  </body>
</script>
```

But as javascript ecosystem has matured, this process has gotten a little complicated (for the better). In this tutorial, you will get to know how to set up this type of project.

### Assumptions

* You know Javascript (preferrably some es6 too).
* You have `nodejs` and `npm` installed on your system ([Tutorial](https://docs.npmjs.com/getting-started/installing-node)).

The full code is available at [https://github.com/brijeshb42/hello-world-tutorial](https://github.com/brijeshb42/hello-world-tutorial).

## Part 1

Open your terminal app or Command Prompt and `cd` to a directory where you would like to create this project. Let's assume the project folder is called `hello-world` in some directory on your disk. Now type these commands -

1. `cd hello-world`
2. `npm init --y`

This will create a `package.json` file in `hello-world` directory. `package.json` is file in your project which is used by `nodejs` and `npm` to keep track of installed packages and your project's metadata. Your `package.json` might look something like this -

```json
{
  "name": "hello-world",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}
```

Now add webpack and dev-server -

```bash
npm install webpack webpack-dev-server --save-dev
```

at the time of writing this, the version of webpack installed was `3.11.1`.

Create a `src` directory inside your project folder and then create `index.js` file inside it.

1. `mkdir src`
2. `echo "console.log('Hello world');" > src/index.js`

This is our hello world program that will print `Hello world` in the browser console when run.

At this point, you can start with writing a webpack config file to bundle your files for browser to load.

Create a `webpack.config.js` file in your project folder with the following content. This file is used by webpack to read your configuration and build project accordingly.

```js
const path = require('path');

module.exports = {
  entry: {
    bundle: './src/index.js'
  },
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'bundle.js'
   }
};
```

Now, you can run this command for webpack to load the entry file and create a bundled js file in `dist` directory in the root of your project directory.
 
```bash
./node_modules/.bin/webpack
```

This is the build command that will bundle up all the dependencies and create a `bundle.js` file as specified in the `output` option of webpack config file. After running this command, you can see a `bundle.js` file in `dist`. You can not load this js file yet as you first have to have an html file. Browser will load that file which in turn will load the js file.
 You can manually create an `index.html` file in `dist` with this content.

```html
<script src="./bundle.js"></script>
```

This is the minimum amount of html required to load and run our bundled js. Now you can double click this html file which will open it in a browser. You can open the browser console using `CMD/CTRL` `+` `SHIFT` `+` `I` to see the output. Let's look at a better way through which you don't have to write the html file.

`npm install html-webpack-plugin --save-dev`

This is a webpack plugin that automatically generates the `index.html` file in `dist` with proper references to all the generated javascript files. To use this plugin, update your `webpack.config.js` with this -

```diff
  const path = require('path');
+ const HtmlWebpackPlugin = require('html-webpack-plugin');

 module.exports = {
   entry: {
    bundle: './src/index.js'
  },
   output: {
     path: path.resolve(__dirname, 'dist'),
      filename: 'bundle.js'
-   }
+  },
+  plugins: [
+    new HtmlWebpackPlugin()
+  ]
 };
```
 
After this, you can run the build command again -

```bash
./node_modules/.bin/webpack
```

This will now create and extra `index.html` file in `dist` directory with proper script tags to include `bundle.js`. This can now be opened in browser directly and it'll work like before, except that you didn't have to create it yourself. 

To make the build command shorter, lets create an alias inside `package.json` so that you only have to type `npm run build` to bundle your files. Update your `package.json` -

```diff
 {
   "name": "hello-world",
   "version": "1.0.0",
    "description": "",
   "main": "index.js",
   "scripts": {
     "test": "echo \"Error: no test specified\" && exit 1"
   },
   "keywords": [],
   "author": "",
   "license": "ISC",
   "devDependencies": {
     "html-webpack-plugin": "^2.30.1",
     "webpack": "^3.11.0",
     "webpack-dev-server": "^2.11.1"
-  }
+  },
+  "scripts": {
+    "build": "webpack"
+  }
 }
```

Till now, webpack bundles the files and exits. This is good when you just want to bundle and deploy to your local or remote server. But during development, this can get frustrating really quickly. To overcome this frustration, you'll use `webpack-dev-server` which constantly watches your files for changes and refreshes the page in browser instantly. It also starts a development server inside `dist` so the html file is loaded from a server instead of a file system (just in case you are using ajax in your js which does not work when opened from filesystem). Install it using -

`npm install webpack-dev-server --content-base dist`

This starts the development server with `dist` as the base directory. The default url is `http://localhost:8080`. Opening this url in your browser will load the `index.html` file and log `Hello World` in the console. Now if you update your console log from `Hello World` to `Hi World` inside `src/index.js`, `webpack-dev-server` will automatically reload the browser and you will be able to see the new output.

`./node_modules/.bin/webpack-dev-server --content-base dist`

Lets add this also as an alias in `package.json` -

```diff
 {
   "name": "hello-world",
   "version": "1.0.0",
   "description": "",
   "main": "index.js",
   "scripts": {
     "test": "echo \"Error: no test specified\" && exit 1"
   },
   "keywords": [],
   "author": "",
   "license": "ISC",
   "devDependencies": {
     "html-webpack-plugin": "^2.30.1",
     "webpack": "^3.11.0",
     "webpack-dev-server": "^2.11.1"
   },
   "scripts": {
     "build": "webpack",
+    "dev": "webpack-dev-server --content-base dist"
   }
 }
```

Now running `npm run dev` will start `webpack-dev-server` with auto reload on changes.

At this point, you cannot use es6 syntax in your js code yet. Let's add that support. This will be done by using `babel`. To add babel support in the build process, let us first install it. The `babel-loader` will require `babel-core` to be installed. And to support es6/7/8/* syntax, you'll add `babel-preset-env`. Run this in your terminal in the project folder -

`npm install babel-core babel-loader babel-preset-env --save-dev`

First create a `.babelrc` file in project directory so that babel can load its configuration. Add this to the file -

```json
{
  "presets": [[
    "env", {
      "targets": {
        "browsers": ["Chrome >= 55"]
      }
    }
  ]]
}
```

This configuration is used deliberately so that you can see the bundled js file in `dist` directory and discern how your es6 code has been transpiled. As browsers started supporting more and more es6 features, `babel`, instead of transpiling all of the code blindly, now smartly identifies which features are supported natively and does not transpile those parts. This reduces the overall bundle size.

The simplest configuration to be used instead of the above (if you don't care about browser version) would have been -

```json
{
  "presets": ["env"]
}
```

Now let's instruct `webpack` to use babel to transpile the `js` files first.

```diff
  const path = require('path');
+ const HtmlWebpackPlugin = require('html-webpack-plugin');

 module.exports = {
   entry: `{
    bundle: './src/index.js'
  },
   output: {
     path: path.resolve(__dirname, 'dist'),
      filename: 'bundle.js'
    },
   plugins: [
     new HtmlWebpackPlugin()
-  ]
+  ],
+  module: {
+    rules: [{
+      test: /\.js$/,
+     exclude: /node_modules/,
+     use: 'babel-loader'
+   }]
+ }
 };
```

Create a new file `src/message.js` and add this - 

```js
export default "Hello World";
```

Now modify `src/index.js` to use the simplest es6 feature of importing -

```js
import message from './message';
 
console.log(message);
```

In above code, es6 module syntax is used. Now running `npm run dev` will create an updated bundle (though the output is same) which you can test in your browser console.

This sums up the first part of the tutorial where you have setup the simplest (seriously simplest) javascript project using webpack for bundling with babel integration for transpiling es6 to es5.

-----------------

## Part 2

Now, let's move on to second part of the tutorial where we'll setup webpack to import `css` files. Through this, you can directly include styles in your javascript files.

First, let's modify `src/index.js` to show some text on the page instead of just logging to console.

```diff
 import message from './message';

-console.log(message);
+const paragraph = document.createElement('p');
+paragraph.innerHTML = message;
+
+document.body.prepend(paragraph);
```

This creates a `p` tag with the imported `message` as the html and adds it to the page.

Now, let's style this `p` tag using css. This requires `css-loader` and `style-loader`. Install it using -

`npm install css-loader style-loader --save-dev`

To support `css` file importing, let's update our `webpack.config.js` with a new rule which tests if an imported file has `css` extension and parses it using `style-loader` and `css-loader` -

```diff
  const path = require('path');
  const HtmlWebpackPlugin = require('html-webpack-plugin');
 
  module.exports = {
   entry: {
     bundle: './src/index.js '
   },
   output: {
     path: path.resolve(__dirname, 'dist'),
      filename: 'bundle.js'
    },
   plugins: [
     new HtmlWebpackPlugin()
   ],
   module: {
     rules: [{ 
        test: /\.js$/,
        exclude: /node_modules/,
       use: 'babel-loader'
+    }, {
+      test: /\.css$/,
+      exclude: /node_modules/,
+      use: [
+        {loader: 'style-loader'},
+        {loader: 'css-loader'}
+       ]
+     }]
   }
 };
```

Now create a css file `src/index.css` and style the `p` tag -

```css
p {
   color: red;
} 
```

Import this css file in `src/index.css` -

```diff
 import message from './message';
+import './index.css';

 const paragraph = document.createElement('p');
 paragraph.innerHTML = message;
 
 document.body.prepend(paragraph);
```

Now, restart dev server using `npm run dev`. You'll be able to see that the page now show `Hello World` in red color. If you change the color from to red to blue in `index.css`, the page will reload and new style will be visible. To see the new style without the actual page reload, modify the dev server command in `package.json` -

```diff
 {
  "name": "hello-world",
  "version": "1.0.0", 
  "description": "",
  "main": "index.js",
  "scripts": {
    "build": "webpack",
-    "dev": "webpack-dev-server --content-base dist"
+    "dev": "webpack-dev-server --content-base dist --hot"
  },
  "keywords": [],
  "author": "" ,
  "license": "ISC",
  "devDependencies": {
    "babel-core": "^6.26.0",
    "babel-loader": "^7.1.2",
    "babel-preset-env": "^1.6.1",
    "css-loader": "^0.28.9",
    "html-webpack-plugin": "^2.30.1",
    "style-loader": "^0.20.2",
    "webpack": "^3.11.0",
    "webpack-de v-server": "^2.11.1"
  }
 }
```

This enables hot module replacement in webpack which shows the new changes in your code (in `css` or `js` or any file as long as webpack knows how to load it) without full reload of the page. Restart the server with `npm run dev` and try to change the color of `p` in css. You'll notice that the color changes in page without actually reloading the page.

If you try to run the build command, `npm run build`, in the `dist` directory, you'll notice that there aren't any css files built. That is because webpack adds the styles in javascript bundles as strings and applies these styles in the page by creating `style` tags. This is fine when you are developing. But during deployment process, it is always a good practice to include your css files in the `head` tag so that the page look is not compromised while the javascript is loading. To fix this, we'll use `extract-text-webpack-plugin` which extracts all the imported css to its own file during the build process. Before this, let's first setup webpack to understand `development` and `production` mode.

```diff
  const path = require('path');
  const HtmlWebpackPlugin = require('html-webpack-plugin');

+ const env = process.env.NODE_ENV || 'development';
+ const isDev = env === 'development';
+ const isProd = env === 'production';

  module.exports = {
    entry: {
      bundle: './src/index.js'
    },
    output: {
      path: path.resolve(__dirname, 'dist'),
      filename: 'bundle.js'
    },
    plugins: [
      new HtmlWebpackPlugin()
    ],
    module: {
      rules: [{
        test: /\.js$/,
        exclude: /node_modules/,
       use: 'babel-loader'
      }, {
        test: /\.css$/,
        exclude: /node_modules/,
        use: [
         {loader: 'style-loader'},
          {loader: 'css-loader'}
        ]
      }]
    }
  };
```

And modify `package.json` to run build command in production mode and dev server in development mode.

```diff
 { 
   "name": "hello-world",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
-    "build": "webpack",
-    "dev": "webpack-dev-server --content-base dist --hot"
+    "build": "NODE_ENV=production webpack",
+    "dev": "NODE_ENV=development webpack-dev-server --content-base dist --hot"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "babel-core": "^6.26.0",
    "babel-loader": "^7.1.2",
    "babel-preset-env": "^1.6.1",
    "css-loader": "^0.28.9",
    "extract-text-webpack-plugin": "^3.0.2",
    "html-webpack-plugin": "^2.30.1",
    "style-loader": "^0.20.2",
    "webpack": "^3.11.0",
    "webpack-dev-server": "^2.11.1"
  }
 }
```

Now install `extract-text-webpack-plugin` using -

`npm install extract-text-webpack-plugin --save-dev`

And update `webpack.config.js` -

```diff
 const path = require('path');
 const HtmlWebpackPlugin = require('html-webpack-plugin');
+const ExtractTextPlugin = require('extract-text-webpack-plugin');

 const env = process.env.NODE_ENV || 'development';
 const isDev = env === 'development';
 const isProd = env === 'production';

+const extractCss = new ExtractTextPlugin({
+  filename: 'index.css',
+  disable: isDev
+});

 module.exports = {
   entry: {
     bundle: './src/index.js'
   },
    output: {
     path: path.resolve(__dirname, 'dist'),
    filename: 'bundle.js'
   },
   plugins: [
-    new HtmlWebpackPlugin()
+    new HtmlWebpackPlugin(),
+    extractCss
   ],
   module: {
     rules: [{
       test: /\.js$/,
      exclude: /node_modules/,
       use: 'babel-loader'
     }, {
       test: /\.css$/,
      exclude: /node_modules/,
       use: extractCss.extract({
         use:[
           {loader: 'css-loader'}
         ],
         fallback: 'style-loader'
      })
     }]
   }
 };
```

This disables `extractCss` in developement mode in which case, `style` tag is used to apply css. In production mode, `extractCss` plugin extracts all the `css` from `js` bundles into their own files which is named according to the value of `filename` used while declaring `extractCss`.

Now running `npm run build` will create 3 files in `dist` - `bundle.js`, `index.css` and `index.html`.

#### Update - Adding scss support

Let's add `scss` parsing support to the webpack config file. For this you'll need `sass-loader` which in turn needs `node-sass`. Install these using -

`npm install node-sass sass-loader --save-dev`

Now, update `webpack.config.js` so that webpack knows how to process imported scss files -

```diff
 const path = require('path');
 const HtmlWebpackPlugin = require('html-webpack-plugin');
 const ExtractTextPlugin = require('extract-text-webpack-plugin');
 
 const env = process.env.NODE_ENV || 'development';
 const isDev = env === 'development';
 const isProd = env === 'production';

-const extractCss = new ExtractTextPlugin({
+const extractScss = new ExtractTextPlugin({
   filename: 'index.css',
   disable: isDev
 });
 
 module.exports = {
   entry: {
     bundle: './src/index.js'
   },
   output: {
     path: path.resolve(__dirname, 'dist'),
     filename: 'bundle.js'
   },
   plugins: [
     new HtmlWebpackPlugin(),
-    extractCss
+    extractScss
   ],
   module: {
     rules: [{
       test: /\.js$/,
       exclude: /node_modules/,
       use: 'babel-loader'
     }, {
-      test: /\.css$/,
+      test: /(\.css|\.scss)$/,
       exclude: /node_modules/,
-      use: extractCss.extract({
+      use: extractScss.extract({
         use:[
-          {loader: 'css-loader'}
+          {loader: 'css-loader'},
+          {loader: 'sass-loader'}
         ],
         fallback: 'style-loader'
       })
     }]
   }
 };
 ```

Now to test this out, rename `index.css` to `index.scss` and update its content with basic scss nesting -

```scss
body {
  p {
    color: red;
  }
}
```

Update the import in `index.js` -

```diff
 import message from './message';
-import './index.css';
+import './index.scss';

 const paragraph = document.createElement('p');
 paragraph.innerHTML = message;

 document.body.prepend(paragraph);
```

Test this by running `npm run dev` and open the url in browser.

This part concludes the usage of importing `css` and `scss` files in `js`.

-----------------

## Part 3

As the codebase of a project increases in size, it can become difficult to maintain a strict coding guideline if not taken care of at early stage. Also, as more people start contributing to a single project, they may bring their own style of coding which can result in code in various files looking different and it becomes painful for new developers to decide which style to follow. This problem is taken care of by using linters. They help in following a single strict guideline for writing code. Linters in javascript show many helpful messages like unused variables, missing semicolon (this may not be a problem in some projects), codes exceeding maximum permitted length, etc. Let's update our project to use `eslint` to throw error when a particular guideline is not followed. For this, we need `eslint` and `eslint-loader`. Install them using -

`npm install eslint eslint-loader --save-dev`

Now update `webpack.config.js` to inform webpack to use `eslint-loader` before passing it through `babel-loader` -

```diff
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const ExtractTextPlugin = require('extract-text-webpack-plugin');

const env = process.env.NODE_ENV || 'development';
const isDev = env === 'development';
const isProd = env === 'production';

const extractScss = new ExtractTextPlugin({
  filename: 'index.css',
  disable: isDev
});

 module.exports = {
   entry: {
     bundle: './src/index.js'
   },
   output: {
     path: path.resolve(__dirname, 'dist'),
     filename: 'bundle.js'
   },
   plugins: [
     new HtmlWebpackPlugin(),
     extractScss
   ],
   module: {
     rules: [{
+      enforce: 'pre',
+      test: /\.js$/,
+      exclude: /node_modules/,
+      use: 'eslint-loader'
+    }, {
       test: /\.js$/,
       exclude: /node_modules/,
       use: 'babel-loader'
     }, {
       test: /(\.css|\.scss)$/,
       exclude: /node_modules/,
       use: extractScss.extract({
         use:[
           {loader: 'css-loader'},
           {loader: 'sass-loader'}
         ],
         fallback: 'style-loader'
       })
     }]
   }
 };
```

Create a new file `.eslintrc` at the top level of your project (alongside `package.json`). In this file, you can define your own custom rules and the parser for `eslint` to follow.

```json
{
  "parserOptions": {
    "ecmaVersion": 6,
    "sourceType": "module"
  },
  "extends": "eslint:recommended"
}
```

`ecmaVersion` allows eslint to recognise ES6 features, `sourceType: module` allows the usage of `import` and `export` keywords. By default, there are no rules set for `eslint`. So `"extends": "eslint:recommended"` tells `eslint` to use default recommended rules.

At this point, you can run `npm run dev`. In the console, you'll see that there are 2 same type of errors -

```txt
4:19  error  'document' is not defined  no-undef
7:1   error  'document' is not defined  no-undef
```

This tells that the variable `document` has not been defined (`no-undef`) anywhere but is still being used. This can be fixed in 2 ways. To fix this, you'll need to use the `globals` key in `.eslintrc`. Update your `.eslintrc` -

```diff
 {
   "parserOptions": {
     "ecmaVersion": 6,
     "sourceType": "module"
   },
-  "extends": "eslint:recommended"
+  "extends": "eslint:recommended",
+. "globals": {
      "document": true
    }
 }
```

This tells `eslint` that the variable `document` is global and will be provided by the JS environment (in this case, browser). Now you can run `npm run dev` without any error. You can also add a linting command to `package.json` to see lint error independently of webpack. Update `package.json` -

```diff
{
  "name": "hello-world",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "build": "NODE_ENV=production webpack",
-   "dev": "NODE_ENV=development webpack-dev-server --content-base dist --hot"
+   "dev": "NODE_ENV=development webpack-dev-server --content-base dist --hot",
+   "lint": "eslint ./src --ext .js"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "babel-core": "^6.26.0",
    "babel-loader": "^7.1.2",
    "babel-preset-env": "^1.6.1",
    "css-loader": "^0.28.9",
    "eslint": "^4.18.1",
    "eslint-loader": "^2.0.0",
    "extract-text-webpack-plugin": "^3.0.2",
    "html-webpack-plugin": "^2.30.1",
    "node-sass": "^4.7.2",
    "sass-loader": "^6.0.6",
    "style-loader": "^0.20.2",
    "webpack": "^3.11.0",
    "webpack-dev-server": "^2.11.1"
  }
}
```

Now you can run `npm run lint` in your console and check for any linting errors regardless of whether you are bundling the project or not. This can also be used in git pre-commit hooks to not allow commits if `eslint` throws any error. `eslint ./src --ext .js` tells `eslint` to check for errors in all files in `src` dirctory with `js` extension. You can also add an optional `--fix` option to this command which automatically tries to fix errors so that you don't have to.

You can also add your own rules in `.eslintrc` file as per your requirements. The `eslint:recommended` option does not allow you to use `console.log` in your code (recommended way is to use a logging module). You can add a rule to tell `eslint` to show a warning in `console.log` statements instead of an error. Update `.eslintrc` file -

```diff
 {
   "parserOptions": {
     "ecmaVersion": 6,
     "sourceType": "module"
   },
   "extends": "eslint:recommended",
   "globals": {
-    "document": true
+    "document": true,
+    "console": true
-  }
+  },
+  "rules": {
+    "no-console": 1
+  }
 }
```

`"no-console": 1` tells `eslint` to show a warning instead of an error. Other values are `0` (turn off `eslint` for this rule) and `2` (throw an error if this rule is violated). There are some standard javascript style guides that a lot of companies use (instead of the default `eslint:recommended`). One of them is airbnb's [javascript style guide](https://github.com/airbnb/javascript) which adds a lot of well accepted linting rules. You can use this instead of the current one. Let's add that to our configuration. It requires the installation of an extra `eslint-plugin-import` dependency. Install `eslint-config-airbnb-base` and it's dependencies using -

`npx install-peerdeps --dev eslint-config-airbnb-base`

Now update `.eslintrc` -

```diff
 {
-  "parserOptions": {
-   "ecmaVersion": 6,
-   "sourceType": "module"
-  },
-  "extends": "eslint:recommended",
+  "extends": "airbnb-base",
   "globals": {
     "document": true,
     "console": true
   },
   "rules": {
     "no-console": 1
   }
 }
```

`airbnb-base` has `parserOptions` internally. So it has been removed. Now, when you run `npm run dev`, you'll get an error-

```text
...hello-world/src/message.js
1:16  error  Strings must use singlequote  quotes
```

That's because `airbnb-base` has a rule to use single quotes for strings instead of double quotes. Running `npm run lint` with `--fix` option will automatically change `"` to `'` in `src/message.js`.

This concludes the usage of `eslint` to enforce code quality in your code.

> Part 4 will update this project to start developing ReactJS apps.
