nodermi
=======

A rmi(Remote Method Invocation) service for node

It is designed to handle complex communication patterns, a group of servers can talk to each other and pass around remote objects.

No messages, just method invocation.

##API
###Initialize
option{host, port}; callback(error, rmiService)

```coffeescript
createRmiService(option, callback)
```
###rmi service object : register object
name : name for this object, client use this name to lookup remote object
```coffeescript
createSkeleton(name, object)

```

###rmi service object : lookup object
option : {host, port, [objName]}
```coffeescript
retrieveObj(option, callback)
```

##Sample

```javascript
var nodermi = require('nodermi');

var serverConf = {
  host: 'localhost',
  port: 7000
};

var clientConf = {
  host: 'localhost',
  port: 8000
};

var serverObj = {
  print: function(str) {
    console.log("print " + str);
  },
  // return the obj in the arguments by callback
  echo: function(obj, callback) {
    console.log("get object: " + obj.name);
    callback(null, obj);
  },
  // this obj could be a remote object!
  invoke: function(obj, callback) {
    console.log("calling doSomething of another object.");
    obj.doSomething(callback);
  }
};

// a cyclic object
var serverObj2 = {
  name: "obj2"
};
serverObj2.ref = serverObj2;


nodermi.createRmiService(serverConf, function(err, server) {
  // register objects with names for clients to lookup
  server.createSkeleton('serverObj', serverObj);
  server.createSkeleton('serverObj2', serverObj2);

  // create client rmi instance after the server rmi service is created
  nodermi.createRmiService(clientConf, function(err, client) {
    // create a request to retive remote object
    var retrieveRequest = serverConf;
    retrieveRequest.objName = 'serverObj';
      
    client.retrieveObj(retrieveRequest, function(err, stub) {
      // call serverObj.print
      stub.print("something on client");
      var localObj = {
        name: 'a local object',
        doSomething: function(callback) {
          console.log("i am doing something");
          callback(null);
        }
      };
      stub.echo(localObj, function(err, returned) {
        // the remote function pass back the local object
        console.log("get from server " + (JSON.stringify(returned)));
        // localObj === returned evaluates as true
        console.log("returned is identical to localObj " + (localObj === returned));
      });
      // the local is passed over to the server, and the server will call its doSomething
      stub.invoke(localObj, function(err) {
        console.log("invoke ends");
      });
    });

    var retrieveAllRequest = serverConf;
    retrieveAllRequest.objName = null;
    // this time retrieve all objects
    client.retrieveObj(retrieveAllRequest, function(err, stub) {
       // prints out "obj2", cyclic reference is handled nicely
       console.log(stub.serverObj2.ref.name);
    });
  });
});
```
