---
{"tags":["blogs"],"Author":"Qiu Weihong","creation date":"2023-11-12 15:41","modification date":"Sunday 12th November 2023 15:41:23","publish":null,"priority":null,"topics":["Backend Essential"],"banner":"https://infitniteloop.s3.ap-southeast-1.amazonaws.com/banner/distributed+lock.png","dg-publish":true,"permalink":"/blogs/backend-development-essentials/understanding-and-implementing-distributed-locks/","dgPassFrontmatter":true,"created":"2023-11-12T15:41:23.574+08:00","updated":"2023-11-12T21:39:21.269+08:00"}
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
<style> .container {font-family: sans-serif; text-align: center;} .button-wrapper button {z-index: 1;height: 40px; width: 100px; margin: 10px;padding: 5px;} .excalidraw .App-menu_top .buttonList { display: flex;} .excalidraw-wrapper { height: 800px; margin: 50px; position: relative;} :root[dir="ltr"] .excalidraw .layer-ui__wrapper .zen-mode-transition.App-menu_bottom--transition-left {transform: none;} </style><script src="https://cdn.jsdelivr.net/npm/react@17/umd/react.production.min.js"></script><script src="https://cdn.jsdelivr.net/npm/react-dom@17/umd/react-dom.production.min.js"></script><script type="text/javascript" src="https://cdn.jsdelivr.net/npm/@excalidraw/excalidraw@0/dist/excalidraw.production.min.js"></script><div id="Understanding_and_Implementing_Distributed_Locks_2023-11-12_1549.23.excalidraw.md1"></div><script>(function(){const InitialData={"type":"excalidraw","version":2,"source":"https://github.com/zsviczian/obsidian-excalidraw-plugin/releases/tag/1.9.27","elements":[{"id":"oWRa1Syv1XxLNteKPaF4_","type":"rectangle","x":-413.2486572265625,"y":-145.68031311035156,"width":171.845703125,"height":67.919921875,"angle":0,"strokeColor":"#1e1e1e","backgroundColor":"transparent","fillStyle":"solid","strokeWidth":2,"strokeStyle":"solid","roughness":1,"opacity":100,"groupIds":[],"frameId":null,"roundness":{"type":3},"seed":825723505,"version":21,"versionNonce":1599715743,"isDeleted":false,"boundElements":[{"type":"text","id":"SyfHUJ9j"},{"id":"rko0fZGvmoj6dtQs2Zqib","type":"arrow"},{"id":"O0WZ-AkFcbX_2nesRZDTM","type":"arrow"}],"updated":1699775478624,"link":null,"locked":false},{"id":"SyfHUJ9j","type":"text","x":-369.05577850341797,"y":-124.22035217285156,"width":83.45994567871094,"height":25,"angle":0,"strokeColor":"#1e1e1e","backgroundColor":"transparent","fillStyle":"solid","strokeWidth":2,"strokeStyle":"solid","roughness":1,"opacity":100,"groupIds":[],"frameId":null,"roundness":null,"seed":1581810321,"version":12,"versionNonce":1283966079,"isDeleted":false,"boundElements":null,"updated":1699775374877,"link":null,"locked":false,"text":"Tread A","rawText":"Tread A","fontSize":20,"fontFamily":1,"textAlign":"center","verticalAlign":"middle","baseline":18,"containerId":"oWRa1Syv1XxLNteKPaF4_","originalText":"Tread A","lineHeight":1.25},{"type":"rectangle","version":54,"versionNonce":600406751,"isDeleted":false,"id":"p9S4WmkaexuZUGbUtP5Nz","fillStyle":"solid","strokeWidth":2,"strokeStyle":"solid","roughness":1,"opacity":100,"angle":0,"x":-434.0191650390625,"y":135.9404754638672,"strokeColor":"#1e1e1e","backgroundColor":"transparent","width":171.845703125,"height":67.919921875,"seed":1720827569,"groupIds":[],"frameId":null,"roundness":{"type":3},"boundElements":[{"type":"text","id":"RYAwxeV9"},{"id":"OAD0PFlsDBWGM0pxIFS_K","type":"arrow"}],"updated":1699775520231,"link":null,"locked":false},{"type":"text","version":50,"versionNonce":1062916049,"isDeleted":false,"id":"RYAwxeV9","fillStyle":"solid","strokeWidth":2,"strokeStyle":"solid","roughness":1,"opacity":100,"angle":0,"x":-390.5362854003906,"y":157.4004364013672,"strokeColor":"#1e1e1e","backgroundColor":"transparent","width":84.87994384765625,"height":25,"seed":1881718929,"groupIds":[],"frameId":null,"roundness":null,"boundElements":[],"updated":1699775515968,"link":null,"locked":false,"fontSize":20,"fontFamily":1,"text":"Tread B","rawText":"Tread B","textAlign":"center","verticalAlign":"middle","containerId":"p9S4WmkaexuZUGbUtP5Nz","originalText":"Tread B","lineHeight":1.25,"baseline":18},{"id":"UYlgfS337Pn_yPfDiLN9u","type":"ellipse","x":21.10028076171875,"y":-33.57417297363281,"width":191.9921875,"height":125.78448486328125,"angle":0,"strokeColor":"#1e1e1e","backgroundColor":"transparent","fillStyle":"solid","strokeWidth":2,"strokeStyle":"solid","roughness":1,"opacity":100,"groupIds":[],"frameId":null,"roundness":{"type":2},"seed":1520557567,"version":84,"versionNonce":601592305,"isDeleted":false,"boundElements":[{"type":"text","id":"OrCBk5rv"},{"id":"rko0fZGvmoj6dtQs2Zqib","type":"arrow"},{"id":"OAD0PFlsDBWGM0pxIFS_K","type":"arrow"},{"id":"lJtzLHn3E3Rn_ZcTmWNNV","type":"arrow"}],"updated":1699775553940,"link":null,"locked":false},{"id":"OrCBk5rv","type":"text","x":91.65691098326404,"y":16.8465383505664,"width":51.11994934082031,"height":25,"angle":0,"strokeColor":"#1e1e1e","backgroundColor":"transparent","fillStyle":"solid","strokeWidth":2,"strokeStyle":"solid","roughness":1,"opacity":100,"groupIds":[],"frameId":null,"roundness":null,"seed":170454047,"version":38,"versionNonce":616442335,"isDeleted":false,"boundElements":null,"updated":1699775445281,"link":null,"locked":false,"text":"Redis","rawText":"Redis","fontSize":20,"fontFamily":1,"textAlign":"center","verticalAlign":"middle","baseline":18,"containerId":"UYlgfS337Pn_yPfDiLN9u","originalText":"Redis","lineHeight":1.25},{"id":"rko0fZGvmoj6dtQs2Zqib","type":"arrow","x":-234.85348510742188,"y":-113.51234436035156,"width":339.6583394381298,"height":85.41028805181574,"angle":0,"strokeColor":"#1e1e1e","backgroundColor":"transparent","fillStyle":"solid","strokeWidth":2,"strokeStyle":"solid","roughness":1,"opacity":100,"groupIds":[],"frameId":null,"roundness":{"type":2},"seed":297085649,"version":150,"versionNonce":1436490463,"isDeleted":false,"boundElements":[{"type":"text","id":"YBVTB7Ai"}],"updated":1699775453843,"link":null,"locked":false,"points":[[0,0],[298.9029846191406,-7.43817138671875],[339.6583394381298,77.97211666509699]],"lastCommittedPoint":[314.9120788574219,68.34307861328125],"startBinding":{"elementId":"oWRa1Syv1XxLNteKPaF4_","focus":0.01410522484212517,"gap":6.549468994140625},"endBinding":{"elementId":"UYlgfS337Pn_yPfDiLN9u","focus":0.18549848401904923,"gap":2.4772895446520167},"startArrowhead":null,"endArrowhead":"arrow"},{"id":"YBVTB7Ai","type":"text","x":-10.65045166015625,"y":-133.4505157470703,"width":149.39990234375,"height":25,"angle":0,"strokeColor":"#1e1e1e","backgroundColor":"transparent","fillStyle":"solid","strokeWidth":2,"strokeStyle":"solid","roughness":1,"opacity":100,"groupIds":[],"frameId":null,"roundness":null,"seed":1863214527,"version":17,"versionNonce":711510719,"isDeleted":false,"boundElements":null,"updated":1699775452532,"link":null,"locked":false,"text":"1) Set Exp 30s","rawText":"1) Set Exp 30s","fontSize":20,"fontFamily":1,"textAlign":"center","verticalAlign":"middle","baseline":18,"containerId":"rko0fZGvmoj6dtQs2Zqib","originalText":"1) Set Exp 30s","lineHeight":1.25},{"id":"hCQYc9pOE26OQztUjFVxm","type":"rectangle","x":454.837158203125,"y":-35.15299987792969,"width":190.6510009765625,"height":93.779296875,"angle":0,"strokeColor":"#1e1e1e","backgroundColor":"#a5d8ff","fillStyle":"hachure","strokeWidth":2,"strokeStyle":"solid","roughness":1,"opacity":100,"groupIds":[],"frameId":null,"roundness":{"type":3},"seed":507836817,"version":52,"versionNonce":1599662111,"isDeleted":false,"boundElements":[{"type":"text","id":"oQVXY71Z"},{"id":"O0WZ-AkFcbX_2nesRZDTM","type":"arrow"},{"id":"lJtzLHn3E3Rn_ZcTmWNNV","type":"arrow"}],"updated":1699775564611,"link":null,"locked":false},{"id":"oQVXY71Z","type":"text","x":471.7427291870117,"y":-0.7633514404296875,"width":156.83985900878906,"height":25,"angle":0,"strokeColor":"#1e1e1e","backgroundColor":"transparent","fillStyle":"solid","strokeWidth":2,"strokeStyle":"solid","roughness":1,"opacity":100,"groupIds":[],"frameId":null,"roundness":null,"seed":515740849,"version":43,"versionNonce":222234609,"isDeleted":false,"boundElements":null,"updated":1699775561667,"link":null,"locked":false,"text":"Target function","rawText":"Target function","fontSize":20,"fontFamily":1,"textAlign":"center","verticalAlign":"middle","baseline":18,"containerId":"hCQYc9pOE26OQztUjFVxm","originalText":"Target function","lineHeight":1.25},{"id":"O0WZ-AkFcbX_2nesRZDTM","type":"arrow","x":-312.8222351074219,"y":-147.46417236328125,"width":864.1353124911572,"height":237.9784927368164,"angle":0,"strokeColor":"#1e1e1e","backgroundColor":"#a5d8ff","fillStyle":"hachure","strokeWidth":2,"strokeStyle":"solid","roughness":1,"opacity":100,"groupIds":[],"frameId":null,"roundness":{"type":2},"seed":334572785,"version":295,"versionNonce":2094719441,"isDeleted":false,"boundElements":[{"type":"text","id":"GaaZLGZc"}],"updated":1699775561667,"link":null,"locked":false,"points":[[0,0],[75.03253173828125,-128.02083587646484],[735.7161560058594,-122.17772674560547],[864.1353124911572,109.95765686035156]],"lastCommittedPoint":[795.5403747558594,110.57942199707031],"startBinding":{"elementId":"oWRa1Syv1XxLNteKPaF4_","focus":-0.06090855137178886,"gap":1.7838592529296875},"endBinding":{"elementId":"hCQYc9pOE26OQztUjFVxm","focus":0.2341325049112612,"gap":2.353515625},"startArrowhead":null,"endArrowhead":"arrow"},{"id":"GaaZLGZc","type":"text","x":-24.546755642766925,"y":-307.3526197467005,"width":256.5597839355469,"height":25,"angle":0,"strokeColor":"#1e1e1e","backgroundColor":"#a5d8ff","fillStyle":"hachure","strokeWidth":2,"strokeStyle":"solid","roughness":1,"opacity":100,"groupIds":[],"frameId":null,"roundness":null,"seed":1111312639,"version":27,"versionNonce":279707871,"isDeleted":false,"boundElements":null,"updated":1699775502729,"link":null,"locked":false,"text":"2) Program execution 40s","rawText":"2) Program execution 40s","fontSize":20,"fontFamily":1,"textAlign":"center","verticalAlign":"middle","baseline":18,"containerId":"O0WZ-AkFcbX_2nesRZDTM","originalText":"2) Program execution 40s","lineHeight":1.25},{"id":"OAD0PFlsDBWGM0pxIFS_K","type":"arrow","x":-258.57745361328125,"y":179.23301696777344,"width":397.880859375,"height":91.79031372070312,"angle":0,"strokeColor":"#1e1e1e","backgroundColor":"#a5d8ff","fillStyle":"hachure","strokeWidth":2,"strokeStyle":"solid","roughness":1,"opacity":100,"groupIds":[],"frameId":null,"roundness":{"type":2},"seed":1502859153,"version":113,"versionNonce":340074481,"isDeleted":false,"boundElements":[{"type":"text","id":"zHq7vduG"}],"updated":1699775543057,"link":null,"locked":false,"points":[[0,0],[369.63214111328125,6.05462646484375],[397.880859375,-85.73568725585938]],"lastCommittedPoint":[397.880859375,-85.73568725585938],"startBinding":{"elementId":"p9S4WmkaexuZUGbUtP5Nz","focus":0.2224153153064531,"gap":3.59600830078125},"endBinding":{"elementId":"UYlgfS337Pn_yPfDiLN9u","focus":-0.42846183260868836,"gap":2.958060635213215},"startArrowhead":null,"endArrowhead":"arrow"},{"id":"zHq7vduG","type":"text","x":-3.5352020263671875,"y":160.2876434326172,"width":229.17977905273438,"height":50,"angle":0,"strokeColor":"#1e1e1e","backgroundColor":"#a5d8ff","fillStyle":"hachure","strokeWidth":2,"strokeStyle":"solid","roughness":1,"opacity":100,"groupIds":[],"frameId":null,"roundness":null,"seed":1421223999,"version":57,"versionNonce":2073987601,"isDeleted":false,"boundElements":null,"updated":1699775541640,"link":null,"locked":false,"text":"3. previous key expired,\nacquire lock","rawText":"3. previous key expired,\nacquire lock","fontSize":20,"fontFamily":1,"textAlign":"center","verticalAlign":"middle","baseline":43,"containerId":"OAD0PFlsDBWGM0pxIFS_K","originalText":"3. previous key expired,\nacquire lock","lineHeight":1.25},{"id":"lJtzLHn3E3Rn_ZcTmWNNV","type":"arrow","x":442.68231201171875,"y":18.490829467773438,"width":227.412109375,"height":9.4921875,"angle":0,"strokeColor":"#e03131","backgroundColor":"#a5d8ff","fillStyle":"hachure","strokeWidth":2,"strokeStyle":"solid","roughness":1,"opacity":100,"groupIds":[],"frameId":null,"roundness":{"type":2},"seed":572349919,"version":102,"versionNonce":653248639,"isDeleted":false,"boundElements":[{"type":"text","id":"FRcUbpNw"}],"updated":1699775564939,"link":null,"locked":false,"points":[[0,0],[-227.412109375,9.4921875]],"lastCommittedPoint":[-160.85284423828125,7.822265625],"startBinding":{"elementId":"hCQYc9pOE26OQztUjFVxm","focus":-0.04458450141573448,"gap":12.15484619140625},"endBinding":{"elementId":"UYlgfS337Pn_yPfDiLN9u","focus":0.043839043907385196,"gap":2.1982696327676763},"startArrowhead":null,"endArrowhead":"arrow"},{"id":"FRcUbpNw","type":"text","x":243.17667388916016,"y":11.571884155273438,"width":105.03990173339844,"height":25,"angle":0,"strokeColor":"#e03131","backgroundColor":"#a5d8ff","fillStyle":"hachure","strokeWidth":2,"strokeStyle":"solid","roughness":1,"opacity":100,"groupIds":[],"frameId":null,"roundness":null,"seed":849492511,"version":11,"versionNonce":1442910193,"isDeleted":false,"boundElements":null,"updated":1699775559832,"link":null,"locked":false,"text":"Delete key","rawText":"Delete key","fontSize":20,"fontFamily":1,"textAlign":"center","verticalAlign":"middle","baseline":18,"containerId":"lJtzLHn3E3Rn_ZcTmWNNV","originalText":"Delete key","lineHeight":1.25},{"id":"qOrohBNs0Hj6KB_HpuCym","type":"arrow","x":-237.84503173828125,"y":-110.69334411621094,"width":347.32745361328125,"height":67.94595336914062,"angle":0,"strokeColor":"#1e1e1e","backgroundColor":"transparent","fillStyle":"solid","strokeWidth":2,"strokeStyle":"solid","roughness":1,"opacity":100,"groupIds":[],"frameId":null,"roundness":{"type":2},"seed":1169975313,"version":102,"versionNonce":1467762993,"isDeleted":true,"boundElements":null,"updated":1699775401590,"link":null,"locked":false,"points":[[0,0],[336.95965576171875,0.712890625],[347.32745361328125,67.94595336914062]],"lastCommittedPoint":[347.32745361328125,67.94595336914062],"startBinding":{"elementId":"oWRa1Syv1XxLNteKPaF4_","focus":0.02453587439024075,"gap":3.55792236328125},"endBinding":{"elementId":"UYlgfS337Pn_yPfDiLN9u","focus":0.44879035512370136,"gap":3.865154801723534},"startArrowhead":null,"endArrowhead":"arrow"},{"id":"DCOS0_UnS07Nh_AdQdauX","type":"arrow","x":-306.6470947265625,"y":-149.75909423828125,"width":784.759033203125,"height":118.974609375,"angle":0,"strokeColor":"#1e1e1e","backgroundColor":"#a5d8ff","fillStyle":"hachure","strokeWidth":2,"strokeStyle":"solid","roughness":1,"opacity":100,"groupIds":[],"frameId":null,"roundness":{"type":2},"seed":979815537,"version":249,"versionNonce":1102972209,"isDeleted":true,"boundElements":null,"updated":1699775471458,"link":null,"locked":false,"points":[[0,0],[101.32159423828128,-118.974609375],[784.759033203125,-98.92253112792969]],"lastCommittedPoint":[784.759033203125,-98.92253112792969],"startBinding":{"elementId":"oWRa1Syv1XxLNteKPaF4_","focus":-0.10201653265671105,"gap":4.0787811279296875},"endBinding":null,"startArrowhead":null,"endArrowhead":"arrow"},{"id":"s97KcFwB5UVFgz2mAekeD","type":"arrow","x":-252.3990478515625,"y":111.21214294433594,"width":355.185546875,"height":30.08465576171875,"angle":0,"strokeColor":"#1e1e1e","backgroundColor":"#a5d8ff","fillStyle":"hachure","strokeWidth":2,"strokeStyle":"solid","roughness":1,"opacity":100,"groupIds":[],"frameId":null,"roundness":{"type":2},"seed":1230770257,"version":63,"versionNonce":1079579903,"isDeleted":true,"boundElements":null,"updated":1699775514115,"link":null,"locked":false,"points":[[0,0],[355.185546875,30.08465576171875]],"lastCommittedPoint":null,"startBinding":{"elementId":"p9S4WmkaexuZUGbUtP5Nz","focus":0.08807113166910104,"gap":3.0947265625},"endBinding":null,"startArrowhead":null,"endArrowhead":"arrow"},{"id":"4RWPoK3zWKOgPz871AJl4","type":"line","x":373.78582763671875,"y":20.681564331054688,"width":157.15167236328125,"height":6.73504638671875,"angle":0,"strokeColor":"#1e1e1e","backgroundColor":"#a5d8ff","fillStyle":"hachure","strokeWidth":2,"strokeStyle":"solid","roughness":1,"opacity":100,"groupIds":[],"frameId":null,"roundness":{"type":2},"seed":2096367551,"version":49,"versionNonce":1647537599,"isDeleted":true,"boundElements":null,"updated":1699775548995,"link":null,"locked":false,"points":[[0,0],[-157.15167236328125,6.73504638671875]],"lastCommittedPoint":[-157.15167236328125,6.73504638671875],"startBinding":null,"endBinding":null,"startArrowhead":null,"endArrowhead":null}],"appState":{"theme":"light","viewBackgroundColor":"#ffffff","currentItemStrokeColor":"#e03131","currentItemBackgroundColor":"#a5d8ff","currentItemFillStyle":"hachure","currentItemStrokeWidth":2,"currentItemStrokeStyle":"solid","currentItemRoughness":1,"currentItemOpacity":100,"currentItemFontFamily":1,"currentItemFontSize":20,"currentItemTextAlign":"left","currentItemStartArrowhead":null,"currentItemEndArrowhead":"arrow","scrollX":619.4205322265625,"scrollY":344.25982666015625,"zoom":{"value":1},"currentItemRoundness":"round","gridSize":null,"gridColor":{"Bold":"#C9C9C9FF","Regular":"#EDEDEDFF"},"currentStrokeOptions":null,"previousGridSize":null,"frameRendering":{"enabled":true,"clip":true,"name":true,"outline":true}},"files":{}};InitialData.scrollToContent=true;App=()=>{const e=React.useRef(null),t=React.useRef(null),[n,i]=React.useState({width:void 0,height:void 0});return React.useEffect(()=>{i({width:t.current.getBoundingClientRect().width,height:t.current.getBoundingClientRect().height});const e=()=>{i({width:t.current.getBoundingClientRect().width,height:t.current.getBoundingClientRect().height})};return window.addEventListener("resize",e),()=>window.removeEventListener("resize",e)},[t]),React.createElement(React.Fragment,null,React.createElement("div",{className:"excalidraw-wrapper",ref:t},React.createElement(ExcalidrawLib.Excalidraw,{ref:e,width:n.width,height:n.height,initialData:InitialData,viewModeEnabled:!0,zenModeEnabled:!0,gridModeEnabled:!1})))},excalidrawWrapper=document.getElementById("Understanding_and_Implementing_Distributed_Locks_2023-11-12_1549.23.excalidraw.md1");ReactDOM.render(React.createElement(App),excalidrawWrapper);})();</script>
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

