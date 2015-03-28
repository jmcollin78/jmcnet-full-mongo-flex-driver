

# Introduction #

This page will explain, how to use the MapReduce command. More explanations can be found @ MongoDB documentation [here](http://www.mongodb.org/display/DOCS/Aggregation#Aggregation-Map%2FReduce)

# Details #
MapReduce is done by using the _mapReduce_ method of the driver. The _mapReduce_ method of the driver has the following signature :
```
/**
 * Map and reduce documents. Example of scope ('init=0' and a JS function called 'myScopedFct=new Date()') : new MongoDocument("init", 0).addKeyValuePair("myScopedFct", new JavaScriptCodeScoped("new Date()"))
 * @param collectionName (String) : the name of the collection to query from,
 * @param map (JavaScriptCode) : the map JS function. JS signature must be "function():void". You must call 'emit(key, value)' in the JS function,
 * @param reduce (JavaScriptCode) : the reduce JS function executed on the documents grouped by key. JS signature must be "function (key, emits):{key, value}",
 * @param out (MongoDocument) : specificy how to output the result. Values can be. See http://www.mongodb.org/display/DOCS/MapReduce#MapReduce-Outputoptions,
 * @param callback (Function) : the callback called when documents are ready to read. Default value is null,
 * @param query (MongoDocument) : a document used to filter documents that will be map/reduced. Default value is null (all documents are considered),
 * @param sort (MongoDocument) : a document specifiying the sort order. Default value is null (no sort),
 * @param limit (uint) : limit the number of documents considered. Usefull only with sort. Default is 0 (no limit),
 * @param finalize (JavaScriptCode) : a JS function used to finalize the result. JS signature must be "function(key, result):result". Default value is null (no finalization).
 * @param scope (MongoDocument) : a document specifiying the scoped variables. Scoped variables are global variables usable in all JS. If the value of a variable is a JS code, you must use JavaScriptCodeScoped to specify the code. Default value is null (no scope variables),
 * @param jsMode (Boolean) : if true, all JS is executed in one JS instance. Default value is false,
 * @param verbose (Boolean) : if true output all 'print' JS command to the server logs. Default is false.
 */
public function mapReduce(collectionName:String, map:JavaScriptCode, reduce:JavaScriptCode, out:MongoDocument, responder:MongoResponder,
	query:MongoDocument=null, sort:MongoDocument=null, limit:uint=0, finalize:JavaScriptCode=null,
	scope:MongoDocument=null, jsMode:Boolean=false, verbose:Boolean=false):void

```

The result of a _mapReduce_ command is a _MongoDocumentResponse_ in which there is only one document which _results_ value contains an Array of result (build by _finalize_).

Example :
```
private var mapFunctionCode:String =
	"function() {\n"+
	"    emit(this.attrString, { total : this.attrInt32, nb : 1 });\n"+
	"}";
private var reduceFunctionCode:String =
	"function (key, emits) {\n"
	"    nb=0;\n"+
	"    total = 0;\n"+
	"    for (var i in emits) {\n"+
	"        nb += emits[i].nb;\n"+
	"        total += emits[i].total;\n"+
	"    }\n"+
	"    return {total : total , nb : nb};\n"+
	"}";

private var finalizeFunctionCode:String =
	"function(key, result) {\n"+
	"    if (result.nb != 0)\n"+
	"         result.avg = result.total/result.nb;\n"+
	"    else result.avg = 0;\n"+
	"    result.scopedVar = myScopedFct();\n"+
	"    result.init = init++;\n"+
	"    return result;\n"+
	"}";

mongoDriver.mapReduce(
	"documentsCollectionName",
	/* map */    new JavaScriptCode(mapFunctionCode),
	/* reduce */ new JavaScriptCode(reduceFunctionCode),
	/* out */ new MongoDocument("inline", 1),
	onResponseReceived,
	/* query */ new MongoDocument("attrString", MongoDocument.exists(true)).
		addKeyValuePair("attrInt32", MongoDocument.exists(true)),
	/* sort */ null,
	/* limit */ 0,
	/* finalize */ new JavaScriptCode(finalizeFunctionCode),
	/* scope */ new MongoDocument("init", 0).
		addKeyValuePair("myScopedFct",
			new JavaScriptCodeScoped("new Date()")),
	/* jsMode */ false,
	/* verbose */ true
);

...

private function onResponseReceived(responseDoc:MongoDocumentResponse):void {
	if (responseDoc.numberReturned == 1) {
		for each (var doc:MongoDocument in responseDoc.getDocument(0).getValue("results")) {
			// do something with doc
		}
	}
```

Please refers to MongoDB documentation to have more explanations and example of the _mapReduce_ command. [Here](http://www.mongodb.org/display/DOCS/Aggregation#Aggregation-Map%2FReduce).

## Scoped variables ##
In the example above, there is two scoped variables :
  * `init=0`
  * `myScopedFct=new Date()` : a JS function.

You can notice that _init_ can be read and modify during map/reduce/finalize JS functions and that _myScopedFct_ is used this way : `xxx = myScopedFct();` in the finalize function.

# See also #
  * [Home](Home.md),
  * [Using responder](Responder.md),
  * [Query](Query.md) (Find, FindOne, getMore, killCursor),
  * [Count](Count.md),
  * [Distinct](Distinct.md),
  * [Group](Group.md),
  * [MapReduce](MapReduce.md)

