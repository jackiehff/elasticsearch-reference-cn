########################################
入门
########################################

Elasticsearch是一个具有高可扩展性的开源的全文搜索和分析引擎. 它允许你快速且准实时地存储、搜索以及分析大规模数据.它通常用作底层的搜索引擎或技术来促使应用程序能够支持复杂的搜索功能和需求。

下面是几个Elasticsearch可以应用到的示例用例:

* 你运行着一个可以允许顾客搜索你所售卖商品的网上商店。在这种情况下, 你可以使用Elasticsearch来存储整个的产品类目以及库存信息并且为顾客提供搜索和自动完成推荐功能。
* 你想要收集日志或者交易数据并且想要分析和挖掘这些数据来用于寻找趋势、统计、汇总或者异常. 在这种情况下, 你可以使用Logstash(Elasticsearch/Logstash/Kibana栈的一部分)来收集、聚合以及解析你的数据, 接着让Logstash将这些数据插入到Elasticsearch中.一旦数据存在于Elasticsearch中, 你就可以运行搜索和聚合来挖掘你感兴趣的信息。
* 你运行一个允许为价格敏感的顾客指定一个像"我想买一款电子产品并且到下个月之内如果有商家售卖的这款电子产品的价格降到X美元以下我都想收到通知"这样的规则的价格提醒平台. 在这种情况下你可以爬取商家的价格, 然后将它们推送到Elasticsearch中，接着使用Elasticsearch的反向搜索(Percolator) 功能来匹配针对顾客查询的价格变动，最后一旦发现有匹配的价格则推送提醒给顾客。
* 你有数据分析或商业智能方面的需求并且想要快速地在大量的数据(想象一下数百万或数十亿条记录)上进行研究、分析、可视化以及找寻特定问题的答案(ask ad-hoc questions?).在这种情况下, 你可以使用Elasticsearch来存储数据，然后使用Kibana (Elasticsearch/Logstash/Kibana栈的一部分) 构建自定义仪表盘，它可以可视化展现对你来说比较重要的数据的各个方面. 另外, 你可以使用Elasticsearch的聚合功能来对数据执行复杂的商业智能查询。
本教程接下来的部分，我会指导你经历Elasticsearch的安装和运行、简单窥探Elasticsearch内部原理以及 执行像索引、搜索和修改数据等基本的一些操作这些过程. 在本教程结束的时候,你应该对Elasticsearch是什么以及它是如何工作的有一个很好的了解, 并且希望能够启发你知道如何使用它来构建复杂的搜索应用程序或是从你的数据中挖掘商业智能。


****************************************
基本概念
****************************************

Elasticsearch有几个比较核心的概念，从一开始就理解这些概念将会极大地帮助简化我们学习Elasticsearch的过程.

[float]
=== 准实时 (NRT)

Elasticsearch是一个准实时的搜索平台，这意味着从你为文档建立索引到文档可以被检索这之间会有轻微的延迟 (通常是1秒)。

[float]
=== 集群

集群是一个或多个节点（服务器）的集合，它持有你所有的数据并且在所有节点间提供联合索引和搜索的功能. 集群使用一个唯一的名称作为标识，默认是"elasticsearch".这个名称非常重要，因为一个节点只能是一个集群的一部分，如果这个节点创建时是通过集群的名称加入到集群中.

请确保你没有在不同的环境中复用相同的集群名称，否则你可能会以节点加入到错的集群中而告终。例如你可以使用logging-dev、logging-stage以及logging-prod分别命名开发环境集群、预发布环境集群以及生产环境集群。

注意，只有单个节点的集群也可以有效并且非常良好的运作。另外，你也可以拥有多个独立的集群，每一个都有唯一的集群名称。

[float]
=== 节点 (Node)

一个节点是一个单一的服务器, 它是集群的一部分, 它存储你的数据并参与集群的索引和搜索功能. 和集群一样, 节点也是由一个名称唯一标识, 在节点启动的时候会默认分配一个随机的Marvel字符串名称。
如果不想使用默认的节点名称你可以定义任何的节点名称.为了便于管理, 当你想要标识网络中的哪台服务器对应的是集群中的哪个节点时, 这个名称非常重要.

节点可以通过配置集群名称加入到一个特定的集群. 每个节点建立时会默认加入到一个名为 `elasticsearch` 的集群中, 这意味着如果你在你的网络环境中启动多个节点并且--假设它们可以发现彼此—它们都会自动的形成并加入到名为 `elasticsearch` 的单一集群 .

在单一集群中, 你想定义多少节点都可以。而且，如果在你当前的网络环境中没有其它Elasticsearch节点在运行, 启动单个节点默认会形成一个名为elasticsearch的单节点集群。

[sect2]
[float]
=== 索引 (index)

索引是具有某些类似特征的文档的一个集合. 例如, 你可以为顾客数据创建一个索引, 为产品类目创建另一个索引，还可以为订单数据创建另一个索引. 索引使用名称(字母必须全部小写)来标识，并且当在索引里的文档上执行索引、搜索、更新以及删除操作时这个名称用于引用这个索引。

在一个单一集群中, 你想定义多少索引都可以。

[float]
=== 类型 (Type)

在索引中，你可以定义一个或多个类型。类型是索引的逻辑分类或分区，并且它的语义完全取决于你.类型通常是为有公共属性集合的文档定义的. 例如, 假设你运行一个博客平台并且将你所有的数据存储在单个索引中. 在这个索引中, 你可以为用户数据定义一种类型, 为博客数据定义另一种类型，并且还为评论数据定义另一种类型。

