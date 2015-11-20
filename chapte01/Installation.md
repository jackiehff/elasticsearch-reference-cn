安装Elasticsearch要求JDK的版本至少是Java 7. 特别是在编写本教程时，推荐使用 Oracle JDK 1.8.0_25版本. 由于Java在不同平台上的安装过程都不一样，所以这里我们不再详细描述JDK的安装细节. 可以在Oracle网站上找到Oracle官方推荐的安装文档. 简单的说, 在安装Elasticsearch之前, 请运行如下命令检查你安装的Java版本(如果需要的话就相应地安装或升级):

<pre><code>java -version
echo $JAVA_HOME
</code></pre>

一旦Java安装完成, 我们就可以下载和运行Elasticsearch了. 可以在www.elastic.co/downloads上下载所有版本的二进制安装文件. 对于每个发布版本, 你都可以在 zip 或 tar 归档文件, 或者  DEB 或 RPM 包之间选择. 为了简单起见，我们就使用tar文件.

可以使用如下方式下载Elasticsearch 2.0.0 tar包 (Windows用户需要下载zip包):

<pre><code>curl -L -O https://download.elastic.co/elasticsearch/release/org/elasticsearch/distribution/tar/elasticsearch/2.0.0/elasticsearch-2.0.0.tar.gz</code></pre>


然后使用如下命令解压 (Windows系统解压zip包):

<pre><code>tar -xvf elasticsearch-2.0.0.tar.gz</code></pre>

它会在当前目录下创建一堆的文件和文件夹. 接着我们进入到bin目录下:

<pre><code>cd elasticsearch-2.0.0/bin</code></pre>

现在我们就可以启动节点和单个集群 (Windows用户需要运行 elasticsearch.bat 文件):

<pre><code>./elasticsearch</code></pre>

如果一切顺利的话, 你会看到类似下面的一堆信息:
<pre><code>./elasticsearch
[2014-03-13 13:42:17,218][INFO ][node           ] [New Goblin] version[2.0.0], pid[2085], build[5c03844/2014-02-25T15:52:53Z]
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
</code></pre>

无需深入了解, 我们可以看到名为 "New Goblin" (在你的例子中将是不同的漫画人物) 的节点已经成功启动并选举她自己为单一集群中的master. 暂时还不用担心master是什么意思. 这里最重要的是我们已经在一个集群中启动了一个节点.

之前提到过我们可以修改集群或者节点的名字. 这可以通过启动Elasticsearch的时候在命令行输入以下命令完成:
<pre><code>./elasticsearch --cluster.name my_cluster_name --node.name my_node_name
</code></pre>

同时注意标记为http的行带有访问节点的HTTP地址(192.168.8.112)和端口 (9200)信息. Elasticsearch默认使用9200 端口来为其REST API提供访问. 如果需要的话这个端口是可配置的.