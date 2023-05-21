## Run a MongoDB container using Docker

1. Pull the MongoDB Docker Image
```
docker pull mongodb/mongodb-community-server
```

2. Run the Image as a Container 
```        
docker run --name mongo -d mongodb/mongodb-community-server:latest
```
3. Connect to the MongoDB Deployment with mongosh
``` 
docker exec -it mongo mongosh
```     
    The default port of MongoDB server is 27017



## Database, Collection & Document
* A MongoDB server has one or more database (Database is similiar to database in a SQL database server) 
* Each database can have one or more collection (Collection is similiar to Table in a SQL database) 
* Each collection can have one or more documents (Document is similiar to Row in a SQL database)
* Each document can have one or more key-value pairs (Key is similiar to a column in a SQL database)


## Mongo Shell

### Database and Collection

The API interface provided by different MongoDB drivers are similiar to mongo shell


**List all databases in a MongoDB server**
```
> show dbs 
```

**Select a database**
```
> use {database_name}
```

If a database is not present in the server with the specified name, a new database is automatically created by the server with the provided name.

Once we have selected a database, we can reference it with  **db.** 

**List all collections in current database**
```
> show collections
```
or
```
> show tables
```

**Select a collection**

To reference a collection, we use collection name after db.  (e.g db.employee)

```
> db.{database_name}.{collection_name}
```

If a collection is not present in the database with the specified name, a new collection is automatically created by the server with the providied name.



**Creating document in a collection**

MongoDB shell provides insertOne() and insertMany() function to create on or many documents in a collection

Create a single document using insertOne() function
```
db.user.insertOne({"name": "alex", age: 18})
```
If document is created successfully, mongo server returns a object with acknowledgement status and id of the created document

```
{
  acknowledged: true,
  insertedId: ObjectId("646a0d760b2c00a3baea10fb")
}

```

**List all documents in a collection**
```
db.user.find().pretty()
```

### Data Types

* **Text** : "hello world"     
* **Boolean** : true/false
* **Integer (32 bit)** :  3
* **NumberLong (64 bit)**: 10000000
* **NumberDecimal** :  3.4
* **ObjectID** : ObjectId("646a0d760b2c00a3baea10fb")
* **ISODate** :  ISODate("2019-09-09")   new Date()
* **Timestamp** : Timestamp(63538922)     new Timestamp()
* **EmbeddedDocument** : {"address":{"pincode": 12201}}
* **Array** :  {"hobbies":["Cycling","Singing", "Gardening"]} 

MongoDB converts and stores JSON objects in BSON format. BSON is a binary format which is more efficent for storage, query operations, compression and data types.

### Document Key and its Value
* For specifying key of a document use of double quotes or single quotes is optional, if the key doesn't contain any whitespace
* If key contains any whitespace then key needs to be enclosed within double quotes or single quotes
* For specifying value of a document, use of double quotes or single quote for string type values is necessary

### Document Schema
Documents in a mongo db collection are schemaless.

### CRUD Operations

### Create

1. insertOne(data, options)
```
> db.user.insertOne({"name": "alex", age: 18})
```
```
{
  acknowledged: true,
  insertedId: ObjectId("646a0d760b2c00a3baea10fb")
}
```


2. insertMany(data, options)
```
> db.user.insertMany([{name:"bob", age: 20},{name:"sam", age: 21}])
```
```
{
  acknowledged: true,
  insertedIds: {
    '0': ObjectId("646a567c3c96fdae53aebbcd"),
    '1': ObjectId("646a567c3c96fdae53aebbce")
  }
}
```

### Read

1. findOne(filter, options)

```
> db.user.find({ _id: ObjectId("646a567c3c96fdae53aebbce") })
```
```
[ { _id: ObjectId("646a567c3c96fdae53aebbce"), name: 'sam', age: 21 } ]


```
2. find(filter, options)

```
> db.user.find({}).pretty()
```
```
[
  { _id: ObjectId("646a567c3c96fdae53aebbcd"), name: 'bob', age: 20 },
  { _id: ObjectId("646a567c3c96fdae53aebbce"), name: 'sam', age: 21 }
]

```

