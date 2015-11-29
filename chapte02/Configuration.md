# Configuration

Configurationedit

On this page
Environment Variables
System Configuration
Elasticsearch Settings
Index Settings
Logging
Elasticsearch Reference:
Getting Started
Setup
Configuration
Running as a Service on Linux
Running as a Service on Windows
Directory Layout
Repositories
Upgrading
Breaking changes
API Conventions
Document APIs
Search APIs
Aggregations
Indices APIs
cat APIs
Cluster APIs
Query DSL
Mapping
Analysis
Modules
Index Modules
Testing
Glossary of terms
Release Notes
Environment Variablesedit

Within the scripts, Elasticsearch comes with built in JAVA_OPTS passed to the JVM started. The most important setting for that is the -Xmx to control the maximum allowed memory for the process, and -Xms to control the minimum allocated memory for the process (in general, the more memory allocated to the process, the better).

Most times it is better to leave the default JAVA_OPTS as they are, and use the ES_JAVA_OPTS environment variable in order to set / change JVM settings or arguments.

The ES_HEAP_SIZE environment variable allows to set the heap memory that will be allocated to elasticsearch java process. It will allocate the same value to both min and max values, though those can be set explicitly (not recommended) by setting ES_MIN_MEM (defaults to 256m), and ES_MAX_MEM (defaults to 1g).

It is recommended to set the min and max memory to the same value, and enable mlockall.

System Configurationedit

File Descriptorsedit

Make sure to increase the number of open files descriptors on the machine (or for the user running elasticsearch). Setting it to 32k or even 64k is recommended.

In order to test how many open files the process can open, start it with -Des.max-open-files set to true. This will print the number of open files the process can open on startup.

Alternatively, you can retrieve the max_file_descriptors for each node using the Nodes Info API, with:

curl localhost:9200/_nodes/process?pretty
Virtual memoryedit

Elasticsearch uses a hybrid mmapfs / niofs directory by default to store its indices. The default operating system limits on mmap counts is likely to be too low, which may result in out of memory exceptions. On Linux, you can increase the limits by running the following command as root:

sysctl -w vm.max_map_count=262144
To set this value permanently, update the vm.max_map_count setting in /etc/sysctl.conf.

Note
If you installed Elasticsearch using a package (.deb, .rpm) this setting will be changed automatically. To verify, run sysctl vm.max_map_count.

Memory Settingsedit

Most operating systems try to use as much memory as possible for file system caches and eagerly swap out unused application memory, possibly resulting in the elasticsearch process being swapped. Swapping is very bad for performance and for node stability, so it should be avoided at all costs.

There are three options:

Disable swap

The simplest option is to completely disable swap. Usually Elasticsearch is the only service running on a box, and its memory usage is controlled by the ES_HEAP_SIZE environment variable. There should be no need to have swap enabled.

On Linux systems, you can disable swap temporarily by running: sudo swapoff -a. To disable it permanently, you will need to edit the /etc/fstab file and comment out any lines that contain the word swap.

On Windows, the equivalent can be achieved by disabling the paging file entirely via System Properties → Advanced → Performance → Advanced → Virtual memory.

Configure swappiness

The second option is to ensure that the sysctl value vm.swappiness is set to 0. This reduces the kernel’s tendency to swap and should not lead to swapping under normal circumstances, while still allowing the whole system to swap in emergency conditions.

Note
From kernel version 3.5-rc1 and above, a swappiness of 0 will cause the OOM killer to kill the process instead of allowing swapping. You will need to set swappiness to 1 to still allow swapping in emergencies.

mlockall

The third option is to use mlockall on Linux/Unix systems, or VirtualLock on Windows, to try to lock the process address space into RAM, preventing any Elasticsearch memory from being swapped out. This can be done, by adding this line to the config/elasticsearch.yml file:

bootstrap.mlockall: true
After starting Elasticsearch, you can see whether this setting was applied successfully by checking the value of mlockall in the output from this request:

curl http://localhost:9200/_nodes/process?pretty
If you see that mlockall is false, then it means that the the mlockall request has failed. The most probable reason, on Linux/Unix systems, is that the user running Elasticsearch doesn’t have permission to lock memory. This can be granted by running ulimit -l unlimited as root before starting Elasticsearch.

Another possible reason why mlockall can fail is that the temporary directory (usually /tmp) is mounted with the noexec option. This can be solved by specifying a new temp directory, by starting Elasticsearch with:

./bin/elasticsearch -Djna.tmpdir=/path/to/new/dir
Warning
mlockall might cause the JVM or shell session to exit if it tries to allocate more memory than is available!

Elasticsearch Settingsedit

