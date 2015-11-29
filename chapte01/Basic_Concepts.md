There are a few concepts that are core to Elasticsearch. Understanding these concepts from the outset will tremendously help ease the learning process.

Elasticsearch有几个比较核心的概念，从一开始就理解这些概念将会极大地帮助简化学习过程.

### 准实时(NRT)

Elasticsearch is a near real time search platform. What this means is there is a slight latency (normally one second) from the time you index a document until the time it becomes searchable.

Elasticsearch是一个准实时的搜索平台，这意味着从你为文档建立索引的那一刻到文档可以被检索的那一刻之间会有轻微的延迟 (通常是1秒)

### 集群(Cluster)

A cluster is a collection of one or more nodes (servers) that together holds your entire data and provides federated indexing and search capabilities across all nodes. A cluster is identified by a unique name which by default is "elasticsearch". This name is important because a node can only be part of a cluster if the node is set up to join the cluster by its name.

集群是一个或多个节点（服务器）的集合，它持有你所有的数据并且在所有节点间提供联合索引和搜索的功能. 集群使用一个唯一的名称标识，默认是"elasticsearch".这个名称非常重要，因为一个节点只能是一个集群的一部分，如果这个节点创建时是通过集群的名称加入到集群中.

Make sure that you don’t reuse the same cluster names in different environments, otherwise you might end up with nodes joining the wrong cluster. For instance you could use logging-dev, logging-stage, and logging-prod for the development, staging, and production clusters.

请确保你没有在不同的环境中复用相同的集群名称，否则你可能会以节点加入到错的集群中而告终。例如你可以使用logging-dev、logging-stage以及logging-prod分别命名开发环境集群、预发布环境集群以及生产环境集群。

Note that it is valid and perfectly fine to have a cluster with only a single node in it. Furthermore, you may also have multiple independent clusters each with its own unique cluster name.

注意，集群中只有一个节点是有效并且非常好的。另外，你也可以拥有多个独立的集群，每一个都有唯一的集群名称。

### 节点(Node)

A node is a single server that is part of your cluster, stores your data, and participates in the cluster’s indexing and search capabilities. Just like a cluster, a node is identified by a name which by default is a random Marvel character name that is assigned to the node at startup. You can define any node name you want if you do not want the default. This name is important for administration purposes where you want to identify which servers in your network correspond to which nodes in your Elasticsearch cluster.

A node can be configured to join a specific cluster by the cluster name. By default, each node is set up to join a cluster named elasticsearch which means that if you start up a number of nodes on your network and—assuming they can discover each other—they will all automatically form and join a single cluster named elasticsearch.



In a single cluster, you can have as many nodes as you want. Furthermore, if there are no other Elasticsearch nodes currently running on your network, starting a single node will by default form a new single-node cluster named elasticsearch.

在单个集群中, 你想定义多少节点都可以。而且，如果在你当前的网络环境中没有其它Elasticsearch节点在运行, 启动单个节点默认会形成一个名为elasticsearch的单节点集群。


### 索引(Index)

An index is a collection of documents that have somewhat similar characteristics. For example, you can have an index for customer data, another index for a product catalog, and yet another index for order data. An index is identified by a name (that must be all lowercase) and this name is used to refer to the index when performing indexing, search, update, and delete operations against the documents in it.

索引是具有某些类似特征的文档的一个集合. 例如, 你可以为顾客数据创建一个索引, 为产品类目创建另一个索引，还可以为订单数据创建另一个索引. 索引使用名称(字母必须全部小写)来标识，并且当在索引里的文档上执行索引、搜索、更新以及删除操作时这个名称用于引用这个索引。

In a single cluster, you can define as many indexes as you want.

在单个集群中, 你想定义多少索引都可以。

### 类型(Type)

Within an index, you can define one or more types. A type is a logical category/partition of your index whose semantics is completely up to you. In general, a type is defined for documents that have a set of common fields. For example, let’s assume you run a blogging platform and store all your data in a single index. In this index, you may define a type for user data, another type for blog data, and yet another type for comments data.

在索引中，你可以定义一个或多个类型。类型是索引的逻辑分类或分区，并且它的语义完全取决于你.类型通常是为有公共属性集合的文档定义的. 例如, 假设你运行一个博客平台并且将你所有的数据存储在单个索引中. 在这个索引中, 你可以为用户数据定义一种类型, 为博客数据定义另一种类型，并且还为评论数据定义另一种类型.


### 文档(Document)

A document is a basic unit of information that can be indexed. For example, you can have a document for a single customer, another document for a single product, and yet another for a single order. This document is expressed in JSON (JavaScript Object Notation) which is an ubiquitous internet data interchange format.

文档是可以被索引的基本信息单元。例如你可以为单个顾客创建一个文档，为单个产品创建另一个文档，并且还可以为单个订单创建另外一个文档。文档使用JSON(JavaScript Object Notation)格式表示，它是一种普遍存在的网络数据交换格式。

Within an index/type, you can store as many documents as you want. Note that although a document physically resides in an index, a document actually must be indexed/assigned to a type inside an index.

在一个索引或类型中，你想存储多少文档都可以。注意，尽管文档物理地存在于索引中，但它实际上必须被索引到或分配给索引中的一个类型。

### 分片和复制(Shards&Replica)

An index can potentially store a large amount of data that can exceed the hardware limits of a single node. For example, a single index of a billion documents taking up 1TB of disk space may not fit on the disk of a single node or may be too slow to serve search requests from a single node alone.

To solve this problem, Elasticsearch provides the ability to subdivide your index into multiple pieces called shards. When you create an index, you can simply define the number of shards that you want. Each shard is in itself a fully-functional and independent "index" that can be hosted on any node in the cluster.

Sharding is important for two primary reasons:

分片之所以很重要有以下两个主要原因：

* It allows you to horizontally split/scale your content volume
* It allows you to distribute and parallelize operations across shards (potentially on multiple nodes) thus increasing performance/throughput

The mechanics of how a shard is distributed and also how its documents are aggregated back into search requests are completely managed by Elasticsearch and is transparent to you as the user.

In a network/cloud environment where failures can be expected anytime, it is very useful and highly recommended to have a failover mechanism in case a shard/node somehow goes offline or disappears for whatever reason. To this end, Elasticsearch allows you to make one or more copies of your index’s shards into what are called replica shards, or replicas for short.

Replication is important for two primary reasons:

复制之所以很重要有以下两个主要原因：

* It provides high availability in case a shard/node fails. For this reason, it is important to note that a replica shard is never allocated on the same node as the original/primary shard that it was copied from.
* It allows you to scale out your search volume/throughput since searches can be executed on all replicas in parallel.

To summarize, each index can be split into multiple shards. An index can also be replicated zero (meaning no replicas) or more times. Once replicated, each index will have primary shards (the original shards that were replicated from) and replica shards (the copies of the primary shards). The number of shards and replicas can be defined per index at the time the index is created. After the index is created, you may change the number of replicas dynamically anytime but you cannot change the number shards after-the-fact.

By default, each index in Elasticsearch is allocated 5 primary shards and 1 replica which means that if you have at least two nodes in your cluster, your index will have 5 primary shards and another 5 replica shards (1 complete replica) for a total of 10 shards per index.

Note
Each Elasticsearch shard is a Lucene index. There is a maximum number of documents you can have in a single Lucene index. As of LUCENE-5843, the limit is 2,147,483,519 (= Integer.MAX_VALUE - 128) documents. You can monitor shard sizes using the _cat/shards api.

With that out of the way, let’s get started with the fun part…