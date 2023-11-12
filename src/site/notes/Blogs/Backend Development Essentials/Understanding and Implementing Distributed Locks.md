---
{"tags":["blogs"],"Author":"Qiu Weihong","creation date":"2023-11-12 15:41","modification date":"Sunday 12th November 2023 15:41:23","publish":null,"priority":null,"topics":["Backend Essential"],"banner":"https://infitniteloop.s3.ap-southeast-1.amazonaws.com/banner/distributed+lock.png","dg-publish":true,"permalink":"/blogs/backend-development-essentials/understanding-and-implementing-distributed-locks/","dgPassFrontmatter":true,"created":"2023-11-12T15:41:23.574+08:00","updated":"2023-11-12T20:00:59.742+08:00"}
---



Distributed locks play a crucial role in ensuring data consistency in distributed systems. In this post, we will explore the core concepts of distributed locks, their importance, and how to implement them with an emphasis on Redis-based locks. 

# 1. Why Distributed Locks?

In scenarios like preventing over-redemption of coupons in a distributed application, ensuring that a piece of code is executed by only one thread at a time across all servers is vital. This is where [[Study/50.041 Distributed Systems and Computing/Distributed Mutual Exclusion\|Distributed Mutual Exclusion]] come in.

# 2. Types of Locks

- **Local Locks**: These include synchronization mechanisms like `synchronize` and `lock` in Java. They are effective within a single process but fail to provide a solution in a distributed environment.
- **Distributed Locks**: These are implemented using technologies like Redis, ZooKeeper, or even databases like MySQL. They provide a locking mechanism that is shared across multiple processes.

# 3. Key Considerations in Designing Distributed Locks

- **Exclusivity**: Only one thread on one machine should be able to execute a specific piece of code at a time.
- **Fault Tolerance**: The lock should be released in case of client crashes or network interruptions.
- **Performance and Availability**: The lock should be efficient and always available.
- **Lock Granularity and Overhead**: Careful consideration is needed to decide the granularity and the associated overhead of the lock.

# 4. Implementing Distributed Locks

Redis often emerges as the best performer for implementing distributed locks due to its simplicity and efficiency.

- **Key-Value Setting**: The lock in Redis is associated with a key-value pair. 
   - Example: For a product's flash sale, the key could be named `seckill_productID`. The value could be a fixed number or a unique identifier.

- **Redis Commands for Lock Management**:
  - **Acquiring a Lock**: Using `SETNX` (SET if Not Exists) which is an atomic operation.
  - **Releasing a Lock**: Using `DEL` to delete the key.
  - **Setting Lock Expiry**: Using `EXPIRE` to ensure the lock is not held indefinitely in case of issues.

- **Pseudo Code Example**:
  ```java
  methodA() {
      String key = "coupon_66";
      if (setnx(key, 1) == 1) {
          expire(key, 30, TimeUnit.MILLISECONDS);
          try {
              // Business logic
          } finally {
              del(key);
          }
      } else {
          // Retry after a short delay
      }
  }
  ```

# 5. Potential Issues and Solutions

- **Non-atomic Operations**: The sequence of `setnx` and `expire` can lead to deadlocks if not handled properly. The solution is to use atomic commands like `SET key 1 EX 30 NX`.
- **Lock Deletion by Wrong Thread**: To prevent a thread from deleting a lock acquired by another thread, use unique identifiers for each lock.
	- If lock expiration is set to 30s and code execution took 40s
	- after lock expires, another thread took the lock
	- code execution finish, delete the lock that does not belong to that program
![](Understanding%20and%20Implementing%20Distributed%20Locks%202023-11-12%2015.49.23.excalidraw.svg)


- **Further Refinement Against Accidental Deletion**: Even with unique identifiers, race conditions can occur. Ensuring atomicity in the check-and-delete operation is crucial.

# Redisson and Redis Distributed Locks

