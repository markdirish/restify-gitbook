# Tutorial 2: REST APIs and the IBM i

## Setup

Of course, what good is creating a RESTful API if you are unable to interact with some sort of data store? Below is an example of a RESTful API created using the **restify** package that can add books to a database, and then can also query that database to see if a user exists. For our database, we are going to use the internal DB2 database on your IBM i machine. In order to easily interface with the database, we will use the **idb-pconnector** package. We are also going to be using the **restify-clients** and **restify-errors** packages. These can be installed by going to the root directory of your Node.js server \(where the **package.json** file resides\) and running these three commands:

```bash
npm install --save idb-pconnector  
npm install --save restify-clients  
npm install --save restify-errors
```

With this package installed, create a new directory called tutorial2 and navigate to it:

```bash
mkdir tutorial2  
cd tutorial2
```

In the **tutorial2** directory, we are going to create two files: **tutorial2-server.js** and **tutorial2-client.js**. The code for these files is given below. 

There are two changes you need to make for your server to run correctly:

1. On line 12 of tutorial2-server.js, change the SCHEMA constant to be the name of your DB2 SCHEMA.
2. On line 4 of tutorial2-client.js, change the beginning of the URL to be the URL to your IBM i machine.

```javascript
let restify = require('restify');
let errors = require('restify-errors');
let dba = require('idb-pconnector');

// setting up the RESTify server
let server = restify.createServer();
server.use(restify.plugins.bodyParser());

// DATABASE
// setting up our database variables
let dbaConnection = new dba.Connection().connect();
const SCHEMA = _YOUR_SCHEMA_;

async function setupDatabaseTables() {
  // setting up the database tables
  try {
    var dbStatement = dbaConnection.getStatement();
    await dbStatement.exec("CREATE OR REPLACE TABLE " + SCHEMA + ".BOOKS(ID INTEGER NOT NULL GENERATED ALWAYS AS IDENTITY (START WITH 1 INCREMENT BY 1), TITLE VARCHAR(32) NOT NULL, AUTHOR VARCHAR(32), PRIMARY KEY (ID));");
  } catch (dbError) {
    console.error("Error is " + dbError);
  } finally {
    dbStatement.close();
  }
}

setupDatabaseTables();

// API ROUTES
// Books
server.get('/books/:id', async function(req, res, next) {

  let id = req.params.id;

  let dbStatement; 
  try {
    dbStatement = dbaConnection.getStatement();
    var result = await dbStatement.exec("SELECT * FROM " + SCHEMA + ".BOOKS WHERE ID = " + req.params.id +";");
    if (result.length > 0) {
      let responsePayload = { id: id, title: result[0]["TITLE"], author: result[0]["AUTHOR"] };
      res.send(responsePayload);
    } else {
      return next(new errors.NotFoundError("Could not find book with id " + id + " in the database."));
    }
  } catch (dbError) {
    console.error("Error is " + dbError);
    return next(new errors.InternalServerError("Internal Server Error"));
  } finally {
    dbStatement.close();
  }
  
  return next();
});

server.post('/books/', async function(req, res, next) {

  // the body of the request should be in the format { title: 'title', author: 'author' }
  let payload = req.body;
  if (!payload.hasOwnProperty('title') || !payload.hasOwnProperty('author')) {
    return next(new errors.BadRequestError("Your request wasn't formatted correctly. Pass a title and an author"));
  }

  let dbStatement; 
  try {
    dbStatement =  dbaConnection.getStatement();

    let payload = req.body;

    // insert the book into the database
    await dbStatement.prepare("SELECT ID FROM FINAL TABLE(INSERT INTO " + SCHEMA + ".BOOKS(TITLE, AUTHOR) VALUES(?, ?));");
    await dbStatement.bind([ [payload.title, dba.SQL_PARAM_INPUT, dba.SQL_BIND_CHAR], [payload.author, dba.SQL_PARAM_INPUT, dba.SQL_BIND_CHAR]]);
    await dbStatement.execute();
    let id = await dbStatement.fetch();
    await dbStatement.commit();

    // get the id from the new book, return it to the caller
    res.send(id.ID);

  } catch (dbError) {
      console.error("Error is " + dbError);
      return next(new errors.InternalServerError("Internal Server Error"));
  } finally {
      dbStatement.close();
  }

  next();
});

// Set up the server
server.listen(3030, function() {
  console.log('%s listening at %s', server.name, server.url);
});
```

```javascript
var clients = require('restify-clients');

var client = clients.createJsonClient({
    url: '_SERVER_LOCATION_:3030/',
    version: '~1.0'
});

// post a new book, then get that book from the database to confirm it was inserted
client.post('/books/', { title: 'Foundation', author: "Isaac Asimov" }, function (err, req, res, obj) {

    console.log('Server returned: %j', obj);
    console.log("Use the returned ID to call our get route:");

    client.get('/books/' + obj, function (err, req, res, obj) {
        console.log('Server returned: %j', obj);
    });
});
```

## Running the Tutorial

To run the tutorial, open up a tutorial in your tutorial2 directory and run

node tutorial2-server.js

Then open another terminal in the same directory and run

node tutorial2-client.js

On the client terminal, you should see output similar to:

`Server returned: "381"  
Use the returned ID to call our get route:  
Server returned: {"id":"381","title":"Foundation","author":"Isaac Asimov"}`

