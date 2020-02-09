---
layout: post
title:  Build desktop apps with Electron and Angular 2
categories: [Javascript,Electron]
excerpt: This week I was in face of a challenge. I had to build a simple desktop application that gives for the user a combinatorial analysis of a number sequence. With the timeline of one day. I ended up doing in Java, using Swing (Yeah, I know, not the best way to do it, but at the moment I just keep thinking simple is always better). Along the way, I find out an awesome platform for build desktop applications call Electron.
---

This week I was in face of a challenge. I had to build a simple desktop application that gives for the user a combinatorial analysis of a number sequence. With the timeline of one day. I ended up doing in Java, using Swing (Yeah, I know, not the best way to do it, but at the moment I just keep thinking simple is always better). Along the way, I find out an awesome platform for build desktop applications call Electron.

Electron is an open-source framework developed by GitHub. It allows the development of desktop applications using Node.js runtime and the Chromium web browser. Electron is the framework behind the text editors Atom and Visual Studio Code. This technology allows us to leverage the power of web technologies and use them to develop desktop applications.

So, after delivering the project I decide to do it again, this time with Electron e Angular 2.

Why I choose Angular 2? Quoting a legendary person “[new is always better](#https://i.pinimg.com/564x/f8/52/3e/f8523eb939dbe28e9fe4d0c67402533a.jpg)”. If you wanna know how it went, follow me, I will give you a step by step of how to initiate a project with Electron and Angular 2.

To begin, make sure you have recent versions of Node.js and npm installed. Create a new directory and create a new project with the command bellow:

```bash
npm init
```

Answer the question asked. After install the electron framework:

```bash
npm install electron-prebuilt — save-dev
```

Now create a new file called index.js and add the code below in it (this is the same code find in the quickstart seed on Electron website)

```javascript
const {app, BrowserWindow} = require('electron')
const path = require('path')
const url = require('url')
// Keep a global reference of the window object, if you don't, the window will
// be closed automatically when the JavaScript object is garbage collected.
let win
function createWindow () {
  win = new BrowserWindow({width: 800, height: 600})
  win.loadURL(url.format({
    pathname: path.join(__dirname, 'app/index.html'),
    protocol: 'file:',
    slashes: true
  }))
  win.webContents.openDevTools()
  win.on('closed', () => {
    win = null
  })
}
app.on('ready', createWindow)
app.on('window-all-closed', () => {
  if (process.platform !== 'darwin') {
    app.quit()
  }
})
app.on('activate', () => {
  if (win === null) {
    createWindow()
  }
})
```

Create a new directory called app and add the file index.html in it. Open the index.html file and add the below code in it.

```html
<html>
    <head>
        <title>First App</title>
    </head>
    <body>
          <h1>Hello World</h1>
    </body>
</html>
```
Now open up package.json file and modify the scripts to include start as shown below.

```javascript
"scripts": {
    "start": "electron ."
},
```

Then you can test and se if works.

```bash
npm start
```

You should see a simple window with Hello world on it. Ok, this is your desktop app, create using electron. Now let’s use angular 2.

Open up package.json and add the below dependencies to it.

```javascript
"devDependencies": {
    "electron-prebuilt": "^0.37.8",
    "ts-loader": "^0.7.2",
    "typescript": "^1.7.3",
    "webpack": "^1.12.9",
    "webpack-dev-server": "^1.14.0"
},
  "dependencies": {
    "angular2": "2.0.0-beta.17",
    "es6-shim": "^0.35.0",
    "reflect-metadata": "0.1.2",
    "rxjs": "5.0.0-beta.6",
    "zone.js": "^0.6.12"
}
```

After adding the dependencies run npm install again.

Now we will be using typescript for writing the app. So to compile the typescript files we need a tsconfig.json which specifies the options. Let’s create a tsconfig.json file. Add the below code into it.

```javascript
{
  "compilerOptions": {
    "target": "ES5",
    "module": "commonjs",
    "removeComments": true,
    "emitDecoratorMetadata": true,
    "experimentalDecorators": true,
    "sourceMap": true
  },
  "files": [
      "app/app.ts"
  ]
}
```

Create a new file called app.ts and add the code bellow into it.

```javascript
///<reference path="../node_modules/angular2/typings/browser.d.ts"/>
import {bootstrap} from 'angular2/platform/browser';
import {Component} from 'angular2/core';
@Component({
    selector: 'myapp',
    template: '<h1>Angular 2 app inside a desktop app</h1>'
})
export class AppComponent {}
bootstrap(AppComponent);
```

We’re using a single component with a simple text the says “Angular 2 app inside a desktop app”.

Open up a webpack.config.js file (out of the app directory) and add the below code in it.

```javascript
var path = require('path');
var webpack = require('webpack');
var CommonsChunkPlugin = webpack.optimize.CommonsChunkPlugin;
module.exports = {
  devtool: 'source-map',
  debug: true,
entry: {
    'angular2': [
      'rxjs',
      'reflect-metadata',
      'angular2/core',
      'angular2/router',
      'angular2/http'
    ],
 'app': './app/app'
  },
output: {
    path: __dirname + '/build/',
    publicPath: 'build/',
    filename: '[name].js',
    sourceMapFilename: '[name].js.map',
    chunkFilename: '[id].chunk.js'
  },
resolve: {
    extensions: ['','.ts','.js','.json', '.css', '.html']
  },
module: {
    loaders: [
      {
        test: /\.ts$/,
        loader: 'ts',
        exclude: [ /node_modules/ ]
      }
    ]
  },
plugins: [
    new CommonsChunkPlugin({ name: 'angular2', filename: 'angular2.js', minChunks: Infinity }),
    new CommonsChunkPlugin({ name: 'common',   filename: 'common.js' })
  ]
};
```

Now open up package.json file again and modify the scripts as shown below.

That’s it. Now run:

```bash
npm run build
```

You should have inside your directory a new build directory with our .js files. Now we need to reference them in the index.html.

```html
<html>
    <head>
        <title>First App</title>
    </head>
    <body>
        <myapp></myapp>
        <script src="../node_modules/angular2/bundles/angular2-polyfills.js"></script>
        <script src="../build/common.js"></script>
        <script src="../build/angular2.js"></script>
        <script src="../build/app.js"></script>
    </body>
</html>
```

See that we’re using the selector for AppComponent.

Now open up your terminal and run:

```
npm start
```

Now you could see that the template in the app.ts file is present on the screen of our app which means that the angular 2 app is built and then made to run along with the desktop app created by electron. This is how angular 2 could be used to make desktop apps along with electron framework.

If you wanna know more [this site](#http://tphangout.com/angular-2-desktop-apps-with-electron/) help me a lot.
