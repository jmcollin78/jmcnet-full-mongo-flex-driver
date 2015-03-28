

# Introduction #

This page will explain, how to manipulate direct native Flex objects without needing to transform them into MongoDocument. This feature permits you to :
  1. save object directly without conversion,
  1. restore those object directly.
Those features are avalaible thanks to Flex introspection.

Special cases are explained for Array and ArrayCollection at the end of this description.

# Details #
We have seen in previous documentation (see [CRUD](CRUD.md)), that _insertDoc_, _queryDoc_ deals with generic MongoDocument.
It is also possible to call them with native Flex object, like VO (Value Object) or DTO (Data Transfert Object) for example.
In this case, all attributes will be saved regarding of it's type.
All native Flex type a supported String, Date, Array, int, Number, ...

## New in 2.0 ##
### Using generic object while inserting and querying ###
You can use directly generic object in MongoDocumentResponse returned by a query. The MongoDocumentResponse provide a `toInterpretedObject` attribute (a ArrayCollection) which contains all `documents` transformed in generic !Object.
Here is a complete example extracted from FlexUnit testing :
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

...
driver.insertDoc("testu", [doc1]);
...
driver.queryDoc("testu",
	new MongoDocumentQuery(MongoDocument.addKeyValuePair("_id", doc1._id)),
	new MongoResponder(onResponseReceived));
...
public function onResponseReceived(rep:MongoDocumentResponse, token:*):void {
	assertTrue(rep.isOk);
	assertEquals(1, rep.interpretedResponse.length);
			
	var repDoc1:Object = rep.interpretedResponse[0];

	assertEquals(doc1.title, repDoc1.title);
	assertEquals(doc1.author, repDoc1.author);
	...
}

```

### Generic objects manipulation ###
In some case, you prefer manipulate more generic Flex objects like Object and ArrayCollection.
You can achieve this using the MongoDocument.toObject() method with a null parameter.

You use it like this :
```
// The arrayCollection to deals with
private var array:ArrayCollection = new ArrayCollection([
	{ attr1:"value1", attr2:12, attr3:true},
	{ attr1:"value2", attr2:13, attr3:false}
]);

...
// We are connected, so write the doc composed of an ArrayCollection
var doc:MongoDocument = new MongoDocument().addKeyValuePair("arrayCollectionName", array);
driver.insertDoc("testu", [doc]);

...
// Query the result
var query:MongoDocumentQuery=new MongoDocumentQuery();
// Ask for all object in collections
driver.queryDoc("testu", query, new MongoResponder(onResponseReceivedArrayCollection));