[float]
=== 文档 (Document)

文档是可以被索引的基本信息单元。例如你可以为单个顾客创建一个文档，为单个产品创建另一个文档，并且还可以为单个订单创建另外一个文档。文档使用JSON(JavaScript Object Notation)格式表示，它是一种普遍存在的网络数据交换格式。

在一个索引或类型中，你想存储多少文档都可以。注意，尽管文档物理地存在于索引中，但它实际上必须被索引到或分配给索引中的一个类型。

[float]
=== 分片和复制 (Shards & Replicas)

一个索引中存储的大量数据很可能会超出单一节点的硬件限制. 例如, 一个拥有十亿个文档的单一索引将会占用1TB的磁盘空间, 它可能不适合放在磁盘上的单一节点或者
从单一节点独自的处理搜索请求过于缓慢.

为了解决这个问题, Elasticsearch提供了将索引细分成多个称为分片的功能.当你创建索引时可以简单地指定你想要的分片数量. 每个分片自身就是一个功能完善且独立的 "索引", 可以将其托管在集群中的任意节点上.

分片之所以很重要有以下两个主要原因:

* 它允许你水平地切分或扩展你的容量

* 它允许你跨分片(可能在多个节点上)分发和并行化操作, 从而提升性能和吞吐量

分片是如何分布的以及它的文档是如何被聚合以支持搜索请求等这些技术细节完全由Elasticsearch来管理并且对于作为用户的你来说是透明的.

在一个随时可能发生故障的网络或云环境, 故障转移机制是非常有用的且强烈推荐使用的, 它可以避免一个分片或节点莫名其妙地脱机或是消失而导致无法响应搜索请求. 出于这个目的,
Elasticsearch允许你创建一个或多个索引的分片的副本, 它们被称作复制分片, 或简称为副本.

复制之所以很重要有以下两个主要原因:

* 在一个分片或节点发生故障的情况下它可以提供高可用性. 基于这个原因, 需要注意到的是复制分片永远不会和它拷贝的原始分片或主分片分配在同一个节点上.

* 由于可以并行的在所有副本上执行搜索，它允许你水平的扩展你的搜索容量和吞吐量.


简而言之, 每个索引都可以分割成多个分片. 一个索引也可以被复制0 (意思没有副本) 次或多次.一旦被复制, 每个索引将会有主分片(最开始被复制的那个分片) 和 复制分片 (主分片的副本).
每个索引的分片和副本的数量可以在创建索引的时候定义. 索引创建之后, 你可以随时动态地修改副本的数量，但是你不能事后修改分片的数量.

Elasticsearch中每个索引默认分配了5个主分片和1个副本, 这意味着如果你的集群中至少有两个节点的话, 你的索引将会有5个主分片和另外5个复制分片 (1个完整副本) , 总共就是每个索引10个分片.

NOTE: 每个Elasticsearch分片都是一个Lucene索引. 单个Lucene索引中允许拥有的文档数量有一个最大值. 在 https://issues.apache.org/jira/browse/LUCENE-5843[`LUCENE-5843`] 这里面有提到, 这个限制是 `2,147,483,519` (= Integer.MAX_VALUE - 128) 个文档.
你可以使用 <<cat-shards,`_cat/shards`>> api监控分片数量.

掌握了这些核心概念以后, 让我开始进入有趣的部分...


****************************************
安装
****************************************

安装Elasticsearch要求JDK的版本至少是Java 7. 特别是在编写本教程时，推荐使用 Oracle JDK 1.8.0_25版本. 由于Java在不同平台上的安装过程都不一样，所以这里我们不再详细描述JDK的安装细节. 可以在 http://docs.oracle.com/javase/8/docs/technotes/guides/install/install_overview.html[Oracle网站]上找到Oracle官方推荐的安装文档。简单的说, 在安装Elasticsearch之前, 请运行如下命令检查你安装的Java版本(如果需要的话就相应地安装或升级):

.. code-block:: sh

    java -version
    echo $JAVA_HOME

一旦Java安装完成, 我们就可以下载和运行Elasticsearch了. 可以在 http://www.elastic.co/downloads[`www.elastic.co/downloads`] 上下载所有版本的二进制安装文件. 对于每个发布版本, 你都可以在 zip 或 tar 归档文件, 或者  DEB 或 RPM 包之间选择. 为了简单起见，我们就使用tar文件。

可以使用如下方式下载Elasticsearch 2.0.0 tar包 (Windows用户需要下载zip包):

.. code-block:: sh

    curl -L -O https://download.elastic.co/elasticsearch/release/org/elasticsearch/distribution/tar/elasticsearch/{version}/elasticsearch-{version}.tar.gz

然后使用如下命令解压 (Windows系统解压zip包):

.. code-block:: sh

    tar -xvf elasticsearch-{version}.tar.gz

它会在当前目录下创建一堆的文件和文件夹. 接着我们进入到bin目录下:

.. code-block:: sh

    cd elasticsearch-{version}/bin

现在我们就可以启动节点和单个集群 (Windows用户需要运行 elasticsearch.bat 文件):

.. code-block:: sh

    ./elasticsearch

如果一切顺利的话, 你会看到类似下面的一堆信息:

