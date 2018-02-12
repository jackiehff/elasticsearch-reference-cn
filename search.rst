= 搜索API

大多数搜索API是 <<search-multi-index-type,多索引&#44; 多类型>>, 除了 <<search-explain>> 端点外.

[float]
[[search-routing]]
== 路由

当执行一个搜索时, 它会被广播到所有的索引分片 (在副本之间轮询调度). 在哪个分片上执行搜索可以由提供的 `routing` 参数控制. 例如, 当索引推文时, 路由值可以是用户名:

[source,js]
--------------------------------------------------
$ curl -XPOST 'http://localhost:9200/twitter/tweet?routing=kimchy' -d '{
    "user" : "kimchy",
    "postDate" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
'
--------------------------------------------------

在这种情况下, 如果我们只想搜索特定用户的推文, 我们可以指定它作为路由值, 这样搜索只会命中相关的分片:

[source,js]
--------------------------------------------------
$ curl -XGET 'http://localhost:9200/twitter/tweet/_search?routing=kimchy' -d '{
    "query": {
        "bool" : {
            "must" : {
                "query_string" : {
                    "query" : "some query string here"
                }
            },
            "filter" : {
                "term" : { "user" : "kimchy" }
            }
        }
    }
}
'
--------------------------------------------------

路由参数可以是多个值, 由一个逗号分隔的字符串表示. 这样会导致搜索命中路由值匹配到的多个相关的分片.

[float]
[[stats-groups]]
== 统计组

搜索可以和统计组关联, 它维护了每个组的一个统计数据聚合. 后面可以使用 <<indices-stats,索引统计数据>> API专门地解析它. 例如, 下面是一个搜索体请求, 它将请求和两个不同的组关联在一起:

[source,js]
--------------------------------------------------
{
    "query" : {
        "match_all" : {}
    },
    "stats" : ["group1", "group2"]
}
--------------------------------------------------

--

include::search/search.asciidoc[]

include::search/uri-request.asciidoc[]

include::search/request-body.asciidoc[]

include::search/search-template.asciidoc[]

include::search/search-shards.asciidoc[]

include::search/suggesters.asciidoc[]

include::search/multi-search.asciidoc[]

include::search/count.asciidoc[]

include::search/exists.asciidoc[]

include::search/validate.asciidoc[]

include::search/explain.asciidoc[]

include::search/percolate.asciidoc[]

include::search/field-stats.asciidoc[]
