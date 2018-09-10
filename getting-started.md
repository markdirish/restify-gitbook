# Getting Started

**Restify** is a powerful framework for creating simple and fast REST APIs for use with Node.js. In this series of tutorials, we will look at how to create a simple REST API to add books to a list, and then query from that list. We we also secure our API endpoints with an API key so that they are secure from unauthorized access. Because these tutorials are intended for users of the IBM i, our database of choice will the DB2 database at the heart of the i. 

This tutorial assumes that you already have **Node.js** installed on your IBM i. If that is not the case, please refer to the Node.js tutorial at [https://testdemogitbook.gitbook.io/node-js/](https://testdemogitbook.gitbook.io/node-js/). 

The source code for this project can be cloned or downloaded [here](https://bitbucket.org/litmis/nodejs/src/d6644e5ee3b54780b2fc9ffc8e74a3a01da1e614/examples/restify/?at=master). Once obtained simply run `npm install` within the project root to install the required node modules. Or continue following along to setup your project manually.

When you have Node.js installed, you will need to create a directory for your application to live in. Navigate to this directory, and then run: 

`npm init`

This will walk you through the creation of a **packages.json** file, which allows all of the dependencies of your application to be rebuilt if they were ever to be lost. After your packages.json file is created, run:

`npm install restify --save`

And that's it! You are all ready to start creating a REST API on your IBM i.

### Simple Setup

If you would like to set up your entire environment now and not worry about installing packages as we go, you can overwrite or create a **packages.json** file with the following code.  Then run `npm install` and all of the packages used will be available to you. With this setup, you can ignore all npm install commands in the following tutorials.

```javascript
{
  "name": "restify-tutorial",
  "version": "1.0.0",
  "description": "",
  "main": "app.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC",
  "dependencies": {
    "dotenv": "^6.0.0",
    "idb-connector": "^1.1.1",
    "idb-pconnector": "0.1.1",
    "itoolkit": "^0.1.2",
    "jsonwebtoken": "^8.3.0",
    "restify": "^7.2.0",
    "restify-clients": "^2.5.0",
    "restify-errors": "^6.0.0",
    "restify-jwt-community": "^1.0.7"
  }
}
```





