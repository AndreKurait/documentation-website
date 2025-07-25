---
layout: default
title: Profile
parent: Search APIs
nav_order: 55
redirect_from:
  - /api-reference/profile/
---

# Profile API
**Introduced 1.0**
{: .label .label-purple }

The Profile API provides timing information about the execution of individual components of a search request. Using the Profile API, you can debug slow requests and understand how to improve their performance. The Profile API does not measure the following:

- Network latency
- Time spent in the search fetch phase
- Amount of time a request spends in queues
- Idle time while merging shard responses on the coordinating node

The Profile API is a resource-consuming operation that adds overhead to search operations.
{: .warning}

## Endpoints

```json
GET /testindex/_search
{
  "profile": true,
  "query" : {
    "match" : { "title" : "wind" }
  }
}
```

## Concurrent segment search

Starting in OpenSearch 2.12, [concurrent segment search]({{site.url}}{{site.baseurl}}/search-plugins/concurrent-segment-search/) allows each shard-level request to search segments in parallel during the query phase. The Profile API response contains several additional fields with statistics about _slices_.

A slice is the unit of work that can be executed by a thread. Each query can be partitioned into multiple slices, with each slice containing one or more segments. All the slices can be executed either in parallel or in some order depending on the available threads in the pool.

In general, the max/min/avg slice time captures statistics across all slices for a timing type. For example, when profiling aggregations, the `max_slice_time_in_nanos` field in the `aggregations` section shows the maximum time consumed by the aggregation operation and its children across all slices. 

## Example requests

### Non-concurrent search

To use the Profile API, include the `profile` parameter set to `true` in the search request sent to the `_search` endpoint:

```json
GET /testindex/_search
{
  "profile": true,
  "query" : {
    "match" : { "title" : "wind" }
  }
}
```
{% include copy-curl.html %}

To turn on human-readable format, include the `?human=true` query parameter in the request:

```json
GET /testindex/_search?human=true
{
  "profile": true,
  "query" : {
    "match" : { "title" : "wind" }
  }
}
```
{% include copy-curl.html %}

The response contains an additional `time` field with human-readable units, for example:

```json
"collector": [
    {
        "name": "SimpleTopScoreDocCollector",
        "reason": "search_top_hits",
        "time": "113.7micros",
        "time_in_nanos": 113711
    }
]
```

The Profile API response is verbose, so if you're running the request through the `curl` command, include the `?pretty` query parameter to make the response easier to understand.
{: .tip}

### Aggregations

To profile aggregations, send an aggregation request and provide the `profile` parameter set to `true`.

#### Global aggregation

```json
GET /opensearch_dashboards_sample_data_ecommerce/_search
{
  "profile": "true",
  "size": 0,
  "query": {
    "match": { "manufacturer": "Elitelligence" }
  },
  "aggs": {
    "all_products": {
      "global": {}, 
      "aggs": {     
      "avg_price": { "avg": { "field": "taxful_total_price" } }
      }
    },
    "elitelligence_products": { "avg": { "field": "taxful_total_price" } }
  }
}
```
{% include copy-curl.html %}

#### Non-global aggregation

```json
GET /opensearch_dashboards_sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "avg_taxful_total_price": {
      "avg": {
        "field": "taxful_total_price"
      }
    }
  }
}
```
{% include copy-curl.html %}


## Example response

### Non-concurrent search

The response contains profiling information:

<details markdown="block">
  <summary>
    Response
  </summary>
  {: .text-delta}

