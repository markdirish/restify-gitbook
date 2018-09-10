# Tutorial 1: Simple REST API Endpoints

One of the first exercises taught to those interested in making a server is usually to create an echo server. Our echo server will use restify to capture a parameter passed through the URL, and return that parameter with a simple messages. Create a file named **echo-server.js** and copy the following code:

```javascript
var restify = require('restify');
var server = restify.createServer();

// add a user to the database
server.get('/echo/:name', async function(req, res, next) {
  res.send("Hello, " + req.params.name);
  return next();
});

server.listen(8080, function() {
  console.log('%s listening at %s', server.name, server.url);
});
```

Using restify, we create a single endpoint '/echo/:name', where :name is a parameter passed through the URI. To run the server, simply open a terminal in your project's directory and run:

`node echo-server.js`

You should see something similar to `restify listening at http://[::]:8080` as a response. Simply navigate to your server in your browser, with a URL similar to **&lt;yourserver&gt;:8080/echo/george**. You should see returned in your browser "Hello, george"**.**

Of course, APIs can be accessed by more than just a browser. Create another file called **echo-client.js** and copy the following code:

```javascript
var clients = require('restify-clients');
 
var client = clients.createJsonClient({
  url: 'http://localhost:8080',
  version: '~1.0'
});
 
client.get('/echo/mark', function (err, req, res, obj) {
  if (err) console.error(err);
  console.log('Server returned: %j', obj);
});
```

Note, we are requiring another package to run our client called 'restify-clients'. To install it in your project, simply run

`npm install restify-clients --save`

Make sure your echoserver is still running. In a new terminal, navigate to the same directory and run 

`node echo-client.js`

You should see a message in the terminal of "Server returned: "Hello, mark"". Our client worked! Notice that because our client is on the same machine as the server, we can just call localhost:8080 instead of the URL to our IBM i. Clients are a great way for your front end code to retrieve data from your back-end server, thereby programatically using your API!

