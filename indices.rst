########################################
索引API
########################################

索引 API 用来管理单个的索引,索引设置, 别名, 映射, 索引模板以及warmers.

[float]
[[index-management]]
== 索引管理:

* <<indices-create-index>>
* <<indices-delete-index>>
* <<indices-get-index>>
* <<indices-exists>>
* <<indices-open-close>>

[float]
[[mapping-management]]
== 映射管理:

* <<indices-put-mapping>>
* <<indices-get-mapping>>
* <<indices-get-field-mapping>>
* <<indices-types-exists>>

[float]
[[alias-management]]
== 别名管理:
* <<indices-aliases>>

[float]
[[index-settings]]
== 索引设置:
* <<indices-update-settings>>
* <<indices-get-settings>>
* <<indices-analyze>>
* <<indices-templates>>
* <<indices-warmers>>

[float]
[[shadow-replicas]]
== 副本配置
* <<indices-shadow-replicas>>

[float]
[[monitoring]]
== 监控:
* <<indices-stats>>
* <<indices-segments>>
* <<indices-recovery>>
* <<indices-shards-stores>>

[float]
[[status-management]]
== 状态管理:
* <<indices-clearcache>>
* <<indices-refresh>>
* <<indices-flush>>
* <<indices-optimize>>
* <<indices-upgrade>>

--

include::indices/create-index.asciidoc[]

include::indices/delete-index.asciidoc[]

include::indices/get-index.asciidoc[]

include::indices/indices-exists.asciidoc[]

include::indices/open-close.asciidoc[]

include::indices/put-mapping.asciidoc[]

include::indices/get-mapping.asciidoc[]

include::indices/get-field-mapping.asciidoc[]

include::indices/types-exists.asciidoc[]

include::indices/aliases.asciidoc[]

include::indices/update-settings.asciidoc[]

include::indices/get-settings.asciidoc[]

include::indices/analyze.asciidoc[]

include::indices/templates.asciidoc[]

include::indices/warmers.asciidoc[]

include::indices/shadow-replicas.asciidoc[]

include::indices/stats.asciidoc[]

include::indices/segments.asciidoc[]

include::indices/recovery.asciidoc[]

include::indices/shard-stores.asciidoc[]

include::indices/clearcache.asciidoc[]

include::indices/flush.asciidoc[]

include::indices/refresh.asciidoc[]

include::indices/optimize.asciidoc[]

include::indices/upgrade.asciidoc[]
