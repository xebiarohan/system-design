Sharding is one of the most important concepts in distributed systems because it answers a simple question:

> **"When one database server is no longer enough, how do we split the data across many servers?"**

Instead of buying a bigger machine forever (vertical scaling), we distribute the data among multiple machines.

Imagine you have a `users` table with **1 billion users**.

One database can't comfortably store and serve all of them.

Instead of this:

```
                Database
+--------------------------------+
| Users 1 ... 1,000,000,000      |
+--------------------------------+
```

you split it into several databases.

```
          Router / Application
                 |
     -----------------------------
     |            |             |
  Shard 1      Shard 2      Shard 3
 Users A       Users B      Users C
```

Each shard stores **only part of the data**.

---

# Why do we shard?

Suppose a single database can handle

* 10 TB storage
* 20,000 reads/sec
* 5,000 writes/sec

Now your application grows.

```
Users: 5 million
```

No problem.

Then:

```
Users: 500 million
```

Now the database starts becoming overloaded.

Problems:

* Storage fills up
* CPU maxes out
* Memory becomes insufficient
* Too many concurrent requests

Instead of one huge database:

```
DB1
```

You create

```
DB1
DB2
DB3
DB4
```

Now every server handles only part of the workload.

---

# What is a shard?

A shard is simply

> **one partition of the total data.**

For example:

```
Shard 1
--------
Users:
1
2
3
4

Shard 2
--------
Users:
5
6
7
8
```

Together they form one logical database.

Applications usually don't care where the data lives.

---

# The biggest question

How do we decide

> **Which user goes to which shard?**

This is where sharding strategies come in.

There are several ways.

---

# Strategy 1: Hash-Based Sharding

This is probably the most common strategy.

Idea:

Take some key

```
UserID
```

Run a hash function on it.

Then choose the shard.

Example:

```
Shard = hash(UserID) % NumberOfShards
```

Suppose we have

```
4 shards
```

Then

```
User 15

hash(15)=8392

8392 % 4 = 0

→ Shard 0
```

Another user

```
User 98

hash(98)=120391

120391 % 4 = 3

→ Shard 3
```

---

Example

```
Users

1
2
3
4
5
6
7
8
9
10
```

Suppose

```
hash(UserID)%3
```

Results

```
Shard 0

3
6
9

Shard 1

1
4
7
10

Shard 2

2
5
8
```

Notice

The users are spread almost evenly.

That is the goal.

---

## Why hashing is good

It distributes data evenly.

Instead of

```
Shard1

1 million users

Shard2

100 users
```

You get

```
Shard1

330k

Shard2

335k

Shard3

335k
```

Balanced load.

---

## Advantages

Very even distribution.

Good for

* user accounts
* sessions
* shopping carts

Anything accessed by ID.

---

## Disadvantages

Imagine we had

```
4 shards
```

Formula

```
hash(id)%4
```

Now we add another server.

Now

```
hash(id)%5
```

Everything changes.

Example

Before

```
15 %4 =3
```

After

```
15 %5 =0
```

Almost every record belongs somewhere else.

That means

* copying billions of rows
* downtime
* expensive migration

This is called **resharding**.

---

## Solution: Consistent Hashing

Instead of modulo arithmetic,

servers are arranged in a ring.

```
           Shard A

      /               \

Shard D               Shard B

      \              /

         Shard C
```

Every key is hashed onto the ring.

Each key belongs to the next server clockwise.

When a new server is added

```
Shard E
```

only nearby keys move.

Instead of moving

```
100%
```

of the data,

maybe only

```
20%
```

moves.

This is why systems like distributed caches (e.g., Redis clusters, CDNs) often use consistent hashing.

---

# Strategy 2: Range-Based Sharding

Instead of hashing,

split data by value ranges.

Example

```
UserID

1-1000000

→ Shard1

1000001-2000000

→ Shard2

2000001-3000000

→ Shard3
```

Simple.

---

Example

