1. **Load balancer**
   - balances the traffic load in a group of servers

2. Benefit
   - Scalability - we can add as many servers as we require behind a load balancer
   - Availability - If a server stops working, load balancer moves the traffic to another server
   - Maintainability - Easy to add or remove servers

3. Types of Load balancer
   1. DNS load balancer
   2. Hardware load balancer
   3. Software load balancer
   4. Global server load balancer

4. **DNS**
   - Domain name system
   - When user enters the URL,
   - It goes to the DNS to resolve the IP
   - Usually DNS returns a list of IPs
   - All referring to different instance of the application
   - Client application usually picks the first IP
   - So DNS balances the load by rotating the IPs in that list

5. Disadvantages of DNS
   - It doesn't monitor the health of server. So even if a server is down, DNS will still share its IP
   - List only change based on TTL configured for that particular record.
   - Balance strategy is always simple round-robin
   - Client application gets IP address of all our servers

6. **Hardware load balancer and software load balancer**
   - Hardware load balancer runs of dedicated devices design for load balancing
   - Software load balancers are just programs that can run of any general purpose computer
   - Features
     - Exposes only the load balancer IP, not the instance IPs
     - Continuously monitors the health of the instances
     - Better balancing strategy based on different factors like
       - load on each instance
       - Type of instance
       - Number of open connections
     - It does not map IP address to human-readable URLs, for that we can use `Global server load balancer`

7. **Global server load balancer**
   - Hybrid between DNS and hardware/Software load balancer
   - It resolves the IP as DNS load balancer
   - It can detect the Geolocation of the user, so shares the IP of server that is closer to the user
   - It also monitors the health of the instances
   - Example Amazon route 53