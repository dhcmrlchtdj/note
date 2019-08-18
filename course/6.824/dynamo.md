# Dynamo

- introduction
    - at this scale, small and large components fail continuously
    - reliability is one of the most important requirements
    - the reliability and scalability of a system is dependent on how its application state is managed
    - build highly-available applications on top of an eventual-consistent storage system
    - Dynamo
        - Dynamo provides a primary-key only interface
        - Dynamo sacrifices consistency under certain failure scenarios
        - consistency is facilitated by object versioning
            - a quorum-like technique
            - a decentralized replica synchronization protocol
        - data is partitioned and replicated using consistent hashing
        - a gissip based distributed failure detection and membership protocol
        - Dynamo is a highly available and scalable data store
            - successful in handling server failures, data center failures and network partitions
            - customize by tuning the parameters N, R, W

- background
    - state is stored as binary objects identified by unique keys (usually less than 1MB)
    - Dynamo does not provide any isolation guarantees and permits only single key updates
        - provide ACID guarantees tend to have poor availability
    - SLA is measured at the 99.9th percentile of the distribution
        - eg. a response within 300ms for 99.9% of its requests for a peak client load of 500 requests per second
    - conflicts, when to resolve and who resolve them
        - when, read or write
            - Dynamo is an "always writeable" data store (better for user experiences)
        - who, the data store or the application
            - the data store can only use simple policies (eg. last write wins)
            - application can choose the better conflict resolution method
    - others
        - symmetry
        - decentralization
        - incremental scalability

- system architecture
    - components
        - data persistence
        - load balance
        - membership
        - failure detecion
        - failure recovery
        - replica syschronization
        - overload handling
        - state transfer
        - concurrency and job schedule
        - request marshall
        - request route
        - system monitor and alerm
        - configuration management
    - Dynamo
        - partitioning
            - By, consistent hashing
            - Can, incremental scalability
        - high availability for writes
            - By, vector clocks with reconciliation during reads
            - Can, version size is decoupled from update rates
        - handling temporary failures
            - By, sloppy quorum and hinted handoff
            - Can, provides high availability and durability guarantee when some of the replicas are not available
        - recovering from permanent failures
            - By, anti-entropy using Merkle trees
            - Can, synchronizes divergent replicas in the background
        - membership and failure detection
            - By, gossip-based membership protocol and failure detection
            - Can, preserves symmetry and avoids having a centralized registry for storing membership and node liveness information
    - interface
        - put(key, context, object)
        - get(key) => object | object list
        - ID = MD5(key), a 128-bit identifier
    - partitioning
        - use consistent hashing to make system scale incrementally
        - Dynamo variant: each node gets assigned to multiple points in the ring
            - 一致性哈希的好处是增删节点，只影响相邻的节点，影响范围较小
            - Dynamo 的做法，让负载更加均衡（比如删节点，其负载会由其他节点平均负担，而不会导致某个节点压力激增
            - 假如节点配置不同，可以分配不同的负载
    - replication
        - data is replicated at N nodes
            - N=3, key between range (node_A, node_B) will be stored in node_B, node_C, node_D
        - (key's) preference list, the list of nodes that store the key
    - data versioning
        - every write request create a new and immutable version of object
            - when a client wishes to update an object, it must specify which version it is updating
        - use vector clocks in order to capture causality between different version of the same object
            - vector clock is a list of (node, counter) pairs
        - the size of vector clocks grow
            - threshold, delete the oldest pair
    - executing get/put operations
        - coordinator: the node handling read/write operations
            - the first among the top N nodes in the preference list
            - any storage node can receive get/put for any key
            - node will forward the request to coordinator of key
        - quorum system
            - R, minimum number of nodes that participate in a successful read
            - W, minimum number of nodes that participate in a successful write
            - latency 由 R/W 中最慢的节点决定
            - 由 coordinator 将操作发送给 N-1 个节点，等待 R/W-1 个节点操作成功
    - handleing failures
        - hinted handoff
            - N=3, if A down, the request will be sent to B,C,D
            - D will keep data in a separate local datatbase
            - when A recovery, D will delivery the replica to A
        - replica synchronization
            - each node maintains a separate Merkle tree for each key range
    - membership and failure detection
        - administrator use CLI to issue a membership change
        - gossip-based protocol
            - each node contacts a peer chosen at random every second
            - the two nodes efficiently reconcile theri persisted membership change histories
            - 管理员修改一个节点，然后被传播到所有节点
        - external discovery
            - seeds are known to all nodes
            - get seeds from static configuration or configuration service
        - failure
            - A consider B failed if B does not respond to A's message
            - A periodically retries B to check for B's recovery

- implementation
    - each storage node has three main component
        - request coordination
        - membership and failure detection
        - local persistence engine
            - can choose storage engine for different application
            - BerkeleryDB, MySQL, in-memory buffer, ...
    - read-your-write
        - the coordinator for a write is chsen to be the node that replied fastest to previous read

- experiences
    - the common (N,R,W) is (3,2,2)
    - uniform load distribution
        - use consistent hashing to partition its key space across its replicas and ensure uniform load distribution
        - 通过 partitioning，read/write skew 带来的负载会被分散到不同节点
        - partitioning scheme
            - T random tokens per node and partition by token value
            - T random tokens per node and equal sized partitions
            - Q/S tokens per node, equal-sized partitions (better)
    - coordination
        - coordination component, use a state machine to handle incoming requests
        - server-driven
            - any Dynamo node can act as a coordination for a read request
            - write requests will be coordinated by a node in the key's preference list
        - client-driven (better performance)
            - move the state machine to the client nodes
            - load balancer is no longer required to uniformly distrubute client load
    - background/foreground tasks