```json
{
  "took": 21,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 2,
      "relation": "eq"
    },
    "max_score": 0.19363807,
    "hits": [
      {
        "_index": "testindex",
        "_id": "1",
        "_score": 0.19363807,
        "_source": {
          "title": "The wind rises"
        }
      },
      {
        "_index": "testindex",
        "_id": "2",
        "_score": 0.17225474,
        "_source": {
          "title": "Gone with the wind",
          "description": "A 1939 American epic historical film"
        }
      }
    ]
  },
  "profile": {
    "shards": [
      {
        "id": "[LidyZ1HVS-u93-73Z49dQg][testindex][0]",
        "inbound_network_time_in_millis": 0,
        "outbound_network_time_in_millis": 0,
        "searches": [
          {
            "query": [
              {
                "type": "BooleanQuery",
                "description": "title:wind title:rise",
                "time_in_nanos": 2473919,
                "breakdown": {
                  "set_min_competitive_score_count": 0,
                  "match_count": 0,
                  "shallow_advance_count": 0,
                  "set_min_competitive_score": 0,
                  "next_doc": 5209,
                  "match": 0,
                  "next_doc_count": 2,
                  "score_count": 2,
                  "compute_max_score_count": 0,
                  "compute_max_score": 0,
                  "advance": 9209,
                  "advance_count": 2,
                  "score": 20751,
                  "build_scorer_count": 4,
                  "create_weight": 1404458,
                  "shallow_advance": 0,
                  "create_weight_count": 1,
                  "build_scorer": 1034292
                },
                "children": [
                  {
                    "type": "TermQuery",
                    "description": "title:wind",
                    "time_in_nanos": 813581,
                    "breakdown": {
                      "set_min_competitive_score_count": 0,
                      "match_count": 0,
                      "shallow_advance_count": 0,
                      "set_min_competitive_score": 0,
                      "next_doc": 3291,
                      "match": 0,
                      "next_doc_count": 2,
                      "score_count": 2,
                      "compute_max_score_count": 0,
                      "compute_max_score": 0,
                      "advance": 7208,
                      "advance_count": 2,
                      "score": 18666,
                      "build_scorer_count": 6,
                      "create_weight": 616375,
                      "shallow_advance": 0,
                      "create_weight_count": 1,
                      "build_scorer": 168041
                    }
                  },
                  {
                    "type": "TermQuery",
                    "description": "title:rise",
                    "time_in_nanos": 191083,
                    "breakdown": {
                      "set_min_competitive_score_count": 0,
                      "match_count": 0,
                      "shallow_advance_count": 0,
                      "set_min_competitive_score": 0,
                      "next_doc": 0,
                      "match": 0,
                      "next_doc_count": 0,
                      "score_count": 0,
                      "compute_max_score_count": 0,
                      "compute_max_score": 0,
                      "advance": 0,
                      "advance_count": 0,
                      "score": 0,
                      "build_scorer_count": 2,
                      "create_weight": 188625,
                      "shallow_advance": 0,
                      "create_weight_count": 1,
                      "build_scorer": 2458
                    }
                  }
                ]
              }
            ],
            "rewrite_time": 192417,
            "collector": [
              {
                "name": "SimpleTopScoreDocCollector",
                "reason": "search_top_hits",
                "time_in_nanos": 77291
              }
            ]
          }
        ],
        "aggregations": []
      }
    ]
  }
}
```
</details>

### Concurrent segment search

The following is an example response for a concurrent segment search with three segment slices:

<details markdown="block">
  <summary>
    Response
  </summary>
  {: .text-delta}

