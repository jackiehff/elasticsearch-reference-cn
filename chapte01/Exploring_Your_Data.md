# Exploring Your Data
Exploring Your Dataedit

On this page
Sample Dataset
Loading the Sample Dataset
Elasticsearch Reference:
Getting Started
Basic Concepts
Installation
Exploring Your Cluster
Modifying Your Data
Exploring Your Data
The Search API
Introducing the Query Language
Executing Searches
Executing Filters
Executing Aggregations
Conclusion
Setup
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
Sample Datasetedit

Now that we’ve gotten a glimpse of the basics, let’s try to work on a more realistic dataset. I’ve prepared a sample of fictitious JSON documents of customer bank account information. Each document has the following schema:

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
For the curious, I generated this data from www.json-generator.com/ so please ignore the actual values and semantics of the data as these are all randomly generated.

Loading the Sample Datasetedit

You can download the sample dataset (accounts.json) from here. Extract it to our current directory and let’s load it into our cluster as follows:

curl -XPOST 'localhost:9200/bank/account/_bulk?pretty' --data-binary "@accounts.json"
curl 'localhost:9200/_cat/indices?v'
And the response:

curl 'localhost:9200/_cat/indices?v'
health index pri rep docs.count docs.deleted store.size pri.store.size
yellow bank    5   1       1000            0    424.4kb        424.4kb
Which means that we just successfully bulk indexed 1000 documents into the bank index (under the account type).



The Search APIedit


Now let’s start with some simple searches. There are two basic ways to run searches: one is by sending search parameters through the REST request URI and the other by sending them through the REST request body. The request body method allows you to be more expressive and also to define your searches in a more readable JSON format. We’ll try one example of the request URI method but for the remainder of this tutorial, we will exclusively be using the request body method.

The REST API for search is accessible from the _search endpoint. This example returns all documents in the bank index:

curl 'localhost:9200/bank/_search?q=*&pretty'
Let’s first dissect the search call. We are searching (_search endpoint) in the bank index, and the q=* parameter instructs Elasticsearch to match all documents in the index. The pretty parameter, again, just tells Elasticsearch to return pretty-printed JSON results.

And the response (partially shown):

