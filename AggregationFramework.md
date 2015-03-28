

# Introduction #

This page will explain, how to use the new aggregation framework. More explanations can be found @ MongoDB documentation [here](http://docs.mongodb.org/manual/reference/aggregation/)
**Note :** all operators and expressions are supported by the driver. Some expressions are available in the _MongoDocument_ class.

# Details #
To use the aggregation you must use the _aggregate_ method of the driver. The _aggregate_ method of the driver has the following signature :
```
/**
 * Do aggregation on documents. More on aggregation framework can be found on the MongoDB Documentation.
 * @param collectionName (String) : the name of the collection to query from,
 * @param pipeline (MongoAggregationPipeline) : the pipeline of aggregation command. See MongoAggregationPipeline
 * @param callback (Function) : the callback called when documents are ready to read. Default value is null,
 * 
 */
public function aggregate(collectionName:String, pipeline:MongoAggregationPipeline, responder:MongoResponder=null):void;
```

The result of a _aggregate_ command is a _MongoDocumentResponse_ in which there is directly interpretedResponse or an array of documents.

## Pipeline ##
The _MongoAggregationPipeline_ is a class which helps to build pipeline commands. Each method has its static version for convenience use.
It contains the following methods :
```
/**
 * Adds a $project operator in the aggregation. A $project operator is used to reshape the documents.
 * @param MongoDocument doc the document containing the $project value. Cf. http://docs.mongodb.org/manual/reference/aggregation/#_S_project
 * @return MongoDocumentAggregationPipeline this to able to chain the operator
 */
public function addProjectOperator(doc:MongoDocument):MongoAggregationPipeline;
public static function addProjectOperator(doc:MongoDocument):MongoAggregationPipeline;
		
/**
 * Adds a $match operator in the aggregation. A $match operator is used to filter the document in the pipeline.
 * @param MongoDocument doc the document containing the $match value. Cf. http://docs.mongodb.org/manual/reference/aggregation/#_S_match
 * @return MongoDocumentAggregationPipeline this to able to chain the operator
 */
public function addMatchOperator(doc:MongoDocument):MongoAggregationPipeline;
public static function addMatchOperator(doc:MongoDocument):MongoAggregationPipeline;
		
/**
 * Adds a $limit operator in the aggregation. Use this to limit the result collection.
 * @param unit nbMaxDoc the maximum number of document returned. Cf. http://docs.mongodb.org/manual/reference/aggregation/#_S_limit
 * @return MongoDocumentAggregationPipeline this to able to chain the operator
 */
public function addLimitOperator(nbMaxDoc:uint):MongoAggregationPipeline
public static function addLimitOperator(nbMaxDoc:uint):MongoAggregationPipeline;
		
/**
 * Adds a $skip operator in the aggregation. Use this to skip the result collection.
 * @param unit nbDocToSkip the maximum number of document returned. Cf. http://docs.mongodb.org/manual/reference/aggregation/#_S_skip
 * @return MongoDocumentAggregationPipeline this to able to chain the operator
 */
public function addSkipOperator(nbDocToSkip:uint):MongoAggregationPipeline;
public static function addSkipOperator(nbDocToSkip:uint):MongoAggregationPipeline;
		
/**
 * Adds a $unwind operator in the aggregation. Use this to peels off the elements of an array individually, and returns a stream of documents.
 * @param String attributeName. The attribute name that will be peeled off. If the attribute don't begins with a '$' then this method adds it automatically. Cf. http://docs.mongodb.org/manual/reference/aggregation/#_S_unwind
 * @return MongoDocumentAggregationPipeline this to able to chain the operator
 */
public function addUnwindOperator(attributeName:String):MongoAggregationPipeline;
public static function addUnwindOperator(attributeName:String):MongoAggregationPipeline;
		
/**
 * Adds a $group operator in the aggregation. A $group operator is used for the purpose of calculating aggregate values based on a collection of documents.
 * @param MongoDocument doc the document containing the $group value. Cf. http://docs.mongodb.org/manual/reference/aggregation/#_S_group
 * @return MongoDocumentAggregationPipeline this to able to chain the operator
 */
public function addGroupOperator(doc:MongoDocument):MongoAggregationPipeline;
public static function addGroupOperator(doc:MongoDocument):MongoAggregationPipeline;
		
/**
 * Adds a $sort operator in the aggregation. The $sort pipeline operator sorts all input documents and returns them to the pipeline in sorted order.
 * @param MongoDocument doc the document containing the $sort value. Cf. http://docs.mongodb.org/manual/reference/aggregation/#_S_sort
 * @return MongoDocumentAggregationPipeline this to able to chain the operator
 */
public function addSortOperator(doc:MongoDocument):MongoAggregationPipeline;
public static function addSortOperator(doc:MongoDocument):MongoAggregationPipeline;

/**
 * Gets the array of aggregation operators
 * @return Array of MongoDocument. Each MongoDocument contains an aggregation's operator in the order of insertion
 */ 
[ArrayElementType("MongoDocument")]
public function get tabPipelineOperators():Array;
```

## Examples ##
A simple sort operation :
```
private var doc1:Object = {
			title : "this is my first title" ,
			author : "bob" ,
			posted : d1,
			pageViews : 5 ,
			tags : [ "fun" , "good" , "marvellous" ] ,
			comments : [
				{ author :"joe" , text : "this is cool" } ,
				{ author :"sam" , text : "this is bad" }
			],
			other : { foo : 10 }
		};
		
private var doc2:Object = {
			title : "this is my second title" ,
			author : "tony" ,
			posted : d2 ,
			pageViews : 6 ,
			tags : [ "fun" , "good" , "sad" ] ,
			comments : [
				{ author :"joe" , text : "this is cool" } ,
				{ author :"sam" , text : "this is bad" }
			],
			other : { foo : 5 }
		};
		
private var doc3:Object = {
			title : "this is my third title" ,
			author : "bob" ,
			posted : d3,
			pageViews : 7,
			tags : [ "fun" , "good" ] ,
			other : { foo : 10 }
		};
		
...
// Build a $sort aggregation
var pipeline:MongoAggregationPipeline =
	MongoAggregationPipeline.
		addSortOperator(new MongoDocument().
			addKeyValuePair("posted", 1).
			addKeyValuePair("other.foo", -1)
		);
driver.aggregate("testu", pipeline, new MongoResponder(onResponseReceived));

...

private function onResponseReceived(responseDoc:MongoDocumentResponse, token:*):void {
	assertTrue(rep.isOk);
	assertEquals(3, rep.interpretedResponse.length);
			
	assertEquals("this is my third title", rep.interpretedResponse[0].title);
	assertEquals("this is my second title", rep.interpretedResponse[1].title);
	assertEquals("this is my first title", rep.interpretedResponse[2].title);
```

Project, Limit and skip together :
```
var pipeline:MongoAggregationPipeline =
	MongoAggregationPipeline.
		addProjectOperator(
			new MongoDocument().addKeyValuePair("title",1).addKeyValuePair("author",1)).
		addSkipOperator(1).
		addLimitOperator(1);
driver.aggregate("testu", pipeline, new MongoResponder(onResponseReceivedTestLimitSkip));
```

Condition :
```
var pipeline:MongoAggregationPipeline =
	MongoAggregationPipeline.
		addProjectOperator(new MongoDocument().
			addKeyValuePair("title",1).
			addKeyValuePair("sixPagesView",MongoDocument.cond(MongoDocument.eq("$pageViews", 6), true, false)).
			addKeyValuePair("notNullcomments", MongoDocument.ifNull("$comments", []))).
		addSortOperator(new MongoDocument("posted",-1));
driver.aggregate("testu", pipeline, new MongoResponder(onResponseReceivedTestCondition));

public function onResponseReceivedTestCondition(rep:MongoDocumentResponse, token:*):void {
	assertTrue(rep.isOk);
	assertEquals(3, rep.interpretedResponse.length);
			
	var repDoc1:Object = rep.interpretedResponse[0];
	var repDoc2:Object = rep.interpretedResponse[1];
	var repDoc3:Object = rep.interpretedResponse[2];
			
	assertEquals(doc1.title, repDoc1.title);
	assertFalse(repDoc1.sixPagesView);
	assertEquals(2, repDoc1.notNullcomments.length);
	assertEquals(doc2.title, repDoc2.title);
	assertTrue(repDoc2.sixPagesView);
	assertEquals(2, repDoc2.notNullcomments.length);
	assertEquals(doc3.title, repDoc3.title);
	assertFalse(repDoc3.sixPagesView);
	assertTrue(repDoc3.hasOwnProperty("notNullcomments"));
	assertEquals(0, repDoc3.notNullcomments.length);			
}
```

# See also #
  * [Home](Home.md),
  * [Using responder](Responder.md),
  * [Query](Query.md) (Find, FindOne, getMore, killCursor),
  * [Count](Count.md),
  * [Distinct](Distinct.md),
  * [Group](Group.md),
  * [MapReduce](MapReduce.md)