...
// Analyse and decode the result
public function onResponseReceivedArrayCollection(rep:MongoDocumentResponse, token:*):void {
	// check is result is ok
	assertTrue(rep.isOk);
	assertEquals(1, rep.documents.length);
	
	var vo1:MongoDocument = rep.documents[0] as MongoDocument;
	assertNotNull(vo1.getValue("_id"));
	assertNotNull(vo1.getValue("arrayCollectionName"));

	// Convert by using toGenericObject
	var genericObject:Object = vo1.toObject();
	assertTrue(genericObject.arrayCollectionName is ArrayCollection);
	assertTrue(genericObject._id is ObjectID);
	var arrayCol1:ArrayCollection = genericObject.arrayCollectionName;
	assertNotNull(arrayCol1);
	assertEquals(2, arrayCol1.length);
			
	// Verify 1rst object returned
	assertEquals("value1", arrayCol1[0].attr1);
	assertEquals(12, arrayCol1[0].attr2);
	assertEquals(true, arrayCol1[0].attr3);
			
	// Verify 2nd object returned
	assertEquals("value2", arrayCol1[1].attr1);
	assertEquals(13, arrayCol1[1].attr2);
	assertEquals(false, arrayCol1[1].attr3);	
```
This use is more simplier that the one explain in next chapter and must be prefer if possible.

Another simple example demonstrating the toObject() method :
```
private var array:ArrayCollection = new ArrayCollection([
	{ attr1:"value1", attr2:12, attr3:true},
	{ attr1:"value2", attr2:13, attr3:false}
]);
...
var m:MongoDocument = new MongoDocument();
m.addKeyValuePair("a1",new String("string1"));
var d:Date = new Date();
m.addKeyValuePair("a2",d);
m.addKeyValuePair("a3",new Number(12));
m.addKeyValuePair("a4",new Boolean(true));
m.addKeyValuePair("a5",new MongoDocument().addKeyValuePair("k", "value").addKeyValuePair("l",15.9));
m.addKeyValuePair("a6", ["string1", "string2"]);
m.addKeyValuePair("a7", array);
			
// toGenericObject conversion and do some verifications
var obj:Object = m.toObject();
assertEquals("string1", obj.a1);
assertEquals(d, obj.a2);
assertEquals(12, obj.a3);
assertEquals(true, obj.a4);
			
assertEquals("value", obj.a5.k);
assertEquals(15.9, obj.a5.l);
			
assertEquals("string1", obj.a6[0]);
assertEquals("string2", obj.a6[1]);
			
assertEquals("value1", obj.a7[0].attr1);
assertEquals(12, obj.a7[0].attr2);
assertEquals(true, obj.a7[0].attr3);
assertEquals("value2", obj.a7[1].attr1);
assertEquals(13, obj.a7[1].attr2);
assertEquals(false, obj.a7[1].attr3);
```

## Complex objects manipulation ##
It's also possible to manipulate complex objects composed of any combination of the type described.

Complete example :
```
// Two complex class to manipulate :
public class TestVO {
	public var attrString:String;
	public var attrInt32:uint;
	public var attrNumber:Number;
	public var attrBoolean:Boolean;
	public var attrArray:Array;
}

public class TestComplexObjectIDVO extends ObjectIDable {
	public var attrInt32:uint;
	public var testvo:TestVO;
	public var arrayTestvo:Array;
	// Notice how to declare a strongly typed Array
	[ArrayElementType("TestVO")]
	public var typedArrayTestvo:Array;		
}

...

// To store the object
var testvo:TestVO = new TestVO("One String", 19, 123.456, false);
var testvo1:TestVO = new TestVO("2nd String", 20, 124.456, true);
var testvo2:TestVO = new TestVO("3rd String", 21, 125.456, false);
var testvo3:TestVO = new TestVO("4rd String", 22, 126.456, true);
var obj:TestComplexObjectIDVO = new TestComplexObjectIDVO(ObjectID.createFromString("myObject"), 16, testvo, new Array(testvo1, testvo2), new Array(testvo3));

driver.insertDoc("testu", [obj]);

...
> db.testu.find()
{
  "_id" : ObjectId("00086d794f626a6563740000"),
  "arrayTestvo" :
  [
       {
         "attrArray" : ["arrayStringValue", 14, 345.678, false ],
         "attrBoolean" : true,
         "attrInt32" : 20,
         "attrNumber" : 124.456,
         "attrString" : "2nd String"
      },
      {
         "attrArray" : [ "arrayStringValue", 14, 345.678, false ],
         "attrBoolean" : false,
         "attrInt32" : 21,
         "attrNumber" : 125.456,
         "attrString" : "3rd String" }
  ],
  "attrInt32" : 16,
  "testvo" :
  {
     "attrArray" : [ "arrayStringValue", 14, 345.678, false ], 
     "attrBoolean" : false,
     "attrInt32" : 19,
     "attrNumber" : 123.456,
     "attrString" : "One String"
  },
  "typedArrayTestvo" :
  [
     {
        "attrArray" : [ "arrayStringValue", 14, 345.678, false ],
        "attrBoolean" : true,
        "attrInt32" : 22,
        "attrNumber" : 126.456,
        "attrString" : "4rd String"
     }
  ]
}

...

// gets the object back
driver.addEventListener(JMCNetMongoDBDriver.EVT_RESPONSE_RECEIVED, onResponseReceived);
driver.queryDoc("testu", new MongoDocumentQuery());

...

public function onResponseReceived(event:EventMongoDB):void {
	// clean up -> delete object
	driver.dropCollection("testu");
	var rep:MongoDocumentResponse = event.result as MongoDocumentResponse; 
	assertEquals(1, rep.documents.length);

	// Retrieve a TestComplexObjectIDVO document
	var vo:TestComplexObjectIDVO = rep.documents[0].toObject(TestComplexObjectIDVO);

	assertEquals("086d794f626a65637400", vo._id.toString());
	assertEquals(16, vo.attrInt32);
			
	assertNotNull(vo.testvo);
	assertTrue(vo.testvo is TestVO);
	assertEquals(19, vo.testvo.attrInt32);
	assertEquals("One String", vo.testvo.attrString);
	assertEquals(123.456, vo.testvo.attrNumber);
	assertFalse(vo.testvo.attrBoolean);
			
	assertEquals(2, vo.arrayTestvo.length);
	// here we have send TestVO but we only get an Object in return because Array are not typed
	assertTrue(vo.arrayTestvo[0] is Object);
	assertEquals(20, vo.arrayTestvo[0].attrInt32);
	assertEquals("2nd String", vo.arrayTestvo[0].attrString);
	assertEquals(124.456, vo.arrayTestvo[0].attrNumber);
	assertTrue(vo.arrayTestvo[0].attrBoolean);
			
	assertTrue(vo.arrayTestvo[1] is Object);
	assertEquals(21, vo.arrayTestvo[1].attrInt32);
	assertEquals("3rd String", vo.arrayTestvo[1].attrString);
	assertEquals(125.456, vo.arrayTestvo[1].attrNumber);
	assertFalse(vo.arrayTestvo[1].attrBoolean);
			
	assertEquals(1, vo.typeArrayTestvo.length);
	// here we have a send TestVO and we get a real TestVO in return !!
	assertTrue(vo.typeArrayTestvo[0] is TestVO);
	assertEquals(22, vo.typeArrayTestvo[0].attrInt32);
	assertEquals("4rd String", vo.typeArrayTestvo[0].attrString);
	assertEquals(126.456, vo.typeArrayTestvo[0].attrNumber);
	assertTrue(vo.typeArrayTestvo[0].attrBoolean);
}

```

In the example we can notice that :
  1. Conversion to object is done by `rep.documents[0].toObject(TestComplexObjectIDVO)`. You get a `TestComplexObjectIDVO` in return,
  1. not typed Array (like _arrayTestvo_ in example) are converted into generic Object in return because the driver can't know the real object type,
  1. strongly typed Array - declared with `[ArrayElementType("TestVO")]` - are converted into strongly typed Array (no lost of type during restore).
  1. `_id` value can be provided before _insertDoc_. This is done by using the _ObjectID_ class Cf. [ObjectID](ObjectID.md).

# Array and ArrayCollection manipulation #
Manipulatin Array or ArrayCollection is a little more tricky.
Here is how you can deal with those objects :
Note : this is part of the JUnit module you can find in browsing source code.
```
// The arrayCollection to deals with
private var array:ArrayCollection = new ArrayCollection([
	{ attr1:"value1", attr2:12, attr3:true},
	{ attr1:"value2", attr2:13, attr3:false}
]);

...
// We are connected, so write the doc composed of an ArrayCollection
var doc:MongoDocument = new MongoDocument().addKeyValuePair("arrayCollectionName", array);
driver.insertDoc("testu", [doc], onTimerArrayCollection);
```

```
// In the database we can see :
> db.testu.find()
{
  "_id" : ObjectId("5033b40c9fd9b604e4b46c32"),
  "arrayCollectionName" : [
    {
       "attr1" : "attr1",
       "attr2" : 12,
       "attr3" : true },
    {
       "attr1" : "attr2",
       "attr2" : 13,
       "attr3" : false }
  ]
}
```

```
...
// Query the result
var query:MongoDocumentQuery=new MongoDocumentQuery();
// Ask for all object in collections
driver.queryDoc("testu", query, onResponseReceivedArrayCollection);
```

```
...
// Analyse and decode the result
public function onResponseReceivedArrayCollection(rep:MongoDocumentResponse):void {
	assertEquals(1, rep.documents.length);
	
	var vo1:MongoDocument = rep.documents[0] as MongoDocument;
	assertNotNull(vo1.getValue("_id"));
	assertNotNull(vo1.getValue("arrayCollectionName"));
	
```

```
...
// Direct manipulation of the returned Array named "arrayCollectionName"		
	// Try to convert the arrayCollectionName into an Array
	// var array:Array = vo1.toObject(Array); // this cannot work because returned doc is not an Array
	assertTrue(vo1.getValue("arrayCollectionName") is Array);
	var array:Array = vo1.getValue("arrayCollectionName") as Array;
	assertNotNull(array);
	assertEquals(2, array.length);
			
	// Verify 1rst object returned
	assertEquals("value1", (array[0] as MongoDocument).getValue("attr1"));
	assertEquals(12, (array[0] as MongoDocument).getValue("attr2"));
	assertEquals(true, (array[0] as MongoDocument).getValue("attr3"));
			
	// Verify 2nd object returned
	assertEquals("value2", (array[1] as MongoDocument).getValue("attr1"));
	assertEquals(13, (array[1] as MongoDocument).getValue("attr2"));
	assertEquals(false, (array[1] as MongoDocument).getValue("attr3"));
	
```

```
...
// Convert the result into ArrayCollection and verify the result		
	// Try to convert in a ArrayCollection
	var arrayCol:ArrayCollection = new ArrayCollection(array);
	assertEquals(2, arrayCol.length);
			
	// Verify 1rst object returned
	assertEquals("value1", (arrayCol[0] as MongoDocument).getValue("attr1"));
	assertEquals(12, (arrayCol[0] as MongoDocument).getValue("attr2"));
	assertEquals(true, (arrayCol[0] as MongoDocument).getValue("attr3"));
			
	// Verify 2nd object returned
	assertEquals("value2", (arrayCol[1] as MongoDocument).getValue("attr1"));
	assertEquals(13, (arrayCol[1] as MongoDocument).getValue("attr2"));
	assertEquals(false, (arrayCol[1] as MongoDocument).getValue("attr3"));
			
```

```
...
// Convert each Object in the array into a generic Object for manipulation
	// 3rd test -> convert into generic object
	log.debug("Converting into generic object");
	var obj1:Object = (arrayCol[0] as MongoDocument).toObject(Object);
	assertEquals("value1", obj1.attr1);
	assertEquals(12, obj1.attr2);
	assertEquals(true, obj1.attr3);
			
	var obj2:Object = (arrayCol[1] as MongoDocument).toObject(Object);
	assertEquals("value2", obj2.attr1);
	assertEquals(13, obj2.attr2);
	assertEquals(false, obj2.attr3);
}
```

So, the structure returned is :
Array of 1 MongoDocument containing an Array named "arrayCollectionName" containing 2 MongoDocument each with attrx attributes.

# Turns off DirectObject feature (mainly for performance reason) #
To turn off this feature you need to use a NullResponseInterpreter which does not try to interprete the response. You use it like this :
```
// Ask for all object in collections but don't interprete them
			var nullResponder:MongoResponder = new MongoResponder(onResponseReceivedTestNullResponseInterpreter);
			nullResponder.responseInterpreter = new NullResponseInterpreter();
			driver.queryDoc("testu", null, nullResponder);
...

public function onResponseReceivedTestNullResponseInterpreter(rep:MongoDocumentResponse, token:*):void {
			log.debug("Calling onResponseReceivedTestNullResponseInterpreter responseDoc="+rep+" token="+token);
			
			assertEquals(3, rep.documents.length);
			assertNull(rep.interpretedResponse);
			log.debug("EndOf onResponseReceivedTestNullResponseInterpreter");
		}

```

# See also #
  * [Home](Home.md),
  * [Using responder](Responder.md),
  * [CRUD](CRUD.md) (Create, Retrieve, Update, Delete),
  * [Query](Query.md) (Find, FindOne, getMore, killCursor),
  * [object identifier](ObjectID.md)

