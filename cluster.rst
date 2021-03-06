= 集群API

== 节点规范

大多数集群级别的 API 允许指定在哪个节点上执行 (例如, 为节点获取节点统计信息). 可以在 API 中标识节点, 要么使用它们内部的节点ID, 节点名称, 地址,
自定义属性, 要么只使用 `_local` 节点接收请求. 例如, 下面是节点信息的一些执行示例:

[source,js]
--------------------------------------------------
# 本地
curl localhost:9200/_nodes/_local
# 地址
curl localhost:9200/_nodes/10.0.0.3,10.0.0.4
curl localhost:9200/_nodes/10.0.0.*
# 名称
curl localhost:9200/_nodes/node_name_goes_here
curl localhost:9200/_nodes/node_name_goes_*
# 属性 (在配置中设置像 node.rack: 2 这样的东西)
curl localhost:9200/_nodes/rack:2
curl localhost:9200/_nodes/ra*:2
curl localhost:9200/_nodes/ra*:2*
--------------------------------------------------
--

include::cluster/health.asciidoc[]

include::cluster/state.asciidoc[]

include::cluster/stats.asciidoc[]

include::cluster/pending.asciidoc[]

include::cluster/reroute.asciidoc[]

include::cluster/update-settings.asciidoc[]

include::cluster/nodes-stats.asciidoc[]

include::cluster/nodes-info.asciidoc[]

include::cluster/nodes-hot-threads.asciidoc[]
