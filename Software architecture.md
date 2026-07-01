## Software architecture

1. It impacts
   - Performance and scalability of product
   - Ease of adding new features
   - Response to failure and security attacks

2. Software architecture of a system is a high level blueprint of the system's structure, its components and 
   how those components communicate with each other

3. Software architecture can have different level of abstraction
   - Classes/structs
   - Modules/packages/libraries
   - Services (processes/ group of processes)

4. Software development Cycle
   - Design --> Implementation --> Testing  --> Deployment
   - Software architecture is the output of the Design phase and the input to the implementation phase

5. Challenges of Software architecture
   - We cannot prove Software architecture to be either 
     - correct
     - optimal
  
6. Requirements
   - When ever we get a requirement, first get as much information as we can get before starting system design
   - Like if we are creating an app for Hitchhiking, we need to ask client
     - Real time or Advance booking
     - Payment method in application or Direct payment
     - Any platform fees or not
     - Mobile application or Desktop application

7. Types of requirements
   - Features of the system
     - functional requirements - Features of the system
   - Quality attributes
     - Non-functional requirements - Example Scalability, Availability, Reliability, security and performance
   - System constraints
     - limitations and boundaries like Budget, time, staffing constraints

8. Methods of gathering functional requirements
   - Use cases
     - Situation/Scenario in which our application can be used
   - User flows
     - Step-by-step or a graphical representation of each use case
   - Identify users
     - who all can access the application and in which way

9. Example of Hitchhiking application
   1.  Rider and Driver are 2 actors (users)
   2.  Rider first time registration, Driver registration, Rider login, etc. are user flows
   3.  We can have a sequence diagram for each user flow

10. Quality requirements
    1.  Quality attributes must be
         - measurable (Good -page change should be done in less than 1 second, Bad - page change should be very quick)
         - Testable
    2.  Trade off - Sometimes the requirements are contradictory, we have to do trade off
         - example Performance - user must login in less than 1 second. Security - user must enter credentials every time
    3.  Feasibility
         - We have to check is it even possible what the client is asking
         - Example Let the data center is in America and we are in Australia, then low latency requirement is not feasible

11. System constraints
    1.  Technical constraints
         - Being locked to a particular cloud vendor
         - Having to use a particular programming language
         - Having to use particular database
         - Having to support a particular browser, etc
    2.  Business constraints
         - Budget
         - Deadline
    3.  Legal constraints
         - in critical applications like defense or medical
         - who can access the data, not all can see data of patients
         - There can be country specific legal constraints

12. Read at the end : https://www.udemy.com/course/software-architecture-design-of-modern-large-scale-systems/learn/lecture/51849711#overview