elasticsearch configuration files can be found under ES_HOME/config folder. The folder comes with two files, the elasticsearch.yml for configuring Elasticsearch different modules, and logging.yml for configuring the Elasticsearch logging.

The configuration format is YAML. Here is an example of changing the address all network based modules will use to bind and publish to:

network :
    host : 10.0.0.4
Pathsedit

In production use, you will almost certainly want to change paths for data and log files:

path:
  logs: /var/log/elasticsearch
  data: /var/data/elasticsearch
Cluster nameedit

Also, don’t forget to give your production cluster a name, which is used to discover and auto-join other nodes:

cluster:
  name: <NAME OF YOUR CLUSTER>
Make sure that you don’t reuse the same cluster names in different environments, otherwise you might end up with nodes joining the wrong cluster. For instance you could use logging-dev, logging-stage, and logging-prod for the development, staging, and production clusters.

Node nameedit

You may also want to change the default node name for each node to something like the display hostname. By default Elasticsearch will randomly pick a Marvel character name from a list of around 3000 names when your node starts up.

node:
  name: <NAME OF YOUR NODE>
The hostname of the machine is provided in the environment variable HOSTNAME. If on your machine you only run a single elasticsearch node for that cluster, you can set the node name to the hostname using the ${...} notation:

node:
  name: ${HOSTNAME}
Internally, all settings are collapsed into "namespaced" settings. For example, the above gets collapsed into node.name. This means that its easy to support other configuration formats, for example, JSON. If JSON is a preferred configuration format, simply rename the elasticsearch.yml file to elasticsearch.json and add:

Configuration stylesedit

{
    "network" : {
        "host" : "10.0.0.4"
    }
}
It also means that its easy to provide the settings externally either using the ES_JAVA_OPTS or as parameters to the elasticsearch command, for example:

$ elasticsearch -Des.network.host=10.0.0.4
Another option is to set es.default. prefix instead of es. prefix, which means the default setting will be used only if not explicitly set in the configuration file.

Another option is to use the ${...} notation within the configuration file which will resolve to an environment setting, for example:

{
    "network" : {
        "host" : "${ES_NET_HOST}"
    }
}
Additionally, for settings that you do not wish to store in the configuration file, you can use the value ${prompt.text} or ${prompt.secret} and start Elasticsearch in the foreground. ${prompt.secret} has echoing disabled so that the value entered will not be shown in your terminal; ${prompt.text} will allow you to see the value as you type it in. For example:

node:
  name: ${prompt.text}
On execution of the elasticsearch command, you will be prompted to enter the actual value like so:

Enter value for [node.name]:
Note
Elasticsearch will not start if ${prompt.text} or ${prompt.secret} is used in the settings and the process is run as a service or in the background.

Index Settingsedit

Indices created within the cluster can provide their own settings. For example, the following creates an index with memory based storage instead of the default file system based one (the format can be either YAML or JSON):

$ curl -XPUT http://localhost:9200/kimchy/ -d \
'
index:
    refresh_interval: 5s
'
Index level settings can be set on the node level as well, for example, within the elasticsearch.yml file, the following can be set:

index :
    refresh_interval: 5s
This means that every index that gets created on the specific node started with the mentioned configuration will store the index in memory unless the index explicitly sets it. In other words, any index level settings override what is set in the node configuration. Of course, the above can also be set as a "collapsed" setting, for example:

$ elasticsearch -Des.index.refresh_interval=5s
All of the index level configuration can be found within each index module.

Loggingedit

Elasticsearch uses an internal logging abstraction and comes, out of the box, with log4j. It tries to simplify log4j configuration by using YAML to configure it, and the logging configuration file is config/logging.yml. The JSON and properties formats are also supported. Multiple configuration files can be loaded, in which case they will get merged, as long as they start with the logging. prefix and end with one of the supported suffixes (either .yml, .yaml, .json or .properties). The logger section contains the java packages and their corresponding log level, where it is possible to omit the org.elasticsearch prefix. The appender section contains the destinations for the logs. Extensive information on how to customize logging and all the supported appenders can be found on the log4j documentation.

Additional Appenders and other logging classes provided by log4j-extras are also available, out of the box.

Deprecation loggingedit

In addition to regular logging, Elasticsearch allows you to enable logging of deprecated actions. For example this allows you to determine early, if you need to migrate certain functionality in the future. By default, deprecation logging is disabled. You can enable it in the config/logging.yml file by setting the deprecation log level to DEBUG.

deprecation: DEBUG, deprecation_log_file
This will create a daily rolling deprecation log file in your log directory. Check this file regularly, especially when you intend to upgrade to a new major version.
