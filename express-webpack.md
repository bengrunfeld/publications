# Creating an Node Express & Webpack Application with Dev and Prod Builds

This is a very long article. I'm sorry - it takes time to explain this stuff.

I **REALLY** struggled with building an error-free Webpack and Express application boilerplate. 

Every time I would pass one hurdle, another error would get generated by the next thing in the heap that was braking. Honestly, it was a very frustrating experience. 

Considering the fact that Express and Webpack are amongst the most used technologies on the web, you'd think that there were a lot more blog posts out there about how to set them up together, but no, you'd be wrong.

There are a couple that look at one aspect or another of integration, but besides [Alejandro Napoles'](https://alejandronapoles.com/) FANTASTIC series on the issue, there's really not much else. 

If you like this article or the projects (Expack and Rexpack), please show some love by hitting the Clap/Applause button, starring it on Github, or just tweeting me a quick thank you note at [@bengrunfeld](https://twitter.com/bengrunfeld). I'm not asking this because of some marketing effort - I couldn't care less about that. The feedback means a lot to me personally, and every single star/clap makes me happy. Thank you!!

Link to code: **[Expack](https://github.com/bengrunfeld/expack)** (Express + Webpack). Similarily, the React version is called **[Rexpack](https://github.com/bengrunfeld/rexpack)** (React + Express + Webpack).

## What We Want To Make

Firstly, we want to have both a development build of our code, as well as a production build, because we want to employ certain tools in one but not the other. 

For our Development build, we want to transpile it from ES6+, lint it, run unit tests on it, run coverage reports for it, and to enable Hot Module Reloading (HMR) for an easier development experience. Re bundling, we do NOT want to minify it or uglify it, so that we can explore features and find bugs more easily. Similarly, we want the images to stay as their own files so that they are more easily identifiable - as opposed to being encoded in Base64 straight into our CSS file. 

For our Production build, we want the file sizes to be as small as possible to increase app loading speed and usage speed (especially on mobile devices, which may have limited bandwidth). We also want there to be as few files as possible to reduce the number of requests from to the server. With all that in mind, we'll want to minify and uglify our code, with comments and blank space stripped out. We will also want to encode images directly into our CSS files as Base64 to reduce the amount of files (as above).

To avoid unnecessary imports and to keep Dev and Prod concepts clearly seperate, we will have a seperate **Express server file** for Dev and Prod, and seperate **Webpack config files** for Dev and Prod application code, and for the server too.

If you're already confused, please don't worry. It will become clearer as we build out the app. Once you've finished the article, read this section again and it will make more sense.

## Our Tech Stack

These are the main technologies we want to employ:

* Express - server
* Webpack 4 - bundling
* Jest - testing
* Babel - ES6+ transpilation
* ESlint - Linting
* Webpack Dev Middleware - Bundle code in memory instead of in a file
* Webpack Hot Middleware - Enables Hot Module Reloading (HMR)
* UglifyJS - uglifies code
* mini-css-extract-plugin - minifies CSS

NOTE: The next article that deals with building a React Express Webpack application will simply add React to this tech stack, with some special configuration changes. 

## Ok, Let's Begin - Step 1: The Express Server

I'm running all of this on macOS Sierra 10.12.6, with Node v10.0.0, NPM 6.0.0, Webpack 4, Express 4.16.3. 

Let's start from scratch with a new directory. We will test incrementally as we go, so you can identify errors in a specific area as opposed to installing everything and then at the end having no clue where it is.

    mkdir express-webpack
    cd express-webpack

First create a `package.json`.

    npm init -y

Now we can install Express.

    npm install --save express

Add the following to your `package.json`.

    "scripts": {
      "start": "node ./server.js"
    },

Let's write a basic Exress server file in the project root directory, `server.js` to test that it works.

    const path = require('path')
    const express = require('express')
    
    const app = express(),
                DIST_DIR = __dirname,
                HTML_FILE = path.join(DIST_DIR, 'index.html')
    
    
    app.use(express.static(DIST_DIR))
    
    app.get('*', (req, res) => {
        res.sendFile(HTML_FILE)
    })
    
    const PORT = process.env.PORT || 8080
    app.listen(PORT, () => {
        console.log(`App listening to ${PORT}....`)
        console.log('Press Ctrl+C to quit.')
    })

And of course, a nice simple HTML file to say Hello.

    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="utf-8">
        <title>Express and Webpack App</title>
        <link rel="shortcut icon" href="#">
    </head>
    <body>
        <h1>Expack</h1>
        <p class="description">Express and Webpack Boilerplate App</p>
    </body>
    </html>

Now, to test that it works, run `npm start` and navigate to `http://localhost:8080`. Page should display the HTML correctly.

NOTE: Make sure to open the Console in Browser Dev Tools to ensure that no Javascript or other errors are being generated.

## Step 2: Install and Enable Webpack

Let's install Webpack (currently at version 4) and configure it. We need the  CLI tools to call it from the command line, and we need `webpack-node-externals` to ignore `node_modules` when creating the build for the server. If we try to build an Express server with `node_modules`, we get hit with a ton of errors. 

    npm install --save-dev webpack webpack-cli webpack-node-externals

And we'll also install Babel to transpile ES6+ to ES5 (what Browsers still need to consume).

    npm install --save-dev babel-core babel-loader babel-preset-env

We're also going to need to install `html-loader` and `html-webpack-plugin` to copy our `index.html` file to the `dist` directory. This plugin will also insert a `script` tag in the HTML file that imports the main Javascript file.

    npm install --save-dev html-loader html-webpack-plugin

Now we must create the Webpack config file - `webpack.config.js`, and it should look like this:

    const path = require('path')
    const webpack = require('webpack')
    const nodeExternals = require('webpack-node-externals')
    const HtmlWebPackPlugin = require("html-webpack-plugin")
    
    module.exports = {
      entry: {
        server: './server.js',
      },
      output: {
        path: path.join(__dirname, 'dist'),
        publicPath: '/',
        filename: '[name].js'
      },
      target: 'node',
      node: {
        // Need this when working with express, otherwise the build fails
        __dirname: false,   // if you don't put this is, __dirname
        __filename: false,  // and __filename return blank or /
      },
      externals: [nodeExternals()], // Need this to avoid error when working with Express
      module: {
        rules: [
          {
            // Transpiles ES6-8 into ES5
            test: /\.js$/,
            exclude: /node_modules/,
            use: {
              loader: "babel-loader"
            }
          },
          {
            // Loads the javacript into html template provided.
            // Entry point is set below in HtmlWebPackPlugin in Plugins 
            test: /\.html$/,
            use: [{loader: "html-loader"}]
          }
        ]
      },
      plugins: [
        new HtmlWebPackPlugin({
          template: "./index.html",
          filename: "./index.html",
          excludeChunks: [ 'server' ]
        })
      ]
    }

Note that `excludeChunks` will exclude a file called `server` which we don't want to be included into our HTML file, since that is the webserver, and not needed in the app itself.

Change `server.js` to have ES6+ `import` syntax instead of Node's `require` to test that Babel transpilation is happening correctly.

    import path from 'path'
    import express from 'express'
    
    const app = express(),
                DIST_DIR = __dirname,
                HTML_FILE = path.join(DIST_DIR, 'index.html')
    
    
    app.use(express.static(DIST_DIR))
    
    app.get('*', (req, res) => {
        res.sendFile(HTML_FILE)
    })
    
    const PORT = process.env.PORT || 8080
    app.listen(PORT, () => {
        console.log(`App listening to ${PORT}....`)
        console.log('Press Ctrl+C to quit.')
    })

Create a file called `.babelrc` in your root and fill it with this code:

    {
      'presets': ['env']
    }

And change `package.json` scripts to the following:

    "scripts": {
      "build": "rm -rf dist && webpack --mode development",
      "start": "node ./dist/server.js"
    },

That way we always start with a fresh `dist` folder, and we declaratively set devlopment mode from the command line. 

Now you can run `npm run build` and `npm start` and navigate to `localhost:8080` to test. At this point, there should be no errors (fingers crossed...). I'm building this step by step as I'm writing this article and the above configuration works for me.

## Step 3: Add CSS and Javascript Functionality to App

We have quite a bit of functionality in place already, but lets add CSS styles, Javascript, and images to our app.

To do this, we are going to need to separate our Webpack config into 2 files, which will later even become 3 files. One will only deal with bundling the server code - `webpack.server.config.js` and the other will deal with bundling the application code - `webpack.config.js`. Later, we'll separate this main config file into Dev and Prod versions, and we'll separate the server file into Dev and Prod versions too.

First, let's install the necessary dependencies:

    npm install --save-dev css-loader file-loader style-loader

Our directory structure should look like this:

    .babelrc
    .git
    .gitignore
    README.md
    dist
    node_modules
    package-lock.json
    package.json
    webpack.config.js
    webpack.server.config.js
    src
        index.js
        html
            index.html
        css
            style.css
        js
            index.js
        img
            awful-selfie.jpg
        server
            server.js

Adjust your `package.json` scripts.

    "scripts": {
      "build": "rm -rf dist && webpack --mode development --config webpack.server.config.js && webpack --mode development",
      "start": "node ./dist/server.js"
    },

Now update `./src/html/index.html`

    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="utf-8">
        <title>Express and Webpack App</title>
        <link rel="shortcut icon" href="#">
    </head>
    <body>
        <h1>Expack</h1>
        <p class="description">Express and Webpack Boilerplate App</p>
        <div class="awful-selfie"></div>
    </body>
    </html>

Now update `./src/css/style.css`

    h1, h2, h3, h4, h5, p {
      font-family: helvetica;
      color: #3e3e3e;
    }
    
    .description {
      font-size: 14px;
      color: #9e9e9e;
    }
    
    .awful-selfie{
      background: url(../img/bg.jpg);
      width: 300px;
      height: 300px;
      background-size: 100% auto;
      background-repeat: no-repeat;
    }

Now update `./src/index.js` to check that imports and styles are working, as well as some basic functionality.

    import logMessage from './js/logger'
    import './css/style.css'
    
    // Log message to console
    logMessage('Welcome to Expack!')

And of course `./src/js/logger.js`

    const logMessage = msg => console.log(msg)
    
    export default logMessage

Just move `server.js` from the root into `./src/server`. This keeps the root clean and keeps server code in a location that's appropriate.

Finally, let's take care of Webpack config. We'll start with `./webpack.server.config.js`.

    const path = require('path')
    const webpack = require('webpack')
    const nodeExternals = require('webpack-node-externals')
    
    module.exports = {
      entry: {
        server: './src/server/server.js',
      },
      output: {
        path: path.join(__dirname, 'dist'),
        publicPath: '/',
        filename: '[name].js'
      },
      target: 'node',
      node: {
        // Need this when working with express, otherwise the build fails
        __dirname: false,   // if you don't put this is, __dirname
        __filename: false,  // and __filename return blank or /
      },
      externals: [nodeExternals()], // Need this to avoid error when working with Express
      module: {
        rules: [
          {
            // Transpiles ES6-8 into ES5
            test: /\.js$/,
            exclude: /node_modules/,
            use: {
              loader: "babel-loader"
            }
          }
        ]
      }
    }

And now let's finish everything off with `./webpack.config.js`.

    const path = require("path")
    const webpack = require('webpack')
    const HtmlWebPackPlugin = require("html-webpack-plugin")
    
    module.exports = {
      entry: {
        main: './src/index.js'
      },
      output: {
        path: path.join(__dirname, 'dist'),
        publicPath: '/',
        filename: '[name].js'
      },
      target: 'web',
      devtool: '#source-map',
      module: {
        rules: [
          {
            test: /\.js$/,
            exclude: /node_modules/,
            loader: "babel-loader",
          },
          {
            // Loads the javacript into html template provided.
            // Entry point is set below in HtmlWebPackPlugin in Plugins 
            test: /\.html$/,
            use: [
              {
                loader: "html-loader",
                //options: { minimize: true }
              }
            ]
          },
          {
            test: /\.css$/,
            use: [ 'style-loader', 'css-loader' ]
          },
          {
           test: /\.(png|svg|jpg|gif)$/,
           use: ['file-loader']
          }
        ]
      },
      plugins: [
        new HtmlWebPackPlugin({
          template: "./src/html/index.html",
          filename: "./index.html",
          excludeChunks: [ 'server' ]
        })
      ]
    }

Notice how we use `target: 'web'` for the app builds. This is VERY important, and you'll get an error if you use `target: 'node'`, so be sure to double check it.

If you run `npm run build`, you should not receive any errors. If you do, hunt them down. 

## Step 4: Seperate App into Dev and Prod Builds

While the above is good general starting point and will give you most of the things you want for a basic app, we're going to want to start using different technologies for Dev than for Prod. For example, we'll want images encoded in Base64 directly into the CSS file in Prod builds, but we'll want them in regular image file format in Dev builds. Also, we'll want to enable Hot Module Reloading in Dev only, and definitely not in Prod. 

So first, let's split `webpack.config.js` into `webpack.dev.config.js` and `webpack.prod.config.js`. Also, split `./src/server/server.js` into `server-dev.js` and `server-prod.js`  They can all have the same code at moment - we'll change it soon.

So the directory structure will look like this (affected files only):

    src
        server
            server-dev.js
            server-prod.js
    webpack.dev.config.js
    webpack.prod.config.js
    webpack.server.config.js

> NOTE: The reasons I split everything out into its own file instead of finding some smart way to have everything in one file is that this is my personal style. I like to be VERY declarative with my projects and my code - some might even say that I'm being overly obvious about what I'm trying to do. This makes it easier to figure out what my initial intentions were later on when I need to return to a part of the project. 

Now let's update the `package.json` scripts accordingly:

    "scripts": {
      "buildDev": "rm -rf dist && webpack --mode development --config webpack.server.config.js && webpack --mode development --config webpack.dev.config.js",
      "buildProd": "rm -rf dist && webpack --mode production --config webpack.server.config.js && webpack --mode production --config webpack.prod.config.js",
      "start": "node ./dist/server.js"
    },

As we said, `server-dev.js` and `server-prod.js` will have the same code.

Let's change `webpack.server.config.js` to the following.

    const path = require('path')
    const webpack = require('webpack')
    const nodeExternals = require('webpack-node-externals')
    
    module.exports = (env, argv) => {
      const SERVER_PATH = (argv.mode === 'production') ?
        './src/server/server-prod.js' :
        './src/server/server-dev.js'
    
      return ({
        entry: {
          server: SERVER_PATH,
        },
        output: {
          path: path.join(__dirname, 'dist'),
          publicPath: '/',
          filename: '[name].js'
        },
        target: 'node',
        node: {
          // Need this when working with express, otherwise the build fails
          __dirname: false,   // if you don't put this is, __dirname
          __filename: false,  // and __filename return blank or /
        },
        externals: [nodeExternals()], // Need this to avoid error when working with Express
        module: {
          rules: [
            {
              // Transpiles ES6-8 into ES5
              test: /\.js$/,
              exclude: /node_modules/,
              use: {
                loader: "babel-loader"
              }
            }
          ]
        }
      })
    }

Here we have used some Webpack 4 goodness that allows the config file to export a function which takes `argv` as a param. `argv` has a property `mode` which tells you what `--mode` flag the `webpack` command was called with (development or production) from the CLI.

We use `argv.mode` to decide which server file to use - `server-dev.js` or `server-prod.js`.

At this stage, everything should work. Notice how when you run `npm run buildDev` and then `npm start`, the source code that appears in the browser (Browser Dev Tools Sources tab - `main.js`) is very different than if you run `npm run buildProd` and then `npm start`. When you use the `--mode production` flag in Webpack 4, your code gets minified and uglified. Woot!

Ok, let's apply some build-specific functionality to Dev and Prod so that separating config and server files has a point. Webpack doesn't have a CSS minifier, so we're going to install and use our own. This will also encode images into Base64 and place them directly into our CSS file, as we discussed above

    npm install --save-dev mini-css-extract-plugin uglifyjs-webpack-plugin optimize-css-assets-webpack-plugin

Here is the updated code for `webpack.prod.config.js`:

    const path = require("path")
    const HtmlWebPackPlugin = require("html-webpack-plugin")
    const MiniCssExtractPlugin = require("mini-css-extract-plugin")
    const UglifyJsPlugin = require("uglifyjs-webpack-plugin")
    const OptimizeCSSAssetsPlugin = require("optimize-css-assets-webpack-plugin");
    
    module.exports = {
      entry: {
        main: './src/index.js'
      },
      output: {
        path: path.join(__dirname, 'dist'),
        publicPath: '/',
        filename: '[name].js'
      },
      target: 'web',
      devtool: '#source-map',
      // Webpack 4 does not have a CSS minifier, although
      // Webpack 5 will likely come with one
      optimization: {
        minimizer: [
          new UglifyJsPlugin({
            cache: true,
            parallel: true,
            sourceMap: true // set to true if you want JS source maps
          }),
          new OptimizeCSSAssetsPlugin({})
        ]
      },
      module: {
        rules: [
          {
            // Transpiles ES6-8 into ES5
            test: /\.js$/,
            exclude: /node_modules/,
            use: {
              loader: "babel-loader"
            }
          },
          {
            // Loads the javacript into html template provided.
            // Entry point is set below in HtmlWebPackPlugin in Plugins 
            test: /\.html$/,
            use: [
              {
                loader: "html-loader",
                options: { minimize: true }
              }
            ]
          },
          {
            // Loads images into CSS and Javascript files
            test: /\.jpg$/,
            use: [{loader: "url-loader"}]
          },
          {
            // Loads CSS into a file when you import it via Javascript
            // Rules are set in MiniCssExtractPlugin
            test: /\.css$/,
            use: [MiniCssExtractPlugin.loader, 'css-loader']
          },
        ]
      },
      plugins: [
        new HtmlWebPackPlugin({
          template: "./src/html/index.html",
          filename: "./index.html"
        }),
        new MiniCssExtractPlugin({
          filename: "[name].css",
          chunkFilename: "[id].css"
        })
      ]
    }

`webpack.dev.config.js` stays the same as before:

    const path = require("path")
    const webpack = require('webpack')
    const HtmlWebPackPlugin = require("html-webpack-plugin")
    
    module.exports = {
      entry: {
        main: './src/index.js'
      },
      output: {
        path: path.join(__dirname, 'dist'),
        publicPath: '/',
        filename: '[name].js'
      },
      target: 'web',
      devtool: '#source-map',
      module: {
        rules: [
          {
            test: /\.js$/,
            exclude: /node_modules/,
            loader: "babel-loader",
          },
          {
            // Loads the javacript into html template provided.
            // Entry point is set below in HtmlWebPackPlugin in Plugins 
            test: /\.html$/,
            use: [
              {
                loader: "html-loader",
                //options: { minimize: true }
              }
            ]
          },
          { 
            test: /\.css$/,
            use: [ 'style-loader', 'css-loader' ]
          },
          {
           test: /\.(png|svg|jpg|gif)$/,
           use: ['file-loader']
          }
        ]
      },
      plugins: [
        new HtmlWebPackPlugin({
          template: "./src/html/index.html",
          filename: "./index.html",
          excludeChunks: [ 'server' ]
        })
      ]
    }

To check that all of the above works, use Browser Dev Tools to look at your CSS and you should see the difference - Prod should be minified with a giant-ass image in it, while Dev should have a link to the image file.

## Step 5: Add Webpack Dev Middleware

As you've noticed so far through iterative development, running `npm run buildDev` every time you make a change to a file gets REALLY annoying.... fast. Fear not, `webpack-dev-middleware` comes to the rescue.

Webpack Dev Middleware (WDM) watches your source files and runs a Webpack build anytime you hit save on a file. All changes are handled in memory, so no files are writted to the disk. Hot Module Reloading is build on top of Webpack Dev Middleware, so we need to do this step first anyway.

Let's install our dev dependencies:

    npm install --save-dev webpack-dev-middleware

Because WDM has not yet caught up with Webpack 4 as of the time of this writing, it still expects an object to be exported from `webpack.dev.config.js`, so we can't export a function that uses `argv.mode` to set the `mode`. Instead, we have to hard-code it into the config file.

Let's update `webpack.dev.config.js`:

    const path = require('path')
    const webpack = require('webpack')
    const HtmlWebPackPlugin = require('html-webpack-plugin')
    
    module.exports = {
      entry: {
        main: './src/index.js'
      },
      output: {
        path: path.join(__dirname, 'dist'),
        publicPath: '/',
        filename: '[name].js'
      },
      mode: 'development',
      target: 'web',
      devtool: '#source-map',
      module: {
        rules: [
          {
            test: /\.js$/,
            exclude: /node_modules/,
            loader: "babel-loader",
          },
          {
            // Loads the javacript into html template provided.
            // Entry point is set below in HtmlWebPackPlugin in Plugins 
            test: /\.html$/,
            use: [
              {
                loader: "html-loader",
                //options: { minimize: true }
              }
            ]
          },
          { 
            test: /\.css$/,
            use: [ 'style-loader', 'css-loader' ]
          },
          {
           test: /\.(png|svg|jpg|gif)$/,
           use: ['file-loader']
          }
        ]
      },
      plugins: [
        new HtmlWebPackPlugin({
          template: "./src/html/index.html",
          filename: "./index.html",
          excludeChunks: [ 'server' ]
        }),
        new webpack.NoEmitOnErrorsPlugin()
      ]
    }

Now we have to update `./src/server/server-dev.js` with the WDM code:
    
    import path from 'path'
    import express from 'express'
    import webpack from 'webpack'
    import webpackDevMiddleware from 'webpack-dev-middleware'
    import config from '../../webpack.dev.config.js'
    
    const app = express(),
                DIST_DIR = __dirname,
                HTML_FILE = path.join(DIST_DIR, 'index.html'),
                compiler = webpack(config)
    
    app.use(webpackDevMiddleware(compiler, {
      publicPath: config.output.publicPath
    }))
    
    app.get('*', (req, res, next) => {
      compiler.outputFileSystem.readFile(HTML_FILE, (err, result) => {
      if (err) {
        return next(err)
      }
      res.set('content-type', 'text/html')
      res.send(result)
      res.end()
      })
    })
    
    const PORT = process.env.PORT || 8080
    
    app.listen(PORT, () => {
        console.log(`App listening to ${PORT}....`)
        console.log('Press Ctrl+C to quit.')
    })

Run `npm run buildDev` and `npm start`, and that should work. Make a change to a file (e.g. `logger.js` or `index.html`) and if you refresh, you should see the change.

## Step 6: Add Hot Module Reloading

One final touch before we move on to Linting and Unit Testing is to enable Hot Module Reloading (HRM). Basically, whenever you save a change to a file, Webpack Dev Middleware creates a new build, and HMR executes the change in the browser without you having to refresh the page manually (i.e. Ctrl + R).

    npm install --save-dev webpack-hot-middleware

Now we need to update `webpack.dev.config.js`:

    const path = require('path')
    const webpack = require('webpack')
    const HtmlWebPackPlugin = require('html-webpack-plugin')

    module.exports = {
      entry: {
        main: ['webpack-hot-middleware/client?path=/__webpack_hmr&timeout=20000', './src/index.js']
      },
      output: {
        path: path.join(__dirname, 'dist'),
        publicPath: '/',
        filename: '[name].js'
      },
      mode: 'development',
      target: 'web',
      devtool: '#source-map',
      module: {
        rules: [
          {
            test: /\.js$/,
            exclude: /node_modules/,
            loader: "babel-loader",
          },
          {
            // Loads the javacript into html template provided.
            // Entry point is set below in HtmlWebPackPlugin in Plugins 
            test: /\.html$/,
            use: [
              {
                loader: "html-loader",
                //options: { minimize: true }
              }
            ]
          },
          { 
            test: /\.css$/,
            use: [ 'style-loader', 'css-loader' ]
          },
          {
           test: /\.(png|svg|jpg|gif)$/,
           use: ['file-loader']
          }
        ]
      },
      plugins: [
        new HtmlWebPackPlugin({
          template: "./src/html/index.html",
          filename: "./index.html",
          excludeChunks: [ 'server' ]
        }),
        new webpack.HotModuleReplacementPlugin(),
        new webpack.NoEmitOnErrorsPlugin()
      ]
    }

And also `./src/server/server-dev.js`:

    import path from 'path'
    import express from 'express'
    import webpack from 'webpack'
    import webpackDevMiddleware from 'webpack-dev-middleware'
    import webpackHotMiddleware from 'webpack-hot-middleware'
    import config from '../../webpack.dev.config.js'
    
    const app = express(),
                DIST_DIR = __dirname,
                HTML_FILE = path.join(DIST_DIR, 'index.html'),
                compiler = webpack(config)
    
    app.use(webpackDevMiddleware(compiler, {
      publicPath: config.output.publicPath
    }))
    
    app.use(webpackHotMiddleware(compiler))
    
    app.get('*', (req, res, next) => {
      compiler.outputFileSystem.readFile(HTML_FILE, (err, result) => {
      if (err) {
        return next(err)
      }
      res.set('content-type', 'text/html')
      res.send(result)
      res.end()
      })
    })
    
    const PORT = process.env.PORT || 8080
    
    app.listen(PORT, () => {
        console.log(`App listening to ${PORT}....`)
        console.log('Press Ctrl+C to quit.')
    })

And finally `./src/index.js`

    import logMessage from './js/logger'
    import './css/style.css'
    
    // Log message to console
    logMessage('A very warm welcome to Expack!')
    
    // Needed for Hot Module Replacement
    if(typeof(module.hot) !== 'undefined') {
      module.hot.accept() // eslint-disable-line no-undef  
    }

Now just run `npm run buildDev` and `npm start` and if you make a change to a JS, CSS, or HTML file and save it, the change will appear almost instantly in the browser without you having to refresh. 

## Step 6: Add ESLint Code Linting

    npm install --save-dev eslint babel-eslint eslint-loader

Create a file in the root called `.eslintrc.js` with the following:

    module.exports = {
      "plugins": [ "react" ],
      "extends": [
        "eslint:recommended",
        "plugin:react/recommended"
      ],
      "parser": "babel-eslint"
    };

Then update your `webpack.dev.config.js`

    const path = require('path')
    const webpack = require('webpack')
    const HtmlWebPackPlugin = require('html-webpack-plugin')
    
    module.exports = {
      entry: {
        main: ['webpack-hot-middleware/client?path=/__webpack_hmr&timeout=20000', './src/index.js']
      },
      output: {
        path: path.join(__dirname, 'dist'),
        publicPath: '/',
        filename: '[name].js'
      },
      mode: 'development',
      target: 'web',
      devtool: '#source-map',
      module: {
        rules: [
          {
            enforce: "pre",
            test: /\.js$/,
            exclude: /node_modules/,
            loader: "eslint-loader",
            options: {
              emitWarning: true,
              failOnError: false,
              failOnWarning: false
            }
          },
          {
            test: /\.js$/,
            exclude: /node_modules/,
            loader: "babel-loader",
          },
          {
            // Loads the javacript into html template provided.
            // Entry point is set below in HtmlWebPackPlugin in Plugins 
            test: /\.html$/,
            use: [
              {
                loader: "html-loader",
                //options: { minimize: true }
              }
            ]
          },
          { 
            test: /\.css$/,
            use: [ 'style-loader', 'css-loader' ]
          },
          {
           test: /\.(png|svg|jpg|gif)$/,
           use: ['file-loader']
          }
        ]
      },
      plugins: [
        new HtmlWebPackPlugin({
          template: "./src/html/index.html",
          filename: "./index.html",
          excludeChunks: [ 'server' ]
        }),
        new webpack.HotModuleReplacementPlugin(),
        new webpack.NoEmitOnErrorsPlugin()
      ]
    }

And add the following ESlint line disabler to the HMR code in `./src/index.js`:

    module.hot.accept() // eslint-disable-line no-undef

And now if you run `npm run buildDev`, you should receive 2 errors:

* `Unexpected console statement  no-console`
* `'console' is not defined      no-undef`

Console statements should not be shipped in production code, which is why the linter is pointing this out. To silence it, you can just add the above comment to disable linting on the offending line in `logger.js`.

Other ESLint plugins you may need are:

* eslint-plugin-import
* eslint-plugin-node
* eslint-plugin-promise
* eslint-plugin-react
* eslint-plugin-standard

## Step 7: Add Jest Unit Testing and Coverage

We will use Jest to unit test our code. Jest has way fewer issues than Mocha and Karma when used with Node, Express, and React, so in my opinion, it's definitely the way to go!

    npm install --save-dev jest

Now create a directory in the root called `__mocks__` and inside it two files: `fileMock.js` and `styleMock.js`. We'll use these to mock out large files and styles, since the originals of these aren't needed for unit testing. 

    src
    dist
    __mocks__
        fileMock.js
        styleMock.js

Place the following code in `fileMock.js`:

    module.exports = 'test-file-stub';

Place the following code in `styleMock.js`:

    module.exports = {};

Now update `package.json`:

    "scripts": {
      "test": "jest",
      "coverage": "jest --coverage",
      "buildDev": "rm -rf dist && webpack --mode development --config webpack.server.config.js && webpack --mode development --config webpack.dev.config.js",
      "buildProd": "rm -rf dist && webpack --mode production --config webpack.server.config.js && webpack --mode production --config webpack.prod.config.js",
      "start": "node ./dist/server.js"
    },
    "jest": {
      "moduleNameMapper": {
        "\\.(css|less)$": "<rootDir>/__mocks__/styleMock.js",
        "\\.(gif|ttf|eot|svg)$": "<rootDir>/__mocks__/fileMock.js"
      }
    },

Let's create an easily testable file. Create a new file `./src/js/adder.js` and a test file in a new directory, `./src/js/test/adder.test.js`

Here's the code for `adder.js`:

    const adder = (x, y) => x + y
    export default adder

And here's the code for `adder.test.js`:

    import adder from '../adder'
    
    describe('Adder', () => {
      test('adds two numbers', () => {
        expect(adder(5, 3)).toEqual(8)
      })
    })

Now you can run `npm test` - the test should pass, and you can also run `npm run coverage` to get a coverage report.

And there you have it, folks! An Express-Webpack application with Hot Module Reloading, Linting, and Unit Testing with Jest. Again, if you enjoyed the article or it helped you, please hit the clap button or hit star on Github!

Peace out. =)