The first line is the response from the server after we POSTed our book. It tells us the ID of the book, which we can then use to call our GET /books/ route. Doing so returns our book, showing that it was correctly added.

## Explanation

Ok, deep breath, I know it looks like a lot. But it really isn't that bad. What we have created is a server with two endpoints: POST\(/books/\) and GET\(/books/:id\). 

* **POST \(/books/\)**: Takes an object in the format `{ title: 'Foundation', author: 'Isaac Asimov' }` and writes it to our DB2 database. It then returns the ID of the book it just added, which is generated by the database itself. This allows the client to call GET on that id to retrieve the entry.
* **GET \(/books/:id\)**: Sends an ID and gets an object in the format `{"id":"381","title":"Foundation","author":"Isaac Asimov"}`.

And that's it! The rest is setting up our DB2 database to have a table with the correct format, managing the data, and handling errors! Lets take a look at some of the important sections of our server for understanding how we are using DB2 with our restify API.

### DB2 in tutorial2-server.js

One of the first things we do on line 3 is require our i**db-pconnector** library and assign it to a variable, which here I call 'dba' to mean database adapter. From there, on line 11 and 12 we connect to our database \(that defaults to the '\*LOCAL' database with our IBM i credentials for the login username and password\) and define our SCHEMA so we don't have to declare it multiple places. Finally, we define the function setupDatabaseTables\(\) on line 14, and call the function on line 26. 

The setupDatabaseTables\(\) function simply creates a table in our SCHEMA called BOOKS, which has three fields: An ID that is auto-generated and auto-incremented, a book title, and an author name. With that our database is all set up and ready to have data added and queried when our endpoints are hit.

### Endpoints

```javascript
server.post('/books/', async function(req, res, next) {
​
  // the body of the request should be in the format { title: 'title', author: 'author' }
  let payload = req.body;
  if (!payload.hasOwnProperty('title') || !payload.hasOwnProperty('author')) {
    return next(new errors.BadRequestError("Your request wasn't formatted correctly. Pass a title and an author"));
  }
​
  let dbStatement; 
  try {
    dbStatement =  dbaConnection.getStatement();
​
    let payload = req.body;
​
    // insert the book into the database
    await dbStatement.prepare("SELECT ID FROM FINAL TABLE(INSERT INTO " + SCHEMA + ".BOOKS(TITLE, AUTHOR) VALUES(?, ?));");
    await dbStatement.bind([ [payload.title, dba.SQL_PARAM_INPUT, dba.SQL_BIND_CHAR], [payload.author, dba.SQL_PARAM_INPUT, dba.SQL_BIND_CHAR]]);
    await dbStatement.execute();
    let id = await dbStatement.fetch();
    await dbStatement.commit();
​
    // get the id from the new book, return it to the caller
    res.send(id.ID);
​
  } catch (dbError) {
      console.error("Error is " + dbError);
      return next(new errors.InternalServerError("Internal Server Error"));
  } finally {
      dbStatement.close();
  }
​
  next();
});
```

Our first endpoint is server.post\('/books/', ...\) which looks at the body of the request object to determine if it is formatted correctly, with a title and an author. If not, we return a BadRequestError with a message for the client, letting them know the issue. If the format is correct, we use our dbConnection to create a dbStatement, which we then use to create a prepared statement, bind our data to it, and execute it. Our statement syntax is a little difficult, so lets cover it.

`"SELECT ID FROM FINAL TABLE(INSERT INTO " + SCHEMA + ".BOOKS(TITLE, AUTHOR) VALUES(?, ?));"`

Our statement is designed to select the ID from the record generated with our nested insert statement. The insert statement inserts into our BOOKS table values for the columns TITLE and AUTHOR, but not the ID field, since it is auto-generated to avoid conflicts. With the ID generated from our statement, we use the **res** object to send the id back to the calling client. 

```javascript
server.get('/books/:id', async function(req, res, next) {
​
  let id = req.params.id;
​
  let dbStatement; 
  try {
    dbStatement = dbaConnection.getStatement();
    var result = await dbStatement.exec("SELECT * FROM " + SCHEMA + ".BOOKS WHERE ID = " + req.params.id +";");
    if (result.length > 0) {
      let responsePayload = { id: id, title: result[0]["TITLE"], author: result[0]["AUTHOR"] };
      res.send(responsePayload);
    } else {
      return next(new errors.NotFoundError("Could not find book with id " + id + " in the database."));
    }
  } catch (dbError) {
    console.error("Error is " + dbError);
    return next(new errors.InternalServerError("Internal Server Error"));
  } finally {
    dbStatement.close();
  }
  
  return next();
});
```

Our other endpoint is server.get\('/books/:id', ...\) where we check to see if the id provided in the parameter points to a book in our database. If so, we return all of the information we have on that book in a JavaScript object. If the book doesn't exist, we return a NotFoundError to let the client know what the issue was.

And that's it! With these two endpoints, we can add and retrieve books from our DB2 database. We call these endpoints with our client, which uses the restify-clients package. First we call the POST method on /books/ to add Foundation by Isaac Asimov. When the result from that action returns, we call our GET method to ensure that the data was added correctly. This POST/GET pattern is a common one for REST APIs, and is a good way to check that the object you sent to the server was added correctly.

This tutorial showed you how to access your DB2 database with an API you created using restify. In the next tutorial, we will look at how to secure our endpoints with an authentication scheme, ensuring that not just anyone can add books to our list.
