

# Introduction #

This page will explain, how to use the _distinct_ driver's command.

# Details #
In order to retrieve some distinct values on a query you must use the _distinct_ method of the driver. The method signature is the following :
```
/**
 * Retrieve distinct documents compliant to a query.
 * @param collectionName (String) : the name of the collection to query from,
 * @param key (String) : the key used to distinct documents,
 * @param query (MongoDocument) : the conditions document,
 * @param responder (MongoResponder) : the callback called when documents are ready to read. Default value is null,
 */
public function distinct(collectionName:String, key:String, query:MongoDocument=null, responder:MongoResponder = null):void
```

The result of the distinct command is a _MongoDocumentResponse_ and is passed to the callback if provided.

Example :
```
// Retrieve distinct attrString key value in a collection
mongoDriver.distinct(Constantes.documentsCollectionName, "attrString", new MongoDocument(), new MongoResponder(onResponseReceived)); 

...
private function onResponseReceived(responseDoc:MongoDocumentResponse, token:*):void {
	// with 1.4 syntax
	if (responseDoc.numberReturned == 1) {
		for each (var doc:MongoDocument in responseDoc.getDocument(0).getValue("values")) {
			// do something with doc
		}
	}
	
	// With 2.0 syntax
	if (responDoc.isOk) {
		for each (var doc:Object in responseDoc.interpretedResponse) {
			// do something with doc
		}
	}	
}

```

# See also #
  * [Home](Home.md),
  * [Using responder](Responder.md),
  * [CRUD](CRUD.md) (Create, Retrieve, Update, Delete),
  * [Query](Query.md) (Find, FindOne, getMore, killCursor),
  * [Count](Count.md),
  * [Distinct](Distinct.md),
  * [Group](Group.md),
  * [MapReduce](MapReduce.md)