**Cursor**

**find()** gives us a cursor object though which we can scroll over results of a query. By defaults cursor gives 20 results. We can navigate over database results through cursor.

**pretty()** we can pretty function to beautify the json result of mongo query in shell

**findOne()** gives us the actual document found for a query. It doesn't return a cursor and therefore we cannot use pretty() function on findOne().

**Filtering search results**
We can pass the key-values in first object to findOne() or find() to filter the search results (This is similiar to where clause in SQL)
```
> db.user.find({name: 'alex'})

find documents where user.name is 'alex'

```

**Projection**

Using projection we can read specific keys of a document (This similiar to select clause in SQL)

```
> db.user.find({}, {name:1}).pretty()

Read all user but for each document only read name key and its value
````
```
[
  { _id: ObjectId("646a567c3c96fdae53aebbcd"), name: 'bob' },
  { _id: ObjectId("646a567c3c96fdae53aebbce"), name: 'sam' }
]
```

**_id key**

_id is always included in result. To omit _id in query result we need to explicitly exclude it 
```
> db.user.find({}, {name:1, _id:0}).pretty()
```
```
[ { name: 'bob' }, { name: 'sam' } ]

```



### Update

1. updateOne(filter, data, options)

**Update value of a key** 
```
>db.user.updateOne({name:'bob'}, {$set:{name:'jerry'}})
```
Here we have use **$set** operator to update a document.

**Add a new key-value in a document**

```
> db.user.updateOne({name:'jerry'}, {$set:{dob:new Date('2010-02-30')}})
```
```
{
  acknowledged: true,
  insertedId: null,
  matchedCount: 1,
  modifiedCount: 1,
  upsertedCount: 0
}

```
'dob' key added
```
{
    _id: ObjectId("646a567c3c96fdae53aebbcd"),
    name: 'jerry',
    age: 20,
    dob: ISODate("2010-03-02T00:00:00.000Z")
}
```
**Add a nested document**
```
> db.user.updateOne({name:'jerry'}, {$set:{address: {city: 'Delhi',country:'India'}}})

