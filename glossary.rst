########################################
名词解释
########################################

[[glossary-analysis]] 分词 ::

  分词是将 <<glossary-text,文本>> 转换为
  如果使用特定的分词器，下面这些短语： `FOO BAR`, `Foo-Bar`, `foo,bar`有可能全部被分词成
  `foo` `bar`。 这些词串会最终被保存在索引中。
  +
  一个包含FoO:bAR的全文查询（不是<<glossary-term,term>>查询）也会被分词成foo bar，
  然后匹配上之前保存在索引中的词串。
  +
  这样的分词过程（同时在索引阶段和搜索阶段分词）使得elasticsearch可以执行全文检索。
  +
请查看<<glossary-text,text>>和<<glossary-term,term>>相关章节。

[[glossary-cluster]] 集群 ::
  一个集群包含一个或多个 <<glossary-node,节点>>，这些节点之间具有相同的集群名称。
  每个集群同一时刻有且只有一个
  主节点，主节点由集群自动选择，并且当前主节点宕掉后系统会自动选择另外的主节点。

[[glossary-document]] 文档 ::

  一篇文档就是存在elasticsearch中的一个JSON文档，类似于关系型数据库中的一行记录。
  每篇文档只会被保存在一个 <<glossary-index,索引>> 中，并且有一个<<glossary-type,类型>>
  和<<glossary-id,id>>。
  +
  文档是一个JSON包含零个或多个 <<glossary-field,字段>>，
  或者键-值对的对象（或者在其他语言中被当做hash/hashmap/关联数组）。
  +
  原始JSON文档将被存储于 <<glossary-source_field, `_source`字段>>中，
该字段会在获取或者查询文档时被默认返回回来。

[[glossary-id]] id ::

  ID是用来唯一标示一篇 <<glossary-document,文档>> 的。 一篇文档的 `索引/类型/id`三
  者的组合必须唯一。
  如果没有提供ID，系统会字段生成一个。 （见 <<glossary-routing,路由>>章节）

[[glossary-field]] 字段 ::

  一篇 <<glossary-document,文档>> 包含一组字段，或者键-值对。
  字段值可以是简单类型值（如字符串，整数，日期）， 也可以是类似于数组或对象的复杂嵌套数据结构。
   一个字段相当于关系型数据库中的一列。
  +
  每个字段有一个字段类型的  <<glossary-mapping,映射>> （不要和文档
   <<glossary-type,类型>> 混淆），
  该类型标明了存储于该字段中的数据的类型，如 `字符串`， `整数`， `对象`等。
字段的映射同时要求你定义如何对一个字段进行分词和其他选项。


[[glossary-index]] 索引 ::

  一个索引好比关系型数据库中的一个 _数据库_。 它有个 <<glossary-mapping,映射>>
  用来定义多个索引 <<glossary-type,类型>>。
  +
索引是逻辑上的命名空间，对应物理上的一个或多个分片，
  每个  <<glossary-primary-shard,主分片>>可以有零个或多个
  <<glossary-replica-shard,从分片>> 。

[[glossary-mapping]] 映射 ::

  映射对应关系型数据库中的 _schema_定义。 每个 <<glossary-index,索引>>有一个映射，
  对索引中的每个索引 <<glossary-type,类型>>进行定义， 外加一系列索引基本的设置选项。
  +
映射可以被显示定义，也可以在文档被索引时自动生成。

[[glossary-node]] 节点 ::

  一个节点是 <<glossary-cluster,集群>> 中运行elasticsearch的一个实例。 测试时可以在
  一台机器上启动多个节点， 但通常你只会一台服务器上启动一个节点。
  +
启动阶段，节点会利用单播机制来发现与其设置的集群名称相同的集群，并尝试加入该集群。


 [[glossary-primary-shard]] 主分片 ::

  每篇文档只会被保存在一个主  <<glossary-shard,分片>>中。 当你索引一个文档时，该文档首先
  在主分片上被索引，然后被分发到所有从分片上索引。
  +
  一个索引默认有5个主分片。 在 <<glossary-index,索引>> 创建时，你可以指定更少或更多的主分片，
  从而提升你的索引处理 <<glossary-document,文档>>数量的能力。
  +
  一旦索引被创建后，主分片数量将不可改变。
  +
见 <<glossary-routing,路由>>

 [[glossary-replica-shard]] 从分片 ::

  每个 <<glossary-primary-shard,分片>> 主可以有零个或多个从分片。 一个从分片是主
  分片的备份， 有下面两个目的：

  1. 增强错误恢复能力： 当主分片宕掉时，从分片可以被提升为主分片。
  2. 增强性能： 获取或者搜索请求能分别被主分片或者从分片处理。
  +
  每个主分片默认有一个从分片，
但索引的从分片数量可以在索引生成以后动态改变。 从分片和主分片不会被分配到同一个节点上。

[[glossary-routing]]  路由 ::

  你索引一篇文档时， 文档将被存储在单个 <<glossary-primary-shard,主分片>> 中。
  选择哪个分片是由 `路由`的hash值决定。 默认情况下， `路由` 值由文档的ID生成，如果该文档有
  父文档，则路由值取父文档的ID（以此确保子文档和父文档被索引到同一个分片中）。
  +
如果在索引时指定了特定的路由值，或者 <<glossary-mapping,映射>> 中设定了
 <<mapping-routing-field,路由字段>> ，
则该值会被设定的值取代。

[[glossary-shard]] 分片 ::

  一个分片是一个独立的Lucene实例， 是elasticsearch自动管理的底层工作单元。
  一个index是一个逻辑命名，其被指向物理 <<glossary-primary-shard,主分片>>和
  <<glossary-replica-shard,从分片>>。
  +
  除了定义主分片和从分片的数量以外， 不需要直接操作分片，而是应该对索引进行操作。
  +
  Elasticsearch会将所有分片在 <<glossary-cluster,集群>>中所有 <<glossary-node,节点>>
之间自动分配，添加节点或者节点失效时，会自动平衡分片。


 [[glossary-source_field]] 源字段 ::

  默认情况下，你索引的JSON文档将会被存储在 `_source` 字段中，并在搜索和获取请求时返回来。
  这让你可以直接从搜索结果中访问原始对象， 而不是通过ID重新请求一次。
  +
  注: 返回的是你所索引的JSON字符串的原文，即使是不规范的JSON字符串。
  这个字段的内容与该对象在索引中是怎么被索引的并没有什么关系。

[[glossary-term]] 词项 ::

   词项是被真正索引在elasticsearch中的值。 `foo`, `Foo`, `FOO`不是同一个词项。
   词项（也就是这些实际被索引的值）可以用 _term_查询搜索到。
   +
  请见 <<glossary-text,文本>>和 <<glossary-analysis,分词>>。


[[glossary-text]] 文本 ::

  文本（或全文本）通常是非结构化文本， 就像这一段的内容一样。 默认情况下，
  文本会被 <<glossary-analysis,分词>> 成 <<glossary-term,词串>>，并最终保存在索引中。
  +
  文本 <<glossary-field,字段>>需要在索引阶段被分词，这些文本如果需要被搜索到，
  搜索阶段也需要被分词成相同的词串。
  +
请见 <<glossary-term,词项>>和 <<glossary-analysis,分词>>。

[[glossary-type]] 类型 ::

  索引类型类似于关系型数据库中的 _表_。 每个类型包含一组用来表示文档的 <<glossary-field,字段>> 。
  <<glossary-mapping,映射>>定义了 <<glossary-document,文档>> 中的每个字段如何被分词。
