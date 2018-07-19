+++ 
draft = false
date = 2018-07-19T08:38:58+02:00
title = "How to handle different domains and subdomains with your (local) node server"
slug = "domains-and-subdomains-node-server" 
tags = ["node", "express", "vhost", "server", "domains", "subdomains"]
categories = ["backend"]
+++

Developing web apps now is not what it was before, and it's a good thing. Not that W/L/X/AMP was such a pain to use (even if sometimes it was), now we install some packages via npm, `npm start` and like magic you can start developing your app.  
But with more complex apps, using subdomains for example, you may have to set-up your own local server to handle the different urls. Or maybe you don't want messy folders containing apps that all respond to `localhost:3000`, but you want different domains for each app, with one server to launch and the right url to put in your browser.

We're going to see here how we can manage that. Let's say we have this organization :

-projects/
: |--app1/
: &nbsp;&nbsp;&nbsp;|-- src
: &nbsp;&nbsp;&nbsp;|-- public
: &nbsp;&nbsp;&nbsp;|-- build
: &nbsp;&nbsp;&nbsp;|-- package.json
: |--app2/
: &nbsp;&nbsp;&nbsp;|-- src
: &nbsp;&nbsp;&nbsp;|-- public
: &nbsp;&nbsp;&nbsp;|-- dist
: &nbsp;&nbsp;&nbsp;|-- package.json
: |--app3/
: &nbsp;&nbsp;&nbsp;|-- src
: &nbsp;&nbsp;&nbsp;|-- public
: &nbsp;&nbsp;&nbsp;|-- package.json
: |--dashboard/
: &nbsp;&nbsp;&nbsp;|-- src
: &nbsp;&nbsp;&nbsp;|-- dist
: &nbsp;&nbsp;&nbsp;|-- package.json
: |--server.js

In our projects folder, we need __*express*__ and __*vhost*__ :

```
$ npm i express vhost
```

Our server.js contains: 

```javascript
// server.js

const express = require('express');
const path = require('path');
const vhost = require('vhost');
const app = express();

// Let initialize the app that we want to deploy
const app1 = express();
const app2 = express();
const app3 = express();
const appDashboard = express();

// Declare the static folder of applications
app1.use(express.static(path.join(__dirname, 'app1', 'build')));
app2.use(express.static(path.join(__dirname, 'app2', 'dist')));
app3.use(express.static(path.join(__dirname, 'app3', 'public')));
appDashboard.use(express.static(path.join(__dirname, 'dashboard', 'dist')));


// Return the right entry point for each application
app1.use((req, res, next) => {
    return res.sendFile(path.resolve(__dirname, 'app1', 'build', 'index.html'));
});
app2.use((req, res, next) => {
    return res.sendFile(path.resolve(__dirname, 'app2', 'dist', 'index.html'));
});
app3.use((req, res, next) => {
    return res.sendFile(path.resolve(__dirname, 'app3', 'public', 'index.html'));
});
appDashboard.use((req, res, next) => {
    return res.sendFile(path.resolve(__dirname, 'dashboard', 'dist', 'index.html'));
});

// Declare the vhost
app.use(vhost('app1.test', app1));
app.use(vhost('app2.test', app2));
app.use(vhost('app3.test', app3));
app.use(vhost('dashboard.app3.test', appDashboard));

app.listen(4000);
```
Ok pretty simple and short code. Note that we used the _.test_ as a TLD (Top Level Domain) since _.dev_ needs to be accessed with https now, which is a pain honestly... But _.test_ does the job here. We also have to use a TLD for __*express*__ to parse correctly the url when we use subdomains.  
Now we have to link the url of our vhost to our localhost. Let's edit our /etc/hosts file: 

```txt
# Virtual hosts
127.0.0.1   app1.test
127.0.0.1   app2.test
127.0.0.1   app3.test
127.0.0.1   dashboard.app3.test
```

And, Well that's it. In the _projects_ directory, open a terminal and launch the server `$ node server.js`.  
Now you can access your apps by typing in your browser the right url with the port 4000:
```
app1.test:4000
```

You can try it [here](https://github.com/rdhox/domains-and-subdomains-node-server.git).