```

'address' key added
```
{
    _id: ObjectId("646a567c3c96fdae53aebbcd"),
    name: 'jerry',
    age: 20,
    dob: ISODate("2010-03-02T00:00:00.000Z"),
    address: { city: 'Delhi', country: 'India' }
}
```
**Update value of a nested key**

```
> db.user.updateOne({name:'jerry'}, {$set:{'address.city': 'Assam'}})
```
A nested key can be referenced using **. dot** operator
```
{
    _id: ObjectId("646a567c3c96fdae53aebbcd"),
    name: 'jerry',
    age: 20,
    dob: ISODate("2010-03-02T00:00:00.000Z"),
    address: { city: 'Assam', country: 'India' }
}
```

**Add a array type key**
```
> db.user.updateOne({name:'jerry'}, {$set:{'hobbies': ['Cycling','Gardening','Singing']}})
```

```
{
    _id: ObjectId("646a567c3c96fdae53aebbcd"),
    name: 'jerry',
    age: 20,
    dob: ISODate("2010-03-02T00:00:00.000Z"),
    address: { city: 'Assam', country: 'India' },
    hobbies: [ 'Cycling', 'Gardening', 'Singing' ]
}
```


2. updateMany(filter, data, options)


3. replaceOne(filter, data, options)

**Delete**
* deleteOne(filter, options)
* deleteMany(filter, options)





We can use custom _id value while creating documents


CRUD Operations:




find() gives us a cursor object though which we can scroll over results of a query. MongoDB curson by defaults gives only 20 results.

findOne() gives us the actual data for the found document. It doesn't return a cursor therefore we could not use pretty() on findOne().

updateOne({}, {$set: {status: 'active'}}) update value of 'status' key as 'active' for all documents in collection


Filter
first argument to find, update, delete is filter object, through which we narrow down the search.
Filters allows to limit the no. of documents in a collection



Embedded Documents
Value of key can be another document. (Nested documents)
We can have a nesting upto 100 level in MongoDB.
Each document can be of max size upto 16 MB in MongoDB
We can also have an Array of embedded docuemnts

Searching in embedded documents
find({"key1.innerkey1.innerKey1": 'value'})  use . dot notation for specifiying an inner object key in case of objects
find({"array_name": "value1"})  find can search for array of string directly 


In a SQL table, if a column doesn't have a value we have null value for that column in row
In a Mongo, if a document doesnt't have a value for a key, we can omit that key because mongo db is schemaless



db.stats()  can be used to view stats information

Relations:

1. Nester/EmbeddedDocument
        with nesting we have data in same document
        reading is compartively easy
customer 
{
   "name": "alex"
   "age": 23,
   "department": "IT",
   "address" : {
        "house_no": "K33",
        "street_address1": "",
        "street_address2": "",
        "city": "",
        "state": "",
        "country": ""
   }
}


2. Reference
        with reference we store id of the other document
        reading requires reading main document and referenced document

customer
{
   "name": "alex"
   "age": 23,
   "department": "IT"
   "addresses": [343]
}

address
{
    "_id": 343
    "house_no": "K33",
    "street_address1": "",
    "street_address2": "",
    "city": "",
    "state": "",
    "country": ""
}


With help of aggregate function we can return referenced in our query result 

db.customer.aggregate([$lookup: {from: "customer", localField: "addresses", foreignField: "_id"}]) 


While modeling data we need to carefully take decision whether storing an embedded document or a reference would be a good approach. If data is related and not changed often, embedded document would might be a good approach. On the other side, if change in referenced document will affect on our main document then storing reference would might be better approach.


mongod   can be used to configure mongo server
mongod --help 
mongod --dbpath can used to change the database storage files
mongod --logpath can used to change the log file path
mongod --for --logpath can be used to start mongo server as a background process

mongo    is a client shell
mongo --help shows help for client shell
in mongo shell
> help       shows help for in shell commands
> db.help()  shows commands available on database object
> db.collection.help()  shows commands available on collection object


MongoDB Compass
Its a UI tool to access MongoDB database
Available for Windows,Linux and Mac OS. Can be downloaded from official website of MongoDB


CRUD
Create Document
insertOne({})       : Insert one document
insertMany([{},{}]) : Insert multiple documents
                      By default, objects are inserted in the order in which they are provided.
                      If insertion fails in between, documents after the faliure are not inserted 
insertMany([{},{}],{ordered: false})  : can be used to allow database to continue processing rest of the 
                                        documents after faliure
                      WriteConcern
insertOne({},{w:1})   w specifies on how many instances we want document insertion to be acknowledged

insertOne({},{w:0})   if w is 0 database will return acknowledge: false as writtern because it accepted the request
                      but whether that write actually got succefully processed or not is known
                      This is extremly fast as db will immediately return the response, but there is no gurantee that
                      whether data got succefully saved on disk db file or not

insertOne({},{w:0, j: true}) j species journaling, if true database will ensure that only when document is written to                    the journal file, then only it acknowledges the insertion response

insertOne({},{w:0, wtimeout:200}) timeout value for w write

Atomicity is applicable at single document insertion (e.g insertOne()) for complete document including embedded document
Atomicity is not applicable on multiple document insertion in insertMany([{},{}])       

insert({})          : Old version of insert, supports single and multiple document insertion, depricated       
mongoimport         : Import data from a json file
mongoimport data.json -d facebook -c user --jsonArray --drop   
                     -d database_name
                     -c collection_name
                     --jsonArray multiple objects in file
                     --drop drop the collection if it already exits, otherwise command will append the documents
                        in existing collection




{
    first_name: 'Lenia',
    last_name: 'Sein',
    email: 'lenia.sein32@gmail.com',
    phone: '316212693',
    active: true
  }




MongoDB Query Operators

Comparison

$eq  : Matches values that are equal to a specified value.
$gt  : Matches values that are greater than a specified value.
$gte : Matches values that are greater than or equal to a specified value.
$in  : Matches any of the values specified in an array.
$lt  : Matches values that are less than a specified value.
$lte : Matches values that are less than or equal to a specified value.
$ne  : Matches all values that are not equal to a specified value.
$nin : Matches none of the values specified in an array.

Syntax
db.collection.find({'key': {$eq:50}}})
db.collection.find({'key': {$in: ['india','china'] }}})



Logical

$and : Joins query clauses with a logical AND returns all documents that match the conditions of both clauses.
$not : Inverts the effect of a query expression and returns documents that do not match the query expression.
$nor : Joins query clauses with a logical NOR returns all documents that fail to match both clauses.
$or  : Joins query clauses with a logical OR returns all documents that match the conditions of either clause.

Syntax
db.collection.find({$or: [{"gender":"male"}, {"age": {"$eq": 50}}]})


Element

$exists : Matches documents that have the specified field.
$type   : Selects documents if a field is of the specified type.

Syntax
db.collection.find({"age": {$exists: true}})  : returns documents where age key is available
db.collection.find({"age": {$type: 'double'}}): returns if value is matched with specifed type
db.collection.find({"age": {$type: ['string', 'number']}}): multiple types can be specified using array


Evaluation

$expr        : Allows use of aggregation expressions within the query language.
$jsonSchema  : Validate documents against the given JSON Schema.
$mod         : Performs a modulo operation on the value of a field and selects documents with a specified result.
$regex       : Selects documents where values match a specified regular expression.
$text        : Performs text search.
$where       : Matches documents that satisfy a JavaScript expression.

Syntax
db.user.find({'company.address.address':{$regex: /Sable/}}) : regex search for 'Sable' string in address
db.collection.find({$expr: {$gt: ["$volume", "$target"]}})   : returns result where value of volume key greater than target key



Geospatial

$geoIntersects  : Selects geometries that intersect with a GeoJSON geometry. The 2dsphere index supports $geoIntersects.
$geoWithin      : Selects geometries within a bounding GeoJSON geometry. The 2dsphere and 2d indexes support $geoWithin.
$near           : Returns geospatial objects in proximity to a point. Requires a geospatial index. The 2dsphere and 2d indexes support $near.
$nearSphere     : Returns geospatial objects in proximity to a point on a sphere. Requires a geospatial index. The 2dsphere and 2d indexes support $nearSphere.


Array

$all       : Matches arrays that contain all elements specified in the query.
$elemMatch : Selects documents if element in the array field matches all the specified $elemMatch conditions.
$size      : Selects documents if the array field is a specified size.

Syntax:
db.collection.find({"document.array.document": "Sports"}) : just like embedded documents we can use dot to look for objects in array


Bitwise

$bitsAllClear : Matches numeric or binary values in which a set of bit positions all have a value of 0.
$bitsAllSet   : Matches numeric or binary values in which a set of bit positions all have a value of 1.
$bitsAnyClear : Matches numeric or binary values in which any bit from a set of bit positions has a value of 0.
$bitsAnySet   : Matches numeric or binary values in which any bit from a set of bit positions has a value of 1.


Projection Operators

$          : Projects the first element in an array that matches the query condition.
$elemMatch : Projects the first element in an array that matches the specified $elemMatch condition.
$meta      : Projects the document's score assigned during $text operation.
$slice     : Limits the number of elements projected from an array. Supports skip and limit slices.


Miscellaneous Operators

$comment   : Adds a comment to a query predicate.
$rand      : Generates a random float between 0 and 1.


Indexes


Explain to view query execution stats
db.collection.explain().queryFunction() : explain can be used to view query plan
db.collection.explain("executionStats").queryFunction() : explain can be used to view query plan

Create Index
db.collection.createIndex({"age": 1})  : key represents field, 1 or -1 represents asc or desc order respectively

Drop Index
db.collection.dropIndex({"age":1})     : drop index
db.collection.dropIndex("index_name")  : drop index by specifying index name


Compound Index
db.user.createIndex({"age":1, gender:1}): compound index
db.user.explain().find({age:50, gender: 'male'}) : uses compound index
db.user.explain().find({age:50})      : will use index to scan, compound index can be partially used from left to right
db.user.explain().find({gender: 'male'}) : will not use index to scan
db.user.explain().find({age:50, gender: 'male'}).sort({'age':1}) : index can be used to get the result in sorted array
if index is not present mongodb will read the records and sort them in in-memory, mongo db has a limit of 32 mb of memory size to to in memory operation

View Indexes on a collection
db.collection.getIndexes() : returns all index on a collection

Unique Index
db.user.createIndex({email:1}, {unique:true}):create a unique index, add unique constraint for a key 

Partial Index
db.user.createIndex({age:1}, {partialFilterExpression: {gender: 'male'}})   create index for age field on where gender is equal to male
 
if in query we only use age:1, mongodb will not use partial index as partial index is associated with gender:male
so it will fallback to perform a sequential scan

Comparison between Compound index and Partial index:
Size of partial index will be smaller as it is application to only some amount of data in a collection.

Problem with unique index for missing keys
If some documents doesn't have the key on which we have created the unique index. MongoDB will consider null value for missing keys. And since two keys cannot have same value which is null in this case, it will restrict document creation without unique index key

Solution to this is to create a partial index so that it only considers documents where unique index key is present
db.user.createIndex({email:1}, {unique: true, partialFilterExpression: {email: {$exists: true}}})
In this approach the docments where unique key is not present will not be indexed

Time-to-Live Index
db.user.createIndex({'created_at': 1}, {expireAfterSeconds: 10}) : mongodb will automatically delete after expiry time
creating the ttl index will not immediately evaluate the document expiry, when a new document is inserted in the collection after the ttl index creation, mongodb will start deleting the documents for which ttl is expired


ttl don't work on single field index, it only works on date object.

Multikey Index

If we create a index on array (embedded document also supported), mongodb will call this a multikey index and will store all values of the array in index

db.user.createIndex({cn:1})
{
     cn: [ 'india', 'us', 'china', 'australia', 'japan' ]
}

db.user.explain('executionStats').find({'cn': 'japan'})


If we have array of objects, then mongodb will be able to use index only if we search for the complete object

db.user.insertOne({'address': [{'city': 'delhi'}, {'city':'gurgaon'}]})

db.user.explain('executionStats').find({'address': {'city': 'gurgaon'}})

If we query using exact field, it will not be able to use index as in index whole object of array is stored
db.user.explain('executionStats').find({'address.city':'gurgaon'})

If we create a multikey index on field of object in an array also like
db.user.createIndex({address.city:1}) and then 
db.user.explain('executionStats').find({'address.city':'gurgaon'}) will use this index

but it will be slower as we nesting more levels. with more nested level index query performance will become slow


we can create compound index with one array, but we cannot create compound index with more than one array multikey index.
as in that case mongodb will have to create cartesian product for all values of two arrays and then store them index


Text Index
If we have text in key, we can use Text Index to search for words in a text efficently (better than regex search)

text index: internally creates an array for all keywors in a text and stores them so that we can search word
            it doesn't automaticaly stores grammatical words like (is are and etc)
            all keywords are stored in lowercase in index, so text search is case-insenstive for client
            we can have only one text index on a collection.

db.user.createIndex({'bio': 'text'}): create text index

db.user.find({"$text: {$search: "engineer"}) : we dont need to specify key name for text indexes
db.user.find({"$text: {$search: "software engineer"}) : return records where keyword software and engineer is found
db.user.find({"$text: {$search: "\"software engineer"\"}): to search the complete word we can enclose it in quotes 

MongoDb assigns a score to each search result, we can use this search score to sort our search result to the document which is more close to our search string

db.user.find({"$text: {$search: "\"software engineer"\"}, {score: {$meta: "textScore"}}).sort({score: {$meta: "textScore"}}).pretty()

To perform search on two different text keys of a collection

db.user.createIndex({'bio': 'text', 'favourites_quotes': text}): can be used in this case in same text index value of both the text fields will be indexed

We can also specify 'language' and 'weight' value while creating compund text index


Building Index 
Foreground:  while index is in creation whole collection is locked, index creation is faster
Background:  while index is in creation whole collection is not locked, index creation is slower




Cursor

db.collection.find().next()  : we can use next method to iterate over results using cursor one by one
db.collection.find().sort({"age": 1}) : we can sort result by any key, we can also use embedded key, we can also use multiple keys

db.collection.find().skip(10).limit(10) : we can use skip() for offset and limit() to limit the cursor result


Update Query
db.user.updateOne({_id: ObjectId("6467e4f5a3eb450924815578")}, {$set:{"firstName": "Nikhil Gautam"}})
db.user.updateOne({_id: ObjectId("6467e4f5a3eb450924815578")}, {$set:{"address" :{pincode: "122017"}}})
db.user.updateOne({_id: ObjectId("6467e4f5a3eb450924815578")}, {$set:{"address.city": "gurgaon"}}) 

db.user.updateOne({_id: ObjectId("6467e4f5a3eb450924815578")}, {$inc: {age:1}}) : increment age by 1

db.user.updateMany({},{$rename: {age: 'totalAge'}}) : change key name usind $rename
db.user.updateOne({filter},{update}, {upsert: true}): upsert can be user to insert a new document with  existing object information


db.user.updateOne({}, {$push: {"hobbies": {"title": "Cycling", "frequency": 'daily'}}}):  one object will be pushed in hobbies array


db.user.updateOne({}, {$pull: {"hobbies": {"title": "Cycling"}}}) : objects will be removed from array where hobbies is equal to 'Cycling'

db.user.updateOne({}, {$pop: {"hobbies": 1}}) : last added object will be removed

db.user.updateOne({'hobbies.title': 'Singing'}, {$set: {'hobbies.$.title': 'Dancing'}}) : update value of a key in a object in an array (using .$. is positional operator)


Delete Query
db.collection.deleteOne({filter})     : deletes first matching document
db.collection.deleteMany({filter})    : deletes all matching documents
db.collection.deleteMany({})          : deletes all documents
db.collection.drop()                  : deletes complete collection
db.databaseDrop()                     : deletes the whole database

db.user.updateOne({_id: ObjectId("6467e4f5a3eb450924815578")}, {$unset:{address: 1}}) : delete a key from document 


Aggregation



db.user.aggregate([ {$match:{gender: 'male'}} ])  is same as db.user.find({gender: 'male'})


db.user.aggregate([ 
    {$match:{gender: 'male'}}, 
    {$group:{_id: {'department': '$company.department'}, EmployeeCount: {$sum: 1}}} 
])


Group objects by department

Count of documents by department
db.emp.aggregate([{$group:{_id: '$department', count:{$count: {}}}}])

Max salary by department
db.emp.aggregate([{$group:{_id: '$department', max_salary:{$max: "$salary" }}}])

Min salary by department
db.emp.aggregate([{$group:{_id: '$department', min_salary:{$min: "$salary" }}}])

Sum of salary by department
db.emp.aggregate([{$group:{_id: '$department', sum_salary:{$sum: "$salary" }}}])


Transactions:
  const session = db.getMongo().startSession()
  session.startTransaction()
  const user = session.getDatabase('fb').user
  const order = session.getDatabase('fb').order
  user.deleteOne({id: 12})
  order.deleteOne({user_id: 12})
  session.commitTransaction() or session.abortTransaction()


User Management

Create User
db.createUser({user: "alex", pwd: 'secretpwd', roles: ['userAdminAnyDatabase']})


Login with user credentials
db.auth('alex', 'secretpwd')

https://www.mongodb.com/docs/manual/reference/built-in-roles/



Performance

Capped Collection: Capped collections are fixed-size collections that support high-throughput operations that insert and retrieve documents based on insertion order. Capped collections work in a way similar to circular buffers: once a collection fills its allocated space, it makes room for new documents by overwriting the oldest documents in the collection.

db.createCollection("test", {capped: true, size:1024 max: 3}): size in bytes, max is optional it means total no. of documents for which limit is set

For capped collection, we get results in the order in which they were added

db.collection.find().sort({$natural: -1}) to read the result in reverse order


Replica Sets

Sharding

Atlas is managed solution provided by mongo db as a cloud service