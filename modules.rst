= 模块

本节包含了Elasticsearch中各个功能模块。每个模块的有如下可能的配置：

_static_::

这些配置必须在节点级别设置，要么通过 `elasticsearch.yml` 文件配置，要么在节点启动时通过环境
变量或者命令行指定，并且必须在集群中得每个相关节点上都配置。

_dynamic_::

这些配置可以在正运行的集群中通过 <<cluster-update-settings,cluster-update-settings>>
API动态更新。

本节包含的模块有：

<<modules-cluster,集群级别的路由和分片分配>>::

    用来控制索引分片被适时适当的分配给正确的节点。

<<modules-discovery,Discovery>>::

    节点间如何互相发现并组成一个集群。

<<modules-gateway,Gateway>>::

    在分片恢复开始前，需要有多少个节点加入集群。

<<modules-http,HTTP>>::

    用来控制HTTP REST接口的配置。

<<modules-indices,Indices>>::

    索引相关的全局设置。

<<modules-network,Network>>::

    控制默认网络配置。

<<modules-node,Node client>>::

    一个Java版的客户端节点可以加入到集群中，但并不保持数据和也没用主节点那样的行为。

<<modules-plugins,Plugins>>::

    通过插件扩展Elasticsearch。

<<modules-scripting,Scripting>>::

    自定义脚本，目前支持Lucene表达式，Groovy，Python和Javascript

<<modules-snapshots,Snapshot/Restore>>::

    利用快照或者复原对数据备份。

<<modules-threadpool,Thread pools>>::

    关于Elasticsearch中固定线程池的相关信息。

<<modules-transport,Transport>>::

    配置网络通信层，以供Elasticsearch内部各节点之间的通信。

<<modules-tribe,Tribe nodes>>::

    一个tribe节点可以加入一个或者多个集群，并且将这些集群联合起来。
--


include::modules/cluster.asciidoc[]

include::modules/discovery.asciidoc[]

include::modules/gateway.asciidoc[]

include::modules/http.asciidoc[]

include::modules/indices.asciidoc[]

include::modules/network.asciidoc[]

include::modules/node.asciidoc[]

include::modules/plugins.asciidoc[]

include::modules/scripting.asciidoc[]

include::modules/advanced-scripting.asciidoc[]

include::modules/snapshots.asciidoc[]

include::modules/threadpool.asciidoc[]

include::modules/transport.asciidoc[]

include::modules/tribe.asciidoc[]
