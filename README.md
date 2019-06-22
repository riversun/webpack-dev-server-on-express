# How to run webpack-dev-server on express.

Example for Auto reloading (live reload) using webpack-dev-middleware and webpack-hot-middleware.

It is licensed under MIT license.

# Overview

■ By including **webpack-dev-server** in **express** , you can do the following things!

- No need to manually reload the browser when you change the front-end' source code.
- No need to manually restart server when you change the server-side' source code.

■ Library
Use webpack-dev-middleware and webpack-hot-middleware on express.

# Let's get started.

## (1) Project creation

Create a project from scratch including both front end and server side.
Create a directory with a suitable name and create as an npm project.

```shell
mkdir webpack-dev-server-on-express
cd webpack-dev-server-on-express
npm init
```

## (2) Installation of required modules

Install required modules

1. Installation of webpack modules

```
npm install --save-dev webpack webpack-cli webpack-dev-server webpack-dev-middleware webpack-hot-middleware
```

2. Installation of server modules

```
npm install --save-dev express multer babel-watch
```

3. Installation of front-end modules

```
npm install --save-dev @babel/core @babel/preset-env babel-loader core-js@3
```

## (3)Make Front-End scripts

**index.html**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Addition</title>
</head>
<body>

<h3>Addition</h3>
<form action="" method="post" id="myform">
    <input type="number" name="firstValue"><br>
    <span>+</span><br>
    <input type="number" name="secondValue"><br>
</form>
<span>=</span><br>
<input type="number" id="result" readonly><br><br>

<button id="btn-clac">Calc</button>
<script src="js/app.js"></script>
</body>
</html>
```

**index.js**

```js
const btnSend = document.querySelector('#btn-clac');

btnSend.addEventListener('click', evt => {

    const xhr = new XMLHttpRequest();

    xhr.addEventListener('load', evt => {

        if (xhr.status == 200) {
            const result = JSON.parse(xhr.response);
            const resultEle = document.querySelector('#result');
            resultEle.value = result.sum;
        }
    });

    xhr.addEventListener('error', evt => {
        console.error(evt);
    });

    xhr.open('post', 'api/add', true);

    const formEle = document.querySelector('#myform');
    const formData = new FormData(formEle);

    xhr.send(formData);

});
```

**webpack.config.js**


```js
const path = require('path');

module.exports = {
    mode: 'development',
    devServer: {
        contentBase: path.join(__dirname, 'public'),
        port: 8080,
        host: `localhost`,
    },
    entry: {
        app: [
            './src_client/index.js'
        ]
    },
    output: {
        path: path.join(__dirname, 'dist'),
        publicPath: '/js/',
        filename: `[name].js`,
    },
    module: {
        rules: [
            {
                test: /\.js$/,
                exclude: /(node_modules|bower_components)/,
                use: [
                    {
                        loader: 'babel-loader',
                        options: {
                            presets: [
                                [
                                    '@babel/preset-env',
                                    {
                                        'modules': 'false',//commonjs,amd,umd,systemjs,auto
                                        'useBuiltIns': 'usage',
                                        'targets': '> 0.25%, not dead',
                                        'corejs': 3
                                    }
                                ]
                            ]
                        }
                    }
                ]
            }
        ]
    },
    resolve: {
        alias: {}
    },
    plugins: [],

};
```

## (4)Make Server-Side scripts

**server.js**

```js
const express = require('express');
const multer = require('multer');
const multipart = multer();

const webpack = require('webpack');
const webpackDevMiddleware = require('webpack-dev-middleware');
const webpackHotMiddleware = require('webpack-hot-middleware');
const config = require('../webpack.config.js');

const app = express();
const port = 8080;

const devServerEnabled = true;

