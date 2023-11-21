---
{"tags":["blogs","Database"],"Author":"Qiu Weihong","creation date":"2023-11-12 20:18","modification date":"Sunday 12th November 2023 20:18:33","publish":null,"priority":null,"topics":["Backend Essential"],"banner":"https://infitniteloop.s3.ap-southeast-1.amazonaws.com/banner/db-transaction.png","dg-publish":true,"permalink":"/blogs/backend-development-essentials/a-guide-to-database-transaction/","dgPassFrontmatter":true,"created":"2023-11-12T20:18:33.000+08:00","updated":"2023-11-20T11:21:05.238+08:00"}
---


## Introduction

In backend development, managing data integrity and consistency is crucial, especially when multiple operations are involved. Spring Framework provides powerful transaction management capabilities, ensuring that a series of operations either succeed together or fail without affecting the system's stability. This post delves into the common transaction management strategies in Spring, highlighting their features and how they maintain the ACID properties - Atomicity, Consistency, Isolation, and Durability.

# What is transaction
A database transaction is a fundamental concept in the field of database systems. It refers to a sequence of operations performed as `a single logical unit of work`, ensuring data `integrity` and `consistency`. Transactions are crucial for maintaining the reliability of a database, especially in systems that handle multiple concurrent users or processes.
## Unit of work
A transaction is treated as a single, indivisible unit of work. This means that all the operations within a transaction either all succeed or all fail. There's no halfway: either the entire transaction is committed (applied to the database), or it is rolled back (undone from the database).

## Concurrency Control
n environments where `multiple transactions` occur simultaneously, databases use concurrency control mechanisms to ensure that transactions `do not interfere` with each other, preventing potential issues like data corruption, lost updates, or inconsistent reads.
## ACID
ACID is an acronym that stands for Atomicity, Consistency, Isolation, and Durability. It's a set of properties that guarantee that database transactions are processed reliably. In the context of database systems, a transaction refers to a single operation or a group of operations. Let's break down each component:

1. **Atomicity**: This property ensures that each transaction is treated as a single "unit", which either succeeds completely or fails completely. If any part of the transaction fails, the entire transaction fails and the database state is left unchanged.
    
2. **Consistency**: Consistency ensures that a transaction can only bring the database from one valid state to another, maintaining database invariants. This means that any data written to the database must be valid according to all defined rules, including constraints, cascades, and triggers.
    
3. **Isolation**: This property ensures that concurrent execution of transactions leaves the database in the same state that would have been obtained if the transactions were executed serially. In other words, transactions are isolated from each other, preventing them from interfering with each other's operations.
    
4. **Durability**: Durability guarantees that once a transaction has been committed, it will remain so, even in the event of power loss, crashes, or errors. In a relational database, for instance, once a group of SQL statements execute, the results need to be stored permanently (even if the database crashes immediately thereafter).
# Why is Transaction is needed
Transactions are essential in database operations for several reasons:

1. **Ensuring Data Integrity and Consistency**: Transactions help maintain a consistent state in the database, even in the face of errors, power failures, or other issues. By grouping related operations into a single transaction, you ensure that either all operations succeed or none do, maintaining data integrity.
    
2. **Handling Concurrent Access**: In systems where multiple users or processes access the database simultaneously, transactions help manage concurrent changes in a controlled manner, preventing data conflicts and inconsistencies.
    
