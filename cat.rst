[[cat]]
= cat API

[partintro]
--

["float",id="intro"]
== 介绍

对于计算机来说 JSON 很棒. 即使它是输出美化的, 视图找到数据中的关系是冗长乏味的. 当人类的眼睛看着ssh终端时,  需要的是紧凑和对齐的文本. cat API 旨在满足这样的需求.

所有的 cat 命令都接受一个查询字符串参数 `help` 来查看它们提供的所有头部和信息, 并且单独使用 `/_cat` 命令可以列出所有可使用的命令.

[float]
[[common-parameters]]
== 通用参数

[float]
[[verbose]]
=== Verbose

每个命令都接受一个查询字符串参数 `v` 来打开详细输出.

[source,sh]
--------------------------------------------------
% curl 'localhost:9200/_cat/master?v'
id                     ip        node
EGtKWZlWQYWDmX29fUnp3Q 127.0.0.1 Grey, Sara
--------------------------------------------------

[float]
[[help]]
=== Help

每个命令都接受一个查询字符串参数 `help`, 它会输出可用的列.

[source,sh]
--------------------------------------------------
% curl 'localhost:9200/_cat/master?help'
id   | node id
ip   | node transport ip address
node | node name
--------------------------------------------------

[float]
[[headers]]
=== 头部

每个命令都接受一个查询字符串参数 `h`, 它强制只显示那些列.

[source,sh]
--------------------------------------------------
% curl 'n1:9200/_cat/nodes?h=ip,port,heapPercent,name'
192.168.56.40 9300 40.3 Captain Universe
192.168.56.20 9300 15.3 Kaluu
192.168.56.50 9300 17.0 Yellowjacket
192.168.56.10 9300 12.3 Remy LeBeau
192.168.56.30 9300 43.9 Ramsey, Doug
--------------------------------------------------

你也可以使用简单的通配符来请求多个字段, 像 `/_cat/thread_pool?h=ip,bulk.*` 这样来获取以 `bulk.`开头的所有头部 (或别名).

[float]
[[numeric-formats]]
=== 数字格式

许多命令都提供了几种类型的数字输出, 要么是字节值要么是时间值. 这些类型默认使用人类可读的格式, 例如, `3.5mb` 而不是 `3763212`.  这些人类可读的值不是可排序的数字, 因此为了在这些值上做排序操作, 你可以改变它.

假设你想要找到集群中最大的索引 (所有分片使用的存储空间, 而不是文档数量). `/_cat/indices` API 很完美.  我们只需要调整两件事.  首先, 我们想要关闭人类模式.我们将会使用一个字节级别的解决方案.  然后我们将使用合适的列将输出用管道输入到 `sort` 中, 在本例中是第八个.

[source,sh]
--------------------------------------------------
% curl '192.168.56.10:9200/_cat/indices?bytes=b' | sort -rnk8
green wiki2 3 0 10000   0 105274918 105274918
green wiki1 3 0 10000 413 103776272 103776272
green foo   1 0   227   0   2065131   2065131
--------------------------------------------------


--

include::cat/alias.asciidoc[]

include::cat/allocation.asciidoc[]

include::cat/count.asciidoc[]

include::cat/fielddata.asciidoc[]

include::cat/health.asciidoc[]

include::cat/indices.asciidoc[]

include::cat/master.asciidoc[]

include::cat/nodeattrs.asciidoc[]

include::cat/nodes.asciidoc[]

include::cat/pending_tasks.asciidoc[]

include::cat/plugins.asciidoc[]

include::cat/recovery.asciidoc[]

include::cat/thread_pool.asciidoc[]

include::cat/shards.asciidoc[]

include::cat/segments.asciidoc[]
