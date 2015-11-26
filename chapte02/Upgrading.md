# Upgrading

Upgradingedit


Important
Before upgrading Elasticsearch:

Consult the breaking changes docs.
Test upgrades in a dev environment before upgrading your production cluster.
Always back up your data before upgrading. You cannot roll back to an earlier version unless you have a backup of your data.
Elasticsearch can usually be upgraded using a rolling upgrade process, resulting in no interruption of service. This section details how to perform both rolling upgrades and upgrades with full cluster restarts.

To determine whether a rolling upgrade is supported for your release, please consult this table:

Upgrade From	Upgrade To	Supported Upgrade Type
0.90.x

1.x, 2.x

Full cluster restart

< 0.90.7

0.90.x

Full cluster restart

>= 0.90.7

0.90.x

Rolling upgrade

1.0.0 - 1.3.1

1.x

Rolling upgrade (if indices.recovery.compress set to false)

>= 1.3.2

1.x

Rolling upgrade

1.x

2.x

Full cluster restart

Tip
Take plugins into consideration as well when upgrading. Most plugins will have to be upgraded alongside Elasticsearch, although some plugins accessed primarily through the browser (_site plugins) may continue to work given that API changes are compatible.




Back Up Your Data!edit

On this page
Backing up 1.0 and later
Backing up 0.90.x and earlier
Elasticsearch Reference:
Getting Started
Setup
Configuration
Running as a Service on Linux
Running as a Service on Windows
Directory Layout
Repositories
Upgrading
Back Up Your Data!
Rolling upgrades
Full cluster restart upgrade
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
Always back up your data before performing an upgrade. This will allow you to roll back in the event of a problem.  The upgrades sometimes include upgrades to the Lucene libraries used by Elasticsearch to access the index files, and after an index file has been updated to work with a new version of Lucene, it may not be accessible to the versions of Lucene present in earlier Elasticsearch releases.

Warning
Always back up your data before upgrading

You cannot roll back to an earlier version unless you have a backup of your data.

Backing up 1.0 and lateredit

To back up a running 1.0 or later system, it is simplest to use the snapshot feature. See the complete instructions for backup and restore with snapshots.

Backing up 0.90.x and earlieredit

To back up a running 0.90.x system:

Step 1: Disable index flushingedit

This will prevent indices from being flushed to disk while the backup is in process:

PUT /_all/_settings
{
  "index": {
    "translog.disable_flush": "true"
  }
}
VIEW IN SENSE
Step 2: Disable reallocationedit

This will prevent the cluster from moving data files from one node to another while the backup is in process:

PUT /_cluster/settings
{
  "transient": {
    "cluster.routing.allocation.disable_allocation": "true"
  }
}
VIEW IN SENSE
Step 3: Backup your dataedit

After reallocation and index flushing are disabled, initiate a backup of Elasticsearch’s data path using your favorite backup method (tar, storage array snapshots, backup software).

Step 4: Reenable allocation and flushingedit

When the backup is complete and data no longer needs to be read from the Elasticsearch data path, allocation and index flushing must be re-enabled:

PUT /_all/_settings
{
  "index": {
    "translog.disable_flush": "false"
  }
}

PUT /_cluster/settings
{
  "transient": {
    "cluster.routing.allocation.disable_allocation": "false"
  }
}
VIEW IN SENSE


Rolling upgradesedit

On this page
Step 1: Disable shard allocation
Step 2: Stop non-essential indexing and perform a synced flush (Optional)
Step 3: Stop and upgrade a single node
Step 4: Start the upgraded node
Step 5: Reenable shard allocation
Step 6: Wait for the node to recover
Step 7: Repeat
Elasticsearch Reference:
Getting Started
Setup
Configuration
Running as a Service on Linux
Running as a Service on Windows
Directory Layout
Repositories
Upgrading
Back Up Your Data!
Rolling upgrades
Full cluster restart upgrade
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
A rolling upgrade allows the Elasticsearch cluster to be upgraded one node at a time, with no downtime for end users. Running multiple versions of Elasticsearch in the same cluster for any length of time beyond that required for an upgrade is not supported, as shards will not be replicated from the more recent version to the older version.

Consult this table to verify that rolling upgrades are supported for your version of Elasticsearch.

To perform a rolling upgrade:

Step 1: Disable shard allocationedit

When you shut down a node, the allocation process will immediately try to replicate the shards that were on that node to other nodes in the cluster, causing a lot of wasted I/O. This can be avoided by disabling allocation before shutting down a node:

PUT /_cluster/settings
{
  "transient": {
    "cluster.routing.allocation.enable": "none"
  }
}
VIEW IN SENSE
Step 2: Stop non-essential indexing and perform a synced flush (Optional)edit

You may happily continue indexing during the upgrade. However, shard recovery will be much faster if you temporarily stop non-essential indexing and issue a synced-flush request:

POST /_flush/synced
VIEW IN SENSE
A synced flush request is a “best effort” operation. It will fail if there are any pending indexing operations, but it is safe to reissue the request multiple times if necessary.

Step 3: Stop and upgrade a single nodeedit

Shut down one of the nodes in the cluster before starting the upgrade.

Tip
When using the zip or tarball packages, the config, data, logs and plugins directories are placed within the Elasticsearch home directory by default.

It is a good idea to place these directories in a different location so that there is no chance of deleting them when upgrading Elasticsearch. These custom paths can be configured with the path.config and path.data settings.

The Debian and RPM packages place these directories in the appropriate place for each operating system.

To upgrade using a Debian or RPM package:

Use rpm or dpkg to install the new package. All files should be placed in their proper locations, and config files should not be overwritten.
To upgrade using a zip or compressed tarball:

