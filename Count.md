

# Introduction #

This page will explain, how to count the number of documents that are compliant with conditions.

# Details #
To count documents you must use the _count_ method of the driver.
The _count_ method signature is the following :
```
/**
 * Count how many documents are compliant to a query.
 * @param collectionName (String) : the name of the collection to query from,
 * @param query (MongoDocument) : the conditions document,
 * @param responder (MongoResponder) : the callback called when documents are ready to read. Default value is null,
 * @param skip (uint) : number of docs to skip. Usefull for pagination. Default is 0,
 * @param limit (uint) : number of docs max to return. Usefull for pagination. Default is 0 (returns default documents number),
 * @param snapshot (Boolean) : if true makes a snapshot before counting.
 */
public function count(collectionName:String, query:MongoDocument=null, responder:MongoResponder= null, skip:uint=0, limit:uint=0, snapshot:Boolean=false):void
```

The query parameter can take any combinations of query criterias (cf. [Query](Query.md) for more details).

Here is some examples about counting :
```
// Count how many docs are satisfying to a regexp.
var query:MongoDocument = new MongoDocument("attrString", MongoDocument.regex("^string.*[12]$",true, false, false, false));
mongoDriver.count(Constantes.documentsCollectionName, query, new MongoResponder(onResponseReceived)); 

// The same counting but ignore the first and limit count to 250. Makes a snapshot to be sure that data are consistent across shards.
mongoDriver.count(Constantes.documentsCollectionName, query, new MongoResponder(onResponseReceived),1,250,true); 

private function onResponseReceived(responseDoc:MongoDocumentResponse, token:*):void {
	if (!responseDoc.isOk) return ; // error in response
	
	var numberReturned:int=responseDoc.numberReturned;
	if (numberReturned == 1) {
		// Prior to 1.4 version
		var ok:int=responseDoc.getDocument(0).getValue("ok");
		if (ok) {
			var nbDocs:int = responseDoc.getDocument(0).getValue("n");
		}
		// the same in 2.0 syntax
		if (responseDoc.interpretedResponse[0].ok) {
			var nbDocs:int = responseDoc.interpretedResponse[0].n;
		}
	}
}
```

The answer received in the callback is a _MongoDocumentResponse_ with one document (numberReturned=1). _`response.document[0].n`_ is the number of docs. Notice that _`response.document[0].ok`_ must be 1.

# See also #
  * [Home](Home.md),
  * [Using responder](Responder.md),
  * [CRUD](CRUD.md) (Create, Retrieve, Update, Delete),
  * [Query](Query.md) (Find, FindOne, getMore, killCursor),
  * [Count](Count.md),
  * [Distinct](Distinct.md),
  * [Group](Group.md),
  * [MapReduce](MapReduce.md),
  * [AggregationFramework](AggregationFramework.md)

