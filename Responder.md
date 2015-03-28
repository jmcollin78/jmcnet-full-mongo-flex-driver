

# Introduction #

This chapter will explain how to use responder with the driver's commands. Responder is a very common way to hold :
  * normal result callback,
  * error callback,
  * a token object.


# Details #
In all drivers commands taking a MongoResponder in parameter, the responder is facultative. If not provided, no callback will be called.

## Callbacks rules ##
When using a responder, you must provide a normal result callback, but error callback and the token are facultatives.
If error callback is not provided, the normal callback is called in case of error. You can test if the command was OK using the _isOk_ attribute of the _MongoDocumentResponse_ object.

Normal and error callbacks must have this signature :
```
public function myCallback(response:MongoDocumentResponse, token:*);
```

## Token rules ##
A token is a facultative object passed to the normal or error callback. It's a very convenient way to pass object throught callback.

## Complete examples ##
```
var myTokenObject:ExampleObject = new ExampleObject(...);
driver.insertDoc("collectionName", [doc], new MongoResponder(onInsertResponse, onErrorResponse, myTokenObject));
...
public function onInsertResponse(response:MongoDocumentResponse, token:*):void {
	// localToken is myTokenObject
	var localToken:ExampleObject = token as ExampleObject;
	// Here response.isOk is always true (because you provide an error callback in responder)

	// Do something usefull
}

public function onErrorResponse(response:MongoDocumentResponse, token:*):void {
	// localToken is myTokenObject
	var localToken:ExampleObject = token as ExampleObject;
	// Do error ...
}
```

The following code do the following in a more shorter way :
```
var myTokenObject:ExampleObject = new ExampleObject(...);
driver.insertDoc("collectionName", [doc], new MongoResponder(onInsertResponse, null, myTokenObject));
...
public function onInsertResponse(response:MongoDocumentResponse, token:*):void {
	// localToken is myTokenObject
	var localToken:ExampleObject = token as ExampleObject;
	// Here response.isOk is true only if command succeeds
	if (response.isOk) {
		// Do something usefull
	}
	else {
		// Do error ...
	}
}
```

## Responder and safeMode ##
Callbakcs handling and safeMode combination can be tricky.
With MongoDB some commands always have an answer, and some don't have any answer by default. The driver permits you to automatically send the _GetLastError_ command when a command has no answer.
This is done by setting the safeMode to one of this value : SAFE\_MODE\_SAFE, SAFE\_MODE\_REPLICAS\_SAFE, SAFE\_MODE\_MAJORITY.
When you set the safeMode in one of those value, all commands have an answer, and thus, if you pass a Responder, it will be called.

If safeMode is SAFE\_MODE\_NONE or SAFE\_MODE\_NORMAL, some commands will never have an answer and thus Responder's callback will never be called.

This table synthetize the command's and type of returns depending on safeMode :
| **Command** | **Type of returns in non safe mode** (SAFE\_MODE\_NONE, SAFE\_MODE\_NORMAL)| **Type of returns in safe mode** (SAFE\_MODE\_SAFE, SAFE\_MODE\_REPLICAS\_SAFE, SAFE\_MODE\_MAJORITY)|
|:------------|:---------------------------------------------------------------------------|:-----------------------------------------------------------------------------------------------------|
|insertDoc, updateDoc, deleteDoc, runCommand | none (no callback called) | getLastError returns depending on command |
| queryDoc, getMoreDoc, killCursor, runQueryCommand | MongoDocument in response.documents and Array of Object in response.interpretedResponse | idem |
| getLastError | getLastError returns depending on previous command | idem |
| createCollection, dropCollection, renameCollection | MongoDocument in response.documents and Array of Object in response.interpretedResponse | idem |
| count, distinct, group, mapReduce, aggregate  | MongoDocument in response.documents and Array of Object in response.interpretedResponse | idem |



# Quick index #
  * [Installation](Installation.md),
  * [Driver initialization](DriverInitialization.md) (connection pooling, safe mode),
  * [Using responder](Responder.md),
  * [Collection manipulation](CollectionManipulation.md) (create, drop),
  * [Running commands](SendingCommand.md),
  * [CRUD](CRUD.md) (Create, Retrieve, Update, Delete),
  * [Query](Query.md) (Find, FindOne, getMore, killCursor).

