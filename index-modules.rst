
= 索引模块


索引模块是用来组织和管理与一个索引相关的所有方面的模块。

[float]
[[index-modules-settings]]
== 索引设置

索引级别的设置可以基于每个索引进行设置， 可以设置的内容如下：

_static_::

这个设置仅仅在索引创建时设置，或者基于 <<indices-open-close,closed index>> 设置.

_dynamic_::

动态设置可以在一个正在正常运行的索引上使用
<<indices-update-settings,update-index-settings>> API 进行设置.

警告: 改变一个已关闭索引的静态或者动态设置有可能导致错误发生，并且这种错误不可恢复，除非删除并重新创建一个索引。

[float]
=== 静态索引设置

下面是针对 _static_ 索引设置的完整列表，这些设置与任何特定索引模块无关：

`index.number_of_shards`::

    一个索引中主分片的数量， 默认是5个。 这个设置可以在索引创建时设置， 已关闭索引的主分片数量
    不能被改变。

`index.shard.check_on_startup`::
+
--
experimental[] 在索引分片打开时是否检查索引已损坏，如果检测到索引损坏，这个分片将不会被打开。接受参数：

`false`::

    (默认) 分片打开时不检查是否损坏。

`checksum`::

  检查物理文件是否损坏

`true`::

    既检查物理损坏也检查逻辑损坏， 这是很耗CPU和内存的检查操作。

`fix`::

    既检查物理损坏也检查逻辑损坏， 损坏的索引段将被自动移除。 这个选项 *可能导致数据丢失*， 使用时需要格外小心。

针对大索引的检查可能会花费很长时间。
--

[[index-codec]] `index.codec`::

    experimental[] 该参数默认值为 +default+ ，此时采用LZ4压缩算法进行压缩，
    但这可以设置成 +best_compression+，该算法使用 https://en.wikipedia.org/wiki/DEFLATE[DEFLATE]
    从而达到更高压缩比，代价就是存储速度变慢。

[float]
[[dynamic-index-settings]]
=== 动态索引设置

下面是针对 _dynamic_ 索引设置的完整列表，这些设置与任何特定索引模块无关：


`index.number_of_replicas`::

    索引分片副本数量，默认为1.

`index.auto_expand_replicas`::

    基于可用节点数量自动扩展副本数量。
    指定副本的上下限，中间用 `-` 分隔(如: `0-5` )，或者使用 `all` 作为上限（如： `0-all`）。
    该选项默认值为 `false`(即 禁用该功能)

`index.refresh_interval`::

    多久执行一次 `refresh` 操作，这个操作可以让索引中的更新反映到搜索结果中。 默认是 `1s`, 如果
    设置成 `-1` 则禁用更新。

`index.blocks.read_only`::

    设成 `true` 则索引和索引元数据变为只读， 设成 `false` 则允许索引和元数据修改。

`index.blocks.read`::

    设置为 `true` 则禁止该索引的读操作。

`index.blocks.write`::

    设置为 `true` 则禁止该索引的写操作。

`index.blocks.metadata`::

    设置为 `true` 则禁止对索引元数据的读写操作。

`index.ttl.disable_purge`::

    experimental[] 禁止对索引中 <<mapping-ttl-field,过期文档>> 的清除操作。

[[index.recovery.initial_shards]]`index.recovery.initial_shards`::
+
--
仅当集群中具有足够多的节点可供分配索引分片的副本以使得主分片和所有副本能满足超过半数的要求时，主分片才会被恢复成可用状态。

    * `quorum` (默认值)
    * `quorum-1` (或 `half`)
    * `full`
    * `full-1`.
    * 数值也是支持的, 如 `1`.
--


[float]
=== 索引模块其他设置

索引模块其他设置如下:

<<analysis,Analysis>>::

    用来设置分词器，tokenizers, token filters 和 character filters。

<<index-modules-allocation,Index shard allocation>>::

    用来控制索引分片如何适时恰当的分配给集群中得节点。

<<index-modules-mapper,Mapping>>::

    开启或禁用索引的动态映射。

<<index-modules-similarity,Similarities>>::

    配置自定义相似度以调节搜索结果的排序。

<<index-modules-slowlog,Slowlog>>::

    定义慢查询或者获取请求如何被记录下来。

<<index-modules-store,Store>>::

    配置索引分片数据的存储方式。

<<index-modules-translog,Translog>>::

    用来控制事务日志和后台刷新操作策略。

--

include::index-modules/analysis.asciidoc[]

include::index-modules/allocation.asciidoc[]

include::index-modules/mapper.asciidoc[]

include::index-modules/similarity.asciidoc[]

include::index-modules/slowlog.asciidoc[]

include::index-modules/store.asciidoc[]

include::index-modules/translog.asciidoc[]