```json
{
  "took": 10,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 5,
      "relation": "eq"
    },
    "max_score": 1.0,
    "hits": [
      ...
    ]
  },
  "aggregations": {
    ...
  },
  "profile": {
    "shards": [
      {
        "id": "[9Y7lbpaWRhyr5Y-41Zl48g][idx][0]",
        "inbound_network_time_in_millis": 0,
        "outbound_network_time_in_millis": 0,
        "searches": [
          {
            "query": [
              {
                "type": "MatchAllDocsQuery",
                "description": "*:*",
                "time_in_nanos": 868000,
                "max_slice_time_in_nanos": 19376,
                "min_slice_time_in_nanos": 12250,
                "avg_slice_time_in_nanos": 16847,
                "breakdown": {
                  "max_match": 0,
                  "set_min_competitive_score_count": 0,
                  "match_count": 0,
                  "avg_score_count": 1,
                  "shallow_advance_count": 0,
                  "next_doc": 29708,
                  "min_build_scorer": 3125,
                  "score_count": 5,
                  "compute_max_score_count": 0,
                  "advance": 0,
                  "min_set_min_competitive_score": 0,
                  "min_advance": 0,
                  "score": 29250,
                  "avg_set_min_competitive_score_count": 0,
                  "min_match_count": 0,
                  "avg_score": 333,
                  "max_next_doc_count": 3,
                  "max_compute_max_score_count": 0,
                  "avg_shallow_advance": 0,
                  "max_shallow_advance_count": 0,
                  "set_min_competitive_score": 0,
                  "min_build_scorer_count": 2,
                  "next_doc_count": 8,
                  "min_match": 0,
                  "avg_next_doc": 888,
                  "compute_max_score": 0,
                  "min_set_min_competitive_score_count": 0,
                  "max_build_scorer": 5791,
                  "avg_match_count": 0,
                  "avg_advance": 0,
                  "build_scorer_count": 6,
                  "avg_build_scorer_count": 2,
                  "min_next_doc_count": 2,
                  "min_shallow_advance_count": 0,
                  "max_score_count": 2,
                  "avg_match": 0,
                  "avg_compute_max_score": 0,
                  "max_advance": 0,
                  "avg_shallow_advance_count": 0,
                  "avg_set_min_competitive_score": 0,
                  "avg_compute_max_score_count": 0,
                  "avg_build_scorer": 4027,
                  "max_set_min_competitive_score_count": 0,
                  "advance_count": 0,
                  "max_build_scorer_count": 2,
                  "shallow_advance": 0,
                  "min_compute_max_score": 0,
                  "max_match_count": 0,
                  "create_weight_count": 1,
                  "build_scorer": 32459,
                  "max_set_min_competitive_score": 0,
                  "max_compute_max_score": 0,
                  "min_shallow_advance": 0,
                  "match": 0,
                  "max_shallow_advance": 0,
                  "avg_advance_count": 0,
                  "min_next_doc": 708,
                  "max_advance_count": 0,
                  "min_score": 291,
                  "max_next_doc": 999,
                  "create_weight": 1834,
                  "avg_next_doc_count": 2,
                  "max_score": 376,
                  "min_compute_max_score_count": 0,
                  "min_score_count": 1,
                  "min_advance_count": 0
                }
              }
            ],
            "rewrite_time": 8126,
            "collector": [
              {
                "name": "QueryCollectorManager",
                "reason": "search_multi",
                "time_in_nanos": 564708,
                "reduce_time_in_nanos": 1251042,
                "max_slice_time_in_nanos": 121959,
                "min_slice_time_in_nanos": 28958,
                "avg_slice_time_in_nanos": 83208,
                "slice_count": 3,
                "children": [
                  {
                    "name": "SimpleTopDocsCollectorManager",
                    "reason": "search_top_hits",
                    "time_in_nanos": 500459,
                    "reduce_time_in_nanos": 840125,
                    "max_slice_time_in_nanos": 22168,
                    "min_slice_time_in_nanos": 5792,
                    "avg_slice_time_in_nanos": 12084,
                    "slice_count": 3
                  },
                  {
                    "name": "NonGlobalAggCollectorManager: [histo]",
                    "reason": "aggregation",
                    "time_in_nanos": 552167,
                    "reduce_time_in_nanos": 311292,
                    "max_slice_time_in_nanos": 95333,
                    "min_slice_time_in_nanos": 18416,
                    "avg_slice_time_in_nanos": 66249,
                    "slice_count": 3
                  }
                ]
              }
            ]
          }
        ],
        "aggregations": [
          {
            "type": "NumericHistogramAggregator",
            "description": "histo",
            "time_in_nanos": 2847834,
            "max_slice_time_in_nanos": 117374,
            "min_slice_time_in_nanos": 20624,
            "avg_slice_time_in_nanos": 75597,
            "breakdown": {
              "min_build_leaf_collector": 9500,
              "build_aggregation_count": 3,
              "post_collection": 3209,
              "max_collect_count": 2,
              "initialize_count": 3,
              "reduce_count": 0,
              "avg_collect": 17055,
              "max_build_aggregation": 26000,
              "avg_collect_count": 1,
              "max_build_leaf_collector": 64833,
              "min_build_leaf_collector_count": 1,
              "build_aggregation": 41125,
              "min_initialize": 583,
              "max_reduce": 0,
              "build_leaf_collector_count": 3,
              "avg_reduce": 0,
              "min_collect_count": 1,
              "avg_build_leaf_collector_count": 1,
              "avg_build_leaf_collector": 45000,
              "max_collect": 24625,
              "reduce": 0,
              "avg_build_aggregation": 12013,
              "min_post_collection": 292,
              "max_initialize": 1333,
              "max_post_collection": 750,
              "collect_count": 5,
              "avg_post_collection": 541,
              "avg_initialize": 986,
              "post_collection_count": 3,
              "build_leaf_collector": 86833,
              "min_collect": 6250,
              "min_build_aggregation": 3541,
              "initialize": 2786791,
              "max_build_leaf_collector_count": 1,
              "min_reduce": 0,
              "collect": 29834
            },
            "debug": {
              "total_buckets": 1
            }
          }
        ]
      }
    ]
  }
}
```
</details>

## Response body fields

The response includes the following fields.

