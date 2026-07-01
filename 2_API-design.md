## API Design

1. API
    - Interface to interact with the application

2. Can we divided into 3 groups
   - Public APIs : exposed to all
   - Private APIs: exposed only to internal organization users
   - Partner APIs: exposed to certain users (internal or external to organization)
     - like someone subscribes to our service, buy subscription

3. Good API design
   - Complete encapsulation of internal implementation
   - Easy to use, Easy to understand, Secure
   - Keeping the operation idempotent
   - API pagination
   - API versioning

4. **RPC**
   - RPC (Remote Procedure Call) is a concept in distributed systems that lets one program call a function on another machine as if it were a normal local function call.
   - But that function actually runs on another server somewhere on the network
   - And you don’t worry about HTTP, sockets, serialization, etc. — RPC handles all of that for you
   - Supports multiple programming language

5. How RPC works
   - Contract is defined in Interface description language
   - Then RPC tool generates Server stub and client stub from this contract
   - When client application hits any method on client side, the client stub takes the information and shares it with the server stub.
   - Then server stub calls the real method and return the result to the client stub
   - Drawback
     - Very slow
     - There is no real way for client to know if server received the message

6. When to use RPC
   - Used between 2 backend systems

7. **REST API**
   - Representational State transfer
   - Best for defining API for web