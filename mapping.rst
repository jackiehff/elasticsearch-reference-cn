= Mapping

Mapping is the process of defining how a document, and the fields it contains,
are stored and indexed.  For instance, use mappings to define:

* which string fields should be treated as full text fields.
* which fields contain numbers, dates, or geolocations.
* whether the values of all fields in the document should be
  indexed into the catch-all <<mapping-all-field,`_all`>> field.
* the <<mapping-date-format,format>> of date values.
* custom rules to control the mapping for
  <<dynamic-mapping,dynamically added fields>>.

[float]
[[mapping-type]]
== Mapping Types

Each index has one or more _mapping types_, which are used to divide the
documents in an index into logical groups. User documents might be stored in a
`user` type, and blog posts in a `blogpost` type.

Each mapping type has:

<<mapping-fields,Meta-fields>>::

Meta-fields are used to customize how a document's metadata associated is
treated. Examples of meta-fields include the document's
<<mapping-index-field,`_index`>>, <<mapping-type-field,`_type`>>,
<<mapping-id-field,`_id`>>,  and <<mapping-source-field,`_source`>> fields.

<<mapping-types,Fields>> or _properties_::

Each mapping type contains a list of fields or `properties` pertinent to that
type.  A `user` type might contain `title`, `name`, and `age` fields, while a
`blogpost` type might contain `title`, `body`, `user_id` and `created` fields.
Fields with the same name in different mapping types in the same index
<<field-conflicts,must have the same mapping>>.


[float]
== Field datatypes

Each field has a data `type` which can be:

* a simple type like <<string,`string`>>, <<date,`date`>>, <<number,`long`>>,
  <<number,`double`>>, <<boolean,`boolean`>> or <<ip,`ip`>>.
* a type which supports the hierarchical nature of JSON such as
  <<object,`object`>> or <<nested,`nested`>>.
* or a specialised type like <<geo-point,`geo_point`>>,
  <<geo-shape,`geo_shape`>>, or <<search-suggesters-completion,`completion`>>.

It is often useful to index the same field in different ways for different
purposes. For instance, a `string` field could be <<mapping-index,indexed>> as
an `analyzed` field for full-text search, and as a `not_analyzed` field for
sorting or aggregations.  Alternatively, you could index a string field with
the <<analysis-standard-analyzer,`standard` analyzer>>, the
<<english-analyzer,`english`>> analyzer, and the
<<french-analyzer,`french` analyzer>>.

This is the purpose of _multi-fields_.  Most datatypes support multi-fields
via the <<multi-fields>> parameter.

[float]
== Dynamic mapping

Fields and mapping types do not need to be defined before being used. Thanks
to _dynamic mapping_, new mapping types and new field names will be added
automatically, just by indexing a document. New fields can be added both to
the top-level mapping type, and to inner <<object,`object`>>  and
<<nested,`nested`>> fields.

The
<<dynamic-mapping,dynamic mapping>> rules can be configured to
customise the mapping that is used for new types and new fields.

[float]
== Explicit mappings

You know more about your data than Elasticsearch can guess, so while dynamic
mapping can be useful to get started, at some point you will want to specify
your own explicit mappings.

You can create mapping types and field mappings when you
<<indices-create-index,create an index>>, and you can add mapping types and
fields to an existing index with the <<indices-put-mapping,PUT mapping API>>.

[float]
== Updating existing mappings

Other than where documented, *existing type and field mappings cannot be
updated*. Changing the mapping would mean invalidating already indexed
documents.  Instead, you should create a new index with the correct mappings
and reindex your data into that index.

[[field-conflicts]]
[float]
== Fields are shared across mapping types

Mapping types are used to group fields, but the fields in each mapping type
are not independent of each other. Fields with:

* the _same name_
* in the _same index_
* in _different mapping types_
* map to the _same field_ internally,
* and *must have the same mapping*.

If a `title` field exists in both the `user` and `blogpost` mapping types, the
`title` fields must have exactly the same mapping in each type.  The only
exceptions to this rule are the <<copy-to>>, <<dynamic>>, <<enabled>>,
<<ignore-above>>, <<include-in-all>>, and <<properties>> parameters, which may
have different settings per field.

Usually, fields with the same name also contain the same type of data, so
having the same mapping is not a problem.  When conflicts do arise, these can
be solved by choosing more descriptive names, such as `user_title` and
`blog_title`.

[float]
== Example mapping

A mapping for the example described above could be specified when creating the
index, as follows:

[source,js]
---------------------------------------
PUT my_index <1>
{
  "mappings": {
    "user": { <2>
      "_all":       { "enabled": false  }, <3>
      "properties": { <4>
        "title":    { "type": "string"  }, <5>
        "name":     { "type": "string"  }, <5>
        "age":      { "type": "integer" }  <5>
      }
    },
    "blogpost": { <2>
      "properties": { <4>
        "title":    { "type": "string"  }, <5>
        "body":     { "type": "string"  }, <5>
        "user_id":  {
          "type":   "string", <5>
          "index":  "not_analyzed"
        },
        "created":  {
          "type":   "date", <5>
          "format": "strict_date_optional_time||epoch_millis"
        }
      }
    }
  }
}
---------------------------------------
// AUTOSENSE
<1> Create an index called `my_index`.
<2> Add mapping types called `user` and `blogpost`.
<3> Disable the `_all` <<mapping-fields,meta field>> for the `user` mapping type.
<4> Specify fields or _properties_ in each mapping type.
<5> Specify the data `type` and mapping for each field.


--

include::mapping/types.asciidoc[]

include::mapping/fields.asciidoc[]

include::mapping/params.asciidoc[]

include::mapping/dynamic-mapping.asciidoc[]

include::mapping/transform.asciidoc[]