Field | Data type | Description
:--- | :--- | :---
`profile` | Object | Contains profiling information.
`profile.shards` | Array of objects | A search request can be executed against one or more shards in the index, and a search may involve one or more indexes. Thus, the `profile.shards` array contains profiling information for each shard that was involved in the search.
`profile.shards.id` | String | The shard ID of the shard in the `[node-ID][index-name][shard-ID]` format.
`profile.shards.searches` | Array of objects | A search represents a query executed against the underlying Lucene index. Most search requests execute a single search against a Lucene index, but some search requests can execute more than one search. For example, including a global aggregation results in a secondary `match_all` query for the global context. The `profile.shards` array contains profiling information about each search execution.
[`profile.shards.searches.query`](#the-query-array) | Array of objects | Profiling information about the query execution.
`profile.shards.searches.rewrite_time` | Integer | All Lucene queries are rewritten. A query and its children may be rewritten more than once, until the query stops changing. The rewriting process involves performing optimizations, such as removing redundant clauses or replacing a query path with a more efficient one. After the rewriting process, the original query may change significantly. The `rewrite_time` field contains the cumulative total rewrite time for the query and all its children, in nanoseconds.
[`profile.shards.searches.collector`](#the-collector-array) | Array of objects | Profiling information about the Lucene collectors that ran the search.
[`profile.shards.aggregations`](#aggregations) | Array of objects | Profiling information about the aggregation execution.

### The `query` array

The `query` array contains objects with the following fields.

Field | Data type | Description
:--- | :--- | :---
`type` | String | The Lucene query type into which the search query was rewritten. Corresponds to the Lucene class name (which often has the same name in OpenSearch).
`description` | String | Contains a Lucene explanation of the query. Helps differentiate queries with the same type.
`time_in_nanos`	| Long | The total elapsed time for this query, in nanoseconds. For concurrent segment search, `time_in_nanos` is the total time spent across all the slices (the difference between the last completed slice execution end time and the first slice execution start time).
`max_slice_time_in_nanos`	| Long | The maximum amount of time taken by any slice to run a query, in nanoseconds. This field is included only if you enable concurrent segment search.	
`min_slice_time_in_nanos`	| Long | The minimum amount of time taken by any slice to run a query, in nanoseconds. This field is included only if you enable concurrent segment search.
`avg_slice_time_in_nanos`	| Long | The average amount of time taken by any slice to run a query, in nanoseconds. This field is included only if you enable concurrent segment search.
[`breakdown`](#the-breakdown-object) | Object | Contains timing statistics about low-level Lucene execution.
`children` | Array of objects | If a query has subqueries (children), this field contains information about the subqueries.

### The `breakdown` object

The `breakdown` object represents the timing statistics about low-level Lucene execution, broken down by method. Timings are listed in wall-clock nanoseconds and are not normalized. The `breakdown` timings are inclusive of all child times. The `breakdown` object comprises the following fields. All fields contain integer values.

Field | Description
:--- | :--- 
`create_weight` | A `Query` object in Lucene is immutable. Yet, Lucene should be able to reuse `Query` objects in multiple `IndexSearcher` objects. Thus, `Query` objects need to keep temporary state and statistics associated with the index in which the query is executed. To achieve reuse, every `Query` object generates a `Weight` object, which keeps the temporary context (state) associated with the `<IndexSearcher, Query>` tuple. The `create_weight` field contains the amount of time spent creating the `Weight` object.
`build_scorer` | A `Scorer` iterates over matching documents and generates a score for each document. The `build_scorer` field contains the amount of time spent generating the `Scorer` object. This does not include the time spent scoring the documents. The `Scorer` initialization time depends on the optimization and complexity of a particular query. The `build_scorer` parameter also includes the amount of time associated with caching, if caching is applicable and enabled for the query.
`next_doc` | The `next_doc` Lucene method returns the document ID of the next document that matches the query. This method is a special type of the `advance` method and is equivalent to `advance(docId() + 1)`. The `next_doc` method is more convenient for many Lucene queries. The `next_doc` field contains the amount of time required to determine the next matching document, which varies depending on the query type.  
`advance` | The `advance` method is a lower-level version of the `next_doc` method in Lucene. It also finds the next matching document but necessitates that the calling query perform additional tasks, such as identifying skips. Some queries, such as conjunctions (`must` clauses in Boolean queries), cannot use `next_doc`. For those queries, `advance` is timed.
`match` | For some queries, document matching is performed in two steps. First, the document is matched approximately. Second, those documents that are approximately matched are examined through a more comprehensive process. For example, a phrase query first checks whether a document contains all terms in the phrase. Next, it verifies that the terms are in order (which is a more expensive process). The `match` field is non-zero only for those queries that use the two-step verification process. 
`score` | Contains the time taken for a `Scorer` to score a particular document.
`shallow_advance` | Contains the amount of time required to execute the `advanceShallow` Lucene method.
`compute_max_score` | Contains the amount of time required to execute the `getMaxScore` Lucene method.
`set_min_competitive_score` | Contains the amount of time required to execute the `setMinCompetitiveScore` Lucene method.
`<method>_count` | Contains the number of invocations of a `<method>`. For example, `advance_count` contains the number of invocations of the `advance` method. Different invocations of the same method occur because the method is called on different documents. You can determine the selectivity of a query by comparing counts in different query components.
`max_<method>`	| The maximum amount of time taken by any slice to run a query method. Breakdown stats for the `create_weight` method do not include profiled `max` time because the method runs at the query level rather than the slice level. This field is included only if you enable concurrent segment search.
`min_<method>`	| The minimum amount of time taken by any slice to run a query method. Breakdown stats for the `create_weight` method do not include profiled `min` time because the method runs at the query level rather than the slice level. This field is included only if you enable concurrent segment search.
`avg_<method>`	| The average amount of time taken by any slice to run a query method. Breakdown stats for the `create_weight` method do not include profiled `avg` time because the method runs at the query level rather than the slice level. This field is included only if you enable concurrent segment search.
`max_<method>_count`	| The maximum number of invocations of a `<method>` on any slice. Breakdown stats for the `create_weight` method do not include profiled `max` count because the method runs at the query level rather than the slice level. This field is included only if you enable concurrent segment search.
`min_<method>_count`	| The minimum number of invocations of a `<method>` on any slice. Breakdown stats for the `create_weight` method do not include profiled `min` count because the method runs at the query level rather than the slice level. This field is included only if you enable concurrent segment search.
`avg_<method>_count`	| The average number of invocations of a `<method>` on any slice. Breakdown stats for the `create_weight` method do not include profiled `avg` count because the method runs at the query level rather than the slice level. This field is included only if you enable concurrent segment search.

### The `collector` array

The `collector` array contains information about Lucene Collectors. A Collector is responsible for coordinating document traversal and scoring and collecting matching documents. Using Collectors, individual queries can record aggregation results and execute global queries or post-query filters. 

Field | Description
:--- | :--- 
`name` | The collector name. In the [example response](#example-response), the `collector` is a single `SimpleTopScoreDocCollector`---the default scoring and sorting collector.
`reason` | Contains a description of the collector. For possible field values, see [Collector reasons](#collector-reasons).
`time_in_nanos` | The total elapsed time for this collector, in nanoseconds. For concurrent segment search, `time_in_nanos` is the total amount of time across all slices (the difference between the last completed slice execution end time and the first slice execution start time).
`children` | If a collector has subcollectors (children), this field contains information about the subcollectors.
`max_slice_time_in_nanos`	|The maximum amount of time taken by any slice, in nanoseconds.	This field is included only if you enable concurrent segment search.
`min_slice_time_in_nanos`	|The minimum amount of time taken by any slice, in nanoseconds.	This field is included only if you enable concurrent segment search.
`avg_slice_time_in_nanos`	|The average amount of time taken by any slice, in nanoseconds.	This field is included only if you enable concurrent segment search.
`slice_count`	|The total slice count for this query. This field is included only if you enable concurrent segment search.
`reduce_time_in_nanos`	|The amount of time taken to reduce results for all slice collectors, in nanoseconds.	This field is included only if you enable concurrent segment search.

Collector times are calculated, combined, and normalized independently, so they are independent of query times.
{: .note}

#### Collector reasons

The following table describes all available collector reasons.

Reason | Description
:--- | :--- 
`search_sorted` | A collector that scores and sorts documents. Present in most simple searches.
`search_count` | A collector that counts the number of matching documents but does not fetch the source. Present when `size: 0` is specified.
`search_terminate_after_count` | A collector that searches for matching documents and terminates the search when it finds a specified number of documents. Present when the `terminate_after_count` query parameter is specified.
`search_min_score` | A collector that returns matching documents that have a score greater than a minimum score. Present when the `min_score` parameter is specified.
`search_multi` | A wrapper collector for other collectors. Present when search, aggregations, global aggregations, and post filters are combined in a single search.
`search_timeout` | A collector that stops running after a specified period of time. Present when a `timeout` parameter is specified.
`aggregation` | A collector for aggregations that is run against the specified query scope. OpenSearch uses a single `aggregation` collector to collect documents for all aggregations.
`global_aggregation` | A collector that is run against the global query scope. Global scope is different from a specified query scope, so in order to collect the entire dataset, a `match_all` query must be run.

### Aggregation responses

#### Response: Global aggregation

The response contains profiling information:

<details markdown="block">
  <summary>
    Response
  </summary>
  {: .text-delta}

```json
{
  "took": 10,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 1370,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "all_products": {
      "doc_count": 4675,
      "avg_price": {
        "value": 75.05542864304813
      }
    },
    "elitelligence_products": {
      "value": 68.4430200729927
    }
  },
  "profile": {
    "shards": [
      {
        "id": "[LidyZ1HVS-u93-73Z49dQg][opensearch_dashboards_sample_data_ecommerce][0]",
        "inbound_network_time_in_millis": 0,
        "outbound_network_time_in_millis": 0,
        "searches": [
          {
            "query": [
              {
                "type": "ConstantScoreQuery",
                "description": "ConstantScore(manufacturer:elitelligence)",
                "time_in_nanos": 1367487,
                "breakdown": {
                  "set_min_competitive_score_count": 0,
                  "match_count": 0,
                  "shallow_advance_count": 0,
                  "set_min_competitive_score": 0,
                  "next_doc": 634321,
                  "match": 0,
                  "next_doc_count": 1370,
                  "score_count": 0,
                  "compute_max_score_count": 0,
                  "compute_max_score": 0,
                  "advance": 173250,
                  "advance_count": 2,
                  "score": 0,
                  "build_scorer_count": 4,
                  "create_weight": 132458,
                  "shallow_advance": 0,
                  "create_weight_count": 1,
                  "build_scorer": 427458
                },
                "children": [
                  {
                    "type": "TermQuery",
                    "description": "manufacturer:elitelligence",
                    "time_in_nanos": 1174794,
                    "breakdown": {
                      "set_min_competitive_score_count": 0,
                      "match_count": 0,
                      "shallow_advance_count": 0,
                      "set_min_competitive_score": 0,
                      "next_doc": 470918,
                      "match": 0,
                      "next_doc_count": 1370,
                      "score_count": 0,
                      "compute_max_score_count": 0,
                      "compute_max_score": 0,
                      "advance": 172084,
                      "advance_count": 2,
                      "score": 0,
                      "build_scorer_count": 4,
                      "create_weight": 114041,
                      "shallow_advance": 0,
                      "create_weight_count": 1,
                      "build_scorer": 417751
                    }
                  }
                ]
              }
            ],
            "rewrite_time": 42542,
            "collector": [
              {
                "name": "MultiCollector",
                "reason": "search_multi",
                "time_in_nanos": 778406,
                "children": [
                  {
                    "name": "EarlyTerminatingCollector",
                    "reason": "search_count",
                    "time_in_nanos": 70290
                  },
                  {
                    "name": "ProfilingAggregator: [elitelligence_products]",
                    "reason": "aggregation",
                    "time_in_nanos": 502780
                  }
                ]
              }
            ]
          },
          {
            "query": [
              {
                "type": "ConstantScoreQuery",
                "description": "ConstantScore(*:*)",
                "time_in_nanos": 995345,
                "breakdown": {
                  "set_min_competitive_score_count": 0,
                  "match_count": 0,
                  "shallow_advance_count": 0,
                  "set_min_competitive_score": 0,
                  "next_doc": 930803,
                  "match": 0,
                  "next_doc_count": 4675,
                  "score_count": 0,
                  "compute_max_score_count": 0,
                  "compute_max_score": 0,
                  "advance": 2209,
                  "advance_count": 2,
                  "score": 0,
                  "build_scorer_count": 4,
                  "create_weight": 23875,
                  "shallow_advance": 0,
                  "create_weight_count": 1,
                  "build_scorer": 38458
                },
                "children": [
                  {
                    "type": "MatchAllDocsQuery",
                    "description": "*:*",
                    "time_in_nanos": 431375,
                    "breakdown": {
                      "set_min_competitive_score_count": 0,
                      "match_count": 0,
                      "shallow_advance_count": 0,
                      "set_min_competitive_score": 0,
                      "next_doc": 389875,
                      "match": 0,
                      "next_doc_count": 4675,
                      "score_count": 0,
                      "compute_max_score_count": 0,
                      "compute_max_score": 0,
                      "advance": 1167,
                      "advance_count": 2,
                      "score": 0,
                      "build_scorer_count": 4,
                      "create_weight": 9458,
                      "shallow_advance": 0,
                      "create_weight_count": 1,
                      "build_scorer": 30875
                    }
                  }
                ]
              }
            ],
            "rewrite_time": 8792,
            "collector": [
              {
                "name": "ProfilingAggregator: [all_products]",
                "reason": "aggregation_global",
                "time_in_nanos": 1310536
              }
            ]
          }
        ],
        "aggregations": [
          {
            "type": "AvgAggregator",
            "description": "elitelligence_products",
            "time_in_nanos": 319918,
            "breakdown": {
              "reduce": 0,
              "post_collection_count": 1,
              "build_leaf_collector": 130709,
              "build_aggregation": 2709,
              "build_aggregation_count": 1,
              "build_leaf_collector_count": 2,
              "post_collection": 584,
              "initialize": 4750,
              "initialize_count": 1,
              "reduce_count": 0,
              "collect": 181166,
              "collect_count": 1370
            }
          },
          {
            "type": "GlobalAggregator",
            "description": "all_products",
            "time_in_nanos": 1519340,
            "breakdown": {
              "reduce": 0,
              "post_collection_count": 1,
              "build_leaf_collector": 134625,
              "build_aggregation": 59291,
              "build_aggregation_count": 1,
              "build_leaf_collector_count": 2,
              "post_collection": 5041,
              "initialize": 24500,
              "initialize_count": 1,
              "reduce_count": 0,
              "collect": 1295883,
              "collect_count": 4675
            },
            "children": [
              {
                "type": "AvgAggregator",
                "description": "avg_price",
                "time_in_nanos": 775967,
                "breakdown": {
                  "reduce": 0,
                  "post_collection_count": 1,
                  "build_leaf_collector": 98999,
                  "build_aggregation": 33083,
                  "build_aggregation_count": 1,
                  "build_leaf_collector_count": 2,
                  "post_collection": 2209,
                  "initialize": 1708,
                  "initialize_count": 1,
                  "reduce_count": 0,
                  "collect": 639968,
                  "collect_count": 4675
                }
              }
            ]
          }
        ]
      }
    ]
  }
}
```
</details>

#### Response: Non-global aggregation

The response contains profiling information:

<details markdown="block">
  <summary>
    Response
  </summary>
  {: .text-delta}

```json
{
  "took": 13,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 4675,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "avg_taxful_total_price": {
      "value": 75.05542864304813
    }
  },
  "profile": {
    "shards": [
      {
        "id": "[LidyZ1HVS-u93-73Z49dQg][opensearch_dashboards_sample_data_ecommerce][0]",
        "inbound_network_time_in_millis": 0,
        "outbound_network_time_in_millis": 0,
        "searches": [
          {
            "query": [
              {
                "type": "ConstantScoreQuery",
                "description": "ConstantScore(*:*)",
                "time_in_nanos": 1690820,
                "breakdown": {
                  "set_min_competitive_score_count": 0,
                  "match_count": 0,
                  "shallow_advance_count": 0,
                  "set_min_competitive_score": 0,
                  "next_doc": 1614112,
                  "match": 0,
                  "next_doc_count": 4675,
                  "score_count": 0,
                  "compute_max_score_count": 0,
                  "compute_max_score": 0,
                  "advance": 2708,
                  "advance_count": 2,
                  "score": 0,
                  "build_scorer_count": 4,
                  "create_weight": 20250,
                  "shallow_advance": 0,
                  "create_weight_count": 1,
                  "build_scorer": 53750
                },
                "children": [
                  {
                    "type": "MatchAllDocsQuery",
                    "description": "*:*",
                    "time_in_nanos": 770902,
                    "breakdown": {
                      "set_min_competitive_score_count": 0,
                      "match_count": 0,
                      "shallow_advance_count": 0,
                      "set_min_competitive_score": 0,
                      "next_doc": 721943,
                      "match": 0,
                      "next_doc_count": 4675,
                      "score_count": 0,
                      "compute_max_score_count": 0,
                      "compute_max_score": 0,
                      "advance": 1042,
                      "advance_count": 2,
                      "score": 0,
                      "build_scorer_count": 4,
                      "create_weight": 5041,
                      "shallow_advance": 0,
                      "create_weight_count": 1,
                      "build_scorer": 42876
                    }
                  }
                ]
              }
            ],
            "rewrite_time": 22000,
            "collector": [
              {
                "name": "MultiCollector",
                "reason": "search_multi",
                "time_in_nanos": 3672676,
                "children": [
                  {
                    "name": "EarlyTerminatingCollector",
                    "reason": "search_count",
                    "time_in_nanos": 78626
                  },
                  {
                    "name": "ProfilingAggregator: [avg_taxful_total_price]",
                    "reason": "aggregation",
                    "time_in_nanos": 2834566
                  }
                ]
              }
            ]
          }
        ],
        "aggregations": [
          {
            "type": "AvgAggregator",
            "description": "avg_taxful_total_price",
            "time_in_nanos": 1973702,
            "breakdown": {
              "reduce": 0,
              "post_collection_count": 1,
              "build_leaf_collector": 199292,
              "build_aggregation": 13584,
              "build_aggregation_count": 1,
              "build_leaf_collector_count": 2,
              "post_collection": 6125,
              "initialize": 6916,
              "initialize_count": 1,
              "reduce_count": 0,
              "collect": 1747785,
              "collect_count": 4675
            }
          }
        ]
      }
    ]
  }
}
```
</details>

#### Response body fields

The `aggregations` array contains aggregation objects with the following fields.

Field | Data type | Description
:--- | :--- | :---
`type` | String | The aggregator type. In the [non-global aggregation example response](#response-non-global-aggregation), the aggregator type is `AvgAggregator`. [Global aggregation example response](#response-global-aggregation) contains a `GlobalAggregator` with an `AvgAggregator` child.
`description` | String | Contains a Lucene explanation of the aggregation. Helps differentiate aggregations with the same type.
`time_in_nanos` | Long | The total elapsed time for this aggregation, in nanoseconds. For concurrent segment search, `time_in_nanos` is the total amount of time across all slices (the difference between the last completed slice execution end time and the first slice execution start time).	
[`breakdown`](#the-breakdown-object-1) | Object | Contains timing statistics about low-level Lucene execution.
`children` | Array of objects | If an aggregation has subaggregations (children), this field contains information about the subaggregations.
`debug` | Object | Some aggregations return a `debug` object that describes the details of the underlying execution.
`max_slice_time_in_nanos`	|Long | The maximum amount of time taken by any slice to run an aggregation, in nanoseconds. This field is included only if you enable concurrent segment search.
`min_slice_time_in_nanos`	|Long |The minimum amount of time taken by any slice to run an aggregation, in nanoseconds. This field is included only if you enable concurrent segment search.
`avg_slice_time_in_nanos`	|Long |The average amount of time taken by any slice to run an aggregation, in nanoseconds. This field is included only if you enable concurrent segment search.

#### The `breakdown` object

The `breakdown` object represents the timing statistics about low-level Lucene execution, broken down by method. Each field in the `breakdown` object represents an internal Lucene method executed within the aggregation. Timings are listed in wall-clock nanoseconds and are not normalized. The `breakdown` timings are inclusive of all child times. The `breakdown` object is comprised of the following fields. All fields contain integer values.

Field | Description
:--- | :--- 
`initialize` | Contains the amount of time taken to execute the `preCollection()` callback method during `AggregationCollectorManager` creation. For concurrent segment search, the `initialize` method contains the total elapsed time across all slices (the difference between the last completed slice execution end time and the first slice execution start time).
`build_leaf_collector`| Contains the time spent running the aggregation's `getLeafCollector()` method, which creates a new collector to collect the given context. For concurrent segment search, the `build_leaf_collector` method contains the total elapsed time across all slices (the difference between the last completed slice execution end time and the first slice execution start time).
`collect`| Contains the time spent collecting the documents into buckets. For concurrent segment search, the `collect` method contains the total elapsed time across all slices (the difference between the last completed slice execution end time and the first slice execution start time).
`post_collection`| Contains the time spent running the aggregation’s `postCollection()` callback method. For concurrent segment search, the `post_collection` method contains the total elapsed time across all slices (the difference between the last completed slice execution end time and the first slice execution start time).
`build_aggregation`| Contains the time spent running the aggregation’s `buildAggregations()` method, which builds the results of this aggregation. For concurrent segment search, the `build_aggregation` method contains the total elapsed time across all slices (the difference between the last completed slice execution end time and the first slice execution start time).
`reduce`| Contains the time spent in the `reduce` phase. For concurrent segment search, the `reduce` method contains the total elapsed time across all slices (the difference between the last completed slice execution end time and the first slice execution start time).
`<method>_count` | Contains the number of invocations of a `<method>`. For example, `build_leaf_collector_count` contains the number of invocations of the `build_leaf_collector` method. 
`max_<method>`	|The maximum amount of time taken by any slice to run an aggregation method. This field is included only if you enable concurrent segment search.
`min_<method>`|The minimum amount of time taken by any slice to run an aggregation method. This field is included only if you enable concurrent segment search.
`avg_<method>`	|The average amount of time taken by any slice to run an aggregation method. This field is included only if you enable concurrent segment search.
`<method>_count`	|The total method count across all slices. For example, for the `collect` method, it is the total number of invocations of this method needed to collect documents into buckets across all slices. 
`max_<method>_count`	|The maximum number of invocations of a `<method>` on any slice. This field is included only if you enable concurrent segment search.
`min_<method>_count`	|The minimum number of invocations of a `<method>` on any slice. This field is included only if you enable concurrent segment search.
`avg_<method>_count`	|The average number of invocations of a `<method>` on any slice. This field is included only if you enable concurrent segment search.
