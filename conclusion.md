---
description: 'What we''ve done, where to go'
---

# Conclusion

In this brief introduction to REST APIs and the IBM i, I've demonstrated how to use **Node.js** with **restify** on your IBM i to query its local DB2 database. We've also secured selective endpoints using JSON Web Tokens. But this only scratches the surface of what is possible with restify and the IBM i. To explore further, I recommend the following resources:

* [https://www.ibm.com/developerworks/ibmi/library/i-native-js-app-ibmi-with-nodejs/index.html](https://www.ibm.com/developerworks/ibmi/library/i-native-js-app-ibmi-with-nodejs/index.html)
* [http://restify.com/docs/home/](http://restify.com/docs/home/)
* [https://tutorials.kode-blog.com/nodejs-rest-api](https://tutorials.kode-blog.com/nodejs-rest-api)
* [https://code.tutsplus.com/tutorials/restful-api-design-with-nodejs-restify--cms-22637](https://code.tutsplus.com/tutorials/restful-api-design-with-nodejs-restify--cms-22637)

Note that many tutorials will suggest you use MongoDB, but anything they suggest there can also be done on your IBM i with DB2, which affords you the security that you have come to rely on and the ability  to access all of the same code you've been using for years. Packages like **idb-pconnector** greatly simplify querying your DB2 database, meaning that developing Node.js applications on the IBM i has never been easier!

