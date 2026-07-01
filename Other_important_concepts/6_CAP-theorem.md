## CAP theorem

1. Consistency, Availability and Partition tolerance

2. What problem CAP solves
    - When we have a single database then life is easy, all the read and writes are from that database itself
    - But when we have multiple databases then we have to consider
        - synchronization of data
        - which database is master and how data will get synchronized
        - managing reads with write requests

3. Consistency
    - Every client sees the same data at the same time
    - After a successful write, all future reads should return the same data

4. Availability
    - Every request receives a response
    - The response may not be the latest data but system must respond

5. Partition tolerance
    - Network communication between database nodes breaks
    - DB1 ------ X ------- DB2
    - Servers are alive, but they cannot talk to each other
    - Can be caused due to network failure, cable issues, cloud outrage, router problems, etc.
    - Any real distributed system must tolerate partitions

6. Partition tolerance example
    - Let database nodes connection is broken
    - Client writes to the DB1 node (abc= 200)
    - nodes cannot communicate, DB2 has old value (abc=100)
    - Now when user reads from DB2, what shall we return 
    - We have 2 choices: Consistency or Availability
    - If we choose consistency then we blocks request as we don't have the latest data
    - If we choose Availability then we can return the old data


7. CAP says
    - When a network partitions occurs then you must choose between Consistency and Availability

8. Examples
    CP -> Bank account details, Apache Zookeeper
    AP -> Instagram likes, `AWS dynamoDB, Cassandra`



## Data consistency modal
    - Eventual consistency
        - When we write to a node then the data is not immediately replicated to all the other nodes
        - First we return success to the client for write operation
        - then eventually synchronize the other nodes
        - It is related to AP (Availabilty and partition tolerance)
    
    - Strong consistency
        - When we write to a node then all the future reads must return the latest value
        - It does not mean that all the time we have to replicate to all the nodes before returning the response to the client
        - Most commonly we use Quorem-Based consensus
            - Where we will return true to client if we replicate to majority of nodes
            - Read operation also requires a Quorem (if the majority of nodes are synchronized) then only it will return response to client for read operation
            - Example of strong consistancy
                - We have a primary node and 5 secondary nodes
                - When we write to primary node then 3 secondary nodes replicated the data and 2 failed to do so
                - Majority is sync so we return success to client
                - When a client request for the read operation then we ask majority of nodes let say 3 to return the response
                - then we compare the metadata of the response to check which one is latest
                - As majority of nodes got synced, So when we read from multiple nodes, it is guaranteed that atleast 1 will return the latest response
                - It is an expensive operation but better than all nodes sync before returning success for write operation.

    - coordinator
        - Usually in case of multiple nodes system, we have a coordinator
        - It receives the requests from client and sends the response back 
        - So in case of strong consistency, coordinator requests data from replica nodes and decides based on metadata which is the latest data
        - This coordinator pattern is present in Cassandra, DynamoDB, Elasticsearch, etc