---
{"tags":["blogs"],"Author":"Qiu Weihong","creation date":"2023-11-18 16:28","modification date":"Saturday 18th November 2023 16:28:15","publish":null,"priority":null,"topics":["Backend Essential"],"banner":null,"dg-publish":true,"permalink":"/blogs/backend-development-essentials/distributed-transactions-concepts-and-importance/","dgPassFrontmatter":true,"created":"2023-11-18T16:28:15.000+08:00","updated":"2023-11-20T23:17:04.000+08:00"}
---

# Introduction
Anyone that has a little bit understand about database will know the concept of a [[Blogs/Backend Development Essentials/A guide to database transaction\|A guide to database transaction]]. To simply put, it is to make sure that a series of database transaction is commit or rollback together base on the execution status of the function to ensure the ACID properties of the database.

However, when we are building large scale application or adopting a `microservices` architecture, you will have multiple machines (nodes) running different services and connecting to different database.

![Distributed Transactions Concepts and Importance 2023-11-20 11.31.32.excalidraw.png](/img/user/Blogs/Backend%20Development%20Essentials/attachments/Distributed%20Transactions%20Concepts%20and%20Importance%202023-11-20%2011.31.32.excalidraw.png)


Sometimes, there needs to be interactions between different services which will carry out a dedicated action in that service's own database. But the problem comes when let say, one of the service `failed`, it performed a `rollback` on its own database actions, but the other service that is called using `rpc` is not aware of this failure and **still continues with the database action instead of rollback**. Therefore violating the `Consistency` of the data

The goal of distributed transaction is such that all operation under the same `gloabltransaction` should be all succeeded or all failed and rollback

# BASE Properties
In order to understand distributed transaction, it is important for us to understand some of the basic properties of [[Study/50.041 Distributed Systems and Computing/Consistency Model\|Consistency Model]].

