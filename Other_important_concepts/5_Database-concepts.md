1. ACID transactions

    - Atomicity - All or nothing, if something fails whole transaction roll back
    - Consistency -> transaction must move data from 1 valid state to another valid state only
    - Isolation -> All transactions must be isolated, must not interfere with each other
    - Durability -> Once something is committed, it should stay committed.

 

2. Data replication (Read replica/ write replica)

    - Data replication is a process of copying data from one database server to other database server
    - Used for improving fault tolerance, performace, reducing the latency
    - We can have different read replicas and write replicas
    - Usually we have 1 node for write, all write request goes to it and there can be several read replicas
    - Replication can be Synchronous or Asynchronous
    - In Synchronous
        - primary database waits for the write-in all the replicas then considers successful transaction
        - If the replication fails then whole transaction rollback
        - Writes are little slower in this case
        - But data is up-to-date

    - In asynchronous
        - Primary database does not wait for replication
        - writes are faster
        - But data can be old in replicas

    - Multi-leader replication
        - We can have more than 1 leader to write the data
        - Can we decided based on Geo location, traffic, etc
        - All the replicas can sync synchronously with each other
        - Can use different conflict resolution algo
            - LWW - > last write wins (overrides the other one)
            - Application level -> merge both and let application decide


3. Sharding

    - Process to split the data of a database on multiple machines known as shards
    - Instead of keeping everything in 1 database, we keep in different instances
    - Example a big table in a database can be split into tables in multiple machines
    - When we want to insert or get some data, we have to first select the shard using the shard key
    - Common sharding strategy
        - Range based sharding
             1 to 10,000 records -> Shard 1
             10,000 to 20,000    -> Shard 2
        - Hash based sharding
            - Apply hashing on record and decide the shard
        - Geo location based sharding
            - All data from 1 location goes to 1 shard

    - Data fetching
        - We can have application level logic
        - Built-in database router

    - Challenges
        - Complexity - harder to maintain
        - Rebalancing
        - Poor shard key - uneven load


-------------------------

4. Relation database
   - Schema is defined in advance for each table and relation between the tables
   - we can use robust query language for querying the database like SQL
   - Advantage
     - Ability to perform complex and flexible queries
     - Efficient storage (using joins, no need to data duplicity between tables)
     - Data in very readable format
     - ACID transactions guaranteed
   - Disadvantage
     - Rigid table structure, it has to be defined before time
     - Changing schema is a costly action
     - We need to know our data in advance
     - More costly to maintain and to scale
     - Slower read operations

5. Non-Relational database
   - Sometimes we don't have all the common data between different records.
   - Different records can have different data which makes it difficult to add it in a relational database
   - So non-relational database allow grouping a set of records which do not have same structure
   - Where we can easily add new attributes or remove attributes from the records.
   - Tables are easier for human readability, but it is difficult for programming language to traverse
     - most programming language don't have tables as its data type whereas we have arrays, maps, lists as data types.
   - Non-relational database are designed for faster queries (read)
   - Disadvantages
     - Finding relation between records become hard
     - ACID transactions are rarely supported
   - Category of non-relational database
     - key/value store: example Redis
     - Document store: example Cassandra, MongoDB
     - Graph database (extension of document): example Amazon Nepture, NEO4J

6. Improving performance
   - Indexing
        - Used for speed up the retrieval operations
        - A database table is a helper table that we can create with the help of a column or a group of columns
        - `Index` table contains column/columns that we selected and the row number
        - It can also be used for non-relational database like for document based databases
        - Indexing tradeoff
          - Additional space required for Index table
          - Speed of write operations decreases (as we need to update index table as well)
   - Database replication
     - Running multiple instances of database to increase fault tolerance
     - If one goes down, other one becomes the leader   
     - Trade off
       - Higher conplexity
   - Database sharding
     - Splitting the data in different instances
     - Trade off
       - Managing which Shard to query
       - Handling equal data between shards
       - Sharding is mainly for the non-relational database as we can have a query that will cover records from different shards, so won't be easy to implement

7. Unstructured data
   - A Data that does not follow a structure or schema
   - Example images, videos, documents, etc
   - We can store them in
     - DFS : Distributed file system
     - Object store

8. Distributed file system
   - It simulates an external file system connected with our system
   - like hard drive
   - But in DFS we have multiple servers working behind the scene that provides replication, fault tolerance, strong consistency
   - We don't need any special API for accessing them
   - Easy to modify, it is present in folder structure like in any file system
   - Limitations
     - Number of files that we can create
     - No easy access to those files through web API

9. Object store
    - Designed for unstructured data at internet scale
    - Can be scale easily just like DFS by adding more devices/servers behind the scene (linear scalability)
    - No limit on the number of object that can be stored in a store
    - Provides the HTTP + REST API
    - Supports versioning out of the box
    - Objects are not stored in a file structure but inside containers called as buckets (all at same level)
    - Example S3 in AWS, Azure Blob
    - Limitations
      - Objects are immutable, we need to replace the object with new version
      - No easy file system like access, need to access using REST API