if (devServerEnabled) {
    //reload=true:Enable auto reloading when changing JS files or content
    //timeout=1000:Time from disconnecting from server to reconnecting
    config.entry.app.unshift('webpack-hot-middleware/client?reload=true&timeout=1000');

    //Add HMR plugin
    config.plugins.push(new webpack.HotModuleReplacementPlugin());

    const compiler = webpack(config);

    //Enable "webpack-dev-middleware"
    app.use(webpackDevMiddleware(compiler, {
        publicPath: config.output.publicPath
    }));

    //Enable "webpack-hot-middleware"
    app.use(webpackHotMiddleware(compiler));
}

app.use(express.static('./public'));

//API
app.post('/api/add', multipart.any(), function (req, res) {

    //execute addition(tasizan)
    const firstValue = parseInt(req.body.firstValue);
    const secondValue = parseInt(req.body.secondValue);
    const sum = firstValue + secondValue;

    //return result
    res.json({sum: sum, firstValue: firstValue, secondValue: secondValue});

});

app.listen(port, () => {
    console.log('Server started on port:' + port);
});
```

<hr>

```js
const express = require('express');
const multer = require('multer');
const multipart = multer();
```

Here, we are importing the express, and multer for processing multi-part form data .
When a form object created by "new FormData()" is sent using the POST method with XMLHttpRequest, the data is encoded in **"multipart/form-data"**.
So we can use "multer" instead of "body-parser" for parsing request body on express.

<hr>

```js
const webpack = require('webpack');
const webpackDevMiddleware = require('webpack-dev-middleware');
const webpackHotMiddleware = require('webpack-hot-middleware');
const config = require('../webpack.config.js');
```

Import webpack , webpack-dev-middleware , webpack-hot-middleware to server.js.
**webpack.config.js** is read as a config object.

<hr>

```js
 //reload=true:Enable auto reloading when changing JS files or content
 //timeout=1000:Time from disconnecting from server to reconnecting
 config.entry.app.unshift('webpack-hot-middleware/client?reload=true&timeout=1000');
```

It means the same as reloading like as follows.

```js
entry: {
    app: [
    'webpack-hot-middleware/client?reload=true&timeout=1000',
    './src_client/index.js'
    ]
},
```

# (5)Starter script into package.json

Add following into package.json.

```
"start": "babel-watch ./src_server/server.js"
```

**package.json**

```JSON
{
  "name": "webpack-dev-server-on-express",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "start:client": "webpack-dev-server --config webpack.config.js",
    "start": "babel-watch ./src_server/server.js"
  },
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "webpack": "^4.34.0",
    "webpack-cli": "^3.3.4",
    "webpack-dev-server": "^3.7.1",
    "webpack-dev-middleware": "^3.7.0",
    "webpack-hot-middleware": "^2.25.0",
    "express": "^4.17.1",
    "multer": "^1.4.1",
    "babel-watch": "^7.0.0",
    "@babel/core": "^7.4.5",
    "@babel/preset-env": "^7.4.5",
    "babel-loader": "^8.0.6",
    "core-js": "^3.1.4"
  }
}
```

Start the server and try the Auto Reload development environment.

```
npm start
```

You can try the app on http://localhost:8080

# TIPS

If you got error like this.

```
WebpackOptionsValidationError: Invalid configuration object. Webpack has been initially used using a configuration object that does not match the API schema.
-Configuration should be an object.
```

This happens when module.exports of webpack.config.js doesn't returns **Object** but returns **function**.
For example, in the following webpack.config.js, the lambda function is returned so that it can branch by an argument at startup.
The above problem occurs at such times.

**webpack.config.js**(returning function style)

```js
const path = require('path');
module.exports = (env, argv) => {
    const conf = {
        mode: 'development',
        devServer: {
            contentBase: path.join(__dirname, 'public'),
            port: 8080,
            host: `localhost`,
        },
        entry: {
            app: ['./src_client/index.js']
        },
        output: {
            path: path.join(__dirname, 'dist'),
            publicPath: '/js/',
            filename: `[name].js`,
        },
        resolve: {
            alias: {}
        },
        plugins: [],
    }
    return conf;
};
```

So you can change your code from this

```
const config = require('../webpack.config.js');
```

to this.

```
const webpackConfigJs = require('../webpack.config.js');
const config = webpackConfigJs();
```