.. code-block:: sh

    ./elasticsearch
    [2014-03-13 13:42:17,218][INFO ][node           ] [New Goblin] version[{version}], pid[2085], build[5c03844/2014-02-25T15:52:53Z]
    [2014-03-13 13:42:17,219][INFO ][node           ] [New Goblin] initializing ...
    [2014-03-13 13:42:17,223][INFO ][plugins        ] [New Goblin] loaded [], sites []
    [2014-03-13 13:42:19,831][INFO ][node           ] [New Goblin] initialized
    [2014-03-13 13:42:19,832][INFO ][node           ] [New Goblin] starting ...
    [2014-03-13 13:42:19,958][INFO ][transport      ] [New Goblin] bound_address {inet[/0:0:0:0:0:0:0:0:9300]}, publish_address {inet[/192.168.8.112:9300]}
    [2014-03-13 13:42:23,030][INFO ][cluster.service] [New Goblin] new_master [New Goblin][rWMtGj3dQouz2r6ZFL9v4g][mwubuntu1][inet[/192.168.8.112:9300]], reason: zen-disco-join (elected_as_master)
    [2014-03-13 13:42:23,100][INFO ][discovery      ] [New Goblin] elasticsearch/rWMtGj3dQouz2r6ZFL9v4g
    [2014-03-13 13:42:23,125][INFO ][http           ] [New Goblin] bound_address {inet[/0:0:0:0:0:0:0:0:9200]}, publish_address {inet[/192.168.8.112:9200]}
    [2014-03-13 13:42:23,629][INFO ][gateway        ] [New Goblin] recovered [1] indices into cluster_state
    [2014-03-13 13:42:23,630][INFO ][node           ] [New Goblin] started

无需深入了解, 我们可以看到名为 "New Goblin" (在你的例子中将是不同的漫画人物) 的节点已经成功启动并选举她自己为单一集群中的master. 暂时还不用担心master是什么意思. 这里最重要的是我们已经在一个集群中启动了一个节点.

之前提到过我们可以修改集群或者节点的名字. 这可以通过启动Elasticsearch的时候在命令行输入以下命令完成:

.. code-block:: sh

    ./elasticsearch --cluster.name my_cluster_name --node.name my_node_name

同时注意标记为http的行带有访问节点的HTTP地址(192.168.8.112)和端口 (9200)信息. Elasticsearch默认使用9200 端口来为其REST API提供访问. 如果需要的话这个端口是可配置的。


****************************************
探索集群
****************************************

=== REST API

现在我们已经启动并运行了节点(和集群), 下一步就是理解如何与它进行通信。幸运的是, Elasticsearch提供了一套非常全面和强大的REST API, 你可以使用它来和你的集群进行交互。使用API可以做的少数几件事情如下:

* 检查集群、节点和索引的健康状况、状态以及统计信息
* 管理集群、节点以及索引数据和元数据
* 针对索引执行CRUD(Create, Read, Update和Delete)以及搜索操作
* 执行像分页、排序、筛选、脚本、聚合以及更多其它高级搜索操作


集群健康
========================================

让我们开始一个基本的健康检查, 这样我们可以了解集群的工作情况. 我们将会使用curl命令来做这件事情, 你也可以使用任何可以允许你发起HTTP或REST调用的工具. 假设我们仍然在启动Elasticsearch的相同节点上, 打开另一个shell命令窗口.

为了检查集群的健康状况, 我们将会使用 <<cat,`_cat` API>>. 记住之前我们节点的HTTP端点的访问端口是 `9200`:

.. code-block:: sh

    curl 'localhost:9200/_cat/health?v'

返回结果如下:

.. code-block:: sh

    epoch      timestamp cluster       status node.total node.data shards pri relo init unassign
    1394735289 14:28:09  elasticsearch green           1         1      0   0    0    0        0

我们可以看到名为"elasticsearch"集群的启动状态是green.

每当请求检查集群健康状况时, 我们会得到green、yellow或者red。green意思是一切正常(集群功能是完善的), yellow 意思是所有的数据都可以访问了但是某些副本
还未被分配(集群功能是完善的), red的意思是不管出于什么原因某些数据都无法访问. 请注意, 即使集群的状态是red, 它仍然有部分功能正常 (例如它会继续处理可访问的分片的搜索请求)，
但是由于你已经在丢失数据所以你想要尽可能快地修复它.

从上面的返回结果中我们还可以看到总共只有1个节点并且由于节点中还没有数据所以没有分片.请注意由于我们正在使用默认的集群名称 (elasticsearch) 并且
由于Elasticsearch默认使用单播网络发现机制来寻找同一机器上的其它节点, 所以有可能你不小心在计算机上启动了多个节点并且让它们都加入到一个集群中.
在这个场景中, 你可能会在上面的返回结果中发现多个节点.

我们可以使用如下方式获取集群中的节点列表:

.. code-block:: sh

    curl 'localhost:9200/_cat/nodes?v'

其返回结果如下:

.. code-block:: sh

    curl 'localhost:9200/_cat/nodes?v'
    host         ip        heap.percent ram.percent load node.role master name
    mwubuntu1    127.0.1.1            8           4 0.00 d         *      New Goblin

我们可以看到名为"New Goblin"的节点是我们集群中目前存在的唯一节点.


列举所有索引
========================================


现在我们来看一下所有的索引:

.. code-block:: sh

    curl 'localhost:9200/_cat/indices?v'

其返回结果如下:

.. code-block:: sh

    curl 'localhost:9200/_cat/indices?v'
    health index pri rep docs.count docs.deleted store.size pri.store.size

也就是说我们的集群中还没有任何的节点.


创建索引
========================================

现在我们创建一个名为"customer"的索引并再次列举出所有的索引:

.. code-block:: sh

    curl -XPUT 'localhost:9200/customer?pretty'
    curl 'localhost:9200/_cat/indices?v'

