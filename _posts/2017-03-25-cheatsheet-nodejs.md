---
title: "Cheat Sheet: Node.js"
subtitle: My personal notes around Node.js
tags: [cheatsheet, nodejs]
lang: en
---

This is my first cheatsheet. I wanted to write a small post about Node.js as a local webserver and proxy to a Oracle APEX development instance. Now it is the starting point for all my Node.js related personal notes. This will be a living document with as less boilerplate text as possible - I should delete this paragraph ;-)


## A Basic Webserver & APEX Proxy

1. Install [Node.js](https://nodejs.org)
2. Create a new directory and place the package.json and server.js in this directory
3. Run `npm install` to install the dependencies
4. Run `npm start` or alternative `node server.js` to start the server
5. Done :-)

```js
{
    "name": "nodejs-dev-server",
    "version": "0.0.0",
    "private": true,
    "dependencies": {
        "express": "^4.15.0",
        "http-proxy-middleware": "^0.17.4"
    }
}
```

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

// APEX image directory: unzip the APEX archive here
app.use('/i', express.static(path.join(__dirname, 'apex510en', 'images')));

// markdown-it test site
app.use('/md', express.static(path.resolve(__dirname, '..', 'markdown-it-test'), {
    extensions: ['html']
}));

// APEX proxy
app.use('/apex', proxy({
    target: 'http://localhost:8080',
    changeOrigin: false
}));

http.createServer(app).listen(3000);
```


## How to upgrade package.json

1. Install command line helper: `npm install -g npm-check-updates`
2. Show any new dependencies: `ncu`
3. Upgrade package.json: `ncu -u`
4. [More Infos](https://www.npmjs.com/package/npm-check-updates)


## How to run ad hoc webserver on any directory

1. `npm install -g http-server`
2. Run: `http-server` or `http-server [path] [options]`
3. HTML files without extension: `http-server -e`
3. [More Infos](https://www.npmjs.com/package/http-server)
