########################################
安装Elasticsearch
########################################

本节包括如何安装和运行 *elasticsearch* 等内容. 如果你还没有安装过Elasticsearch, 那么首先
 http://www.elastic.co/downloads[下载]它, 然后查阅 <<setup-installation,安装>> 文档.

NOTE: Elasticsearch也可以通过使用 `apt` 或者 `yum` 从我们的仓库安装.参见 <<setup-repositories,仓库>>.

== 支持的平台

官方所支持的操作系统和JVM矩阵可以在这里找到:
link:/support/matrix[平台支持矩阵].  Elasticsearch已经在列举出来的平台上测试过, 但是它也有可能工作于其它平台上.

== 安装

在 link:/downloads/elasticsearch[下载] 了Elasticsearch最新发布的版本并解压之后, 可以使用如下命令来启动
*elasticsearch* :

.. code-block:: sh

    $ bin/elasticsearch

在类Unix系统上, 该命令会在前台启动Elasticsearch进程.

=== 作为守护进程运行

如果想要在后台运行它, 可以增加 `-d` 选项:

.. code-block:: sh

    $ bin/elasticsearch -d

=== PID

Elasticsearch进程在启动的时候可以将其PID写到一个指定的文件中, 这样便于后面关闭这个进程:

.. code-block:: sh

    $ bin/elasticsearch -d -p pid <1>
    $ kill `cat pid` <2>

<1> PID被写入到 `pid` 文件中.
<2> `kill` 命令给存储在 `pid` 文件中的PID发送了一个 `TERM` 信号.

NOTE: 为 <<setup-service,Linux>> 和 <<setup-service-win,Windows>>系统提供的启动脚本可以帮你处理Elasticsearch进程的启动和停止.

.*NIX
当使用 `elasticsearch` 的 shell 脚本时有几个新增的特性.
第一个我们之前已经描述过, 就是能够很简单的在前台或后台运行Elasticsearch进程.

另一个特性是能够直接给脚本传递 `-D` 或 getopt 长风格配置参数. 如果设置了, 都将覆盖使用 `JAVA_OPTS` 或 `ES_JAVA_OPTS` 设置的任何参数. 例如:

.. code-block:: sh

    $ bin/elasticsearch -Des.index.refresh_interval=5s --node.name=my-node

== Java (JVM) 版本

Elasticsearch使用的是 Java 构建的, 并且如果想要运行它, 需要至少使用
http://www.oracle.com/technetwork/java/javase/downloads/index.html[Java 8] . Elasticsearch 仅支持 Oracle Java 和 OpenJDK. 所有 Elasticsearch 节点和客户端
应该使用相同的 JVM 版本.

我们推荐安装 *Java 8 update 20 或 后续* 版本, 或者 *Java 7 update 55 或后续* 版本.
Java 7之前的版本有已知的会导致索引损坏以及数据丢失的bug. 如果使用一个已知的不良版本的Java, Elasticsearch会拒绝启动.

使用的Java版本可以通过设置 `JAVA_HOME` 环境变量来配置.


include::setup/configuration.asciidoc[]

include::setup/as-a-service.asciidoc[]

include::setup/as-a-service-win.asciidoc[]

include::setup/dir-layout.asciidoc[]

include::setup/repositories.asciidoc[]

include::setup/upgrade.asciidoc[]
