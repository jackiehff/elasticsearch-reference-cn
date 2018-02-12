= API约定

*elasticsearch* REST API 使用 <<modules-http,HTTP协议上的JSON>>来暴露.

本章列举出来的约定适用于整个 REST API, 除非另外指定.

* <<multi-index>>
* <<date-math-index-names>>
* <<common-options>>

--

[[multi-index]]
== 多索引

大多数引用 `index` 参数的 API 通过使用简单的 `test1,test2,test3` 记法 (或者 `_all` 表示所有索引) 支持在多索引间执行. 它还支持通配符, 例如: `test*`, 以及 "增加" ( `+`) 和 "移除" ( `-`) , 例如: `+test*,-test3`.

所有的多索引 API 都支持下面的url查询字符串参数:

`ignore_unavailable`::

控制如果任意指定的索引不可用时是否忽略, 包括不存在的或关闭的索引. 可以指定 `true` 或 `false`.

`allow_no_indices`::

控制如果通配符索引表达式没有转化成具体的索引是否失败. 可以指定 `true` 或 `false`. 例如, 如果指定了通配符表达式 `foo*` 并且没有以 `foo` 开头的索引, 那么根据这个设置请求将会失败.
当指定 `_all`, `*` 或没有索引时, 这个设置也适用. 这个设置也适用于别名, 以防止别名指向一个已关闭的索引.

`expand_wildcards`::

控制通配符索引表达式扩展到的具体那种类型的索引. 如果指定为 `open` , 那么通配符表达式只会扩展到开启的索引, 而如果指定为 `closed` , 那么通配符表达式只会扩展到已关闭的索引. 还可以指定两个值 ( `open,closed`) 来扩展到所有索引.

如果指定为 `none`, 那么通配符扩展将会被禁用. 而如果指定为 `all` , 通配符表达式将会扩展到所有索引 (这等价于指定 `open,closed`).

上述参数的默认设置取决于使用的api.

NOTE: 像 <<docs>> 以及 <<indices-aliases,单索引`alias`API>> 这样的单个索引不支持多索引.

[[date-math-index-names]]
== 索引名称中的日期数学支持

日期数学索引名称解析允许你搜索时间序列索引的一个范围, 而不是搜索所有时间序列的索引以及过滤返回结果或维护别名.限制搜索的索引数量可以减少集群负载以及提高执行性能. 例如, 如果你在每日日志中搜索错误, 你可以使用一个日期数学名称模板来限制搜索过去两天的.

几乎所有API 都有一个 `index` 参数, `index` 参数值支持日期数学.

一个日期数学索引名称使用下面的格式:

[source,txt]
----------------------------------------------------------------------
<static_name{date_math_expr{date_format|time_zone}}>
----------------------------------------------------------------------

其中:

[horizontal]
`static_name`:: 是名称的静态文本部分.
`date_math_expr`:: 是可动态地计算日期的一个动态的日期数学表达式.
`date_format`:: 是计算的日期展现的可选格式. 默认值是 `YYYY.MM.dd`.
`time_zone`:: 是可选的时区. 默认值是 `utc`.

你必须使用尖括号将日期数学索引名称表达式包住. 例如:

[source,js]
----------------------------------------------------------------------
curl -XGET 'localhost:9200/<logstash-{now/d-2d}>/_search' {
  "query" : {
    ...
  }
}
----------------------------------------------------------------------

下面的示例展示了日期数学索引名称的不同格式以及使用给定当前时间为国际协调时间2024年3月22日午时解析的最终的索引名称.

[options="header"]
|======
| 表达式                		      |解析后的值
| `<logstash-{now/d}>`      		      | `logstash-2024.03.22`
| `<logstash-{now/M}>`      		      | `logstash-2024.03.01`
| `<logstash-{now/M{YYYY.MM}}>`           | `logstash-2024.03`
| `<logstash-{now/M-1M{YYYY.MM}}>`        | `logstash-2024.02`
| `<logstash-{now/d{YYYY.MM.dd\|+12:00}}`  | `logstash-2024.03.23`
|======