The above method to implement a distributed lock using redis is too much hassle, well redis got us covered. The have offical packages to help us with distributed lock integration into our system just like our native lock. The [documentation](https://redis.io/docs/manual/patterns/distributed-locks/)explains the different usages and the different packages that they have for different languages.

I will be showing a simple [Redisson](https://github.com/mrniko/redisson) setup and usage demonstration.
Redisson provides an advanced and feature-rich implementation of distributed locks using Redis. It simplifies the process of implementing and managing distributed locks, abstracting the complexity behind a straightforward API.

## Key Advantages

- **Automatic Renewal**: Redisson can automatically renew the lock's expiration while it's held.
- **Fair Locking**: Queueing lock requests to prevent starvation.
- **Watchdog**: Monitoring and extending lock expiration if needed.

## Implementing a Distributed Lock with Redisson

Here's a step-by-step example of how to use Redisson to implement a distributed lock in a Java application.

### Step 1: Add Redisson Dependency

First, include Redisson in your project. If you're using Maven, add this to your `pom.xml`:

```xml
<dependency>
    <groupId>org.redisson</groupId>
    <artifactId>redisson</artifactId>
    <version>LATEST_VERSION</version>
</dependency>

```

### Step 2: Configure Redisson

Next, configure Redisson to connect to your Redis server:

```java
Config config = new Config();
config.useSingleServer().setAddress("redis://127.0.0.1:6379"); // you can also use multiple server
RedissonClient redisson = Redisson.create(config);

```
### Step 3: Implement the Distributed Lock

Now, let's use Redisson's `RLock` object, which represents a reentrant lock:


```java
RLock lock = redisson.getLock("myLock");
try {
    // Acquire the lock
    lock.lock();

    // Your business logic here, e.g., updating a shared resource

} finally {
    // Release the lock
    lock.unlock();
}

```

This example demonstrates acquiring a lock, executing some operations, and then releasing the lock.


### Strategies to Solve Redis Lock Expiration Issues

When implementing distributed locks with Redis, ensuring that the lock does not expire during lengthy business operations is a critical challenge. Redisson offers two main strategies to address this issue: the **Watch Dog mechanism** and **Specified Lock Time**. Let's delve into these two methods in detail.

---

#### 1. Watch Dog Mechanism

The Watch Dog mechanism is an inbuilt feature of Redisson designed to prevent the lock from expiring while business logic is being executed.

##### How It Works:

- Once a Redisson client successfully acquires a lock, it starts a watch dog thread.
- This thread `periodically` checks the status of the lock.
- If the client still holds the lock, the watch dog automatically extends the lock's expiration time, preventing it from expiring before the completion of business operations.

##### Features:

- **Automatic Extension**: If the business processing time is extended, the watch dog continually prolongs the lock’s expiration time.
- **Default Timeout**: By default, the watch dog checks for timeout every 30 seconds, but this can be adjusted by modifying `Config.lockWatchdogTimeout`.

```java
// Example code showing basic usage of the watch dog
RLock lock = redisson.getLock("myLock");
try {
    lock.lock();
    // Perform business logic
} finally {
    lock.unlock();
}
// Explicit extension is not needed here, the watch dog handles it automatically
```

#### 2. Specifying Lock Time

In addition to the watch dog mechanism, Redisson also allows you to specify the lock's lifetime directly when acquiring the lock.

##### How to Use:

- **Directly Specify the Lock's Expiry Time**: You can set the lock's duration at the time of locking, after which it will automatically be released.
- **Applicable Scenarios**: This method is highly effective when you can estimate the business execution time.

##### Example Code:

```java
// Example 1: Auto-unlock after 10 seconds
lock.lock(10, TimeUnit.SECONDS);

// Example 2: Attempt to lock, waiting up to 100 seconds, auto-unlock after 10 seconds
if (lock.tryLock(100, 10, TimeUnit.SECONDS)) {
    try {
        // Execute business logic
    } finally {
        lock.unlock();
    }
}
```


#### Conclusion

Implementing distributed locks requires a careful balance between ensuring data consistency and maintaining system performance. Redis offers an efficient and relatively straightforward way to achieve this, making it a popular choice in distributed systems.

