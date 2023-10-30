---
{"dg-publish":true,"permalink":"/projects/dynamo-clone/consistent-hashing/","tags":["distributedSystems","Project"],"created":"2023-09-24T14:20:26.188+08:00","updated":"2023-10-31T00:10:14.739+08:00"}
---

# Consistent Hashing
In Dynamo's partitioning scheme, consistent hashing is used to determine which node is responsible for each partition. Consistent hashing assigns each node and partition a unique identifier in a hash ring. When a request comes in for a specific partition, the consistent hashing algorithm maps it to the responsible node based on its identifier.

## Advantages
Consistent hashing has several advantages over traditional hash-based partitioning schemes:

1. Load Balancing: The use of a hash ring ensures that each node's load is evenly distributed across multiple partitions. As new nodes are added or removed from the system, only a small fraction of partitions need to be reassigned.

2. Incremental Expansion: Consistent hashing allows for incremental expansion without affecting existing assignments. When a new node is added, only its adjacent partitions in the hash ring need to be reassigned, minimizing the impact on the system.

3. Graceful Degradation: In case of node failures, consistent hashing provides graceful degradation by redistributing only the affected partitions. This minimizes the disruption to the system and ensures continued availability.


## Implementations 
>[!warning]
>The content in the section came from chat gpt, have not being test before
>

Consistent hashing is a technique used in distributed computing to efficiently distribute data across multiple nodes in a consistent manner, even when the number of nodes changes.

1. Determine the number of virtual nodes: Each physical node in the system is assigned a certain number of virtual nodes. The more virtual nodes a physical node has, the more evenly distributed the data will be.

2. Create a hash ring: A hash ring is a circular data structure that represents all possible hash values. Each virtual node is mapped to a position on the hash ring using their hash value.

3. Hash the keys: Each data item (key) that needs to be stored in the system is hashed using a consistent hashing algorithm, such as MD5 or SHA-1. The resulting hash value corresponds to a position on the hash ring.

4. Find the appropriate node: Starting from the position corresponding to the hashed key, find the nearest virtual node on the hash ring in clockwise direction. This virtual node represents the physical node responsible for storing that particular key.

5. Store or retrieve data: Once the appropriate physical node is found, store or retrieve data on that node based on its responsibility for that particular key.

6. Handle node failures and additions: If a physical node fails or becomes unavailable, its virtual nodes are redistributed among other available nodes in order to maintain load balancing and minimal data migration. Similarly, if new nodes are added to the system, their virtual nodes are created and distributed among all existing nodes.
### Code
```go
package main

import (
	"crypto/md5"
	"encoding/binary"
	"sort"
	"strconv"
)

type Node struct {
	ID       int
	IP       string
	Replicas int
}

type VirtualNode struct {
	Hash      uint32
	PhysicalID int
}

type ConsistentHashRing struct {
	Nodes       []Node
	VirtualNodes []VirtualNode
}

func NewConsistentHashRing() *ConsistentHashRing {
	return &ConsistentHashRing{}
}

func (chr *ConsistentHashRing) AddNode(node Node) {
	chr.Nodes = append(chr.Nodes, node)
	for i := 0; i < node.Replicas; i++ {
		vNode := VirtualNode{
			Hash:      hash(node.IP + ":" + strconv.Itoa(i)),
			PhysicalID: node.ID,
		}
		chr.VirtualNodes = append(chr.VirtualNodes, vNode)
	}
	sort.Slice(chr.VirtualNodes, func(i, j int) bool {
		return chr.VirtualNodes[i].Hash < chr.VirtualNodes[j].Hash
	})
}

func (chr *ConsistentHashRing) Get(key string) *Node {
	hashVal := hash(key)
	i := sort.Search(len(chr.VirtualNodes), func(i int) bool {
		return chr.VirtualNodes[i].Hash >= hashVal
	})

	if i == len(chr.VirtualNodes) {
		i = 0
	}

	return &chr.Nodes[chr.VirtualNodes[i].PhysicalID]
}

func hash(key string) uint32 {
	h := md5.New()
	h.Write([]byte(key))
	hashBytes := h.Sum(nil)
	return binary.BigEndian.Uint32(hashBytes[:4])
}

func main() {
	ring := NewConsistentHashRing()
	ring.AddNode(Node{ID: 1, IP: "192.168.0.1", Replicas: 10})
	ring.AddNode(Node{ID: 2, IP: "192.168.0.2", Replicas: 10})

	key := "exampleKey"
	node := ring.Get(key)
	println("Key", key, "goes to Node", node.ID)
}
	```
