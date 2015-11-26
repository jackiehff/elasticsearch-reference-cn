# Setup
This section includes information on how to setup elasticsearch and get it running. If you haven’t already, download it, and then check the installation docs.

Note
Elasticsearch can also be installed from our repositories using apt or yum. See Repositories.

Supported platformsedit

The matrix of officially supported operating systems and JVMs is available here: Support Matrix. Elasticsearch is tested on the listed platforms, but it is possible that it will work on other platforms too.

Installationedit

After downloading the latest release and extracting it, elasticsearch can be started using:

$ bin/elasticsearch
On *nix systems, the command will start the process in the foreground.

Running as a daemonedit

To run it in the background, add the -d switch to it:

$ bin/elasticsearch -d
PIDedit

The Elasticsearch process can write its PID to a specified file on startup, making it easy to shut down the process later on:

$ bin/elasticsearch -d -p pid
$ kill `cat pid`


The PID is written to a file called pid.



The kill command sends a TERM signal to the PID stored in the pid file.

Note
The startup scripts provided for Linux and Windows take care of starting and stopping the Elasticsearch process for you.

*NIX

There are added features when using the elasticsearch shell script. The first, which was explained earlier, is the ability to easily run the process either in the foreground or the background.

Another feature is the ability to pass -D or getopt long style configuration parameters directly to the script. When set, all override anything set using either JAVA_OPTS or ES_JAVA_OPTS. For example:

$ bin/elasticsearch -Des.index.refresh_interval=5s --node.name=my-node
Java (JVM) versionedit

Elasticsearch is built using Java, and requires at least Java 8 in order to run. Only Oracle’s Java and the OpenJDK are supported. The same JVM version should be used on all Elasticsearch nodes and clients.

We recommend installing the Java 8 update 20 or later, or Java 7 update 55 or later. Previous versions of Java 7 are known to have bugs that can cause index corruption and data loss. Elasticsearch will refuse to start if a known-bad version of Java is used.

The version of Java to use can be configured by setting the JAVA_HOME environment variable.