BASE theory is a framework proposed by architects at eBay to address the limitations of achieving strong consistency in distributed systems, as highlighted in the [[Blogs/Backend Development Essentials/The CAP Theroem\|cap]] theorem. The CAP theorem posits a trade-off between `Consistency` and `Availability` in distributed systems. 
BASE theory suggests a balanced approach, focusing on achieving [[Study/50.041 Distributed Systems and Computing/Consistency Model#Eventual Consistency\|eventual consistency]] tailored to the specific needs of an application. 

## Basically Available (BA)
This principle implies that a system remains `operational` even during unforeseen failures, albeit with **potential performance or functionality degradation**. For instance, a system's response time might increase from 10ms to 50ms but remains functional.

## Soft state
BASE allows for a 'soft state' in systems, where data can exist in `intermediate` states without significantly affecting the system's overall availability. It accepts the fact that data replication across different nodes might have latency.

## Eventual Consistency
BASE ensures that, barring any new updates, data across the system will eventually become consistent. This means all clients will ultimately access the most up-to-date data.

# Common solutions for distributed transaction
- **2PC and 3PC**: Two-phase and three-phase commit protocols based on the XA protocol.
- **TCC (Try, Confirm, Cancel)**: A transactional pattern involving trial, confirmation, and cancellation stages.
- **Transactional Messaging**: An approach using "best effort" notifications.
## Types of distributed transaction
- **Rigid Transactions**: Adhere to the [[Blogs/Backend Development Essentials/A guide to database transaction#ACID\|ACID]] properties.
- **Flexible Transactions**: Follow BASE theory.
## Common frameworks
- **TX-LCN**: Supports multiple patterns including 2PC, TCC, etc. [GitHub](https://github.com/codingapi/tx-lcn). It seems to have slow updates.
- **Seata**: Supports AT, TCC, SAGA, and XA patterns. Backed by Alibaba with a dedicated team and commercial product GTS. [GitHub](https://github.com/seata/seata) | [Aliyun GTS](https://www.aliyun.com/aliware/txc)
- **RocketMQ**: Integrates transactional messaging for distributed transactions. [GitHub](https://github.com/apache/rocketmq)
- **Distributed Transaction Manager(DTM)**: It provides saga, tcc, xa, 2-phase message, outbox, workflow patterns for a variety of application scenarios.[Github](https://github.com/dtm-labs/dtm)

# X/Open DTP Transaction Model

- **Definition**: The X/Open DTP model, defined by the X/Open organization, is a set of standards and APIs for distributed transactions. It's a framework followed by various vendors for implementation.
- **Purpose**: It aims to facilitate distributed transaction processing, ensuring that transactions across different systems are handled consistently and reliably.
# XA Protocol
- **Origin**: The XA protocol is a part of the X/Open DTP model, proposed by the X/Open organization.
- **Functionality**: It primarily defines the `interface` between a Transaction Manager (TM) and Resource Managers (RMs).
- **Implementation**: Major database products have implemented the XA interface. It acts as a two-way system interface between the TM and multiple RMs, serving as a communication bridge.

## Java Transaction API (JTA)
- **Overview**: JTA is Java's implementation of the XA specifications for transaction processing.
- **Role**: It provides a standard for handling transactions in Java, integrating the principles laid out by the XA protocol.

## Components of the X/Open DTP Model
- **AP (Application)**: This represents the application layer, including business logic, microservices, etc.
- **RM (Resource Manager)**: Typically a database, but it can also include other types of resource managers like message queues or file systems.
- **TM (Transaction Manager)**: Acts as the transaction coordinator, responsible for receiving XA transaction commands from the AP and coordinating all RMs involved in the transaction to ensure its proper completion.
![Distributed Transactions Concepts and Importance 2023-11-20 15.18.45.excalidraw.png](/img/user/Blogs/Backend%20Development%20Essentials/attachments/Distributed%20Transactions%20Concepts%20and%20Importance%202023-11-20%2015.18.45.excalidraw.png)


# Transaction Model in Distributed Systems
- **Individual Node Awareness**: Each node in a distributed system knows the outcome of its transaction operations but can't directly ascertain the results of transactions on other nodes.
- **Need for Coordination**: To maintain the ACID properties of transactions across multiple distributed nodes, a "coordinator" (TM) is introduced.
- **Role of TM**: The TM orchestrates the actions of distributed nodes (APs) and ultimately decides whether these APs should commit the transactions to their respective RMs.
# XA Protocol
## Two-Phase Commit Protocol (2PC)
The XA protocol uses 2PC for coordinating multiple resources in a global transaction, supported by MySQL version 5.5 and above.
![Distributed Transactions Concepts and Importance 2023-11-20 20.57.24.excalidraw.png](/img/user/Blogs/Backend%20Development%20Essentials/attachments/Distributed%20Transactions%20Concepts%20and%20Importance%202023-11-20%2020.57.24.excalidraw.png)

### Preparation Phase
Transaction managers(TM) send a 'Prepared' message to each participant, who then execute the transaction locally and write `Undo/Redo` logs, without committing the transaction yet.
	- _Undo Logs_: Record pre-modification data for potential `rollbacks`.
	- _Redo Logs_: Record post-modification data for `committing` the transaction.
###  Commit Phase
- On receiving `failure` or `timeout` messages from participants, the transaction manager instructs a rollback; otherwise, it sends a commit message.
- Participants follow these instructions and release any locks held during the transaction.
- Locks are released only in this final stage.

## Characteristics and Limitations
- XA protocol is simple and minimally invasive to business processes.
- It's transparent to users, who can handle distributed transactions as they would local transactions, ensuring strict adherence to ACID properties.
- Known as a rigid transaction (adheres to ACID), it's **not ideal for high-concurrency** scenarios due to extensive lock usage and suboptimal performance.
- Better supported by commercial databases; MySQL's support is still evolving.
- XA also includes other protocols like 3PC (Three-Phase Commit), which addresses some 2PC limitations but has its own drawbacks, such as increased network communication.
# TCC (Try-Confirm-Cancel) Transactions

Divides transaction into serveral stages: 
1. **Try** (business checks and resource reservation)
2. **Confirm**: Commit the changes, by default, confirm will succeed once try succeed since the resources are reserved
3. **Cancel**: Business logic fails and perform rollback. Release the resources
![Distributed Transactions Concepts and Importance 2023-11-20 21.06.49.excalidraw.png](/img/user/Blogs/Backend%20Development%20Essentials/attachments/Distributed%20Transactions%20Concepts%20and%20Importance%202023-11-20%2021.06.49.excalidraw.png)


| Operation | Meaning                                |
| --------- | -------------------------------------- |
| Try       | Reserve resources and data verfication |
| Confirm   | confirm execution and commit data      |
| cancel    | roll back data                                       |
## Advantages 
- Offers greater control over transaction lock strength and avoids resource blockages.
## Challenges:
- Requires significant changes to business logic, as one operation now needs three methods (Try, Confirm, Cancel) to support the two-stage commit.
- It depends a lot on the business logic and hence bad reusablility

Ensures idempotence for each operation to handle potential retries or network issues.

# Transactional Messaging
Uses message queues to provide a mechanism similar to XA for achieving eventual consistency in distributed transactions.
![Distributed Transactions Concepts and Importance 2023-11-20 21.43.54.excalidraw.png](/img/user/Blogs/Backend%20Development%20Essentials/attachments/Distributed%20Transactions%20Concepts%20and%20Importance%202023-11-20%2021.43.54.excalidraw.png)

## Transaction message type
- **Half Transaction Messages**: Messages that are `sent` but not yet `confirmed` for delivery.
- **Message Callback**: In case of network issues or producer restarts, the message server may ask the producer to confirm the final state (Commit or Rollback) of the message.
## Advantage
- Facilitates decoupling between applications and ensures data consistency eventually.
- Splits large transactions into smaller ones, enhancing efficiency and system availability.
- One related application failure and result in entire transaction rollback and hence ensure core availability to the greatest extend
## Disadvantage
- Realtime consistency is not guaranteed

# Conclusion
In the realm of distributed systems, there are multiple solutions to handle distributed transactions, including the XA's Two-Phase Commit (2PC), TCC (Try-Confirm-Cancel), and transactional messaging using MQ. Frameworks like Seata provide support for these various modes, offering a range of options for different scenarios.

However, it's crucial to approach the implementation of these solutions with caution and careful consideration. Except for specific scenarios requiring strong data consistency, it's advisable to avoid using distributed transactions whenever possible. This is because, regardless of how performance-optimized these solutions might be, the introduction of distributed transactions into a project can significantly reduce overall efficiency. This drawback becomes particularly apparent in high-concurrency environments.

For operations involving multiple links, alternative strategies or perspectives should be sought to circumvent the need for distributed transactions. Examples include changing the approach to locking stock during order placement or modifying how discount coupons are recorded. These alternatives can often achieve the desired outcome without resorting to the complexities of distributed transactions.

In summary, both distributed transactions and distributed locks should be used sparingly. If there's a need to use them, the preference should be towards flexible (soft) transactions, resorting to rigid (hard) transactions only when absolutely necessary. Similarly, when using distributed locks, it's advisable to minimize the granularity of the locks. This cautious approach ensures that the system remains efficient and scalable while still meeting the essential requirements of data consistency and integrity.