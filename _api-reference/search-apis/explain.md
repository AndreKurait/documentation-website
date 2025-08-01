---
layout: default
title: Explain
parent: Search APIs
nav_order: 40
redirect_from: 
 - /opensearch/rest-api/explain/
 - /api-reference/explain/
---

# Explain API
**Introduced 1.0**
{: .label .label-purple }

If you want to know why a specific document ranks higher (or lower) in a given query, you can use the Explain API to produce an explanation of how the relevance score (`_score`) was calculated for every result.

OpenSearch uses a probabilistic ranking framework called [Okapi BM25](https://en.wikipedia.org/wiki/Okapi_BM25) to calculate relevance scores. Okapi BM25 is based on the original [TF/IDF](https://lucene.apache.org/core/{{site.lucene_version}}/core/org/apache/lucene/search/package-summary.html#scoring) framework used by Apache Lucene.

Using the Explain API is expensive in terms of both resources and time. For production clusters, we recommend using it sparingly for the purpose of troubleshooting.
{: .warning }


## Endpoints

```json
GET <index>/_explain/<id>
POST <index>/_explain/<id>
```

## Path parameters

Parameter | Type | Description | Required
:--- | :--- | :--- | :---
`<index>` | String | Name of the index. You can only specify a single index. | Yes
`<id>` | String | A unique identifier to attach to the document. | Yes

## Query parameters

You must specify the index and document ID. All other parameters are optional.

Parameter | Type | Description | Required
:--- | :--- | :--- | :---
`analyzer` | String | The analyzer to use for the `q` query string. Only valid when `q` is used. | No
`analyze_wildcard` | Boolean | Whether to analyze wildcard and prefix queries in the `q` string. Only valid when `q` is used. Default is `false`. | No
`default_operator` | String | The default Boolean operator (`AND` or `OR`) for the `q` query string. Only valid when `q` is used. Default is `OR`.  | No
`df` | String | The default field to search if no field is specified in the `q` string. Only valid when `q` is used. | No
`lenient` | Boolean | Specifies whether OpenSearch should ignore format-based query failures (for example, querying a text field for an integer). Default is `false`. | No
`preference` | String | Specifies a preference of which shard to retrieve results from. Available options are `_local`, which tells the operation to retrieve results from a locally allocated shard replica, and a custom string value assigned to a specific shard replica. By default, OpenSearch executes the explain operation on random shards. | No
`q` | String | A query string in [Lucene syntax]({{site.url}}{{site.baseurl}}/query-dsl/full-text/query-string/#query-string-syntax). When used, you can configure query behavior using the `analyzer`, `analyze_wildcard`, `default_operator`, `df`, and `stored_fields` parameters. | No
`stored_fields` | String | A comma-separated list of stored fields to return. If omitted, only `_source` is returned. | No
`routing` | String | Value used to route the operation to a specific shard. | No
`_source` | String | Whether to include the `_source` field in the response body. Default is `true`. | No
`_source_excludes` | String | A comma-separated list of source fields to exclude in the query response. | No
`_source_includes` | String | A comma-separated list of source fields to include in the query response. | No

## Example requests

To see the explain output for all results, set the `explain` flag to `true` either in the URL or in the body of the request:

```json
POST opensearch_dashboards_sample_data_ecommerce/_search?explain=true
{
  "query": {
    "match": {
      "customer_first_name": "Mary"
    }
  }
}
```
{% include copy-curl.html %}

More often, you want the output for a single document. In that case, specify the document ID in the URL:

```json
POST opensearch_dashboards_sample_data_ecommerce/_explain/EVz1Q3sBgg5eWQP6RSte
{
  "query": {
    "match": {
      "customer_first_name": "Mary"
    }
  }
}
```
{% include copy-curl.html %}

## Example response

```json
{
  "_index" : "kibana_sample_data_ecommerce",
  "_id" : "EVz1Q3sBgg5eWQP6RSte",
  "matched" : true,
  "explanation" : {
    "value" : 3.5671005,
    "description" : "weight(customer_first_name:mary in 1) [PerFieldSimilarity], result of:",
    "details" : [
      {
        "value" : 3.5671005,
        "description" : "score(freq=1.0), computed as boost * idf * tf from:",
        "details" : [
          {
            "value" : 2.2,
            "description" : "boost",
            "details" : [ ]
          },
          {
            "value" : 3.4100041,
            "description" : "idf, computed as log(1 + (N - n + 0.5) / (n + 0.5)) from:",
            "details" : [
              {
                "value" : 154,
                "description" : "n, number of documents containing term",
                "details" : [ ]
              },
              {
                "value" : 4675,
                "description" : "N, total number of documents with field",
                "details" : [ ]
              }
            ]
          },
          {
            "value" : 0.47548598,
            "description" : "tf, computed as freq / (freq + k1 * (1 - b + b * dl / avgdl)) from:",
            "details" : [
              {
                "value" : 1.0,
                "description" : "freq, occurrences of term within document",
                "details" : [ ]
              },
              {
                "value" : 1.2,
                "description" : "k1, term saturation parameter",
                "details" : [ ]
              },
              {
                "value" : 0.75,
                "description" : "b, length normalization parameter",
                "details" : [ ]
              },
              {
                "value" : 1.0,
                "description" : "dl, length of field",
                "details" : [ ]
              },
              {
                "value" : 1.1206417,
                "description" : "avgdl, average length of field",
                "details" : [ ]
              }
            ]
          }
        ]
      }
    ]
  }
}
```

## Response body fields

Field | Description
:--- | :---
`matched` | Indicates if the document is a match for the query.
`explanation` | The `explanation` object has three properties: `value`, `description`, and `details`. The `value` shows the result of the calculation, the `description` explains what type of calculation is performed, and the `details` shows any subcalculations performed.
Term frequency (`tf`) | How many times the term appears in a field for a given document. The more times the term occurs the higher is the relevance score.
Inverse document frequency (`idf`) | How often the term appears within the index (across all the documents). The more often the term appears the lower is the relevance score.
Field normalization factor (`fieldNorm`) | The length of the field. OpenSearch assigns a higher relevance score to a term appearing in a relatively short field.

The `tf`, `idf`, and `fieldNorm` values are calculated and stored at index time when a document is added or updated. The values might have some (typically small) inaccuracies as it’s based on summing the samples returned from each shard.

Individual queries include other factors for calculating the relevance score, such as term proximity, fuzziness, and so on.
