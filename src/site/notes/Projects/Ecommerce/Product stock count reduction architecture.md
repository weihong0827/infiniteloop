---
{"tags":["ecommerce","Backend","MQ"],"creation date":"2023-11-29 21:06","modification date":"Wednesday 29th November 2023 21:06:21","target date":null,"publish":null,"dg-publish":true,"permalink":"/projects/ecommerce/product-stock-count-reduction-architecture/","dgPassFrontmatter":true,"created":"2023-11-29T21:06:21.965+08:00","updated":"2023-11-30T11:20:46.072+08:00"}
---





# Introduction
We have previously discussed the [[Blogs/Backend Development Essentials/Distributed Transactions Concepts and Importance\|importance of distributed transaction]] and how we can use a framework such as [[Blogs/Backend Development Essentials/Introduction to seata\|Introduction to seata]] to effectively solved this challenge. However, solutions offered by seata uses a global distributed locking mechanism that requires acquiring and release of resource and machines waiting in idle while the resources are being held. This will result in the reduction in performance, hence solutions like seata is suitable in SMEs or services that does not face with huge traffic. For larger applications like requires a higher QPS need to opt for solutions like `message queue` for [[Study/50.041 Distributed Systems and Computing/Consistency Model#Eventual Consistency\|Eventual consistency]] 

# Core logic
In a standard ecommerce system, we will need a order placement service and a stock tracking service. We will also be connecting to a third party payment service to process the payment for a certain order. If payment is successful, we will fulfil the order and reduce the stock. However if it is not successful we will need to 'rollback' the stock. Sounds easy? lets take a look at the architecture diagram below.

![Product stock count reduction architecture 2023-11-30 10.23.44.excalidraw.png](/img/user/Projects/Ecommerce/attachments/Product%20stock%20count%20reduction%20architecture%202023-11-30%2010.23.44.excalidraw.png)


## Order section
The above chart outlines the entire flow of placing a order. Lets take a closer look at the top half that describes the order placement and how it interacts with the payment service 
1. The user place a order though the order endpoint
2. The order endpoint will then create a order record in the database 
3. The end point will create a `task` in the message queue with a `X` min of delay
4. It will request the payment service to receive payment for the current order. if the payment is made, a callback from the payment service will update the order status in the database
5. The user will be allowed to make the payment within `X` mins.
6. The `close order listener` will consume the `task` after `X` mins, 
	1. it will check against the database to see the current status of the order
	2. if the order is not canceled, it will query the payment status from the payment service
	3. if the payment is not fulfilled, mark the order as cancelled
## Stock Locking
Now lets move on to the bottom half of the chart.
1. The order endpoint will ask the stock endpoint to reduce the stock for the products that is in the order
2. The stock endpoint will increase the lock stock count in the database
3. It will send a `task` indicating this locked stock to the message queue.
4. After `X+N` mins, the task is consumed by the stock recovery service
5. the stock recovery service will query the order endpoint for the order status
	1. if order is fulfilled. release the lock and reduce the stock count
	2. if not release the locked stock
6. In case the message queue's task time out or the message queue having some fault resulting in the message not properly served. We will have a listener that periodically (At a much longer interval than `X+N`) scan the database for unattended lock records

>[!question] 
>Why do we use a MQ instead of just using a cron job or listening service to watch for the waiting states
>>[!answer]
>>This is the same concept as polling and webhook idea in HTTP.
>>For cronjob or a listener, you are constantly querying for changes in the database and hence hogging the compute resources
>>For a message queue, it only query after a predefined TTL




