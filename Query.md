

# Introduction #

This page will explain, how to use send more complex query. This means :
  1. use of operators,
  1. regexp query,
  1. cursor functions :
    1. getMore function for pagination,
    1. killCursor
  1. more complex queries using "orderby" and "returnFields"

# Details #

## Complex queries using operator ##
Queries to database are send using the _queryDoc_ driver's method. [here](CRUD.md) you will found simples examples of using _queryDoc_.

For more complex query, the driver offers the possibility to use all available MongoDB's operators. Cf. http://www.mongodb.org/display/DOCS/Querying
For this, use addQueryCriteria with criterias chained.

Here are examples :
```
// Find documents where attrInt32 >= 19 and attrInt32 < 22
var query:MongoDocumentQuery = new MongoDocumentQuery();
query.addQueryCriteria("attrInt32", MongoDocument.gte(19).lt(22));
mongoDriver.queryDoc("documentsCollectionName", query, new MongoResponder(onResponseReceived)); 

// Find documents where attrArray values are all included in "arrayStringValue" and "false" values
var query:MongoDocumentQuery = new MongoDocumentQuery();
query.addQueryCriteria("attrArray", MongoDocument.all(new Array("arrayStringValue", false)));
mongoDriver.queryDoc("documentsCollectionName", query, new MongoResponder(onResponseReceived));

// Find documents where attrInt32==18 OR attr1==1
var query:MongoDocumentQuery = new MongoDocumentQuery();
query.addDocumentCriteria(
	MongoDocument.or(
		new MongoDocument("attrInt32", 18),
		new MongoDocument("attr1", 1)));
mongoDriver.queryDoc("documentsCollectionName", query, new MongoResponder(onResponseReceived));

// Find docs where (attrInt32 mod 2 != 1) AND (attrInt32 exists)
var query:MongoDocumentQuery = new MongoDocumentQuery();
query.addQueryCriteria("attrInt32", MongoDocument.not(MongoDocument.mod(2, 1)).exists(true));
mongoDriver.queryDoc("documentsCollectionName", query, new MongoResponder(onResponseReceived));
```

Each operator have two methods in the MongoDocument class. One is static and can be used like this :
```
query.addDocumentCriteria(MongoDocument.or(...));
```
The other is applied to a MongoDocument instance. You use it like this :
```
query.addDocumentCriteria(new MongoDocument("first", "criteria").or(...));
```
In this last case, the first criteria and the or criteria must be true all together (and condition).

## regexp query ##
Regexp query permits the user to select documents where an attribute match a regexp expression.
The regexp can be chained with other conditions.
regexp method have the following signature :
```
/**
 * Add a regexp condition. More explanations on options can be found here : http://www.mongodb.org/display/DOCS/Advanced+Queries#AdvancedQueries-RegularExpressions
 * @param expression (String) : the regexp expression,
 * @param caseInsensitive (Boolean) : if true the case is insensitive. Default is false,
 * @param multiline (Boolean) : if true, search is done on a multiline attribute. Default is false,
 * @param extended (Boolean) : if true use the extended mode. 
 * @param dotAll (Boolean) : if true all caracters all matched by a dot even new lines.
 */
public [static] function regex(expression:String, caseInsensitive:Boolean=false, multiline:Boolean=false,
	extended:Boolean=false, dotAll:Boolean=false):MongoDocument
```