第一个命令使用PUT这个动作创建一个名为"customer"的索引. 我们简单的在调用的最后追加`pretty`就可以告诉它
以更加美观的方式输出JSON格式的返回结果(有的话).

其返回结果如下:

.. code-block:: sh

    curl -XPUT 'localhost:9200/customer?pretty'
    {
      "acknowledged" : true
    }

    curl 'localhost:9200/_cat/indices?v'
    health index    pri rep docs.count docs.deleted store.size pri.store.size
    yellow customer   5   1          0            0       495b           495b

第二个命令的结果告诉我们现在有1个名为customer的索引, 它有5个主分片和1个副本(缺省值), 并且它里面没有文档.

你可能还注意到了索引customer有一个yellow的健康标记.回想我们之前讨论的, yellow的意思是某些副本还未被分配.之所以customer索引会这样是因为Elasticsearch默认为其创建了一个副本.
由于目前我们只有一个节点在运行, 所以这个副本暂时还不能被分配 (为了高可用性), 一直到后面的某个时间点时另一个节点加入到集群中.一旦副本被分配到另外一个节点上, 这个索引的健康状态将会变成green.


索引并查询文档
========================================

现在我们往customer索引中放一些东西. 还记得之前说过, 为了索引一个文档, 我们必须告诉Elasticsearch应该将其放置到索引中的哪个类型中.

在下面的例子中, 我们在customer索引、"external"类型中索引一个ID为1的简单customer文档:

JSON文档为: { "name": "John Doe" }

.. code-block:: sh

    curl -XPUT 'localhost:9200/customer/external/1?pretty' -d '
    {
      "name": "John Doe"
    }'

其返回结果如下:

.. code-block:: sh

    curl -XPUT 'localhost:9200/customer/external/1?pretty' -d '
    {
      "name": "John Doe"
    }'
    {
      "_index" : "customer",
      "_type" : "external",
      "_id" : "1",
      "_version" : 1,
      "created" : true
    }

从上面我们可以看到, 在customer索引和external类型内部已经成功创建了一个新的customer文档, 该文档还有我们在索引时指定的一个值为1的内部ID.

需要注意的是Elasticsearch并没有要求你在可以索引文档之前必须先要显示地创建一个索引. 在前面示例中, 如果customer索引事先不存在, Elasticsearch会自动创建它.

现在我们来检索一下我们刚索引过的文档:

.. code-block:: sh

    curl -XGET 'localhost:9200/customer/external/1?pretty'

其返回结果如下:

.. code-block:: sh

    curl -XGET 'localhost:9200/customer/external/1?pretty'
    {
      "_index" : "customer",
      "_type" : "external",
      "_id" : "1",
      "_version" : 1,
      "found" : true, "_source" : { "name": "John Doe" }
    }

上面的返回结果中除了 `found` 字段以外没有其它与众不同的地方, 它声明了我们成功找到带有请求ID值为1的一个文档。另外一个字段 `_source`, 它返回的是我们在之前的步骤中索引过的整个的JSON文档.

删除索引
========================================

现在我们删除刚创建的索引, 然后再次列举出所有的索引:

.. code-block:: sh

    curl -XDELETE 'localhost:9200/customer?pretty'
    curl 'localhost:9200/_cat/indices?v'

其返回结果如下:

.. code-block:: sh

    curl -XDELETE 'localhost:9200/customer?pretty'
    {
      "acknowledged" : true
    }
    curl 'localhost:9200/_cat/indices?v'
    health index pri rep docs.count docs.deleted store.size pri.store.size

它的意思是索引已经删除成功, 而且我们又回到了最开始集群中什么都没有的状态.

在我们继续之前, 让我们再仔细看看目前为止我们已经学习过的一些API命令:

.. code-block:: sh

    curl -XPUT 'localhost:9200/customer'
    curl -XPUT 'localhost:9200/customer/external/1' -d '
    {
      "name": "John Doe"
    }'
    curl 'localhost:9200/customer/external/1'
    curl -XDELETE 'localhost:9200/customer'

如果我们仔细学习了上面的那些命令, 我们就会得出Elasticsearch中访问数据的一个格式. 这个格式可以总结如下:

.. code-block:: sh

    curl -X<REST Verb> <Node>:<Port>/<Index>/<Type>/<ID>

如果你可以简单地记住这个REST访问格式将贯穿于所有的API命令, 那么在掌握Elasticsearch的过程中你将会有一个良好的开端.


****************************************
数据更新
****************************************

Elasticsearch可以提供准实时的数据操作和搜索功能.从你索引/更新/删除你的数据那一刻到它出现在你的搜索结果中的那一刻之间默认会有1秒的延迟
 (刷新间隔).它与其它平台之间有个非常重要的区别，像在SQL中，一旦事务完成之后数据将会被立刻返回.

**索引/替换文档**

我们之前已经学习了如何索引单个文档. 让我们再次回顾一下这个命令:

.. code-block:: sh

    curl -XPUT 'localhost:9200/customer/external/1?pretty' -d '
    {
      "name": "John Doe"
    }'

上面的示例将再次在customer索引、"external"类型中索引一个ID为1的指定文档.
接着如果我们再次在不同的(或相同的)文档上执行上面的命令, Elasticsearch将会在现有ID为1的文档上替换(例如reindex)一个新的文档:

.. code-block:: sh

    curl -XPUT 'localhost:9200/customer/external/1?pretty' -d '
    {
      "name": "Jane Doe"
    }'

