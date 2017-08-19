---
title: "Cheat Sheet: Node.js"
subtitle: My personal notes around Node.js
tags: [cheatsheet, nodejs]
lang: en
last_modified_at: 2017-08-12
---

This is my first cheatsheet. I wanted to write a small post about Node.js as a local webserver and proxy to a Oracle APEX development instance. Now it is the starting point for all my Node.js related personal notes. This will be a living document with as less boilerplate text as possible - I should delete this paragraph ;-)


## A Basic Webserver & APEX Proxy

- Install [Node.js](https://nodejs.org)
- Create a new directory and place the package.json and server.js in this directory
- Run `npm install` to install the dependencies
- Run `npm start` or alternative `node server.js` to start the server
- Done :-)

File package.json:

```js
{
    "name": "nodejs-dev-server",
    "version": "0.0.0-ignored",
    "private": true,
    "dependencies": {
        "express": "^4.15.0",
        "http-proxy-middleware": "^0.17.4"
    }
}
```

File server.js

```js
var path = require('path');
var http = require('http');
var express = require('express');
var proxy = require('http-proxy-middleware');
var app = express();

// local dev instance of ogobrecht.github.io parallel to this server directory
app.use('/', express.static(path.resolve(__dirname, '..', 'ogobrecht.github.io', '_site'), {
    extensions: ['html'] // nicer URL's: html files need no extension
}));

// markdown-it test site
app.use('/md', express.static(path.resolve(__dirname, '..', 'markdown-it-test'), {
    extensions: ['html']
}));

// APEX image directory: unzip the APEX archive here
app.use('/i', express.static(path.join(__dirname, 'apex510en', 'images')));

// APEX proxy
app.use('/apex', proxy({
    target: 'http://localhost:8080',
    changeOrigin: false
}));

http.createServer(app).listen(3000);
```


## Install packages

- A global package: `npm install -g grunt-cli` [more about grunt-cli...](https://www.npmjs.com/package/grunt-cli)
- Save dependency in package.json: `npm install -S express`
- Save dev dependency: `npm install -D grunt-contrib-uglify`


## Upgrade package.json

- Install command line helper: `npm install -g npm-check-updates`
- Show any new dependencies: `ncu`
- Upgrade package.json: `ncu -u`
- [More infos...](https://www.npmjs.com/package/npm-check-updates)


## Run ad hoc webserver on any directory

- `npm install -g http-server`
- Run: `http-server` or `http-server [path] [options]`
- HTML files without extension: `http-server -e`
- [More infos...](https://www.npmjs.com/package/http-server)
