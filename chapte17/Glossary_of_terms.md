# 名词解释
###分词
Analysis is the process of converting full text to terms. Depending on which analyzer is used, these phrases: FOO BAR, Foo-Bar, foo,bar will probably all result in the terms foo and bar. These terms are what is actually stored in the index. A full text query (not a term query) for FoO:bAR will also be analyzed to the terms foo,bar and will thus match the terms stored in the index. It is this process of analysis (both at index time and at search time) that allows elasticsearch to perform full text queries. Also see text and term.

分词是将文本转换为词串的过程。 如果使用特定的分词器，下面这些短语： FOO BAR, Foo-Bar, foo,bar有可能全部被分词成foo bar。 这些词串会最终被保存在索引中。 一个包含FoO:bAR的全文查询（不是term 查询）也会被分词成foo bar，然后匹配上之前保存在索引中的词串。 这样的分词过程（同时在索引阶段和搜索阶段分词）使得elasticsearch可以执行全文检索。 请查看text和term相关章节。

###集群
A cluster consists of one or more nodes which share the same cluster name. Each cluster has a single master node which is chosen automatically by the cluster and which can be replaced if the current master node fails.

一个集群包含一个或多个节点，这些节点之间具有相同的集群名称。 每个集群同一时刻有且只有一个主节点，主节点由集群自动选择，并且当前主节点宕掉后系统会自动选择另外的主节点。

###文档
A document is a JSON document which is stored in elasticsearch. It is like a row in a table in a relational database. Each document is stored in an index and has a type and an id. A document is a JSON object (also known in other languages as a hash / hashmap / associative array) which contains zero or more fields, or key-value pairs. The original JSON document that is indexed will be stored in the _source field, which is returned by default when getting or searching for a document.

一篇文档就是存在elasticsearch中的一个JSON文档，类似于关系型数据库中的一行记录。 每篇文档只会被保存在一个索引中，并且有一个类型和id。 文档是一个JSON包含零个或多个字段，或者键-值对的对象（或者在其他语言中被当做hash/hashmap/关联数组）。 原始JSON文档将被存储于_source字段中，该字段会在获取或者查询文档时被默认返回回来。

###id
The ID of a document identifies a document. The index/type/id of a document must be unique. If no ID is provided, then it will be auto-generated. (also see routing)

ID是用来唯一标示一篇文档的。 一篇文档的索引/类型/id三者的组合必须唯一。 如果没有提供ID，系统会字段生成一个。 （见路由章节）

###字段
A document contains a list of fields, or key-value pairs. The value can be a simple (scalar) value (eg a string, integer, date), or a nested structure like an array or an object. A field is similar to a column in a table in a relational database. The mapping for each field has a field type (not to be confused with document type) which indicates the type of data that can be stored in that field, eg integer, string, object. The mapping also allows you to define (amongst other things) how the value for a field should be analyzed.

一篇文档包含一组字段，或者键-值对。 字段值可以是简单类型值（如字符串，整数，日期）， 也可以是类似于数组或对象的复杂嵌套数据结构。 一个字段相当于关系型数据库中的一列。每个字段有一个字段类型的映射（不要和文档类型混淆），该类型标明了存储于该字段中的数据的类型，如字符串，整数，对象等。 字段的映射同时要求你定义如何对一个字段进行分词和其他选项。

###索引
An index is like a database in a relational database. It has a mapping which defines multiple types. An index is a logical namespace which maps to one or more primary shards and can have zero or more replica shards.

一个索引好比关系型数据库中的一个数据库。 它有个映射用来定义多个索引类型。 索引是逻辑上的命名空间，对应物理上的一个或多个主分片，每个主分片可以有零个或多个从分片。

###映射
A mapping is like a schema definition in a relational database. Each index has a mapping, which defines each type within the index, plus a number of index-wide settings. A mapping can either be defined explicitly, or it will be generated automatically when a document is indexed.

映射对应关系型数据库中的schema定义。 每个索引有一个映射，对索引中的每个索引类型进行定义， 外加一系列索引基本的设置选项。 映射可以被显示定义，也可以在文档被索引时自动生成。

###节点
A node is a running instance of elasticsearch which belongs to a cluster. Multiple nodes can be started on a single server for testing purposes, but usually you should have one node per server. At startup, a node will use unicast to discover an existing cluster with the same cluster name and will try to join that cluster.

一个节点是集群中运行elasticsearch的一个实例。 测试时可以在一台机器上启动多个节点， 但通常你只会一台服务器上启动一个节点。 启动阶段，节点会利用单播机制来发现与其设置的集群名称相同的集群，并尝试加入该集群。

