---
{"tags":["blogs"],"Author":"Qiu Weihong","creation date":"2023-09-20 22:28","modification date":"Wednesday 20th September 2023 22:28:32","publish":null,"topics":["Explore"],"dg-publish":true,"banner":"https://images.unsplash.com/photo-1555066931-4365d14bab8c?auto=format&fit=crop&q=80&w=3270&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D","permalink":"/blogs/explore/why-is-hashtable-o-1/","dgPassFrontmatter":true,"created":"2023-09-20T22:28:32.419+08:00","updated":"2023-10-31T23:10:09.451+08:00"}
---


# Why is Hashtable O(1)?

In computer science, a **hashtable** (also known as a hash map) is a data structure that provides fast access to values based on their associated keys. It allows for efficient storage and retrieval of key-value pairs.

One of the major advantages of a hashtable is its **constant time complexity** for basic operations, such as insertions, deletions, and lookups. This means that regardless of the number of elements stored in the hashtable, the time it takes to perform these operations remains constant.

The reason behind this efficiency lies in how hashtables are implemented. Behind the scenes, a hashtable uses an array to store the key-value pairs. When inserting or searching for an element, the hashtable applies a hash function to the key to determine its index in the array. This process is fast because it only requires a few computations.

Let's take a closer look at each operation and why they have constant time complexity:

## Insertion

When inserting an element into a hashtable, the key is first hashed using the hash function. The resulting hash code is then used as an index to store the value in the underlying array. Since accessing an array element by index has constant time complexity, insertion can be done quickly.

If there are no collisions (i.e., multiple keys mapping to the same index), insertion takes O(1) time on average. However, if there are collisions, additional steps are required to handle them properly.

## Lookup

To search for an element in a hashtable, the key is again hashed using the same hash function. The resulting hash code is used to locate the corresponding value in the array. Similar to insertion, this lookup process has constant time complexity since accessing an array element by index is efficient.

In case of collisions, where multiple keys map to the same index due to different hash codes but identical values after applying modulo arithmetic with respect to array size, the hashtable employs techniques like chaining or open addressing to resolve them. These techniques ensure that the time complexity remains constant even in the presence of collisions.

## Deletion

Deleting an element from a hashtable follows a similar process as lookup and insertion. The key is hashed to determine the index of the value in the array. Once again, this operation has constant time complexity because accessing an array element by index is efficient.

In summary, hashtables achieve a constant time complexity for basic operations due to their efficient hash-based indexing mechanism. However, it's important to note that certain factors