Written by *Martin Kleppmann*
1st. Ed. 2018
O'Reilly Media

Table of Contents
* [1 Foundations of Data Systems](#1-foundations-of-data-systems)
  * [Reliable Scalable Maintainable](#reliable-scalable-maintainable)
  * [Data Models and Query Languages](#data-models-and-query-languages)

# 1 Foundations of Data Systems

## Reliable Scalable Maintainable

An application is data intensive if data is the primary challenge, as opposed to compute intensive applications, where CPU cycles are the bottleneck.

This book focusses on the architecture of data systems and how they get integrated. 

Reliability: Tolerating hardware and software faults and human error

Scalability: Measureing the load and performance, the latency percentiles, and the throughput

Maintainability: Operabiltity, simplicity, evolvability

Most applications need to do some combination of:
* Storing data (databases)
* Remembering the results of expensive operations to speed up reads (cache)
* Allowing users to search the data (search indexes)
* Sending messages to other processes (stream processing)
* Periodically crunching large amounts of data (batch processing)

### Thinking About Data Systems

When you combine multiple tools together in order to provide a service, the API hides the implementation details, meaning that you have hidden the special-purpose data system from the smaller components, making a whole data system. The three aspects to this are the reliability, the scalability, and the maintainability. 

### Reliability

Reliability tends to mean "continuing to work correctly even when things go wrong". If something goes wrong there is a fault, and a system that anticipates this is fault-tolerant/resilient. A faut is not a failure, and is where one component deviates from spec, rather than a stoppage in the whole system. This is partly through testing those faults regularly. 

Hardware faults get solved by:
* Adding redundancy to individual hardware components such as dual-power supplies and swappable CPUs
* Adding in tolerance for the machines being entirely lost, where the fault-tolerance is in the software, rather than the hardware

Software errors are systematic errors in the system, such as a software bug or a runaway process, and is particularly an issue with cascading failures. These bugs can be dormant for a long time, until something odd happens, showing up assumptions that the software is making about its environment. To solve these, we can work on thinking through these assumptions, conduct testing, isolate processes, allow restarts, and analyse production system behaviour. It is important to have some guarantees in the system that can be regularly checked.

Human errors can be modulated through a combination of:
* Designing systems in such a way that minimises the opportunities from error through abstractions, APIs, and admin interfaces. These shouldn't be so restrictive that people have to work around them.
* Decoupling where people make mistakes from where they cause failures, such as by making sandbox environments.
* Testing thoroughly at all levels from unit tests to whole-system integration testing
* Allowing for quick recovery from human errors, such as being able to roll back configuration changes and to roll out new code gradually
* Setting up detailed, clear monitoring, such as performance metrics and error rates
* Implementing good management practices and training

### Scalability

This concerns how a system is able to cope with an increased load. Load can be defined by load parameters, depending on the architecture of the system and can either concern the bottleneck or the average.

Performance can be asessed by:
* Looking at how when you increase a load parameter with the system resource constant, there is an effect on the performance of the system
* When you increase a load parameter, how much do you need to increase the resources to keep the performance unchanged

Response time is the actual client facing time, including the time to process the request, the network delays and the queuing delays, wheras the latency is just the duration of time where the request is waiting to be handled.

We need to think about the distribution of performance over time and under different conditions, rather than as a single number.

The mean is not a good metric of performance, because this doesn't tell you about the outliars, so it is good to use percentiles, median values, etc. Tail latencies (the high percentile latencies) are particularly important because these are often the most important use cases, but sometimes it can all get quite marginal. 

We can set service level objectives and service level agreements that define the expected performance and availability of a service and form a contract between the service and its users.

Queueing delays are often a large amount of response time at high percentiles as servers are limited in their parallel processing, so a couple of slow requests hold everyone up (head-of-line-blocking). When you are testing performance, you need to keep on submitting requests regardless of response time or it clears the queue.

To cope with load you need to start thinking about how that changes with order of magnitude differences in scale. There is a distinction between:
* Scaling Up (moving to a more powerful machine)
* Scaling Out (distributing the load among more machines, also called shared-nothing architectures)

An elastic system automatically adds computing resources as the load increases, wheras some systems are scaled manually. Manually scaled systems can be simpler and have fewer operational surprises. 

Moving static, single node systems into a distributed setup can introduce a lot of complexxity, so often databases were scaled up rather than out, but there are recent movess to make that more of a scaling out procedure as distributed data systems improve.

### Maintainability

A majority of the cost of software is not its initial development, but its ongoing maintenance such as: fixing bugs; keeping systems operational; investigating failures; adapting to new platforms; modifying for new use cases; repaying tech debt; and adding new features.

Three design principles:
* Operability: operations teams should keep the system running smoothly
* Simplicity: making it easy for new engineers to understand the system
* Evolvability: making it easy for engineers to make changes to the system in the future in an adaptable way for unanticipated use cases (also called extensibility, modifiability or plasticity)

#### Operability

Operations teams should be responsible for the following:
* Monitoringthe health of the syustem and restoring the service when it goes into a bad state
* Tracking down the cause of problems
* Keeping software and platforms up to date (incl. security)
* Keeping tabs on how systems interact with each other
* Anticipating future problems
* Establishing good practices and tools
* Performing complex maintenance
* Maintaining the security of the system
* Defining processes that make operations predictable
* Preserving organisational knowledge

Good operability makes routine tasks easy by:
* Providing visibility into runtime behaviour
* Providing good support for automation and integration
* Avoiding dependency on individual machines
* Providing good documentation and an operational model
* Providing good default behaviour and the ability to override these
* Self-healing and manual control
* Making the system predictable

A *big ball of mud* is a software project that is mired in complexity and is a cause of budget and schedule overunning. A large part of this is removing accidental complexity, which is not inherent to the problem that the software solves, but is instead from the implementation.

One key way of doing this is abstraction, where we hide implementation detail (e.g. with high-level programming languages like SQL). 

Agilie organisation practices can provide a way of adapting to change and the agility of a data system can be thought of as its evolvability.

There is a distinction between an applications functional requirements (what it should do) and nonfunctional requirements (security, maintainability etc).

## 2 Data Models and Query Languages

We build a program by layering data models on top of one another, and we need to know about how it is represented in the lower level, such as the APIs built upon APIs. In general, "each layer hides the complexity of the layers below it by providing a clean data model". This chapter focusses on general-purpose data models for storing and querying data.

### Relational vs document models.

In SQL, data is organised into relations (tables) where each relation is an unordered collection of tuples (rows). These came out of business data processing on mainframes in the 60s and 70s, such as transaction and batch processing. Alternatives in the 70s and 80s include the network model and the hierarchical model, and then in the late 80s and early 90s there were object databases. There was an emergence of XML databases in the 2000s too. 

The most recent shift is towards NoSQL in the 2010s for "open source, distributed, nonrelational databases". The push for this has been through:
* Needing more scalability than relational databases
* Preference for FOSS over commecial products
* Specialised queries that are unsuported by relationals
* Frustration with restrictive relational schemas

As application development is usually done in object-oriented programming languages, there has to be a translation layer between objects in application code and the database, which is called the impedence mismatch. There are frameworks fro this like ActiveRecord and Hibernate. 

An example of wher ethere are issues are in one-to-many relationships, which can be represented in one of the following three ways:
* Prior to SQL:1999, there were seperate tables for each sort of information with foreign key references
* Later there were structured datatypes and XML data which means that multi-valued data can be stored in a row
* You could also just encode in JSON or XML and then store it in the txt column of the database but you can't query inside that

JSON is relatively simple and has a lack of a schema, but this can have issues still with impedence. It has a better locality than mutli-table schema as it is all in one place, and can represent tree-like structures.

It is good to have standardised lists for data such as industries or geographic regions because:
* There is consistent style and spelling
* It makes it unambiguous
* It is easy to update
* There is better localisation support
* There is better search as it can encode further data

"Anything that is meaningful to humans may need to change sometime in the future". Removing duplication of the places where you need to change data is the idea behind normalisation, where "if you're duplicating values that could be stored in just one place, the schema is not normalised".

The above requires many-to-one relationships, however, which doesn't fit into a document model (like JSON) as joins are difficult. The join might instead have to be emulated by doing multiple queries.

In the 1970s, IBM had the Information Management System, which is a hierarcical model similar to JSON where all of the data is a tree of records nested within records. This is good for one-to-many but not many-to-many relationships and is not able to do joins, meaning they had to work out whether they had to duplicate / denormalise data or manually resolve references from records.The two solutions were the relational and the network model.

The network model was formalised by the Conference on Data Systems Languages (CODASYL) which generalises the hierarchical model so that a record could have multiple parent records. The links between these records were more like pointers than foreign keys and you had to follow a path from a root called an access path. This could get very complicated and you had to move through the database by iterating over lists of records and this meant that changing the application's data model was really hard.

This is compared to relational models which do not have any nested structures or complicated access paths. To do a new query you just declare a new index, making it easier to add features. 

There is an element of document databases which are hierarchical, where there are nested records within a parent record, but they are connected by document references (basically foreign keys). 

Arguments for document data models:
* Schema flexibility
* Performance due to locality
* They can be closer to the data structures of the application itself

Relational models have the benefits of:
* Better support for joins
* Many-to-one and many-to-many relationships

If you have a document-like structure in your application, where there is a tree of one-to-many relationships and you load the entire tree at once, a document model is easier, and the technique of *shredding* a document structure into multiple tables can be unneceserily schematic and messy. However, document models have the issues of not being able to refer directly to nested items, and there is poor support for joins. 

JSON does not enforce a schema and XML has it optional, which does mean that arbitrary keys and values can be added. 

Changing schemas in a relational database can be slow and require downtime, especially in MySQL (using `ALTER TABLE`), and running `UPDATE` is almost always slow. 

There is a distinction between databases that have a schema-on-read and a schema-on-write. Generally, where all records have the same structure, it is useful to have a schema.

There cna be a benefit to having the storage be as local as possible (i.e. in a document model), but this is usually only the case if you need large parts of the document at once. It is obviously wasteful to load everything if you only need a part of it. 

Most relational systems other than MySQL support XML which includes making modifications and querying them and there is similar for JSON documents. Document databases also have started supporting relational-like joins and there is generally an increase in hybridisation.

### Query Languages for Data

SQL is a declarative query language, whilst IMS and CODASYL was imperative. Imperative languages tell a computer to perform operations in an order, stepping through the code line by line, whilst declarative query languages specify the pattern of the data you want without stating exactly how to achieve that, with the optimiser working it out. Declarative code hides the implementation details, which means that performance improvements do not require changes to the queries. SQL is more limited in functionality, which means there is more room for automatic operations. Declarative languages are also more able to be used for parallel execution because it does not require the order to be specified. 

MapReduce querying was popularised by Google and is neither declarative nor fully imperatiev, where the logic of the query is expressed with snippets of code, and is based on the `map`/`collect` and `reduce`/`fold`/`inject` functions within functional programming languages. These are pure fucntions, only using the data that is passed to them as input, which means that they cannot perform additional database queries. MapReduce is fairly low level, so you can run SQL on top of it. You do have to write two coordinated JavaScript functions when you use MapReduce and it would be useful to run a declarative query language, which is why MondoDB 2.2 added the aggregation pipeline, which is as expressive as SQL but uses JSON-based syntax, meaning that the NoSQL system ends up almost reinventing SQL.

### Graph-Like Data Models

If there are very comon many-to-many relationships in the data, then it is useful to map the data as a graph.

A graph has *verticies* (or nodes / entities) and *edges* (or relationships / arcs). This means that you can map between verticies via edges. Note that you do not need to use homogeneous data, but can instead store many different types of objects in one store. 

#### Property Graphs 

One model of this is the *property graph*. 

Here, each vertex has:
* A unique identifier
* A set of outgoing edges
* A set of incoming edges
* A collection of properties as key-value pairs

Each edge has:
* The vertext identifier
* The vertex where the edge starts (tail)
* The vertex where the edge ends (head)
* A label to describe the relationsip
* A collection of properties as key-value pairs

The graph store has two relational tables, one for vertices and one for edges.

Important aspects of the model:
* Any vertex can have an edge connecting it with any other vertex
* Given any vertex, you can find its incoming and outgoing edges and therefore traverse the graph
* You can store multiple different kinds of relationships and kinds of information in a single graph

The *Cypher* query language was developed for property graphs, which allows us to trace our way through the graph. 

We can also, on the other hand, put graph data into a relational structure and query it with SQL, but as you don't know in advance what joins you need to make, this can be more challenging. In SQL this is called recusive common table expressions.

### Triple-Store and SPARQL

This is a similar model to the property graph model, where the data is stored in three-part statements: subject, predicate, object. The subject is equivalent to a vertex in a graph. The object is either a primitive datatype like a string or a number (the predicate being the key), or another vertex in the graph (the predicate being the edge, the subject the tail vertex and the object the head vertex). 

there has been lots of writing about the semantic web, which is the idea that as websites publish information as text and pictures, they should also publish machine-readable information in the form of the Resource Description Framework, which would allow all of the web to be combined into a database of everything. This has not been realised. The RDF can be expressed in the Turtle language. 

SPARQL is a query language for triple-stores using the RDF data model, and is similar in structure to Cypher. RDF doesn't distinguish between properties and edges, but instead uses predicates for both. 

Graph databases are not like network models because:
* In CODASYL there was a database schema which noted which record type could be nested in another
* CODASYL required you to traverse to a particular record through the access pathes, but in graph databases you can refer to them directly
* CODASYL hd childrens in an ordered set
* CODASYL had imperative queries rather than declarative languages

### Datalog

Datalog was the original model, which was similar to the triple-store, but instead of (subject, predicate, object) it had predicate(subject, object). Datalog defines rules that tell the database about new predictes derived from the data, which can refer to other rules like functions. 

### Summary

Data models started as one big tree hierarchical models which moved to relational models. From there two NoSQL approaches have been tried, document databases and graph databases, neither of which enforce a schema. 

## 3 Storage and Retrieval

This is how we can store the data that we are given and then retrieve it when we need it. There is a distinction between optimising for transactional workloads and for analytics. 

The two major storage engine types are log-structured and page-oriented storage engines.

The most basic databse is a simple key-value store where new values are appended to the database (which is like the log data file, where an application is simple an append-only sequence of records). The cost of a lookup in an append-only table is directly proportional to the size of the database. To reduce this we use indices, of which there are many types. By keeping additional metadata on the side we have a signpost to where the data is. Indices are additional structures derived from the primary data, and the more indices you use, the slower the write is (as that also has to be updated). Well chosen indexes speed up read queries but every index slows down writes. 

The simplest possible strategy is to keep an in-memory hash map where every key is mapped to the byte offset in the data file, the location. This is very fast as long as all the keys fit in the RAM, so if you have a large number of writes per key this is fast. 

Compaction is where you throw away duplicate keys in the log and only keep the most recent update for each key. This can be done in a background thread and use the old segment files while that is happening. There are some considerations for doing this:
* File format. This is best done in a binary format that encodes the length of a string in bytes and is followed by the raw string
* Deleting records. When you delete a key and its associates value you have to add a deletion record called a tombstone which tells the merging process to discard previous values for the deleted key
* Crash recovery. When the database is restarted, the in-memory hash maps are lost, and whilst you cab restore the segment's hash map by reading the segment file, this can make server restarts expensive, so Bitcask stores a snapshot on disk
* Partially written records. If there is a crash during the appending process, there needs to be checksums to make sure the corrupted parts are detected and ignored
* Concurrency control. As the writes are appended in order, usually there is only one writer thread

Append-only seems wasteful in that you could replace the old values directly, but appending / segment merging (being sequential writes) are a lot faster than random writes, and concurrency / crash recovery are simpler where the segment files are immutable, and merging the old segments means you don't have over-time fragmentation. However the limitations are that the hash table has to fit in memory and range queries are pretty inefficient (in that you have to look up each key in turn rather than run through the sequence directly).

If we order our keys we have a Sorted String Table (SSTable). The requirement here is that each key only appears once in each merged segment file through compaction. When we merge these segments it is simple and efficient as you go through each input file, copy the lowest key and move to the next file, and as the segments were created sequentially, you can just take the value of the most recent segment. You also do not have to keep the index in memory as you can use other keys to find the target key - this means the in-memory index can be sparse. Read requests have to scan over several key-pairs anyway, so you can group those records in a block and compress them. The sorting is usually done by maintaining the sorted structure in memory using a tree structure like red-black trees / AVL trees. This makes the workflow as follows:
* When the write comes in, add it to the in-memory balanced tree data structure (the memtable)
* When the memtable gets bigger than a threshold write it out to an SSTable file on disk as the most recent segment
* When you are serving a read request you try to find the key in the memtable and then the last segment, the segment before etc
* In the background run merging and compaction to combine segments and discard overwritten / deleted values

The problem with this comes in crashes where the most recent writes not written out to disk are lost, meaning that we need to keep a seperate log on disk, unsorted, which has every write appended. Examples of these algorithms are LevelDB and RocksDB. 

This indexing structure was called Log-Structured Merge-Trees (LSM-Trees). 

The performance optimisations for this come from the following places:
* The use of Bloom filters which approximate the contents of a set to look up whether the key is somewhere in the database, to avoid searching everything for a nonexistent key
* The use of size-tiered or leveled compaction which either merges newer / smaller tables into older / larger tables, or splitting up the key range into seperate levels, both of which use less disk space

The most common indexing structure is called the B-tree. These date back to 1970 and are used in most relationnal databases. These keep key-value pairs sorted by key but have a different design philosophy. Instead os having variable-sized, megabyte or greater indexes, the B-trees break the database down into 4KB blocks / pages and read/write one page at a time. The page is identified using an address / location which allows the pages to refer to one another, stored on disk. This makes a tree of pages, with one page being the root of the B-tree which has its own child pages, each with a continuous range of keys. The page with the individual keys are called leaf trees and the number of references to child pages of a B-tree is its branching factor. When you update the value you go to the leaf page, change it and then split any pages into half-pages and update the parent pages. This is a balanced tree, where *n* keys has a depth of *O(log n)* depth. A four-level tree of 4KB pages with a branching factor of 500 can store 256TB. 

A write operation of a B-tree is to overwrite the page on disk with the new data and to keep all of the references to that page. If there is a crash whilst you are splitting a page and trying to update the parent page, you have a corrupted index where there is an orphan page with no parent, so there is a write-ahead log append-only file that you have to write to before you can apply a modification, which means you can restore to that state. Careful concurrency control is required if you are accessing the database concurrently. which is done with latches.

We can optimise B-trees by:
* Instead of using a write-ahead log, using a copy-on-write scheme where the modified page is written to a different location and the new version of the parent page is created that points to it
* NNot storing the entire tyree but abbreviating it, such as by just showing the boundaries between key ranges. This means you can have a larger branching factor and therefore fewer levels.
* Positioning the pages on disk in such a way that the leaf pages are roughly sequential
* Adding more pointers to the tree, such as references to the left and right sibling pages
* Using variants such as fractal trees, which borrow some log-structured ideas

LSM-trees are generally faster for writes and B-trees are faster for reads.

