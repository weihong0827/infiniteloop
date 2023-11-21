---
{"tags":["distributedSystems","Project"],"creation date":"2023-09-20 11:00","modification date":"Wednesday 20th September 2023 11:00:09","target date":null,"publish":true,"dg-publish":true,"permalink":"/projects/dynamo-clone/introduction/","dgPassFrontmatter":true,"created":"2023-09-20T11:00:09.000+08:00","updated":"2023-10-31T00:10:29.000+08:00"}
---

# Dynamo
![[Dynamo.pdf]]

# Introduction
This series studies the design and implementation of amazon's `dynamo db`, and we will try to replicate a more simplified version using `go`

The design and implementation of `dynamo db` is targeted at applications with a very specific use case - those only need a simple `primary key interface` with `high availability`. The design consideration that goes behind dynamo is that they want to make sure that they database is still accessible and writable even during node addition or node failure. 
To achieve `high availability`, Dynamo `sacrifices consistency` under certain failure scenarios
Dynamo is targeted mainly at applications that need an `"always writeable"` data store where `no updates are rejected` due to failures or concurrent writes

# Design considerations
Amazon first used this as a internal service for some of their use cases on their e-commerce site and they built dynamo with these services that they are serving
# System Architecture
Here is the table with the problems Dynamo is solving:

| Problem                            | Technique                                              | Advantage                                                                                |
| ---------------------------------- | ------------------------------------------------------ | ---------------------------------------------------------------------------------------- |
| Partitioning                       | Consistent Hashing                                     | Incremental Scalability, High Availability for writes                                    |
| Versioning                         | Vector clocks with reconciliation during reads         | Version size is decoupled from update rates                                              |
| Handling temporary failures        | Sloppy Quorum and hinted handoff                       | Provides high availability and durability guarantee when some replicas are not available |
| Recovering from permanent failures | Anti-entropy using Merkle trees                        | Synchronizes divergent replicas in the background                                        |
| Membership and failure detection   | Gossip-based membership protocol and failure detection | Preserves symmetry and avoids centralized registry for membership and liveness info      |

# Implementations
Dynamo’s local persistence component allows for different storage engines to be plugged in. Engines that are in use are Berkeley Database (BDB) Transactional Data Store2, BDB Java Edition, MySQL, and an in-memory buffer with persistent backing store.

I will be splitting the entire implementation plan into the different sections below
- We will need to [[Projects/Dynamo Clone/Partitioning\|partition]] our distributed systems into different sections and each machine handles a dedicated section of the system using [[Projects/Dynamo Clone/Consistent Hashing\|Consistent Hashing]]
- How do we [[Projects/Dynamo Clone/Interfacing\|interface]] with the database system and the different considerations behind every read and write operations
- Making sure that we [[Projects/Dynamo Clone/Replication\|replicate]] our data to different nodes in the distributed systems, so that in any case of failure, we know that our data is safe
- Since there are different copies of the same data in our system, we need to handle the different [[Projects/Dynamo Clone/Data versioning\|versions]] of data carefully
- One of the most important thing to consider in a distributed system is [[Projects/Dynamo Clone/Failure handling\|Failure handling]]. What do we do when one or a few of the nodes failed.


