= 文档 API

本节描述了下面的 CRUD API:

.单文档 API
* <<docs-index_>>
* <<docs-get>>
* <<docs-delete>>
* <<docs-update>>

.多文档 API
* <<docs-multi-get>>
* <<docs-bulk>>

NOTE: 所有CRUD API 都是单索引 API. `index` 参数接受单个索引名称或者是指向单个索引的一个 `alias`.

--

include::docs/index_.asciidoc[]

include::docs/get.asciidoc[]

include::docs/delete.asciidoc[]

include::docs/update.asciidoc[]

include::docs/multi-get.asciidoc[]

include::docs/bulk.asciidoc[]

include::docs/termvectors.asciidoc[]

include::docs/multi-termvectors.asciidoc[]