###主分片
Each document is stored in a single primary shard. When you index a document, it is indexed first on the primary shard, then on all replicas of the primary shard. By default, an index has 5 primary shards. You can specify fewer or more primary shards to scale the number of documents that your index can handle. You cannot change the number of primary shards in an index, once the index is created. See also routing

每篇文档只会被保存在一个主分片中。 当你索引一个文档时，该文档首先在主分片上被索引，然后被分发到所有从分片上索引。 一个索引默认有5个主分片。 在索引创建时，你可以指定更少或更多的主分片，从而提升你的索引处理文档数量的能力。一旦索引被创建后，主分片数量将不可改变。 见路由章节

###从分片
Each primary shard can have zero or more replicas. A replica is a copy of the primary shard, and has two purposes:

1. increase failover: a replica shard can be promoted to a primary shard if the primary fails
2. increase performance: get and search requests can be handled by primary or replica shards. By default, each primary shard has one replica, but the number of replicas can be changed dynamically on an existing index. A replica shard will never be started on the same node as its primary shard.

每个主分片可以有零个或多个从分片。 一个从分片是主分片的备份， 有下面两个目的：

1. 增强错误恢复能力： 当主分片宕掉时，从分片可以被提升为主分片。
2. 增强性能： 获取或者搜索请求能分别被主分片或者从分片处理。 每个主分片默认有一个从分片，但索引的从分片数量可以在索引生成以后动态改变。 从分片和主分片不会被分配到同一个节点上。

###路由
When you index a document, it is stored on a single primary shard. That shard is chosen by hashing the routing value. By default, the routing value is derived from the ID of the document or, if the document has a specified parent document, from the ID of the parent document (to ensure that child and parent documents are stored on the same shard). This value can be overridden by specifying a routing value at index time, or a routing field in the mapping.

当你索引一篇文档时， 文档将被存储在单个主分片中。 选择哪个分片是由路由的hash值决定。 默认情况下，路由值由文档的ID生成，如果该文档有父文档，则路由值取父文档的ID（以此确保子文档和父文档被索引到同一个分片中）。 如果在索引时指定了特定的路由值，或者映射中设定了路由字段，则该值会被设定的值取代。

###分片
A shard is a single Lucene instance. It is a low-level “worker” unit which is managed automatically by elasticsearch. An index is a logical namespace which points to primary and replica shards. Other than defining the number of primary and replica shards that an index should have, you never need to refer to shards directly. Instead, your code should deal only with an index. Elasticsearch distributes shards amongst all nodes in the cluster, and can move shards automatically from one node to another in the case of node failure, or the addition of new nodes.

一个分片是一个独立的Lucene实例， 是elasticsearch自动管理的底层工作单元。 一个index是一个逻辑命名，其被指向物理主分片和从分片。 除了定义主分片和从分片的数量以外， 不需要直接操作分片，而是应该对索引进行操作。 Elasticsearch会将所有分片在集群中所有节点之间自动分配， 添加节点或者节点失效时，会自动平衡分片。

###源字段
By default, the JSON document that you index will be stored in the _source field and will be returned by all get and search requests. This allows you access to the original object directly from search results, rather than requiring a second step to retrieve the object from an ID. Note: the exact JSON string that you indexed will be returned to you, even if it contains invalid JSON. The contents of this field do not indicate anything about how the data in the object has been indexed.

默认情况下，你索引的JSON文档将会被存储在_source字段中，并在搜索和获取请求时返回来。 这让你可以直接从搜索结果中访问原始对象， 而不是通过ID重新请求一次。

###词项
A term is an exact value that is indexed in elasticsearch. The terms foo, Foo, FOO are NOT equivalent. Terms (i.e. exact values) can be searched for using term queries. See also text and analysis.

词项是被真正索引在elasticsearch中的值。 foo, Foo, FOO不是同一个词项。 词项（也就是这些实际被索引的值）可以用term查询搜索到。 请见文本和分词章节


###文本
Text (or full text) is ordinary unstructured text, such as this paragraph. By default, text will be analyzed into terms, which is what is actually stored in the index. Text fields need to be analyzed at index time in order to be searchable as full text, and keywords in full text queries must be analyzed at search time to produce (and search for) the same terms that were generated at index time. See also term and analysis.

文本（或全文本）通常是非结构化文本， 就像这一段的内容一样。 默认情况下，文本会被分词成词串，并最终保存在索引中。 文本字段需要在索引阶段被分词，这些文本如果需要被搜索到，搜索阶段也需要被分词成相同的词串。 请见词项和分词章节。

###类型
A type is like a table in a relational database. Each type has a list of fields that can be specified for documents of that type. The mapping defines how each field in the document is analyzed.

索引类型类似于关系型数据库中的表。 每个类型包含一组用来表示文档的字段。 映射定义了文档中的每个字段如何被分词。