```
Orders

OrderID

1-9999

Shard1

10000-19999

Shard2

20000-29999

Shard3
```

Easy.

---

Advantages

Very easy to understand.

Queries like

```
Find all users

between

1000

and

5000
```

touch only one shard.

Excellent for range queries.

---

Disadvantages

Imagine new users always get increasing IDs.

```
Newest IDs

999999999
1000000000
1000000001
```

They all go to

```
Last shard
```

Now

```
Shard3

CPU 95%

Shard1

CPU 10%
```

One shard becomes a hotspot.

---

Real example

Suppose Twitter stored tweets by TweetID range.

Newest tweets would all hit the last database.

Bad.

---

# Strategy 3: Geographic Sharding

Very common in global companies.

Split by location.

```
USA users

→ US database

Europe users

→ EU database

Asia users

→ Asia database
```

Example

```
Paris user

Europe DB

Tokyo user

Asia DB

New York

US DB
```

Advantages

Low latency.

Legal compliance (e.g., data residency requirements).

Traffic is naturally distributed.

---

Disadvantages

What if a user moves?

```
India

↓

Canada
```

Should their data move?

That becomes a migration problem.

---

# Strategy 4: Directory-Based Sharding

Instead of computing the shard,

look it up.

Example

```
User123

↓

Lookup Service

↓

Shard7
```

There is a mapping table.

```
User1 -> Shard4

User2 -> Shard9

User3 -> Shard1
```

---

Advantages

Very flexible.

You can move one user

without moving everyone.

Large companies often use this approach because it supports rebalancing with minimal disruption.

---

Disadvantages

Need another service.

If the directory goes down,

everything breaks unless it is highly available.

---

# Comparison

| Strategy        | Best For                 | Pros                             | Cons                                                 |
| --------------- | ------------------------ | -------------------------------- | ---------------------------------------------------- |
| Hash-based      | Even distribution        | Balanced load                    | Hard to add/remove shards without consistent hashing |
| Range-based     | Sequential/range queries | Simple, efficient for ranges     | Hotspots with uneven growth                          |
| Geographic      | Global systems           | Low latency, regional compliance | Cross-region moves and queries are harder            |
| Directory-based | Very large systems       | Flexible, easy to rebalance      | Extra lookup service and complexity                  |

---

# Cross-Shard Queries

One challenge with sharding is that not all queries stay within a single shard.

Suppose data is sharded by `UserID`, and you run:

```sql
SELECT * FROM users WHERE country = 'France';
```

The application doesn't know which shard contains French users, so it must query **every shard**:

```
          Query
            |
     -----------------
     |    |    |    |
   S1    S2   S3   S4
     \    |    |   /
      Aggregate Results
```

This is called a **scatter-gather query**. It is slower because every shard participates.

---

# Rebalancing

As your system grows, some shards will inevitably become larger or busier than others.

Rebalancing is the process of redistributing data to achieve a more even load.

Example:

```
Before

Shard A : 800 GB
Shard B : 150 GB
Shard C : 200 GB
```

After rebalancing:

```
Shard A : 380 GB
Shard B : 390 GB
Shard C : 380 GB
```

Good sharding strategies and tools aim to make rebalancing as inexpensive as possible.

---

# Choosing a Strategy

There is no universally "best" sharding strategy. The choice depends on your application's access patterns:

* **Hash-based sharding**: Best when requests are mostly lookups by a unique key (e.g., user profiles, shopping carts).
* **Range-based sharding**: Best when range scans are common (e.g., time-series data or sequential order IDs).
* **Geographic sharding**: Best for globally distributed applications where users are primarily served from their own region.
* **Directory-based sharding**: Best for very large systems that need fine-grained control over data placement and online rebalancing.

The key takeaway is that **the shard key (the field you partition on) is the most important design decision**. A poor shard key can create hotspots, require expensive migrations, and make common queries inefficient, while a good shard key evenly distributes data and keeps most requests confined to a single shard.
