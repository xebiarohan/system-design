## Quality Attributes

1. It includes : Performance, security, testability, scalability, deployability, maintainability, etc

2. **Performance**
   - Response time
     - Client sending a request and receiving a response.
     - Response time = processing time + network time
   - Throughput
     - Amount of work performed by our system per unit of time
       - Example amount of data processed per seconds (bytes/sec, MBytes/sec, etc)

3. Considerations for measuring Response time
   - It is not just the processing time, we have to consider other factors like network latency, waiting time
   - For example our application can handle only 1 request at a time, and we got 2 request simultaneously. So, the second request has to wait for the time until first finishes.
   - We describe the response time in terms of percentile
     - Examples : 50% of request is under 100ms, 80% of request under 150ms, 99% of the request under 200ms
     - if we have to keep it under 150ms, anything above that is called tail latency.
     - We have to set goals like 95% of request to be under 50ms (we cannot commit for 100% requests)

4. **Scalability**
   - Load on the system never stays the same. So to increase the resources when we have high load and decreasing resources when we have low load.
   - Degradation point - It is the point where if we increase more load the performance degrades

5. Types of scalability
   - Vertical scalability
     - Improving the current resources like using faster CPU, more memory, better network card, better dababase, etc.
   - Horizontal scalability
     - Adding more instance of resources instead of upgrading the current ones.
   - Team/Organization scalability
     - like Adding more engineers to the team
     - It can increase productivity at a certain level after that it decreases it.

6. **Availability**
   - The probability that an application/service is functional and accessibly to the user.
   - Uptime - when application is available
   - Downtime - when application is not available
   - Availability is measure in percentage (uptime/application running time)
   - Other matrix to measure
     - MTBF - Mean time between failures (average time system is operational)
     - MTTR - Mean time to recover (time taken by system to detect and recover from a failure)

7. **Fault Tolerance & High Availability**
   - Fault tolerance enables our system to remain operational and available to the users despite failures
   - Reasons of faults
     - Human error
       - Pushing a faulty configuration
       - Running the wrong script
       - Deploying without proper testing
     - Software error
       - Long garbage collection
       - Out of memory issue
       - Null pointer exception
     - Hardware failures
       - Server/ router/ storage device breaking down
       - Power outrage
       - Natural disaster

8. Fault tolerance revolves around 3 tactics
   - Failure prevention
   - Failure detection and isolation
   - Failure recovery

9. Failure prevention
    - Eliminating any single point of failure example Not running whole application on a single server
    - Strategy of redundancy and replication
      - Active-active architecture : like multiple database are active at the same time
      - Active-passing architecture : One replica is main and other one is waiting state

10. Failure Detection
    - If any one of the instance fails then we must be able to detect it and replace it with a new one
    - Should not have any impact on other instance
    - We usually use monitoring service to check the health of each instance
    - Other way is to check the number of errors each host get per minute, if the number is high then we can replace that instance with a new one.
    - Other way is to check the time taken by each instance to respond to a request.

11. Failure recovery
    - Taking action after finding the fault in the system like detecting a faulty instance
    - Like stop sending traffic to that host, then try to restart it
    - If the problem goes then send traffic again else replace with a new instance
    - Another example : database rollback to previous version

12. SLA
    - Service level agreement
    - It is an agreement between service provider and the client.
    - It is a legal contract between parties related to quality service
      - Availability
      - Performance
      - Data durability
      - Time to respond 
      - and so on

13. SLO
    - Service level objectives
    - Individual goals that we set of our system
    - Each SLO represents a target value that our system needs to meet
    - Example 
      - Availability service objective of 3 nines (99.9%)
      - Response time objectives of less than 100 ms for all types of requests
      - Issue resolution objective of 24-48 hours
    - Even if we don't have SLA, we must have SLO because client relies on this document for what to expect

14. SLI
    - Service level indicator
    - Quantitative measures of our compliance with a service level objective
    - It measures what we promised and what is the current value
    - Example response time, down-time, etc., etc.
