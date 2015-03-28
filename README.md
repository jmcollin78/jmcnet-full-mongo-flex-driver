A full Flex/Flash AS3 Driver for MongoDB database V 2.0, 2.1, 2.2.rc1, 2.2
Connect your AS3 application directly to a MongoDB database. Now you don't need a middleware to access your data.

Main features of the driver are :

Connexion pooling,
Auth mode compatibility,
Safe mode enabling,
Direct CRUD object manipulation,
Full Aggregation support (group, MapReduce, ...),
BSON encoder and decoder,
ObjectID helper
New with 2.0 :

Native support for generic object DirectObject#Using_generic_object_while_inserting_and_querying,
New aggregation framework helpers AggregationFramework,
DBRef support and automatic DBRef deferencement with cross database DBRef possibility DBRef,
Testing via FlexUnit and logging capabilities TestingAndLogging.
See wiki documentation for more information or the test application provided to show uses cases (source/browse/MongoDBTest).

The complete source of FlexUnit testing is also provided in source/browse/JMCNetMongoFlex_Test.

Forum : you can interact with us in the Community forum.

V 2.1-SNAPSHOT Enhancements
correct  issue15  (suppress unwanted .toString()
correct  issue16  (option not to call .toObject() for performance issue),
correct  issue18  (multiple update modifiers now possible),
add response to issue17.
Add Synchrone Runner class to unable synchrone message sending. See handling synchrone messages,
Correct CollectionInterpreter for operation on collection.
V 2.0 Enhancements
Adding native support of generic object like Object and ArrayCollection in DirectObject. See DirectObject#Using_generic_object_while_inserting_and_querying for more informations.
Adding Responder and Token capabilities to driver's calls. This is more standard and so you can now give a token in the call which is given back in the callback. Be careful, this enhancement leads to break compatibility with previous version of the driver.. Hopefully, changes are easy to correct and are documented in the wiki. See Responder and PortabilityNotes for more informations.
Add response interpreter to make response decoding very much simplier. For example, mapReduce and aggregation has "results", group has "retval", distinct has "values" for return values.
Add the support of the new aggregation framework with helpers. New operator must be also provided to complete the aggregation framework.
Add helpers for full DBRef support and automatic deferencement upon find documents.
Add multi-driver capabilities. Use one driver per database and DBRef cross databases support. See DriverInitialization#Multiple_drivers_manipulations,
Add a FlexUnit base class which provide easy testing capabilities. See TestingAndLogging,
Enforce logging capabilities to make debug easier. See TestingAndLogging,
Compatibility with MongoDB 2.2 server,
Issues :  issue 5 ,  issue 8 ,  issue 10 ,  issue 12 ,  issue 11 ,  issue 7 .
Note : this release must be used with the V 1.3 JMCNet libCommun library which can be found here.

V 1.4 Enhancements
correct issue in auth mode when authentication event can be thrown after CONNECT_OK event,
add a EVT_AUTH_ERROR event to easily handle case where authentication is NOK.
Tested with MongoDB 2.x in auth mode and in non auth mode. Note : this release must be used with the V 1.2 JMCNet libCommun library.

V1.3 Enhancements
correct "orderby" directive in MongoDocumentQuery,
add JUnit tests to verify "orderby" and "returnFields" attribute in driver.queryDoc.
V1.2 Enhancements
add ObjectID.fromStringRepresentation method to initialize an ObjectID from its String representation,
correct ObjectID.toString method. If byte value < 17, no preceeding 0 was added leading to incorrect String representation.
See ObjectID for more information about ObjectID manipulation.

V1.1 Enhancements
Makes the driver waits for free socket in the pool when the pool is empty. So you can configure your application with less socket in the pool. See Driver initialization

Licensing
This library is available under 2 licensing mode :

GNU GPL V3 : you can freely use, redistribute, modify this code if your product is open source and GNU GPL too,
Commercial license : commercial applications can use this library without being open source but must paid a licensing fee. Contact me at the[dot]jmcnet[dot]team[at]gmail[dot]com for more information about this commercial license mode.
They use our driver
Clouderial company uses our driver to create an invoice online or to create a quote online. La société Clouderial utilise nos services pour creer une facture en ligne ou creer un devis en ligne.

The JMCNet Team