上面的示例将ID为1的文档的name从 "John Doe" 改成 "Jane Doe". 另一方面, 如果我们使用一个不同的ID, 一个新的文档将会被索引，而索引中已存在的文档将会保持不变.

.. code-block:: sh

    curl -XPUT 'localhost:9200/customer/external/2?pretty' -d '
    {
      "name": "Jane Doe"
    }'

上面的命令索引一个ID为2的新文档.

当索引的时候, ID部分是可选的. 如果不指定ID的话, Elasticsearch将会生成一个随机的ID, 然后使用这个随机的ID来索引文档.
Elasticsearch生成的实际的ID (或者在之前的示例中我们显示的指定的ID) 会作为索引API调用的部分而返回.

下面的例子展示了如何不用显示指定ID来索引一个文档:

.. code-block:: sh

    curl -XPOST 'localhost:9200/customer/external?pretty' -d '
    {
      "name": "Jane Doe"
    }'

请注意在上面的例子中, 我们使用的是POST而不是PUT，因为我们没有指定一个ID.


更新文档
========================================

除了能索引和替换文档之外, 我们还可以更新文档.请注意，Elasticsearch实际上并不是在后台做就地更新.
无论我们何时执行一个更新操作, Elasticsearch会一次性的删除旧的文档并索引一个已经应用更新的新文档.

下面的示例展示了如何通过改变name字段的值为"Jane Doe"来更新我们之前的文档(ID为1):

.. code-block:: sh

    curl -XPOST 'localhost:9200/customer/external/1/_update?pretty' -d '
    {
      "doc": { "name": "Jane Doe" }
    }'

下面的示例展示了如何通过改变name字段的值为"Jane Doe"并且同时增加一个age字段来更新我们之前的文档(ID为1):

.. code-block:: sh

    curl -XPOST 'localhost:9200/customer/external/1/_update?pretty' -d '
    {
      "doc": { "name": "Jane Doe", "age": 20 }
    }'

也可以通过使用简单的脚本来执行更新操作. 注意在 `1.4.3` 版本中像下面的动态脚本默认是禁用的, 想了解更多细节可以查看 <<modules-scripting, 脚本文档>>.
下面的示例使用脚本来将age增加5:

.. code-block:: sh

    curl -XPOST 'localhost:9200/customer/external/1/_update?pretty' -d '
    {
      "script" : "ctx._source.age += 5"
    }'

在上面的示例中, `ctx._source`指的是当前将要被更新的源文档。

注意在写本文的时候, 在单个文档上一次只能执行一次更新操作.未来Elasticsearch可能会提供通过
给定查询条件来更新多个文档的功能 (像一个 `SQL UPDATE-WHERE` 语句).


删除文档
========================================

删除一个文档相当简单. 下面的示例展示了如何删除之前ID为2的customer:

.. code-block:: sh

    curl -XDELETE 'localhost:9200/customer/external/2?pretty'

`delete-by-query`插件可以删除所有匹配一个指定查询的文档。


批处理
========================================

除了能够索引、更新以及删除单个文档之外, 通过使用<<docs-bulk,`_bulk` API>>, Elasticsearch还提供了批量执行以上任意操作的功能.
这个功能很重要, 因为它提供了一个非常高效的机制来使用更少的网络往返更快的执行多个操作.

作为一个简单的示例, 下面的调用在一个批量操作中索引了两个文档 (ID 1 - John Doe和ID 2 - Jane Doe):

.. code-block:: sh

    curl -XPOST 'localhost:9200/customer/external/_bulk?pretty' -d '
    {"index":{"_id":"1"}}
    {"name": "John Doe" }
    {"index":{"_id":"2"}}
    {"name": "Jane Doe" }
    '

下面的示例中, 在一个批量操作中更新了第一个文档(ID为1), 接着删除了第二个文档(ID为2):

.. code-block:: sh

    curl -XPOST 'localhost:9200/customer/external/_bulk?pretty' -d '
    {"update":{"_id":"1"}}
    {"doc": { "name": "John Doe becomes Jane Doe" } }
    {"delete":{"_id":"2"}}
    '

注意到上面命令中的删除操作, 它后面没有相应的源文档, 因为删除操作只需要要删除文档的ID.

批处理API依次并按顺序执行所有的操作. 无论出于何原因, 如果单个操作失败了, 它会继续执行后面剩余的操作.
当批处理API返回时, 它会为每个操作提供执行状态信息 (与发送时操作的顺序相同), 这样你就可以检查某个特定的操作是否失败.


****************************************
数据检索
****************************************

**示例数据集**

现在我们已经粗略的看了一些基础知识, 让我们尝试一个更加真实的数据集.我已经准备好了虚构的顾客银行账户信息的JSON文档样本.
每个文档都有以下schema:

.. code-block:: sh

    {
        "account_number": 0,
        "balance": 16623,
        "firstname": "Bradshaw",
        "lastname": "Mckenzie",
        "age": 29,
        "gender": "F",
        "address": "244 Columbus Place",
        "employer": "Euron",
        "email": "bradshawmckenzie@euron.com",
        "city": "Hobucken",
        "state": "CO"
    }

处于好奇, 我从 http://www.json-generator.com/[`www.json-generator.com/`] 上生成了这些数据，请忽略数据的实际值和语义，因为这些都是随机生成的.

**加载示例数据集**

你可以从 https://github.com/bly2k/files/blob/master/accounts.zip?raw=true[这里]下载示例数据集(accounts.json) .
将其解压到当前目录并且使用如下放弃将其加载到集群中:

