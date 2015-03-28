

# Introduction #

This page will explain, how to initialize the driver. This means :
  1. Initialize the connection with the server,
  1. Initialize the pool of connection,
  1. Initialize the safe mode,
  1. Initialize the logging mode,
  1. Uses mutiple drivers (for connecting with multiple database)

# Details #

First, you must be aware that there must be one JMCNetMongoDBDriver object par database.

## Driver instantiation ##
Initialization of the driver for one database is done by this block of code :
```
<fx:Declarations>
    <driver:JMCNetMongoDBDriver id="mongoDriver"
				hostname="[String]"
				port="[uint]"
				databaseName="[String]"
				socketPoolMin="[uint]"
				socketPoolMax="[uint]"
				socketTimeOutMs="[uint]"
				username="[String]"
				password="[String]"
				logBSON="[Boolean]"
				logDocument="[Boolean]"
				logSocketPool="[Boolean]"/>
</fx:Declarations>
```

The parameters are the following :
  * **hostname**(1) (String) : the MongoDB server's hostname,
  * **port** (uint) : the MongoDB server's port. Default value is 27017,
  * **databaseName**(1) (String) : the database's name this driver is dedicated to,
  * **socketPoolMin** (uint) : the min number of connection in the pool. Default value is 2,
  * **socketPoolMax** (uint) : the max number of connection in the pool. Default value is 5,
  * **socketTimeOutMs** (uint) : the socket timeout in milliseconds. Default value is 10 seconds,
  * **username** (String) : username used for authentication. If null, driver is in non authenticate mode. Default value is null,
  * **password** (String) : password of the user for authentication. This password must be non encrypted. Default value is null,
  * logBSON (Boolean) : if true trace BSON encoding/decoding. Default value is false,
  * logDocument (Boolean) : if true trace all Document manipulation. Default value is false.
  * logSocketPool (Boolean) : if true trace all Sockets manipulation. Default value is false.

(1) : parameter **MUST** be provided.

Examples :

Full initialization :
```
<fx:Declarations>
    <driver:JMCNetMongoDBDriver id="mongoDriver"
				hostname="myserverhostname"
				port="27080"
				databaseName="mydatabase"
				socketPoolMin="5"
				socketPoolMax="10"
				socketTimeOutMs="60000"
				username="myuser"
				password="mypassword"
				logBSON="false"
				logDocument="true"
				logSocketPool="false"/>
<fx:Declarations>
```

Mininal initialization :
```
<fx:Declarations>
    <driver:JMCNetMongoDBDriver id="mongoDriver"
				hostname="myserverhostname"
				databaseName="mydatabase"/>
<fx:Declarations>
```

## Safe mode ##
The driver can be configured to use safe mode (cf. mongoDB documentation for more information) .

There is 5 safe mode you can use (from less to more secure) :
  1. SAFE\_MODE\_NONE : writes returns immediately without any check. Don't even check for network error,
  1. SAFE\_MODE\_NORMAL :  don't call _getLastError_ by default but check for network error. This is default value,
  1. SAFE\_MODE\_SAFE : call _getLastError_ after update, insert or delete, but don't wait for slave. This call is done in the same socket,
  1. SAFE\_MODE\_MAJORITY : writes returns when the data are replicated on a majority of servers,
  1. SAFE\_MODE\_REPLICAS\_SAFE : call _getLastError_ and wait for slaves to complete operation

When safe mode is > SAFE\_MODE\_SAFE, you can provide a callback function that will be called on each update, insert or delete operation. See writes operations for more informations.

Safe mode is done by calling the _setWriteConcern_ of the driver :
```
// Set safeMode
mongoDriver.setWriteConcern(JMCNetMongoDBDriver.SAFE_MODE_SAFE);
```

_setWriteConcern_ takes some others parameters to complete the safe mode :
  * wTimeout (uint) : the time max to wait for an answer in millisecond. If 0 the wait is infinite. Default value is 10 seconds.
  * fsync (Boolean) : if true, wait for data to be written on disk. Default value is false,
  * journal (Boolean) : if true, wait for the journalization of data. Default value is false.

## Start connection to MongoDB server ##
To start a connection to MongoDB server simply call _connect()_ on the driver. This instantiate all connections in the pool and try to authenticate them if needed.

You can be aware of the connection process by adding listener to the driver.
Errors while connecting or authenticating are signaled by throwing an _ExceptionJMCNetMongoDB_ exception. The _message_ give the reason of the error.

