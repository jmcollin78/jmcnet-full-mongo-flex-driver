

# Introduction #

MongoDB offers the possibility to reference a document/collection into another document/collection/database. This is called DBRef.
This is usefull to prevent data duplication. The driver is able to automatically (or manually) deference those DBRef objects to transparently manipulate them.
This page will explain how to deal with DBRef.

# Details #
## DBRef class interface ##
DBRef are handled with the DBRef class. The interface of this class is the following :
```
/**
 * Construct a new DBRef. if databaseName is null this mean, in the current database (use it only if you are mono databaseName).
 * @param collectionName (String) The name of the collection to search in. Equivalent to $ref parameter in DBRef
 * @param id (ObjectID or String) The id of object to search. Equivalent to $id parameter in DBRef. If String, ObjectID.createFromString is called to set the $id attribute.
 * @param databaseName (String) The name of the database to search in. Equivalent to $db parameter in DBRef. If null, suppose to search in the only open database.
 */
public function DBRef(collectionName:String, id:Object, databaseName:String=null);
		
/**
 * Retrieve the value of the DBRef and store it. If this operation is successfull, value and documentValue are set.
 * @throws ExceptionJMCNetMongoDB if no driver if found for databaseName of this DBRef
 * @see value The Object object representing the referenced doc
 * @see documentValue The MongoDocument object representing the referenced doc
 */
public function fetch():void;
		
/**
 * Convert a DBRef into a MongoDocument document containing $ref, $id and optionnaly $db attribute.
 * @return MongoDocument
 */
public function toMongoDocument():MongoDocument;
		
/**
 * A debug string representing the DBRef.
 */
override public function toString():String;

public function get collectionName():String;
public function set collectionName(value:String):void;

public function get id():ObjectID;
public function set id(value:ObjectID):void;

public function get databaseName():String;
public function set databaseName(value:String):void;

/**
 * The value when this DBRef has been fetched (else it is null).
 * @return Object A generic Object containing the value deferenced or null if fetch has not been called with success.
 */
public function get value():Object;

/**
 * The MongoDocument value when this DBRef has been fetched (else it is null).
 * @return MongoDocument A MongoDocument object containing the value deferenced or null if fetch has not been called with success.
 */
public function get documentValue():MongoDocument;

/**
 * The depth in a object, this DBRef has been found
 */
public function get depth():uint;
public function set depth(value:uint):void;
```

## Managing DBRef in MongoDocument ##
For adding a DBRef to MongoDocument, there is 3 methods in MongoDocument class :
```
/**
 * Returns a DBRef document. A DBRef can link a document from one collection to another document in another collection.
 * @see http://www.mongodb.org/display/DOCS/Updating+Data+in+Mongo for more informations
 */
public function addDBRef(key:String, collectionName:String, id:Object, databaseName:String=null):MongoDocument;
/**
 * Add a key value pair to the MongoDocument object.
 * @param key String The key used to name the value
 * @param value Object The object to store in the MongoDocument. Any type is possible for value.
 * @return this
 */
public function addKeyValuePair(key:String, value:Object):MongoDocument; // value can be a DBRef
/**
 * The static version of addKeyPairValue.
 * see addKeyPairValue for more informations.
 */
public static function addKeyValuePair(key:String, value:Object):MongoDocument;  // value can be a DBRef
```

You can also test if a MongoDocument represents a DBRef and convert it into a DBRef if needed :
```
/**
 * Check if document is a DBRef. For being a DBRef it must have one $ref and $id attribute. It could have a $db attribute and must nor have any other attribute.
 * @return true if the doc represent a DBRef
 */
public function isDBRef():Boolean;

/**
 * Convert the MongoDocument to DBRef if the document represent a DBRef. Else it returns null.
 * @see isDBRef
 * @return a DBRef object or null if the document is not a DBRef
 */
public function toDBRef():DBRef;
```

**Note :** when driver receive a response from database, it convert automatically MongoDocument document that represents DBRef (see isDBRef) into DBRef (instead of MongoDocument).