.. code-block:: sh

    curl -XPOST 'localhost:9200/bank/account/_bulk?pretty' --data-binary "@accounts.json"
    curl 'localhost:9200/_cat/indices?v'

其返回结果如下:

.. code-block:: sh

    curl 'localhost:9200/_cat/indices?v'
    health index pri rep docs.count docs.deleted store.size pri.store.size
    yellow bank    5   1       1000            0    424.4kb        424.4kb

它的意思是我们刚刚成功地批量索引了1000个文档到bank索引中 (在account类型下).


搜索API
========================================

现在我们开始一些简单的搜索. 有两种基本的方式来运行查询: 一种是通过 <<search-uri-request,REST请求URI>> 发送查询参数，另一种
是通过<<search-request-body,REST请求主体>>发送发送查询参数. 请求主体方法允许你更具表现力并且还允许你以一种更具可读性的JSON格式
定义你的查询。我们将会尝试一个请求URI方法的示例，但是在本教程剩余部分, 我们只会使用请求主体方法。

搜索的REST API可以从 `_search` 端点访问. 下面的示例返回bank索引中的所有文档:

.. code-block:: sh

    curl 'localhost:9200/bank/_search?q=*&pretty'

我们先来仔细分析一下这个搜索调用. 我们在bank索引中执行搜索 (`_search` 端点), `q=*` 参数指示Elasticsearch去匹配索引中的所有文档.
 `pretty` 参数, 只是告诉Elasticsearch返回更易阅读的JSON结果.

其返回结果(只展示部分)如下:

.. code-block:: js

    {
      "took" : 63,
      "timed_out" : false,
      "_shards" : {
        "total" : 5,
        "successful" : 5,
        "skipped" : 0,
        "failed" : 0
      },
      "hits" : {
        "total" : 1000,
        "max_score" : null,
        "hits" : [ {
          "_index" : "bank",
          "_type" : "_doc",
          "_id" : "0",
          "sort": [0],
          "_score" : null,
          "_source" : {"account_number":0,"balance":16623,"firstname":"Bradshaw","lastname":"Mckenzie","age":29,"gender":"F","address":"244 Columbus Place","employer":"Euron","email":"bradshawmckenzie@euron.com","city":"Hobucken","state":"CO"}
        }, {
          "_index" : "bank",
          "_type" : "_doc",
          "_id" : "1",
          "sort": [1],
          "_score" : null,
          "_source" : {"account_number":1,"balance":39225,"firstname":"Amber","lastname":"Duke","age":32,"gender":"M","address":"880 Holmes Lane","employer":"Pyrami","email":"amberduke@pyrami.com","city":"Brogan","state":"IL"}
        }, ...
        ]
      }
    }

对于上面的返回结果, 我们看下下面的部分:

* `took` – Elasticsearch执行搜索耗费的时间(单位毫秒)
* `timed_out` – 告诉我们搜索是否超时
* `_shards` – 告诉我们搜索了多少个分片, 以及搜索成功或失败的分片的计数
* `hits` – 搜索结果
* `hits.total` – 匹配搜索条件的文档总数
* `hits.hits` – 实际的搜索结果数组 (默认返回前10个文档)
* `hits.sort` – sort key for results (missing if sorting by score)
* `hits._score` 和 `max_score` – 暂时忽略这些字段

下面是和上面完全相同的使用请求主体方法的搜索:

.. code-block:: sh

    GET /bank/_search
    {
      "query": { "match_all": {} },
      "sort": [
        { "account_number": "asc" }
      ]
    }

这里的区别是我们POST一个JSON风格的查询请求主体给 `_search` API, 而不是在URI中传递 `q=*` .
我们将会在下一节讨论JSON查询.


理解这一点很重要，即一旦得到你的搜索结果, Elasticsearch就完全地完成了搜索请求，并且不会维护任何类型的服务器端资源
或是打开游标到你的结果中(open cursors into your results?).
这和许多其它平台完全相反，比如在SQL中你可能最开始在前面得到你查询结果的部分子集，如果你想要获取 (或分页查询) 剩余的数据
那么你必须要使用某种有状态的服务器端游标来不断地请求服务器.


查询语言介绍
========================================

Elasticsearch提供了一种可以用来执行查询的JSON风格的领域特定语言. 它被称为 <<query-dsl,Query DSL>>.
这个查询语言非常全面并且第一眼看上去会很吓人，但是学习它的最好方式就是从一些基本的示例开始.

回到我们上一个示例, 我们执行的这个查询:

.. code-block:: js

    {
      "query": { "match_all": {} }
    }

仔细分析上面的搜索命令, `query` 部分告诉我们查询定义是什么，而 `match_all` 部分只是我们想要运行的查询类型. `match_all` 查询只是简单地在指定的索引中搜索所有的文档.

除了`query`参数外我们还可以传递其它参数来改变查询结果. 例如, 下面的语句执行了一个`match_all`查询并且仅返回了第一个文档:

.. code-block:: sh

    curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
    {
      "query": { "match_all": {} },
      "size": 1
    }'

请注意如果没有指定`size`, 它默认是10.

下面的示例执行了一个`match_all`查询并且返回了第11到第20个文档:

.. code-block:: sh

    curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
    {
      "query": { "match_all": {} },
      "from": 10,
      "size": 10
    }'

`from`参数(从0开始)指定了从哪个文档开始，而 `size` 参数指定了从from参数开始返回的文档个数. 当实现分页的搜索结果时这个功能是很有用的.
注意到如果没有指定 `from` , 它默认就是 0.