Extract the zip or tarball to a new directory, to be sure that you don’t overwrite the config or data directories.
Either copy the files in the config directory from your old installation to your new installation, or use the --path.config option on the command line to point to an external config directory.
Either copy the files in the data directory from your old installation to your new installation, or configure the location of the data directory in the config/elasticsearch.yml file, with the path.data setting.
Step 4: Start the upgraded nodeedit

Start the now upgraded node and confirm that it joins the cluster by checking the log file or by checking the output of this request:

GET _cat/nodes
VIEW IN SENSE
Step 5: Reenable shard allocationedit

Once the node has joined the cluster, reenable shard allocation to start using the node:

PUT /_cluster/settings
{
  "transient": {
    "cluster.routing.allocation.enable": "all"
  }
}
VIEW IN SENSE
Step 6: Wait for the node to recoveredit

You should wait for the cluster to finish shard allocation before upgrading the next node. You can check on progress with the _cat/health request:

GET _cat/health
VIEW IN SENSE
Wait for the status column to move from yellow to green. Status green means that all primary and replica shards have been allocated.

Important
During a rolling upgrade, primary shards assigned to a node with the higher version will never have their replicas assigned to a node with the lower version, because the newer version may have a different data format which is not understood by the older version.

If it is not possible to assign the replica shards to another node with the higher version — e.g. if there is only one node with the higher version in the cluster — then the replica shards will remain unassigned and the cluster health will remain status yellow.

In this case, check that there are no initializing or relocating shards (the init and relo columns) before proceding.

As soon as another node is upgraded, the replicas should be assigned and the cluster health will reach status green.

Shards that have not been sync-flushed may take some time to recover. The recovery status of individual shards can be monitored with the _cat/recovery request:

GET _cat/recovery
VIEW IN SENSE
If you stopped indexing, then it is safe to resume indexing as soon as recovery has completed.

Step 7: Repeatedit

When the cluster is stable and the node has recovered, repeat the above steps for all remaining nodes.



Full cluster restart upgradeedit

On this page
Step 1: Disable shard allocation
Step 2: Perform a synced flush
Step 3: Shutdown and upgrade all nodes
Step 4: Start the cluster
Step 5: Wait for yellow
Step 6: Reenable allocation
Elasticsearch Reference:
Getting Started
Setup
Configuration
Running as a Service on Linux
Running as a Service on Windows
Directory Layout
Repositories
Upgrading
Back Up Your Data!
Rolling upgrades
Full cluster restart upgrade
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
Elasticsearch requires a full cluster restart when upgrading across major versions: from 0.x to 1.x or from 1.x to 2.x. Rolling upgrades are not supported across major versions.

The process to perform an upgrade with a full cluster restart is as follows:

Step 1: Disable shard allocationedit

When you shut down a node, the allocation process will immediately try to replicate the shards that were on that node to other nodes in the cluster, causing a lot of wasted I/O. This can be avoided by disabling allocation before shutting down a node:

PUT /_cluster/settings
{
  "persistent": {
    "cluster.routing.allocation.enable": "none"
  }
}
VIEW IN SENSE
If upgrading from 0.90.x to 1.x, then use these settings instead:

PUT /_cluster/settings
{
  "persistent": {
    "cluster.routing.allocation.disable_allocation": false,
    "cluster.routing.allocation.enable": "none"
  }
}
VIEW IN SENSE
Step 2: Perform a synced flushedit

Shard recovery will be much faster if you stop indexing and issue a synced-flush request:

POST /_flush/synced
VIEW IN SENSE
A synced flush request is a “best effort” operation. It will fail if there are any pending indexing operations, but it is safe to reissue the request multiple times if necessary.

Step 3: Shutdown and upgrade all nodesedit

Stop all Elasticsearch services on all nodes in the cluster. Each node can be upgraded following the same procedure described in Step 3: Stop and upgrade a single node.

Step 4: Start the clusteredit

If you have dedicated master nodes — nodes with node.master set to true(the default) and node.data set to false —  then it is a good idea to start them first. Wait for them to form a cluster and to elect a master before proceeding with the data nodes. You can check progress by looking at the logs.

As soon as the minimum number of master-eligible nodes have discovered each other, they will form a cluster and elect a master. From that point on, the _cat/health and _cat/nodes APIs can be used to monitor nodes joining the cluster:

GET _cat/health

GET _cat/nodes
VIEW IN SENSE
Use these APIs to check that all nodes have successfully joined the cluster.

Step 5: Wait for yellowedit

As soon as each node has joined the cluster, it will start to recover any primary shards that are stored locally. Initially, the _cat/health request will report a status of red, meaning that not all primary shards have been allocated.

Once each node has recovered its local shards, the status will become yellow, meaning all primary shards have been recovered, but not all replica shards are allocated. This is to be expected because allocation is still disabled.

Step 6: Reenable allocationedit

Delaying the allocation of replicas until all nodes have joined the cluster allows the master to allocate replicas to nodes which already have local shard copies. At this point, with all the nodes in the cluster, it is safe to reenable shard allocation:

PUT /_cluster/settings
{
  "persistent": {
    "cluster.routing.allocation.enable": "all"
  }
}
VIEW IN SENSE
If upgrading from 0.90.x to 1.x, then use these settings instead:

PUT /_cluster/settings
{
  "persistent": {
    "cluster.routing.allocation.disable_allocation": true,
    "cluster.routing.allocation.enable": "all"
  }
}
VIEW IN SENSE
The cluster will now start allocating replica shards to all data nodes. At this point it is safe to resume indexing and searching, but your cluster will recover more quickly if you can delay indexing and searching until all shards have recovered.

You can monitor progress with the _cat/health and _cat/recovery APIs:

GET _cat/health

GET _cat/recovery
VIEW IN SENSE
Once the status column in the _cat/health output has reached green, all primary and replica shards have been successfully allocated.