Here is a complete example :
```
...
mongoDriver.addEventListener(JMCNetMongoDBDriver.EVT_CONNECTOK, onConnectOK);
mongoDriver.addEventListener(JMCNetMongoDBDriver.EVT_AUTH_ERROR, onAuthError);
mongoDriver.connect();
...

protected function onConnectOK(event:EventMongoDB):void {
	// connected to MongoDB server
	if (mongoDriver.isConnecte()) {
	...
	}
}

protected function onAuthError(event:EventMongoDB):void {
	// Handle Authentication error
	...
}
```

When (and only when) the message JMCNetMongoDBDriver.EVT\_CONNECTOK is thrown, you can use drivers methods like query, update, ...

The EVT\_AUTH\_ERROR indicates that username or password provided in Driver initialization is incorrect.

## Stopping connection to server ##
Stopping connection to server is done by calling _disconnect()_ method on driver.
All connection with the server are closed, and the socket pool is reinitialized.

You can be aware on complete disconnection by adding a listener to the driver.
Here is a complete example :
```
...
mongoDriver.addEventListener(JMCNetMongoDBDriver.EVT_CLOSE_CONNECTION, onCloseConnexion);
mongoDriver.disconnect();
...
protected function onCloseConnexion(event:EventMongoDB):void {
	// Now we are disconnected
}
```

## Pooling considerations ##
MongoDB and FLEX are massively asynchronous. This means that when you post a command to the database, the answer will come later in an asynchronous way.

The driver uses socket pooling to interact with the server. So, each command are thrown in a separate socket and each socket waits for an answer if there is one.

So, if you send 2 commands in sequence, you are not sure that the first is terminated before the second is thrown. This can be particularly a problem when you throw a _getLastError_ command after an insert ou update or delete.

In safe mode, the driver always send the _getLastError_ command in the **same socket** than the command itself. This ensure that those commands are sequential.

If you want to sequentially call the server :
  * you MUST use callback method and send the 2nd command when callback is called,
  * or force to driver to use only one socket.

A socket is released in the pool when the answer is received (for command with answer). So there is no risk of mixing answers of different command.

When the pool is empty (all socket are in use), the command you're trying to send is put in a Fifo Stack to wait for a free avalaible socket in pool. This adds delay in command but limits the number of server ressources used. So, be careful to size the pool considering the performances of your application (more sockets, give more performances) and the server resources used (more sockets, uses more server resources).

The pool always try to optimize the number of open sockets. Normally the number of open sockets is always equals to **socketPoolMin** value. The number of open socket can grow until **socketPoolMax** value is reach if needed by the application. Supplementary opens socket are closed automatically when not used for a while (1 minute).

Example :
```
...
mongoDriver.insertDoc("documentsCollectionName", [obj1], null, true);
// this command is send without waiting for the first one
mongoDriver.insertDoc("documentsCollectionName", [obj2], new MongoResponder(onInsertCallback), true);
...

protected function onInsertCallback(response:MongoDocumentResponse):void {
	// Called when obj2 is created 
	mongoDriver.insertDoc("documentsCollectionName", [obj3], ...);
}

// obj2 can be created before obj1
// obj3 cannot be created before obj2
```

## Multiple drivers manipulations ##
In case you deals with multiple databases, you need to instanciate one driver per database. `JMCNetMongoDBDriver` holds references to all drivers instanciated.
You can access all drivers by using the `findDriver` method.

```
/**
 * Find a driver in the list of connected driver.
 * If databaseName is null, the first driver in the list is returned (usefull for mono-driver applications)
 * @param databaseName (String) : the name of the database's driver to find or null to return the first one.
 * @return driver The JMCNetMongoDBDriver corresponding to databaseName or the first if databaseName is null
 * @throws ExceptionJMCNetMongoDB if no driver if found for databaseName provided
 */
public static function findDriver(databaseName:String=null):JMCNetMongoDBDriver;
```
Note : for mono driver applications, simply use findDriver without parameter to have access to the unique driver.

**IMPORTANT note :**
Drivers are registred when calling _connect_ for the first time. So, _findDriver_ will not find your driver before _connect_ is called.

# See also #
  * [Home](Home.md),
  * [Installation](Installation.md),
  * [Using responder](Responder.md),
  * [Collection manipulation](CollectionManipulation.md) (create, drop),

