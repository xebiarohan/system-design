## SQL vs NonSQL

1. SQL
   - Based on fixed schemas (very costly to change later)
   - Relationships between tables
   - ACID transactions
     - Atomicity: All or nothing
     - Consistent: All is available almost immediately after write
     - Isolation: Transaction works in isolation
     - Durability: Data remains safe, data is not lost
   - Used in applications where
     - Transactions are very important
     - Relationship is well-defined
     - Complex queries needed
     - Data integrity is needed
   - Drawbacks
     - Hard to scale Horizontally
     - Rigid schema (hard to change)
   - Usage example
     - Banking application
     - Airline application

2. NonSQL
    - There are several types of non-sql database
      - MongoDB for document based storage
      - Cassandra for wide-column based data
      - Redis for key-value
      - Neo4j, GraphQL for Graph based data
    - Advantage
      - Easy to scale horizontally
      - No schema required upfront
      - More flexible
      - Good for storing very large scale data 
      - Distributed system
    - Disadvantage
      - Not that consistent : data will not reflect immediately after write
      - Data duplicacy
      - No joins in data (no direct relation can be formed)
    - Usage example
      - Instagram
      - Twitter
      - Data mining application

3. Why scaling horizontally hard for MQL based database
   - In SQL based database we usually have 1 main node and remaining multiple replica nodes
   - To follow ACID transactions we cannot have more than 1 main node for write operations
   - In horizontal scaling we need multiple main nodes, that is not possible in SQL database
   - We can have reads from replicas
   - So we can have horizontal scaling but just for reads, not for writes

4. Indexing in SQL database
   - Avoid scanning the whole table, it kind of gives a shortcut to find data in tables
   - Index is a separate data structure that helps database find rows faster
   - Think of a book, without index you have to search each page to find something, with index we can go directly to a particular page
   - SQL index is a table of pointer to actual data
   - Let say we have a table of email addresses
   - If we create an index on this table, it will be like
```java
        email        pointer to row
        a@a.com       ROW - 1
        b@b.com       ROW - 2
        c@c.com       ROW - 5

```

5. Types of Index
   - Primary index: Automatically creates on primary key
   - Unique index: Ensures no duplicate values
   - Composite index: 
   - Secondary index: Index on non-primary column

6. Drawbacks of Indexing
    - When even we do INSERT/UPDATE/DELETE, we have to update these indexes as well
    - So these process becomes slow
    - Indexes takes extra space

7. Use index when
    - Lots of data in a table
    - Column is used in WHERE clause
    - Column us used in JOIN clause
    - Column is used in ORDER BY clause

8. How indexing works
   - Index data structure contains all the rows data with only 1 column and a pointer to row where it is present
   - These then get stored in B-Tree based on the data of column
   - In B-Tree, each node can have many children
   - data is stored based on the data of the column, on which we create an index

9. Types of Index
  Cluster index
    - The table data itself is stored in the order of the index
    - No separate index data required
  Non cluster index
    - Separate structure that stores (column value -> pointer to row)
  Composite index.
    - Index on multiple columns

10. ACID transactions
    - Atomicity - All or nothing, if something fails whole transaction roll back
    - Consistency -> transaction must move data from 1 valid state to another valid state only
    - Isolation -> All transactions must be isolated, must not interfere with each other
    - Durability -> Once something is committed, it should stay committed.

 

2. Data replication (Read replica/ write replica)
    - Data replication is a process of copying data from one database server to other database server
    - Used for improving fault tolerance, performance, reducing the latency
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
            - LWW -> last write wins (overrides the other one)
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