下面的示例执行了一个 `match_all` 查询并且将查询结果按照账户的余额的倒序排序，最后返回前10(默认值)个文档.

.. code-block:: sh

    curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
    {
      "query": { "match_all": {} },
      "sort": { "balance": { "order": "desc" } }
    }'


执行搜索
========================================

现在我们已经了解了几个基本的搜索参数, 让我们深入探究一下Query DSL.
我们首先看一下返回的文档字段. 整个的JSON文档默认作为所有搜索的一部分返回. 这被称为源文档 (搜索结果中的 `_source` 字段). 如果我们不想要
返回整个的源文档, 我们可以请求只返回源文档中的几个字段.

下面的示例展示了如何返回两个字段：`account_number`以及`balance` (在`_source`内), from the search:

.. code-block:: sh

    curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
    {
      "query": { "match_all": {} },
      "_source": ["account_number", "balance"]
    }'

注意到上面的示例只是简单地减少了 `_source` 字段. 它仍然只会返回一个名为 `_source` 的字段，但是在它里面只包括 `account_number` 和 `balance` 字段.

如果你学过SQL就知道上面的例子与 `SQL SELECT FROM` 字段列表 的概念有些类似.

现在让我们转移到查询部分. 之前我们已经了解到 `match_all` 查询是如何用来匹配所有文档的.
现在我们引入一种新的叫做 <<query-dsl-match-query,`match` 查询>> 的查询,它可以被认为是一个基本的分类搜索查询 (例如 一个针对某个特定字段或字段集合的搜索).

下面的示例返回account_number为20的账户:

.. code-block:: sh

    curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
    {
      "query": { "match": { "account_number": 20 } }
    }'

下面的示例返回地址中包含术语 "mill" 的所有账户:

.. code-block:: sh

    curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
    {
      "query": { "match": { "address": "mill" } }
    }'

下面的示例返回地址中包含术语 "mill" 或 "lane" 的所有账户:

.. code-block:: sh

    curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
    {
      "query": { "match": { "address": "mill lane" } }
    }'

下面的示例是 `match` (`match_phrase`) 的一个变体，它返回地址中包含 "mill lane" 短语的所有账户:

.. code-block:: sh

    curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
    {
      "query": { "match_phrase": { "address": "mill lane" } }
    }'

现在我们介绍一下 <<query-dsl-bool-query,`bool`(ean) query>>. `bool` 查询允许我们使用布尔逻辑来将较小的查询组合成较大的查询.

下面的示例由两个 `match` 查询组成并返回地址中包含 "mill" 和 "lane" 的所有账户:

.. code-block:: sh

    curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
    {
      "query": {
        "bool": {
          "must": [
            { "match": { "address": "mill" } },
            { "match": { "address": "lane" } }
          ]
        }
      }
    }'

在上面的示例中, `bool must` 子句指定了所有的查询必须为true文档才可以匹配上.

与此相反, 下面的示例由两个 `match` 查询组成并返回地址中包含"mill" 或 "lane" 的所有账户:

.. code-block:: sh

    curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
    {
      "query": {
        "bool": {
          "should": [
            { "match": { "address": "mill" } },
            { "match": { "address": "lane" } }
          ]
        }
      }
    }'

在上面的示例中, `bool should` 子句指定了一个查询列表，其中只要有一个为true那么文档就可以匹配上.

下面的示例由两个 `match` 查询组成并返回地址中既不包含 "mill" 和 "lane" 的所有账户:

.. code-block:: sh

    curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
    {
      "query": {
        "bool": {
          "must_not": [
            { "match": { "address": "mill" } },
            { "match": { "address": "lane" } }
          ]
        }
      }
    }'

在上面的示例中, `bool must_not` 子句指定了一个查询列表，其中所有都不为true文档才可以匹配上.

我们可以在一个 `bool` 查询中同时组合 `must`, `should`以及 `must_not` 语句.
另外, 我们可以在任何这些 `bool` 子句中组合 `bool` 查询以模拟任何复杂的多层次的布尔逻辑.

下面的示例返回年龄在40岁但是不住在ID(aho)的所有账户:

.. code-block:: sh

    curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
    {
      "query": {
        "bool": {
          "must": [
            { "match": { "age": "40" } }
          ],
          "must_not": [
            { "match": { "state": "ID" } }
          ]
        }
      }
    }'


执行过滤器
========================================

在之前的章节中, 我们跳过了被称为文档分数 (搜索结果中的 `_score` 字段) 的这个小细节.
这个分数是一个数值，它表示的是一个有关文档和我们指定的搜索查询匹配程度的相对测量值. 分数越高，文档的相关度越高, 分数越低, 文档的相关度也越低.

但是查询并不总是需要产生分数, 特别是当它们仅用于 "filtering" 文档集合.Elasticsearch会检测到这些情况并且自动地优化查询的执行以免进行无用的分数计算.

在上一节我们介绍的<<query-dsl-bool-query,`bool` 查询>> 也支持 `filter` 子句，
它允许我们使用一个查询来限制将被其它子句匹配到的文档数量, 而不用改变分数计算规则. 我们用一个例子来介绍一下 <<query-dsl-range-query,`range` 查询>>, 它允许我们根据值的范围来过滤
文档. 它通常用于数字或日期过滤.

下面的示例使用一个 bool 查询来返回余额在20000到30000之间的所有账户, 包括20000和30000. 换句话说, 我们想要找到余额大于等于20000并且小于等于30000的账户.