为了在索引名称模板的静态文本部分中使用字符 `{` 和 `}` , 使用反斜线 `\` 进行转义, 例如:

 * `<elastic\\{ON\\}-{now/M}>` 解析为 `elastic{ON}-2024.03.01`

下面的示例展示了一个搜索过去三天的 Logstash 索引的搜索请求, 假设索引使用了默认的 Logstash 索引名称格式 `logstash-YYYY.MM.dd`.

[source,js]
----------------------------------------------------------------------
curl -XGET 'localhost:9200/<logstash-{now/d-2d}>,<logstash-{now/d-1d}>,<logstash-{now/d}>/_search' {
  "query" : {
    ...
  }
}
----------------------------------------------------------------------

[[common-options]]
== 通用选项

下面的选项可以应用于所有的 REST API.

[float]
=== Pretty返回结果

当附加 `?pretty=true` 到任何请求时, 返回的JSON串将会被漂亮的格式化 (只为了调试使用它!). 另一个选项是设置 `?format=yaml` , 它会导致结果以更易读的yaml格式返回.


[float]
=== 人类可读的输出

统计数据以一种适合人类 (比如 `"exists_time": "1h"` 或 `"size": "1kb"`)和计算机 (比如 `"exists_time_in_millis": 3600000` 或 `"size_in_bytes": 1024`)的格式返回.
人类可读的值可以通过在查询字符串后面增加 `?human=false` 来关闭. 当统计结果被监控工具消费而非用于人类消费目的时, 这是很有意义的. `human` 标识的默认值是 `false`.

[[date-math]]
[float]
=== 日期数学

大多数接受一个格式化日期值的参数 -- 像 <<query-dsl-range-query,范围查询>> `range` 查询中的 `gt` 和 `lt`, 或 <<search-aggregations-bucket-daterange-aggregation,`daterange`聚合>> 中的 `from` 和 `to`-- 理解为日期数学.

表达式使用一个基准日期作为开始, 它既可以是 `now`, 也可以是一个以 `||` 结束的日期字符串. 这个基准日期后面可以选择跟着一个或多个数学表达式:

* `+1h` - 增加1个小时
* `-1d` - 减去一天
* `/d`  - 四舍五入到最近的一天

支持的 <<time-units,时间单位>> 是: `y` (年), `M` (月), `w` (周), `d` (日), `h` (时), `m` (分) 以及 `s` (秒).

下面是一些示例:

[horizontal]
`now+1h`::              当前时间加1小时, 使用毫秒级.
`now+1h+1m`::           当前时间加1小时1分钟, 使用毫秒级.
`now+1h/d`::            当前时间加1小时, 四舍五入到最近的一天.
`2015-01-01||+1M/d`::   `2015-01-01` 加1个月, 四舍五入到最近的一天.

[float]
=== 响应过滤

所有的REST API 都接受一个 `filter_path` 参数, 它可以用来减少  elasticsearch 返回的响应. 这个参数值是一个以逗号分隔的过滤器列表, 而过滤器是使用点记法表示的:

[source,sh]
--------------------------------------------------
curl -XGET 'localhost:9200/_search?pretty&filter_path=took,hits.hits._id,hits.hits._score'
{
  "took" : 3,
  "hits" : {
    "hits" : [
      {
        "_id" : "3640",
        "_score" : 1.0
      },
      {
        "_id" : "3642",
        "_score" : 1.0
      }
    ]
  }
}
--------------------------------------------------

它还支持 `*` 通配符来匹配任何字段或字段名称的一部分:

[source,sh]
--------------------------------------------------
curl -XGET 'localhost:9200/_nodes/stats?filter_path=nodes.*.ho*'
{
  "nodes" : {
    "lvJHed8uQQu4brS-SXKsNA" : {
      "host" : "portable"
    }
  }
}
--------------------------------------------------

而且 `**` 通配符可以用来包含字段而不需要知道字段的精确路径. 例如, 我们可以使用下面这个请求来返回每个分段的 Lucene 版本:

[source,sh]
--------------------------------------------------
curl 'localhost:9200/_segments?pretty&filter_path=indices.**.version'
{
  "indices" : {
    "movies" : {
      "shards" : {
        "0" : [ {
          "segments" : {
            "_0" : {
              "version" : "5.2.0"
            }
          }
        } ],
        "2" : [ {
          "segments" : {
            "_0" : {
              "version" : "5.2.0"
            }
          }
        } ]
      }
    },
    "books" : {
      "shards" : {
        "0" : [ {
          "segments" : {
            "_0" : {
              "version" : "5.2.0"
            }
          }
        } ]
      }
    }
  }
}
--------------------------------------------------

请注意, elasticsearch 有时候会直接返回字段的原始值, 像 `_source` 字段. 如果你想要过滤 `_source` 字段, 你应该考虑像下面这样组合已经存在的 `_source` 参数 (想要了解更多详细信息参见<<get-source-filtering,Get API>>) 和 `filter_path` 参数:

[source,sh]
--------------------------------------------------
curl -XGET 'localhost:9200/_search?pretty&filter_path=hits.hits._source&_source=title'
{
  "hits" : {
    "hits" : [ {
      "_source":{"title":"Book #2"}
    }, {
      "_source":{"title":"Book #1"}
    }, {
      "_source":{"title":"Book #3"}
    } ]
  }
}
--------------------------------------------------


[float]
=== Flat设置

`flat_settings` 标识影响设置列表的展现.当 `flat_settings` 标识为 `true` 时, 设置以flat格式返回:

[source,js]
--------------------------------------------------
{
  "persistent" : { },
  "transient" : {
    "discovery.zen.minimum_master_nodes" : "1"
  }
}
--------------------------------------------------

当 `flat_settings` 标识为 `false` 时, 设置将会以一种人类更加可读的结构化的格式返回:

[source,js]
--------------------------------------------------
{
  "persistent" : { },
  "transient" : {
    "discovery" : {
      "zen" : {
        "minimum_master_nodes" : "1"
      }
    }
  }
}
--------------------------------------------------

`flat_settings` 默认设置为 `false`.

[float]
=== 参数

Rest 参数 (当使用 HTTP 协议时, 映射到 HTTP URL 参数) 遵循使用下划线风格的约定.

[float]
=== 布尔值

所有 REST API参数 (包括请求参数和JSON体) 支持将 `false`, `0`, `no` 以及 `off`
这些值作为布尔值 "false".所有其它值作为 "true". 注意, 这和被索引的文档中当做布尔字段的字段无关.

[float]
=== 数字值

所有REST API除了支持本地的 JSON 数字类型外还支持提供数字编号的参数作为 `string`.

[[time-units]]
[float]
=== 时间单位

只要需要指定时间间隔, 比如对于 `timeout` 参数, 那么这个时间间隔必须指定单位, 就像 `2d` 代表 2 天.  支持的单位有:

[horizontal]
`y`::   年
`M`::   月
`w`::   周
`d`::   日
`h`::   时
`m`::   分
`s`::   秒
`ms`::  毫秒

[[distance-units]]
[float]
=== 距离单位

只要需要指定距离, 比如 <<query-dsl-geo-distance-query>> 中的 `distance` 参数, 如果没有指定的话, 默认的单位是米. 距离可以指定其它的单位, 比如 `"1km"` 或 `"2mi"` (2 英里).

下面列出了单位的完整列表:

[horizontal]
Mile::          `mi` 或 `miles`
Yard::          `yd` 或 `yards`
Feet::          `ft` 或 `feet`
Inch::          `in` 或 `inch`
Kilometer::     `km` 或 `kilometers`
Meter::         `m` 或 `meters`
Centimeter::    `cm` 或 `centimeters`
Millimeter::    `mm` 或 `millimeters`
Nautical mile:: `NM`, `nmi` 或 `nauticalmiles`

<<query-dsl-geohash-cell-query>> 中的 `precision` 参数接受带有上面的单位的距离, 但是如果没有指定单位, 那么precision被解释为 geohash 的长度.

[[fuzziness]]
[float]
=== 模糊

有些查询和 API 通过使用 `fuzziness` 参数支持允许非精确的 _fuzzy_ 匹配. `fuzziness` 参数是上下文敏感的, 这意味着它取决于被查询字段的类型:

[float]
==== 数字, 日期和 IPv4 字段

当查询数字, 日期和 IPv4 字段, `fuzziness` 被解释为一个 `+/-` 极限. It behaves like a <<query-dsl-range-query>> where:

    -fuzziness <= 字段值 <= +fuzziness

`fuzziness` 参数可以设置成一个数字值, 比如 `2` 或 `2.0`. `date` 字段将一个long值解释成, 但是也接受一个包含时间值的字符串 -- `"1h"` -- 正如 <<time-units>> 中说明的那样. `ip` 字段接受一个 long 值或另一个 IPv4 地址 (它将会被转换成一个long 值).

[float]
==== 字符串字段

当查询 `string` 字段时, `fuzziness` 被解释为一个
http://en.wikipedia.org/wiki/Levenshtein_distance[Levenshtein编辑距离]
-- 为了和另一个字符串一样, 一个字符串需要改变的字符次数.

`fuzziness` 参数可以指定为:

`0`, `1`, `2`::

允许 Levenshtein编辑距离 (或者编辑次数) 的最大值

`AUTO`::
+
--
基于 term 的长度生成一个编辑距离. 对于以下长度:

`0..2`:: 必须精确匹配
`3..5`:: 允许编辑一次
`>5`:: 允许编辑两次

对于 `fuzziness` 通常应该优先使用 `AUTO` 值.
--

[float]
=== 返回结果大小写

所有的 REST APIs 都接受 `case` 参数. 当设置为 `camelCase` 时, 所有返回结果中的字段名称将会以驼峰式大小写风格返回, 否则,
将会使用下划线风格. 注意, 这将不适用于被索引的 source 文档.

[float]
=== 查询字符串中的请求体

对于不接受一个非POST请求的请求类库来说, 你可以将请求体当作 `source` 查询字符串参数来传递.

[[url-access-control]]
== 基于URL的访问控制

许多用户使用一个基于URL访问控制的代理来确保对 Elasticsearch 索引的访问是安全的. 对于 <<search-multi-search,multi-search>>,
<<docs-multi-get,multi-get>> 以及 <<docs-bulk,bulk>> 请求, 用户可以选择在URL中和在请求体中的每个独立的请求中指定一个索引. 这会使基于URL的访问控制变得更具挑战性.

为了防止用户覆盖了URL中已经指定的索引, 可以在 `config.yml` 文件中增加下面这个设置:

    rest.action.multi.allow_explicit_index: false

默认值是 `true`, 但是当设置为 `false` 时, Elasticsearch 将会拒绝请求体中显示地指定了索引的请求.