curl 'localhost:9200/bank/_search?q=*&pretty'
{
  "took" : 63,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "failed" : 0
  },
  "hits" : {
    "total" : 1000,
    "max_score" : 1.0,
    "hits" : [ {
      "_index" : "bank",
      "_type" : "account",
      "_id" : "1",
      "_score" : 1.0, "_source" : {"account_number":1,"balance":39225,"firstname":"Amber","lastname":"Duke","age":32,"gender":"M","address":"880 Holmes Lane","employer":"Pyrami","email":"amberduke@pyrami.com","city":"Brogan","state":"IL"}
    }, {
      "_index" : "bank",
      "_type" : "account",
      "_id" : "6",
      "_score" : 1.0, "_source" : {"account_number":6,"balance":5686,"firstname":"Hattie","lastname":"Bond","age":36,"gender":"M","address":"671 Bristol Street","employer":"Netagy","email":"hattiebond@netagy.com","city":"Dante","state":"TN"}
    }, {
      "_index" : "bank",
      "_type" : "account",
As for the response, we see the following parts:

took – time in milliseconds for Elasticsearch to execute the search
timed_out – tells us if the search timed out or not
_shards – tells us how many shards were searched, as well as a count of the successful/failed searched shards
hits – search results
hits.total – total number of documents matching our search criteria
hits.hits – actual array of search results (defaults to first 10 documents)
_score and max_score - ignore these fields for now
Here is the same exact search above using the alternative request body method:

curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
{
  "query": { "match_all": {} }
}'
The difference here is that instead of passing q=* in the URI, we POST a JSON-style query request body to the _search API. We’ll discuss this JSON query in the next section.

And the response (partially shown):

curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
{
  "query": { "match_all": {} }
}'
{
  "took" : 26,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "failed" : 0
  },
  "hits" : {
    "total" : 1000,
    "max_score" : 1.0,
    "hits" : [ {
      "_index" : "bank",
      "_type" : "account",
      "_id" : "1",
      "_score" : 1.0, "_source" : {"account_number":1,"balance":39225,"firstname":"Amber","lastname":"Duke","age":32,"gender":"M","address":"880 Holmes Lane","employer":"Pyrami","email":"amberduke@pyrami.com","city":"Brogan","state":"IL"}
    }, {
      "_index" : "bank",
      "_type" : "account",
      "_id" : "6",
      "_score" : 1.0, "_source" : {"account_number":6,"balance":5686,"firstname":"Hattie","lastname":"Bond","age":36,"gender":"M","address":"671 Bristol Street","employer":"Netagy","email":"hattiebond@netagy.com","city":"Dante","state":"TN"}
    }, {
      "_index" : "bank",
      "_type" : "account",
      "_id" : "13",
It is important to understand that once you get your search results back, Elasticsearch is completely done with the request and does not maintain any kind of server-side resources or open cursors into your results. This is in stark contrast to many other platforms such as SQL wherein you may initially get a partial subset of your query results up-front and then you have to continuously go back to the server if you want to fetch (or page through) the rest of the results using some kind of stateful server-side cursor.



Introducing the Query Languageedit

Elasticsearch Reference:
Getting Started
Basic Concepts
Installation
Exploring Your Cluster
Modifying Your Data
Exploring Your Data
The Search API
Introducing the Query Language
Executing Searches
Executing Filters
Executing Aggregations
Conclusion
Setup
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
Elasticsearch provides a JSON-style domain-specific language that you can use to execute queries. This is referred to as the Query DSL. The query language is quite comprehensive and can be intimidating at first glance but the best way to actually learn it is to start with a few basic examples.

Going back to our last example, we executed this query:

{
  "query": { "match_all": {} }
}
Dissecting the above, the query part tells us what our query definition is and the match_all part is simply the type of query that we want to run. The match_all query is simply a search for all documents in the specified index.

In addition to the query parameter, we also can pass other parameters to influence the search results. For example, the following does a match_all and returns only the first document:

curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
{
  "query": { "match_all": {} },
  "size": 1
}'
Note that if size is not specified, it defaults to 10.

This example does a match_all and returns documents 11 through 20:

curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
{
  "query": { "match_all": {} },
  "from": 10,
  "size": 10
}'
The from parameter (0-based) specifies which document index to start from and the size parameter specifies how many documents to return starting at the from parameter. This feature is useful when implementing paging of search results. Note that if from is not specified, it defaults to 0.

This example does a match_all and sorts the results by account balance in descending order and returns the top 10 (default size) documents.

curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
{
  "query": { "match_all": {} },
  "sort": { "balance": { "order": "desc" } }
}'



Executing Searchesedit

Elasticsearch Reference:
Getting Started
Basic Concepts
Installation
Exploring Your Cluster
Modifying Your Data
Exploring Your Data
The Search API
Introducing the Query Language
Executing Searches
Executing Filters
Executing Aggregations
Conclusion
Setup
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
Now that we have seen a few of the basic search parameters, let’s dig in some more into the Query DSL. Let’s first take a look at the returned document fields. By default, the full JSON document is returned as part of all searches. This is referred to as the source (_source field in the search hits). If we don’t want the entire source document returned, we have the ability to request only a few fields from within source to be returned.

This example shows how to return two fields, account_number and balance (inside of _source), from the search:

curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
{
  "query": { "match_all": {} },
  "_source": ["account_number", "balance"]
}'
Note that the above example simply reduces the _source field. It will still only return one field named _source but within it, only the fields account_number and balance are included.

If you come from a SQL background, the above is somewhat similar in concept to the SQL SELECT FROM field list.

Now let’s move on to the query part. Previously, we’ve seen how the match_all query is used to match all documents. Let’s now introduce a new query called the match query, which can be thought of as a basic fielded search query (i.e. a search done against a specific field or set of fields).

This example returns the account numbered 20:

curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
{
  "query": { "match": { "account_number": 20 } }
}'
This example returns all accounts containing the term "mill" in the address:

curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
{
  "query": { "match": { "address": "mill" } }
}'
This example returns all accounts containing the term "mill" or "lane" in the address:

curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
{
  "query": { "match": { "address": "mill lane" } }
}'
This example is a variant of match (match_phrase) that returns all accounts containing the phrase "mill lane" in the address:

curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
{
  "query": { "match_phrase": { "address": "mill lane" } }
}'
Let’s now introduce the bool(ean) query. The bool query allows us to compose smaller queries into bigger queries using boolean logic.

This example composes two match queries and returns all accounts containing "mill" and "lane" in the address:

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
In the above example, the bool must clause specifies all the queries that must be true for a document to be considered a match.

In contrast, this example composes two match queries and returns all accounts containing "mill" or "lane" in the address:

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
In the above example, the bool should clause specifies a list of queries either of which must be true for a document to be considered a match.

This example composes two match queries and returns all accounts that contain neither "mill" nor "lane" in the address:

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
In the above example, the bool must_not clause specifies a list of queries none of which must be true for a document to be considered a match.

We can combine must, should, and must_not clauses simultaneously inside a bool query. Furthermore, we can compose bool queries inside any of these bool clauses to mimic any complex multi-level boolean logic.

This example returns all accounts of anybody who is 40 years old but don’t live in ID(aho):

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



Executing Filtersedit

In the previous section, we skipped over a little detail called the document score (_score field in the search results). The score is a numeric value that is a relative measure of how well the document matches the search query that we specified. The higher the score, the more relevant the document is, the lower the score, the less relevant the document is.

But queries do not always need to produce scores, in particular when they are only used for "filtering" the document set. Elasticsearch detects these situations and automatically optimizes query execution in order not to compute useless scores.

The bool query that we introduced in the previous section also supports filter clauses which allow to use a query to restrict the documents that will be matched by other clauses, without changing how scores are computed. As an example, let’s introduce the range query, which allows us to filter documents by a range of values. This is generally used for numeric or date filtering.

This example uses a bool query to return all accounts with balances between 20000 and 30000, inclusive. In other words, we want to find accounts with a balance that is greater than or equal to 20000 and less than or equal to 30000.

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
Dissecting the above, the bool query contains a match_all query (the query part) and a range query (the filter part). We can substitute any other queries into the query and the filter parts. In the above case, the range query makes perfect sense since documents falling into the range all match "equally", i.e., no document is more relevant than another.

In addition to the match_all, match, bool, and range queries, there are a lot of other query types that are available and we won’t go into them here. Since we already have a basic understanding of how they work, it shouldn’t be too difficult to apply this knowledge in learning and experimenting with the other query types.



Executing Aggregationsedit

Elasticsearch Reference:
Getting Started
Basic Concepts
Installation
Exploring Your Cluster
Modifying Your Data
Exploring Your Data
The Search API
Introducing the Query Language
Executing Searches
Executing Filters
Executing Aggregations
Conclusion
Setup
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
Aggregations provide the ability to group and extract statistics from your data. The easiest way to think about aggregations is by roughly equating it to the SQL GROUP BY and the SQL aggregate functions. In Elasticsearch, you have the ability to execute searches returning hits and at the same time return aggregated results separate from the hits all in one response. This is very powerful and efficient in the sense that you can run queries and multiple aggregations and get the results back of both (or either) operations in one shot avoiding network roundtrips using a concise and simplified API.

To start with, this example groups all the accounts by state, and then returns the top 10 (default) states sorted by count descending (also default):

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
In SQL, the above aggregation is similar in concept to:

SELECT state, COUNT(*) FROM bank GROUP BY state ORDER BY COUNT(*) DESC
And the response (partially shown):

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
}
We can see that there are 21 accounts in AL(abama), followed by 17 accounts in TX, followed by 15 accounts in ID(aho), and so forth.

Note that we set size=0 to not show search hits because we only want to see the aggregation results in the response.

Building on the previous aggregation, this example calculates the average account balance by state (again only for the top 10 states sorted by count in descending order):

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
Notice how we nested the average_balance aggregation inside the group_by_state aggregation. This is a common pattern for all the aggregations. You can nest aggregations inside aggregations arbitrarily to extract pivoted summarizations that you require from your data.

Building on the previous aggregation, let’s now sort on the average balance in descending order:

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
This example demonstrates how we can group by age brackets (ages 20-29, 30-39, and 40-49), then by gender, and then finally get the average account balance, per age bracket, per gender:

curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
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
            "field": "gender"
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
}'
There are a many other aggregations capabilities that we won’t go into detail here. The aggregations reference guide is a great starting point if you want to do further experimentation.



Conclusionedit

Elasticsearch is both a simple and complex product. We’ve so far learned the basics of what it is, how to look inside of it, and how to work with it using some of the REST APIs. I hope that this tutorial has given you a better understanding of what Elasticsearch is and more importantly, inspired you to further experiment with the rest of its great features!

