# Getting Started

**Restify** is a powerful framework for creating simple and fast REST APIs for use with Node.js. In this series of tutorials, we will look at how to create a simple REST API to add books to a list, and then query from that list. We we also secure our API endpoints with an API key so that they are secure from unauthorized access. Because these tutorials are intended for users of the IBM i, our database of choice will the DB2 database at the heart of the i. 

This tutorial assumes that you already have **Node.js** installed on your IBM i. If that is not the case, please download Node.js onto your system by running:

```bash
yum install nodejs
```
There are two ways to follow this tutorial: You can download all the source code straight off and have working examples out of the box, or you can follow along and create files and install dependencies as we go. The choice is yours, but if you are new to Node.js it might be helpful to follow the instructions under **Building your Project from Scratch**.

### Downloading this Repository

If you just want to have working programs that you can run and manipulate as your read these tutorials, or you don't want to go through the steps of copying code or installing new packages, simply [download this repository](https://bitbucket.org/litmis/nodejs/src/d6644e5ee3b54780b2fc9ffc8e74a3a01da1e614/examples/restify/?at=master) onto your IBM i system using git. With the code downloaded, you only have to navigate to the directory and run

`npm install`

to install all of the dependencies that the project requires. You shold now be able to run the programs as dictated in the following tutorials.

### Building your Project from Scratch

To begin creating a Node.js project, the first step is to create a directory for your project (something like restify-tutorial), navigate to it, and run:

`npm init`

This walks you through the creation of a **packages.json** file, which tracks the packages you download and allows you to rebuild all of the project's dependencies.

When your package.json file has been created, you can now download and install packages into your project. To begin, we want to install the restify npm package. To achieve this, in your project directory (where the packages.json was created), run:

`npm install restify --save`

This command installs the restify package in your project directory (in the ./node-modules directory) so that it can be used by your Node.js project. The --save option tells npm to save this package under the 'dependencies' property of your packages.json file, so that your project can be rebuilt from packages.json without having to track all of the files that get created in the node-modules directory. And that's it! You are all ready to start creating a REST API on your IBM i.

### Building your Project from Scratch++

If you would like to set up all of your project's dependencies now and not worry about installing packages as we go, you can overwrite or create a **packages.json** file with the following code.  Then run `npm install` and all of the packages used throughout the tutorial will be available to you. With this setup, you can ignore all npm install commands in the following tutorials.

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