When having a MongoDocument containing DBRefs you can force fetching the DBRef by calling :
```
/**
 * Find all DBRef values and fetch them until the depth of research max in found (for circular reference inhibition).
 * The EVENT_FETCH_COMPLETE is send after completion.
 * @param maxDBRefDepth int. the max depth when fetching nested DBRef. -1 means that value is taken from driver.
 * @event EVENT_FETCH_COMPLETE is send after completion.
 * @event EVENT_FETCH_ERROR is send after fetching in error.
 * @see JMCNetMongoDBDriver.maxDBRefDepth 
 */
public function fetchDBRef(maxDBRefDepth:int=-1):void;
```

**Important note :** if a DBRef valued contains any other DBRef (an so on ...), all nested DBRef will be fetched.
> You can control the depth of this recursive fetching by setting the _maxDBRefDepth_ attribute or setting a global _maxDBRefDepth_ attribute in the driver.
> See [DBRef#Nested\_DBRef\_management](DBRef#Nested_DBRef_management.md) for more informations.

## Nested DBRef management ##
A DBRef value (after fetching) can contains other DBRef. When fetching DBRef of a MongoDocument, all nested DBRef can be fetched automatically.
To control the depth of DBRef's fetching you have two way :
  1. Set a default fetching depth in driver : call _JMCNetMongoDBDriver.maxDBRefDepth_ and pass the max depth allowed,
  1. If fetching is manual (i.e _JMCNetMongoDBDriver.maxDBRefDepth_ = 0), you can control the depth by passing it to the _MongoDocument.fetchDBRef_ method.

**WARNING : in case of circular reference (A referencing B that references A for example) the resulting document can be very important depending on _maxDBRefDepth_ value.
> You should always consider with suspicion maxDBRefDepth > 3.**

## Automatically fetching DBRefs ##
This feature is very powerfull. You can tell the driver to automatically fetch DBRefs in all document received from MongoDB.
This is done by calling this method :
```
/**
 * Sets the max depth when fetching nested DBRef. This applis to all drivers.
 * - 0 means that driver won't try to deference automatically DBRefs. This is default value.
 * - 1 means that only the first level of DBRef are fetched. 
 * @return uint
 */
public static function set maxDBRefDepth(maxDBRefDepth:uint):void;

/**
 * returns the max depth when fetching nested DBRef. This applis to all drivers.
 * - 0 means that driver won't try to deference automatically DBRefs.
 * - 1 means that only the first level of DBRef are fetched. 
 * @return uint
 */   
public static function get maxDBRefDepth():uint;
```

## Code examples (from FlexUnit project) ##
### Manually fetching document ###
```
private var doc1:Object = {
			_id : ObjectID.createFromString("doc1"),
			title : "this is my first title" ,
			author : "bob" ,
			posted : new Date(),
			pageViews : 5 ,
			tags : [ "fun" , "good" , "marvellous" ] ,
			comments : [
				{ author :"joe" , text : "this is cool" } ,
				{ author :"sam" , text : "this is bad" }
			],
			other : { foo : 10 }
		};
		
private var doc2:Object = {
			_id : ObjectID.createFromString("doc2"),
			title : "this is my second title" ,
			author : "tony" ,
			posted : new Date(),
			pageViews : 6 ,
			tags : [ "fun" , "good" , "sad" ] ,
			comments : [
				{ author :"joe" , text : "this is cool" } ,
				{ author :"sam" , text : "this is bad" }
			],
			other : { foo : 5 }
		};
		
private var doc3:Object = {
			_id : ObjectID.createFromString("doc3"),
			title : "this is my third title" ,
			author : "will smith" ,
			posted : new Date(),
			pageViews : 7 ,
			tags : [ "fun" , "sad" ] ,
			other : { foo : 2 }
		};

...
// Don't fetch automatically DBRefs
JMCNetMongoDBDriver.maxDBRefDepth = 0;
			
// Test principle : 2 docs with a dbRef
doc1.doc2Ref = new DBRef("testu2", doc2._id);
driver.dropCollection("testu2");
driver.insertDoc("testu", [doc1], new MongoResponder(onInsertFetchDoc));
driver.insertDoc("testu2", [doc2], new MongoResponder(onInsertFetchDoc));
driver.queryDoc(
	"testu",
	new MongoDocumentQuery(MongoDocument.addKeyValuePair("_id", doc1._id)),
	new MongoResponder(onResponseReceivedFetchDoc));
...
public function onResponseReceivedFetchDoc(rep:MongoDocumentResponse, token:*):void {
	assertTrue(rep.isOk);
	assertEquals(1, rep.interpretedResponse.length);
			
	var repDoc:MongoDocument = rep.documents[0];
	var repDoc1:Object = rep.interpretedResponse[0];
	var dbRef:DBRef = repDoc1.doc2Ref as DBRef;			
	assertEquals(doc1.title, repDoc1.title);
	assertNotNull(dbRef);
	// dbRef has not been fetched, so it's null
	assertNull(dbRef.value);
			
	// Fetch the doc
	Async.failOnEvent(this, dbRef, MongoDocument.EVENT_DOCUMENT_FETCH_ERROR);
	repDoc.addEventListener(MongoDocument.EVENT_DOCUMENT_FETCH_COMPLETE, onFetchDoc);
	repDoc.fetchDBRef(1);
}
		
public function onFetchOnOtherCollection(event:EventMongoDB):void {
	var doc:MongoDocument = event.target as MongoDocument;
	var docObj:Object = doc.toObject();
	assertEquals("testu2", docObj.doc2Ref.collectionName);
	assertEquals(doc2._id.toString(), docObj.doc2Ref.id);
	assertNull(docObj.doc2Ref.databaseName);
			
	assertNotNull(docObj.doc2Ref.value);
	assertEquals(doc2.title, docObj.doc2Ref.value.title);
	assertEquals(doc2.author, docObj.doc2Ref.value.author);
	assertEquals(doc2.pageViews, docObj.doc2Ref.value.pageViews);
	assertEquals(doc2.comments.length, docObj.doc2Ref.value.comments.length);
	assertEquals(doc2.other.foo, docObj.doc2Ref.value.other.foo);
}
```

### Automatically fetching document ###
```
// Fetch automatically DBRefs until depth 3
JMCNetMongoDBDriver.maxDBRefDepth = 3;

...
// Test principle : 3 docs. doc1.doc2ref=DBRef on doc2, doc2.doc3ref = DBRef on doc3, doc3.docRefs[ DBRef on doc1, and doc2], doc3.doc1ref = DBRef on doc1 (circular reference) 
doc1.doc2Ref = new DBRef("testu2", doc2._id);
doc2.doc3Ref = new DBRef("testu2", doc3._id);
doc3.docRefs = new Array(new DBRef("testu", doc1._id), new DBRef("testu2", doc2._id));
doc3.doc1Ref = new DBRef("testu", doc1._id);
			
driver.dropCollection("testu2");
driver.insertDoc("testu", [doc1], new MongoResponder(onInsertFetchDocComplex));
driver.insertDoc("testu2", [doc2, doc3], new MongoResponder(onInsertFetchDocComplex));
driver.queryDoc(
	"testu",
	new MongoDocumentQuery(MongoDocument.addKeyValuePair("_id", doc1._id)),
	new MongoResponder(onResponseReceivedFetchDocComplex));

...

public function onResponseReceivedFetchDocComplex(rep:MongoDocumentResponse, token:*):void {
	assertTrue(rep.isOk);
	assertEquals(1, rep.interpretedResponse.length);
			
	var repDoc1:Object = rep.interpretedResponse[0];
	assertEquals(doc1.title, repDoc1.title);
			
	// Level 1
	assertNotNull(repDoc1.doc2Ref);
	// dbRef has been fetched, so it's not null
	assertNotNull(repDoc1.doc2Ref.value);
	assertEquals(doc2.title, repDoc1.doc2Ref.value.title);
			
	// Level2
	var dbref3:DBRef = repDoc1.doc2Ref.value.doc3Ref;
	assertNotNull(dbref3);
	assertNotNull(dbref3.value);
	assertEquals(doc3.title, dbref3.value.title);
			
	// Level3
	assertNotNull(dbref3.value.doc1Ref);
	assertNotNull(dbref3.value.doc1Ref.value);
	assertEquals(doc1.title, dbref3.value.doc1Ref.value.title);
			
	assertTrue(dbref3.value.docRefs is ArrayCollection);
	assertEquals(2, dbref3.value.docRefs.length);
	assertNotNull(dbref3.value.docRefs[0]);
	assertNotNull(dbref3.value.docRefs[1]);
	assertEquals(doc1.title, dbref3.value.docRefs[0].value.title);
	assertEquals(doc2.title, dbref3.value.docRefs[1].value.title);
			
	// Level4 => null
	assertNull(dbref3.value.doc1Ref.value.doc2ref);
	assertNull(dbref3.value.docRefs[1].value.doc3ref);
	assertNull(dbref3.value.docRefs[0].value.doc2ref);
}
```

## Fetching errors ##
When automatic fetching is activated (cf. preceding chapter), errors are signaled by calling the error callback of the responder.
Example :
```
// Fetch automatically DBRefs until depth 3
JMCNetMongoDBDriver.maxDBRefDepth = 3;
			
// Test principle : 3 docs. doc1.doc2ref=DBRef on doc2, doc2.doc3ref = DBRef on doc3, doc3.doc1ref = DBRef on a unknown document 
doc1.doc2Ref = new DBRef("testu2", doc2._id);
doc2.doc3Ref = new DBRef("testu2", doc3._id);
doc3.doc1Ref = new DBRef("testu", ObjectID.createFromString("doesn't exists"));
			
driver.insertDoc("testu", [doc1], new MongoResponder(onInsertFetchDocError));
driver.insertDoc("testu2", [doc2, doc3], new MongoResponder(onInsertFetchDocError));
driver.queryDoc(
	"testu",
	new MongoDocumentQuery(MongoDocument.addKeyValuePair("_id", doc1._id)),
	new MongoResponder(onResponseReceived, onResponseReceivedFetchDocError));
	
...
	
public function onResponseReceivedFetchDocError(rep:MongoDocumentResponse, token:*):void {
	assertFalse(rep.isOk);
}

```

When manual fetching is done, error can be catched by adding a listener to a EVENT\_DOCUMENT\_FETCH\_ERROR.
Here is an example :
```
// Don't fetch automatically
JMCNetMongoDBDriver.maxDBRefDepth = 0;
			
// Test principle : 3 docs. doc1.doc2ref=DBRef on doc2, doc2.doc3ref = DBRef on doc3, doc3.doc1ref = DBRef on a unknown document 
doc1.doc2Ref = new DBRef("testu2", doc2._id);
doc2.doc3Ref = new DBRef("testu2", doc3._id);
doc3.doc1Ref = new DBRef("testu", ObjectID.createFromString("doesn't exists"));
			
driver.insertDoc("testu", [doc1], new MongoResponder(onInsertFetchDocErrorManually));
driver.insertDoc("testu2", [doc2, doc3], new MongoResponder(onInsertFetchDocErrorManually));
driver.queryDoc(
	"testu",
	new MongoDocumentQuery(MongoDocument.addKeyValuePair("_id", doc1._id)),
	new MongoResponder(onResponseReceivedFetchDocErrorManuallyOK, onResponseReceivedFetchDocErrorManuallyError));

...

public function onResponseReceivedFetchDocErrorManuallyOK(rep:MongoDocumentResponse, token:*):void {
	// Fetch the doc
	var repDoc:MongoDocument = rep.documents[0];
	Async.failOnEvent(this, repDoc, MongoDocument.EVENT_DOCUMENT_FETCH_COMPLETE);
	repDoc.addEventListener(MongoDocument.EVENT_DOCUMENT_FETCH_ERROR, onFetchDocError);
	repDoc.fetchDBRef(3);
}
		
// This should not be called because fetching is not automatic
public function onResponseReceivedFetchDocErrorManuallyError(rep:MongoDocumentResponse, token:*):void {
	fail("This method should not be called because manual fetching");
}
		
// This should not be called because fetching is not automatic
public function onFetchDocError(event:EventMongoDB):void {
	// Normal to be here ...
	assertTrue(true);
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
  * [MapReduce](MapReduce.md),
  * [AggregationFramework](AggregationFramework.md)

