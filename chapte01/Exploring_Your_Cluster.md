# Cluster Health

  Let’s start with a basic health check, which we can use to see how our cluster is doing. We’ll be using curl to do this but you can use any tool that allows you to make HTTP/REST calls. Let’s assume that we are still on the same node where we started Elasticsearch on and open another command shell window.

  To check the cluster health, we will be using the _cat API. Remember previously that our node HTTP endpoint is available at port 9200:

  <pre><code>curl 'localhost:9200/_cat/health?v'</code></pre>

  And the response:


  <pre><code>epoch      timestamp cluster       status node.total node.data shards pri relo init unassign
1394735289 14:28:09  elasticsearch green           1         1      0   0    0    0        0</code></pre>

  We can see that our cluster named "elasticsearch" is up with a green status.

  Whenever we ask for the cluster health, we either get green, yellow, or red. Green means everything is good (cluster is fully functional), yellow means all data is available but some replicas are not yet allocated (cluster is fully functional), and red means some data is not available for whatever reason. Note that even if a cluster is red, it still is partially functional (i.e. it will continue to serve search requests from the available shards) but you will likely need to fix it ASAP since you have missing data.

  Also from the above response, we can see and total of 1 node and that we have 0 shards since we have no data in it yet. Note that since we are using the default cluster name (elasticsearch) and since Elasticsearch uses unicast network discovery by default to find other nodes on the same machine, it is possible that you could accidentally start up more than one node on your computer and have them all join a single cluster. In this scenario, you may see more than 1 node in the above response.

  We can also get a list of nodes in our cluster as follows:

  <pre><code>curl 'localhost:9200/_cat/nodes?v'</code></pre>
  And the response:


  <pre><code>curl 'localhost:9200/_cat/nodes?v'
host         ip        heap.percent ram.percent load node.role master name
mwubuntu1    127.0.1.1            8           4 0.00 d         *      New Goblin</code></pre>
  Here, we can see our one node named "New Goblin", which is the single node that is currently in our cluster.

* * *

# List All Indices

Now let’s take a peek at our indices:

<pre><code>curl 'localhost:9200/_cat/indices?v'</code></pre>
And the response:

<pre><code>curl 'localhost:9200/_cat/indices?v'
health index pri rep docs.count docs.deleted store.size pri.store.size</code></pre>
Which simply means we have no indices yet in the cluster.

* * *

# Create an Index

Now let’s create an index named "customer" and then list all the indexes again:

<pre><code>curl -XPUT 'localhost:9200/customer?pretty'
curl 'localhost:9200/_cat/indices?v'</code></pre>
The first command creates the index named "customer" using the PUT verb. We simply append pretty to the end of the call to tell it to pretty-print the JSON response (if any).

And the response:

<pre><code>curl -XPUT 'localhost:9200/customer?pretty'
{
  "acknowledged" : true
}

curl 'localhost:9200/_cat/indices?v'
health index    pri rep docs.count docs.deleted store.size pri.store.size
yellow customer   5   1          0            0       495b           495b</code></pre>

The results of the second command tells us that we now have 1 index named customer and it has 5 primary shards and 1 replica (the defaults) and it contains 0 documents in it.

You might also notice that the customer index has a yellow health tagged to it. Recall from our previous discussion that yellow means that some replicas are not (yet) allocated. The reason this happens for this index is because Elasticsearch by default created one replica for this index. Since we only have one node running at the moment, that one replica cannot yet be allocated (for high availability) until a later point in time when another node joins the cluster. Once that replica gets allocated onto a second node, the health status for this index will turn to green.

* * *

# Index and Query a Document
Let’s now put something into our customer index. Remember previously that in order to index a document, we must tell Elasticsearch which type in the index it should go to.

Let’s index a simple customer document into the customer index, "external" type, with an ID of 1 as follows:

Our JSON document: { "name": "John Doe" }

<pre><code>curl -XPUT 'localhost:9200/customer/external/1?pretty' -d '
{
  "name": "John Doe"
}'</code></pre>

And the response:
<pre><code>curl -XPUT 'localhost:9200/customer/external/1?pretty' -d '
{
  "name": "John Doe"
}'
{
  "_index" : "customer",
  "_type" : "external",
  "_id" : "1",
  "_version" : 1,
  "created" : true
}</code></pre>

From the above, we can see that a new customer document was successfully created inside the customer index and the external type. The document also has an internal id of 1 which we specified at index time.

It is important to note that Elasticsearch does not require you to explicitly create an index first before you can index documents into it. In the previous example, Elasticsearch will automatically create the customer index if it didn’t already exist beforehand.

Let’s now retrieve that document that we just indexed:
<pre><code>curl -XGET 'localhost:9200/customer/external/1?pretty'</code></pre>
And the response:

<pre><code>curl -XGET 'localhost:9200/customer/external/1?pretty'
{
  "_index" : "customer",
  "_type" : "external",
  "_id" : "1",
  "_version" : 1,
  "found" : true, "_source" : { "name": "John Doe" }
}</code></pre>
Nothing out of the ordinary here other than a field, found, stating that we found a document with the requested ID 1 and another field, _source, which returns the full JSON document that we indexed from the previous step.

* * *

# Delete an Index

Now let’s delete the index that we just created and then list all the indexes again:

<pre><code>curl -XDELETE 'localhost:9200/customer?pretty'
curl 'localhost:9200/_cat/indices?v'</code></pre>
And the response:

<pre><code>curl -XDELETE 'localhost:9200/customer?pretty'
{
  "acknowledged" : true
}
curl 'localhost:9200/_cat/indices?v'
health index pri rep docs.count docs.deleted store.size pri.store.size</code></pre>
Which means that the index was deleted successfully and we are now back to where we started with nothing in our cluster.

Before we move on, let’s take a closer look again at some of the API commands that we have learned so far:

<pre><code>curl -XPUT 'localhost:9200/customer'
curl -XPUT 'localhost:9200/customer/external/1' -d '
{
  "name": "John Doe"
}'
curl 'localhost:9200/customer/external/1'
curl -XDELETE 'localhost:9200/customer'</code></pre>
If we study the above commands carefully, we can actually see a pattern of how we access data in Elasticsearch. That pattern can be summarized as follows:

<pre><code>curl -X<REST Verb> <Node>:<Port>/<Index>/<Type>/<ID>
</code></pre>
This REST access pattern is pervasive throughout all the API commands that if you can simply remember it, you will have a good head start at mastering Elasticsearch.