Example :
```
// find all doc where attrString like "^string.*[12]$"
var query:MongoDocumentQuery = new MongoDocumentQuery();
query.addQueryCriteria("attrString", MongoDocument.regex("^string.*[12]$",true, false, false, false));
mongoDriver.queryDoc(Constantes.documentsCollectionName, query, new MongoResponder(onResponseReceived)); 
```
MongoDB supports PCRE (Pearl Compatible Regular Expressions). More informations on regexp can be found [there](http://perldoc.perl.org/perlre.html).

## Cursors methods ##
When finding documents with _queryDoc_, depending on how many documents match the query, MongoDB opens a cursor. You can access to the next documents by calling _getMoreDoc_ giving the cursorID given in the _queryDoc_ result.

_getMoreDoc_ signature is the following :
```
/**
 * Retrieve more documents on an open cursor. To open Cursor, you can call queryDoc and gets the cursorID in the response.
 * @param collectionName (String) : the name of the collection to query from,
 * @param cursorID (Cursor) : the cursor to fetch datas from. Cames from a preceding call to queryDoc,
 * @param responder (MongoResponder) : the callback called when documents are ready to read. Default value is null,
 * @param numberToReturn (uint) : number of docs to return. Usefull for pagination. Default is 0 (returns default documents number)
 */
public function getMoreDoc(collectionName:String, cursorID:Cursor, responder:MongoResponder=null, numberToReturn:int=0):void
```

Here is a complete example :
```
private var cursorID:Cursor=null;

// retrieve all docs in a collection
mongoDriver.queryDoc("documentsCollectionName", null, new MongoResponder(onResponseReceived, onErrorReceived, myToken), null, 0, 10 );
...

private function onResponseReceived(responseDoc:MongoDocumentResponse, myToken:*):void {
	...
	cursorID = responseDoc.cursorID;
	...
}

private function onErrorReceived(responseDoc:MongoDocumentResponse, myToken:*):void {
	...
}

...
// Gets more docs
if (cursorID.isset()) {
	// The cursor is set, there is more data to read
	mongoDriver.getMoreDoc("documentsCollectionName", cursorID, onResponseReceived, 10);
}

...
// Killing the cursor when no more necessary (see below)
mongoDriver.killCursors(new MongoDocumentKillCursors([cursorID]), onLastErrorCallback);
```

Because cursors uses server ressources we must kill them after use. If not the server will do it for you after a while (10 mins).
To kill a cursor, just call the _killCursor_ method of the driver and give the cursorID.
The signature is the following :
```
/**
 * Kills one or more existing cursors.
 * @param doc (MongoDocumentKillCursors) : the document containing the cursor(s) to kill,
 * @param safeCallback (Function) : the callback called when operation is finished (depending on safe mode)
 */
public function killCursors(doc:MongoDocumentKillCursors, safeCallback:Function = null):void
```

## Complex queries using orderby and returnFields ##
You can order the result using any of the field in the document using the DocumentQuery.addOrderByCriteria. An orderByCriteria is an key/value pair containing the attribute used to order and a boolean representing the ascending (true) or descending (false) order.
Here is an example :
```
// query all docs with attrString="a value to filter" ordered by attrInt32 field in ascending order
var query:MongoDocumentQuery=new MongoDocumentQuery();
query.addQueryCriteria("attrString", "a value to filter");
query.addOrderByCriteria("attrInt32", true);
driver.queryDoc("collection", query);
```

In the previous example, all fields of the document are returned. If you need only a subset all the document's fields, you must use the "returnFields" attribute of the queryDoc method. This is a MongoDocument field containing key/value pairs for each field you want to return. The value 1 means you want the field, the value 0 means, you don't want the field to be returned.
Examples (combined with OrderBy) :
```

// Query all doc order by attrInt32 attribute (ascending) and returns only the attrString field
var query:MongoDocumentQuery=new MongoDocumentQuery();
query.addOrderByCriteria("attrInt32", true);
// Ask for "attrString" attribute only
driver.queryDoc("collection", query, new MongoResponder(onResponseReceivedCallback), new MongoDocument("attrString", 1));

// Query all doc order by attrInt32 attribute (descending) and returns all fields but attrString
var query:MongoDocumentQuery=new MongoDocumentQuery();
query.addOrderByCriteria("attrInt32", true);
// Ask for "attrString" attribute only
driver.queryDoc("collection", query, new MongoResponder(onResponseReceivedCallback), new MongoDocument("attrString", 0));
```

## Complete example ##
Suppose you want to program a query that does (in SQL) : select attrString from Collection where attrBoolean='true' order by attrInt32 desc :
```
var query:MongoDocumentQuery=new MongoDocumentQuery();
query.addQueryCriteria("attrBoolean", true);
query.addOrderByCriteria("attrInt32", false);
driver.queryDoc("collection", query, new MongoResponder(onResponseReceivedCallback), new MongoDocument("attrString", 1));
```

# See also #
  * [Home](Home.md),
  * [Installation](Installation.md),
  * [Driver initialization](DriverInitialization.md) (connection pooling, safe mode),
  * [Using responder](Responder.md),
  * [Collection manipulation](CollectionManipulation.md) (create, drop),
  * [Running commands](SendingCommand.md),
  * [CRUD](CRUD.md) (Create, Retrieve, Update, Delete),
  * [Query](Query.md) (Find, FindOne, getMore, killCursor),
  * [Count](Count.md),
  * [Distinct](Distinct.md),
  * [Group](Group.md),
  * [MapReduce](MapReduce.md),
  * [AggreagationFramework](AggreagationFramework.md)

