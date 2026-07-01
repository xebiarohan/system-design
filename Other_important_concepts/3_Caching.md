## Caching

1. Storing frequent used data in a faster accessible place

    - Like storing it in Browser cache, or application level cache instead of database

2. Why Cache is fast ?

    - Cache systems like Redis and Memcached shores data in RAM (memory) instead of disk

    - RAM is extremely fast as compare to disk based databases

3. Benefits

    - Faster response

    - Decreases load on database

    - Lower infrastructure cost (fewer database servers needed)

4. Flow of request

    - Each layer can cache data

```bash

    Browser -> CDN -> Server -> Database

```

5. Browser Cache

    - images

    - CSS

    - Fonts

    - sometimes API responses

    - Javascript

6. How browser cache works

    - Server sends the header in the response

        - Cache-control :  max-age: 3600

    - Solution of stale data

        - Cache busting

            - When then data of a file change, change the name of the file like app-123.js can be changed with app-345.js

            - Modern build systems like webpack, Vite rewrites all the references of that file

        - Cache controlled headers

            - add the max-age of cache in a header

        - TTL

How to solve the problem of Stale data in Browser ?

7. CDN Cache

    - Content delivery network

    - We can cache data in CDN

    - Imaging the server is in US and we are opening website in India

    - Without CDN, the latency will be high

    - But CDN can cache some frequenctly used data to decrease the latency

    - Example of CDN : Amazon CloudFront, CloudFlare, etc

    - Caches data like images, videos, CSS, JS, downloadable files, etc

    - Real world exaple : A viral video on youtube

    - CDN are localted around the world, they are called edge servers or edge locations

8. Cache invalidation

    - TTL

    - purge cache in case data gets updated

9. Server Cache

    - Stores data between application server and database

    - Usually implemented using Redis/Memcached

    - Eviction policies

        - LFU - least frequently used

        - TTL - time to live

        - LRU -Least recently used

10. Cache Strategies

    - We need to keep the cache and database synchronized

    - Different strategies to do that

        - Cache Aside

        - Write through cache

        - Write-Back

 

11. Cache aside (lazy loading)

    - On get first check if data is available in cache then return from there

    - If not available, then fetch it from DB and add it in cache and return it to client

    - In case of POST or PUT, invalidate the cache, to clear stale data

    - Best for read heavy systems

 

12. Write through cache

    - Idea is that every write goes through Cache first and then to Database

    - App writes to cache -> Cache writes to database

    - Cache and database always synchronized

    - Best for Frequently read update data

 

13. Write back

    - App writes to the cache

    - Cache immidiately responds success

    - Then database is updated asynchronously

    - Excellent for heavy write systems

    - Disadvantage : If cache crashes before database flush

    - Complex logic, need retry and queues