3. **Recovery from Failures**: Transactions provide a mechanism to recover from system failures by rolling back ongoing transactions to their previous stable state.
## Before transaction
```java
LoginUser loginUser = LoginInterceptor.threadLocal.get();  
CouponDO couponDO = couponMapper.selectOne(new QueryWrapper<CouponDO>()  
        .eq("id", couponId)  
        .eq("category", category.name())  
);  
  
checkCoupon(couponDO, loginUser.getId());  
  
CouponRecordDO couponRecordDO = new CouponRecordDO();  
BeanUtils.copyProperties(couponDO, couponRecordDO);  
couponRecordDO.setCreateTime(new Date());  
couponRecordDO.setUseState(CouponUseStateEnum.NEW.name());  
couponRecordDO.setUserId(Long.valueOf(loginUser.getId()));  
couponRecordDO.setUserName(loginUser.getName());  
couponRecordDO.setId(null);  
couponRecordDO.setCouponId(couponId);  
  
int rows = couponMapper.reduceStock(couponId);  
int flag = 1/0 //<- simulate program failure
if (rows == 1) {  
    couponRecordMapper.insert(couponRecordDO);  
} else {  
    log.warn("Claim token failed:{}, user:{}", couponDO, loginUser);  
    throw new BizException(BizCodeEnum.COUPON_GET_FAIL);  
}
```
If the above code snippet is executed. when the program failed on the line `flag = 1/0` it will have already reduce the stock count but no record is written to the database. Hence the data is not `consistent`


## Understanding Spring Transaction Management

Spring offers two primary approaches to transaction management:

### 1. **Programmatic Transaction Management**

In this approach, transactions are managed programmatically. You explicitly call methods like `beginTransaction()`, `commit()`, and `rollback()`. This is achieved using `TransactionTemplate` but is less commonly used due to its manual nature.

```java
// Example of programmatic transaction management
TransactionTemplate transactionTemplate = new TransactionTemplate();

transactionTemplate.execute(status -> {
    // Business logic here
    // commit or rollback based on business logic outcome
});
```

### 2. **Declarative Transaction Management**

This is the more popular method in Spring, leveraging Aspect-Oriented Programming (AOP). Transactions are managed declaratively either through configuration files or annotations, making it cleaner and less intrusive.

```java
// Example of declarative transaction management
@Transactional(rollbackFor = Exception.class,propagation = Propagation.REQUIRED)
public void someTransactionalMethod() {
    // Business logic here
}
```

## The Essence of Declarative Transaction Management

Declarative transaction management in Spring works fundamentally by intercepting methods. Here's what happens:

- Before the target method starts, Spring creates or joins a transaction.
- After the method execution, Spring commits or rolls back the transaction based on the outcome.

## Transaction Propagation Behaviors

Spring allows you to specify how transactions should be propagated. Here are some common propagation behaviors:

- `@Transactional(propagation = Propagation.REQUIRED)`: Join an existing transaction or create a new one if none exists.
- `@Transactional(propagation = Propagation.NOT_SUPPORTED)`: Execute without a transaction.
- `@Transactional(propagation = Propagation.REQUIRES_NEW)`: Always create a new transaction, suspending any existing one.
- `@Transactional(propagation = Propagation.MANDATORY)`: Must be executed within an existing transaction, or an exception is thrown.
- `@Transactional(propagation = Propagation.NEVER)`: Must be executed without an existing transaction, or an exception is thrown.
- `@Transactional(propagation = Propagation.SUPPORTS)`: Use a transaction if one exists.
- `@Transactional(propagation = Propagation.NESTED)`: Execute within a nested transaction if a transaction exists.

## Transaction Isolation Levels

Transaction isolation levels determine how transactions are isolated from each other:

- `@Transactional(isolation = Isolation.READ_UNCOMMITTED)`: Allows reading uncommitted data.
- `@Transactional(isolation = Isolation.READ_COMMITTED)`: Allows reading only committed data.
- `@Transactional(isolation = Isolation.REPEATABLE_READ)`: Guarantees repeatable reads.
- `@Transactional(isolation = Isolation.SERIALIZABLE)`: Ensures complete isolation.

## Conclusion

Spring's transaction management, especially its declarative approach, offers a robust way to handle transactions in enterprise applications. By understanding the nuances of propagation behaviors and isolation levels, developers can ensure data integrity and consistency across multiple operations in a complex system.

---

As an accompaniment to this post, let's create a custom banner image that encapsulates the theme of Spring transaction management. The image will feature symbolic representations of the key concepts discussed, such as ACID properties and the Spring framework, in a visually appealing layout.