= Analysis

The index analysis module acts as a configurable registry of Analyzers
that can be used in order to both break indexed (analyzed) fields when a
document is indexed and process query strings. It maps to the Lucene
`Analyzer`.


Analyzers are composed of a single <<analysis-tokenizers,Tokenizer>> 
and zero or more <<analysis-tokenfilters,TokenFilters>>. The tokenizer may 
be preceded by one or more <<analysis-charfilters,CharFilters>>. The
analysis module allows one to register `TokenFilters`, `Tokenizers` and
`Analyzers` under logical names that can then be referenced either in
mapping definitions or in certain APIs. The Analysis module
automatically registers (*if not explicitly defined*) built in
analyzers, token filters, and tokenizers.

Here is a sample configuration:

[source,js]
--------------------------------------------------
index :
    analysis :
        analyzer : 
            standard : 
                type : standard
                stopwords : [stop1, stop2]
            myAnalyzer1 :
                type : standard
                stopwords : [stop1, stop2, stop3]
                max_token_length : 500
            # configure a custom analyzer which is 
            # exactly like the default standard analyzer
            myAnalyzer2 :
                tokenizer : standard
                filter : [standard, lowercase, stop]
        tokenizer :
            myTokenizer1 :
                type : standard
                max_token_length : 900
            myTokenizer2 :
                type : keyword
                buffer_size : 512
        filter :
            myTokenFilter1 :
                type : stop
                stopwords : [stop1, stop2, stop3, stop4]
            myTokenFilter2 :
                type : length
                min : 0
                max : 2000
--------------------------------------------------

[float]
[[backwards-compatibility]]
=== Backwards compatibility

All analyzers, tokenizers, and token filters can be configured with a
`version` parameter to control which Lucene version behavior they should
use. Possible values are: `3.0` - `3.6`, `4.0` - `4.3` (the highest
version number is the default option).

--

include::analysis/analyzers.asciidoc[]

include::analysis/tokenizers.asciidoc[]

include::analysis/tokenfilters.asciidoc[]

include::analysis/charfilters.asciidoc[]

include::analysis/icu-plugin.asciidoc[]

