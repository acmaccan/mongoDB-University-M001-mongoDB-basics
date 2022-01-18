  # M001: mongoDB basics

  ## The MongoDB Database
  - Database is a structured way to store and access data.
  - NoSQL is a way to organized data store that doesn t use legacy approach of related tables of data, does not have rows and columns.
  - MongoDB is a NoSQL document database. Is stored as documents.
  - These documents are in turn stored in what we call collections of documents.
  - That s why MongoDB is categorized as a NoSQL document database.

  ## Documents
  - Way to organize and store data as a set of field-value pairs
  - The field is a unique identifier for some data point
  - The value is data related to a given identifier
  - A collection would contain many documents
  - A database would contain multiple collections

  ## MongoDB Atlas
  - Database in the cloud
  - MongoDB is used at the core of Atlas for data storage and retrieval
  - Database as a service
  - Clusters: group of servers that stores your data
  - Replica set: a few connected MongoDB instances that store the same data
  - Instance: a single machine locally or in the cloud, running a certain software.
    Si hubiera un error en alguno de los miembros del replica set, la data se mantendría
    intacta y disponible para la aplicación gracias a los miembros aún activos del réplica.

  ## Documents
  - Stored BSON and Viewed JSON
  - Valid JSON format
  - Is friendly, readable, familiar
  - Text-based (slow), space-consuming, limited of basic datatypes
  - Es por eso que la forma en que la data se guarda en memoria es en BSON, binary JSON
  - BSON: optimized for speed, space and flex. High performance and General-purpose focus

  ### JSON:
  - Encoding: UTF-8 String
  - Data Support: String, Boolean, Number, Array
  - Readability: Human and Machine

  ### BSON:
  - Encoding: Binary
  - Data Support: String, Boolean, Number (Int, Long, Float, ...), Array, Date, Raw Binary
  - Readability: Machine only

  ## Data Explorer / Accesing data
  There is multiple ways to explore data:
  - Atlas - online GUI
  - Compass - local GUI
  - mongo shell - local CLI with MQL

  Structure:
  ```
  cluster
  ┕ databases
  ┕ collections
  ┕ documents
  ┕ subdocuments
  ┕ arrays
  ```
  <br/>

  Atlas:
  - browse collections
  - select database / collection
  - filter: {"state":"NY","city":"ALBANY"}

  mongosh:
  - show db: show list of databases in the cluster
  - use \<db-name>: choose database we are going to use
  - show collections: show list of collections in the db
  - db.\<collection-name>.find(\<query>): return all results that match with our query
    This query has a show limit of 20 documents. To see next ones type it (iterate)
  - db.\<collection-name>.find(\<query>).count(): return the number of documents that match
  - db.\<collection-name>.findOne(\<query>): check if one document exist

  - it: iterates through a cursor
  - cursor: pointer to a result set of a query
  - pointer: a direct address of the memory location

  Example:
  ```
  use sample_training                                   // Point to db we are going to use
  show collections                                      // List collections
  db.zips.find({"state":"NY"})                          // Return matching documents
  db.zips.find({"state":"NY","city":"ALBANY"}).count()  // Return number of matching documents
  db.zips.find({"state":"NY","city":"ALBANY"}).pretty() // Easy to read
  db.zips.find({})                                      // Any 20 documents without particular order
  ```
  <br/>

  ## Insert new documents
  
  Atlas:
  browse collections
  insert document

  mongosh:
  - use \<db-name>: choose database we are going to use
  - db.\<collection-name>.findOne(\<query>): check if document exist
  - db.\<collection-name>.insert({document}): if we copy an existing doc return duplicate key error
  - If we erase the _id field from the doc to insert, it is gonna be created with a new _id value

  Example:
  ```
  db.inspections.insert([{ "test": 1 }, { "test": 2 }, { "test": 3 }]) // Bulk result - 3 docs added
  db.inspections.insert([{ "test": 1 }, { "test": 2 }, { "test": 3 }]) // Bulk result - 3 docs added
  ```
  <br/>

  If we repeat the operation, 3 new docs will be added to the collection with the same data, but differents _id

  ```
  db.inspections.insert([{ "_id": 1, "test": 1 },
  { "_id": 1, "test": 2 },
  { "_id": 3, "test": 3 }]) // One document was inserted
  ```
  <br/>

  The documents will be inserted in the order they are listed. With order (default unespecified), if it fails, it stops.

  ```
  db.inspections.insert([{ "_id": 1, "test": 1 },
  { "_id": 1, "test": 2 },
  { "_id": 3, "test": 3 }],
  { "ordered": false }) // Two documents were inserted
  ```
  <br/>

  With ordered false, it will insert all documents that have not duplicate _id.

  ```
  db.inspection.insert([{ "_id": 1, "test": 1 },
  { "_id": 1, "test": 2 },
  { "_id": 3, "test": 3 }]) // Again three documents were inserted
  ```
  <br/>

  If we misspelled a collection - inspections no inspection - the collection will be created with the three documents and no errors will show.

  ## Update documents
  
  Atlas:
  browse collections
  edit fields and type fields

  mongosh:
  updateOne() - findOne()
  updateMany() - find()

  Example:
  ```
  use sample_training
  show collections
  db.zips.find({"city":"HUDSON"}).count() // 16
  db.zips.find({"city":"HUDSON"},{"$inc": {"pop": 10}})
  // {"acknowledged": true, "matchedCount": 16, "modifiedCount": 16}
  ```
  <br/>

  Example:
  ```
  use sample_training
  show collections
  db.zips.find({"zip":"12534"}) // "pop": 21215
  db.zips.updateOne({"zip":"12534"},{"$set": {"pop": 17630}})
  // {"acknowledged": true, "matchedCount": 1, "modifiedCount": 1}
  ```
  <br/>
  Remember that typos will insert new key/values with no errors.

  Example:

  ```
  db.grades.find({"student_id": 250, "class_id": 339})
  // Object with array of scores, with type and score fields

  db.grades.updateOne({"student_id": 250, "class_id": 339},
  {"$push": {"scores": {"type": "extra credit", "score": 100}}})

  // {"acknowledged": true, "matchedCount": 1, "modifiedCount": 1}
  ```
  <br/>

  ### MQL Update operators
  
  \$inc: increment the specify field by the indicated amount
  ```
  { "$inc": {"<field 1>": <increment value 1>, "<field 2>": <increment value 2>, ...} }
  ```
  <br/>

  \$set: set a key/value. If does not exist create one, and if already exist will update it.
  ```
  { "$set": {"<field 1>": <value 1>, "<field 2>": <value 2>, ...} }
  ```
  <br/>

  \$unset

  \$push: to add an element to an array field. If doesn not exist, it will create it
  ```
  { "$push": {"<field 1>": <value 1>, "<field 2>": <value 2>, ...} }
  ```
  <br/>

  ## Delete collections

  Atlas:
  Through garbage sign we can delete databases, collections and documents

  mongosh
  - Delete documents:
  ```
  deleteOne({"<field to match>": "<value to match>"})
  deleteMany({"<field to match>": "<value to match>"})
  ```
  <br/>

  - Delete collections:
  ```
  db.<collection>.drop()
  ```

  Example:
  ```
  db.inspections.find({"test": 3}).count()  // 3
  db.inspections.deleteMany({"test": 3})    // {"acknowledged": true, "deletedCount": 3}
  db.inspection.drop()                      // true
  ```
  <br/>

  ## Query operators
  There is another MQL operators beside the update operators, query operators. Provide additional ways to locate data within the database.

  \$
  - Precedes MQL operators
  - Precedes aggregation pipeline stages
  - Allow access to field values

  ## Comparison operators

  - $eq = equal to
  - $ne = not equal to
  - $gt > greater than
  - $lt < less than
  - $gte >= greater than or equal to
  - $lte <= less than or equal to

  Syntax: { <field>: { <operator>: <value> } }

  Atlas:
  ```
  filter: {"tripduration": {"$lte": 70}, "usertype": {"$ne": "Suscriber"}}
  ```
  <br/>

  mongosh:
  ```
  db.trips.find({"tripduration": {"$lte": 70}, "usertype": {"$ne": "Customer"}})
  ```
  <br/>

  ## Logic operators

  - $and: match all of the specified query clauses - This and this - all true
  - $or: at least one of the query clauses is matched - This or this - one true
  - $nor: fail to match both given clauses, exclude all clauses - Not this and not this - all false
  - $not: negates the query requirements - Not this - one false

  \$and, $or, $nor syntax:
  ```
  { <operator>: [{<statement 1>}, {<statement 2>}, ... ]} // array syntax neccesary
  ```
  <br/>

  \$not syntax:
  ```
  { $not : {<statement 1>} } // array syntax not neccesary
  ```
  <br/>

  \$nor example:
  ```
  { $nor: [{result: "No Violation Issued"}, {result: "Violation Issued"}] }
  // Every results except these ones
  ```
  <br/>

  ## Implicit $and:
  Is implicit by default in all queries (the commas)
  
  ```
  { sector: "Mobile Food Vendor - 881", result: "Warning" }
  { $and: [{ sector: "Mobile Food Vendor - 881" }, { result: "Warning" }] }

  { "$and": [{"student_id": {"$gt": 25}}, {"student_id": {"$lt": 100}}] }
  { "student_id": {"$gt": 25, "$lt": 100} }
  ```

  ## Explicit $and:
  Used when need to include the same operator more than once in a query
  
  ```
  db.routes.find({ "$and":
    [
        {"$or": [{"dst_airport": "KZN"}, {"src_airport": "KZN"}] },
        {"$or": [{"airplane": "CR2"}, {"airplane": "A81"}] }
    ]
  }).count() 
  // 18
  ```
  <br/>

  ## Expressive query operator
  \$expr can do more than a simple operation.
  Allows the use of aggregation expressions within the query language.
  Allow us to use variables and conditional statements.
  Compare fields within the same document to each other.

  Syntax:
  ```
  { $expr: { <expression> } }
  ```
  <br/>

  Example:
  ```
  filter: { "$expr": { "$eq": ["$start station id", "$end station id"] } }
  db.trips.find("$expr": { "$eq": ["$start station id", "$end station id"]).count() // 316
  ```
  <br/>

  \$
  - Denote the use of an operator
  - Addresses the field value (not a key / field name itself)

  Example:
  ```
  { 
    "$expr": {
        "$and": [
          { "$gt": ["$tripduration", 1200] },
          { "$eq": ["$end station id", "$start station id"] }
        ]
      }
  }
  ```
  <br/>

  ```
  // MQL Syntax: 
  { <field>: { <operator>: <value> }}

  // Aggregation Syntax: 
  { <operator>: { <field>, <value> }}
  ```
  <br/>

  ## Array operators
  \$push
  - to add an element to an array.
  - turns a field into an array field if it was previously a different type.

  \$all
  - to return all matches with a single element from an array. Returns a cursor
    with all documents in which the specified array field contains all the given
    elements regardless of their order in the array.
    ```
    { <array field> : { "$all": [<item 1>, <item 2>, ...] } }
    ```
    <br/>

  \$size
  - to limit the search by the array length. Returns a cursor with all documents
    where the specified array is exactly the given length.
    ```
    { <array field> : { "$size": <number> } }
    ```
    <br/>

  ## Accessing data in arrays

  Example
  ```
  Into sample_airbnb filter: { "amenities": "Shampoo" }
  // All matchs. There is no distinction that amenities is an array
  ```
  <br/>

  How can we indicate that amenities is an array?
  Into sample_airbnb filter: { "amenities": ["Shampoo"] }
  // Returns just amenities array, but with only one element "Shampoo". Looks for an exact match

  Does the order matters?
  Into sample_airbnb filter: { "amenities": ["Wifi", "Internet", "Kitchen", ...] }
  // Yes. If we turn the order the results will be different

  How do we find all results that match with a single element without worrying about the order?
  Into sample_airbnb filter: { "amenities": { "$all": ["Shampoo"] } }
  // It will return all matches with include Shampoo in the amenities array

  If we want to search arrays with less than 20 elements that include Shampoo
  Into sample_airbnb filter: { "amenities": { "$size": 20 }, { "$all": ["Shampoo"] } }

  Projection
  ══════════
  Allow us to choose which document fields will be part of the resulting cursor.

  Example
  ```
  db.listingAndReviews.find(
    {
      "amenities": {      // Search
        "$size": 20,
        "$all": [ "Internet", "Wifi", "Kitchen", "Heating", ... ]
        }
    },
    {
      "price": 1,         // Projection: fields included in the cursor result
      "address": 1
    }
  )
  .pretty()
  ```

  Syntax:
  1 - include field / 0 - exclude field
  Projections can not have mixed inclusions or exclusions
  ```
  db.\<collection>.find( { \<query> }, { \<field1>: 1, \<field2>: 1 } ) // Inclusion
  db.\<collection>.find( { \<query> }, { \<field1>: 0, \<field2>: 0 } ) // Exclusion
  db.\<collection>.find( { \<query> }, { \<field1>: 0, "_id": 0 } ) // Exception
  ```
  <br/>

  \$elemMatch
  Allow us to access elements in a subdocument of an array field.

  Example:
  We want to access just scores > score values.
  ```
  {
    "_id": ObjectId("0000000000000000000"),
    "student_id": 1,
    "scores": [
      {
        "type": "exam",
        "score": 89
      },
      {
        "type": "homework",
        "score": 92
      }
    ],
    "class_id": 265
  }
  ```
  <br/>

  We can use $elemMatch for projection:
  ```
  db.grades.find({"class_id": 265}, {"scores": {"$elemMatch": {"score": {"$gt": 85}}}})
  // We get all results. Only the objects where score greater than 85, will show fully. The others just will get the _id field.
  ```
  <br/>

  Or we can use it in the find:
  ```
  db.grades.find({"scores": {"$elemMatch": {"type": "extra credit"}}})
  // We get all the results where exist at least one element in scores array, with value "extra credit"
  ```
  <br/>

  ## Querying arrays and subdocuments
  How to get to the array fields in nested documents? MQL use dot notation, as deep as is needed.

  Example
  ```
  db.trips.findOne({"start station location.type": "Point"})
  // db level

  trips {                                     // collection level
    "id": ObjectId("000000000000000000"),
    "tripduration": 280,
    "start station id": 512,
    "start station location" {                // Object that stores a subdocument
       "type": "Point",                       // Subdocument level
       "coordinates": [
         -73.99,
         40.75
        ]
    }
  }
  ```
  <br/>

  Example
  All Zuckerbergs
  ```
  db.companies.find({ "relationships.0.person.last_name": "Zuckerberg" }, { "name": 1 })
  ↑ find ↑ projection
  ```
  <br/>

  All Marks that are CEOs
  ```
  db.companies.find({ "relationships.0.person.first_name": "Mark", "relationships.0.title": {"$regex": "CEO"} }, { "name": 1 })
  ↑ find ↑ match expression ↑ projection
  ```
  <br/>

  All seniors named Mark who no longer work at their company
  ```
  db.companies.find({ "relationships": {"$elemMatch": {"is_past": true, "person.first_name": "Mark"}} }, { "name": 1 }).count() // 256
  ```
  <br/>

  ## Aggregation Framework
  Another way to query data in mongoDB. Everything that is possible to do in MQL also it is possible in the aggregation framework. Allow us to do more complex data pipelines.

  ### MQL
  Allow us to filter and update data.

  ```
  db.listingAndReviews.find(
    {"amenities": "Wifi"},                               // Search
    {"price": 1, "address": 1, "\_id: 0"}                // Projection
  )
  ```
  <br/>

  ## Aggregation Framework
  Allow us to compute and reshape data.

  ```
  db.listingAndReviews.aggregate([                       // Allow us to do more than one query at once
    { $match: {"amenities": "Wifi"} },                   // Search
    { $project: {"price": 1, "address": 1, "_id: 0"} }   // Projection
  ])                                                     // Like every array, order is important to the pipeline
  ```
  <br/>

  listingAndReviews > $match > $project
  ↑ 1° filter ↑ 2° filter

  \$group
  Takes the incoming stream of data, and siphons it into multiple distinct reservoirs.
  Non-filtering stages do not modify the original data. Instead they work with data in the cursor.

  ```
  db.listingAndReviews.aggregate([
    { $project: {"address": 1, "_id: 0"} },
    { $group: {id_: "$address.country"} }                // Group by expression
  ])
  // 9 countries
  ```
  <br/>

  ## Sort and limit
  ```
  db.zips.find().sort({"pop": 1}).limit(1) // First result from cursor, asc order
  db.zips.find().sort({"pop": -1}).limit(1) // First result from cursor, desc order
  db.zips.find().sort({"pop": -1}).limit(10) // First ten results from cursor, desc order
  ```
  <br/>

  ## Cursor methods
  This not modify the data in the database. Apply to the results set in the cursor.

  - sort() → After the cursor is populated with the filter data from the find command,
  then applies the sort method, 1 increasing / -1 decreasing order
  - limit() → Apply after the sort
  - pretty() → Better reading
  - count() → Count resulting documents

  ## Indexes
  Make queries more efficient. In a DB is a special data structure that stores a small portion of the collection s data set in an easy to traverse form. Sorting is high consuming.

  Not use Index
  - Find all mentions of Toni Morrison
  - Look through every page in a book

  Using an Index
  - Find all mentions of Toni Morrison
  - Go to M in the Index
  - Locate Morrison
  - Go to referenced page

  Given
  1° Query - db.trips.find({"birth year": 1989})                            // Finds by birth year
  2° Query - db.trips.find({"start station id": 476}).sort("birth year": 1) // Sorts by birth year

  For 1° Query - Create Single Index
  db.trips.createIndex({"birth year": 1})                                   // This index order by birth year
  db.trips.find({"birth year": 1989})                                       // Now this query goes directly to 1989 docs

  For 2° Query - Create Compound Index
  db.trips.createIndex({"start station id": 1, "birth year": 1}) // This index first order by start station id, then by birth year
  db.trips.find({"start station id": 476}).sort("birth year": 1)

  ## Data Modeling
  Shape and structure for data. A way to organize fields in a document to support your application performance and querying capabilities.

  Rule:
  - Data is stored in the way that it is used.

  Main conditerations:
  - What we will store?
  - How it will be queried?
  - Who is using our app?

  Data that is used together should be stored together.
  Evolving app implies an evolving data model.

  ## Upsert
  Everything in MQL that is used to locate a document in a collection can also be used to modify this document.
  
  ```
  db.\<collection>.updateOne({\<query to locate>}, {\<update>})
  ```
  <br/>

  Upsert is a hybrid of update and insert, it should only be used when it is needed.
  Third option. By default is set to false.
  Is there a match? Then update the matched document.
  Is not there a match? Then insert a new document.

  ```
  db.\<collection>.updateOne({\<query>}, {\<update>}, {"upsert": true})
  ```
  <br/>