.. code-block:: sh

    curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
    {
      "query": {
        "bool": {
          "must": { "match_all": {} },
          "filter": {
            "range": {
              "balance": {
                "gte": 20000,
                "lte": 30000
              }
            }
          }
        }
      }
    }'

仔细分析一下上面的示例, bool查询包含一个 `match_all` 查询 (query部分) 和一个 `range` 查询 (filter部分).
我们可以将query和filter部分替换成其它任何查询. 在上面的例子中, 由于所有文档落入这个范围的几率都是平等的，所以
range 查询非常有意义, 例如, 没有文档比另一个更具相关性.

除了`match_all`, `match`, `bool` 以及 `range` 查询以外, 还有其它很多的查询类型我们这里就不再详细描述了.
既然对于它们是如何工作的我们已经有了一个基本的了解, 相信将这些知识应用到学习和实践其它查询类型上应该不会太难.


执行聚合
========================================

聚合提供了从你的数据中组合和分解统计数据的能力.理解聚合最简单的方式就是将其大概地等同于SQL GROUP BY 和 SQL 聚合函数.
在Elasticsearch中, 你能够执行搜索并在一次响应中返回搜索结果同时返回和搜索结果分开的聚合的结果.你只使用一个简洁和简单的API
就能执行查询和多个聚合操作并一次性得到这两个操作的返回结果以避免网络往返，这是非常强大和高效的.

下面的示例首先按照state组合所有的账户, 然后返回按照count的降序(默认排序方式)排序后的前10个(默认值)state:

.. code-block:: sh

    curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
    {
      "size": 0,
      "aggs": {
        "group_by_state": {
          "terms": {
            "field": "state"
          }
        }
      }
    }'

在SQL中, 上面的聚合在概念上类似于:

.. code-block:: sql

    SELECT state, COUNT(*) FROM bank GROUP BY state ORDER BY COUNT(*) DESC

其返回结果(只展示部分)如下:

.. code-block:: sh

  "hits" : {
    "total" : 1000,
    "max_score" : 0.0,
    "hits" : [ ]
  },
  "aggregations" : {
    "group_by_state" : {
      "buckets" : [ {
        "key" : "al",
        "doc_count" : 21
      }, {
        "key" : "tx",
        "doc_count" : 17
      }, {
        "key" : "id",
        "doc_count" : 15
      }, {
        "key" : "ma",
        "doc_count" : 15
      }, {
        "key" : "md",
        "doc_count" : 15
      }, {
        "key" : "pa",
        "doc_count" : 15
      }, {
        "key" : "dc",
        "doc_count" : 14
      }, {
        "key" : "me",
        "doc_count" : 14
      }, {
        "key" : "mo",
        "doc_count" : 14
      }, {
        "key" : "nd",
        "doc_count" : 14
      } ]
    }
  }

我们可以看到在AL(abama)中有21个账户 , 接着是TX中有17个账户 , 接着是ID(aho)中有15个账户 , 如此等等.

请注意我们通过设置 `size=0` 来隐藏搜索命中结果, 因为我们只想在返回结果中看到聚合操作结果.

在前面聚合操作的基础上, 下面的示例按照state计算了账户平均余额 (同样的只取按数量倒序排序后的前10个state):

.. code-block:: sh

    curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
    {
      "size": 0,
      "aggs": {
        "group_by_state": {
          "terms": {
            "field": "state"
          },
          "aggs": {
            "average_balance": {
              "avg": {
                "field": "balance"
              }
            }
          }
        }
      }
    }'

请注意我们是如何将 `average_balance` 聚合操作嵌入到 `group_by_state` 聚合操作中的.
对于所有聚合操作来说这是一个通用模式. 你可以随意地在聚合操作中嵌入聚合操作来从你的数据中提取你所要的汇总信息.

在前面聚合操作的基础上, 现在让我们按照降序来排序平均余额:

.. code-block:: sh

    curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
    {
      "size": 0,
      "aggs": {
        "group_by_state": {
          "terms": {
            "field": "state",
            "order": {
              "average_balance": "desc"
            }
          },
          "aggs": {
            "average_balance": {
              "avg": {
                "field": "balance"
              }
            }
          }
        }
      }
    }'

下面的示例展示了我们可以怎样按照年龄段来分组 (年龄段 20-29, 30-39以及40-49), 然后按照性别, 最后得到每个年龄段每种性别的平均账户余额:

.. code-block:: sh

    GET /bank/_search
    {
      "size": 0,
      "aggs": {
        "group_by_age": {
          "range": {
            "field": "age",
            "ranges": [
              {
                "from": 20,
                "to": 30
              },
              {
                "from": 30,
                "to": 40
              },
              {
                "from": 40,
                "to": 50
              }
            ]
          },
          "aggs": {
            "group_by_gender": {
              "terms": {
                "field": "gender.keyword"
              },
              "aggs": {
                "average_balance": {
                  "avg": {
                    "field": "balance"
                  }
                }
              }
            }
          }
        }
      }
    }

在此, 我们不再详细介绍其它更多的聚合功能, 如果你想要进一步去实践的话, <<search-aggregations,aggregations参考指南>> 将是一个非常好的起点.


****************************************
结语
****************************************

Elasticsearch是一个既简单又复杂的产品。到目前为止我们已经学习了Elasticsearch是什么、如何深入了解它以及如何利用REST APIs来使用它等这些基础知识.
我希望这篇教程已经让你对Elasticsearch是什么有了一个更好的了解, 更重要的是, 激发了你进一步去实践它所包含的其他的一些很棒的特性!
