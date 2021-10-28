---
layout: post
title: Personal Elastic Search Cheat Sheet
date:   2021-10-20 14:34:25
categories: elastic-search
tags: elastic search 
image: /assets/article_images/2021-09-15-elastic-search/search.jpg
image2: /assets/article_images/2021-09-15-elastic-search/search.jpg
image-credit: istockphoto.com
image-credit-url: https://media.istockphoto.com/
---
Elastic Search is great but the API syntax is a `little` weird. So, I noted down some important APIs I came across in the book [Elasticsearch: The Definitive Guide](https://www.goodreads.com/book/show/21557290-elasticsearch).

This is `personal` cheat sheet, might not make sense to everybody.

# GET

## _source
Individual fields can be requested by using the _source parameter. 
Multiple fields can be specified in a comma-separated list

{% highlight javascript %}
 GET /website/blog/123?_source=title,text
{% endhighlight %}

## _mget 

The mget API expects a docs array, each element of which specifies the _index, _type, and _id metadata of the document you wish to retrieve. You can also specify a _source parameter if you just want to retrieve one or more specific fields: 
{% highlight javascript %}
GET /_mget 
{
  "docs": [
    {
      "_index": "website",
      "_type": "blog",
      "_id": 2
    },
    {
      "_index": "website",
      "_type": "pageviews",
      "_id": 1,
      "_source": "views"
    }
  ]
}
{% endhighlight %}
The response body also contains a docs array.

If all the documents have the same _index and _type, you can just pass an array of ids instead of the full docs array: 
{% highlight javascript %}
GET /website/blog/_mget { "ids" : [ "2", "1" ] }
{% endhighlight %}


## timeout
Times are more important to you than complete results, you can specify a timeout as 10 or 10ms (10 milliseconds), or 1s (1 second): 
{% highlight javascript %}
GET /_search?timeout=10ms
{% endhighlight %}

 Elasticsearch will return any results that it has managed to gather from each shard before the requests timed out. 
 
 WARNING: It should be noted that this timeout does not halt the execution of the query; it merely tells the coordinating node to return the results collected so far and to close the connection. In the background, other shards may still be processing the query even though results have been sent. Use the time-out because it is important to your SLA, not because you want to abort the execution of long-running queries.

## Pagination
{% highlight javascript %}
GET /_search?size=5

GET /_search?size=5&from=5

GET /_search?size=5&from=10
{% endhighlight %}

## _search Query Inline
{% highlight javascript %}
GET /_all/tweet/_search?q=tweet:elasticsearch 
{% endhighlight %}

The next query looks for john in the `name` field and mary in the `tweet` field. 
{% highlight javascript %}
GET /_all/tweet/_search?q=+name:john +tweet:mary
{% endhighlight %}

This simple search returns all documents that contain the word mary: 
{% highlight javascript %}
GET /_search?q=mary
{% endhighlight %}

## _analyze
When you are new to Elasticsearch, it is sometimes difficult to understand what is actually being tokenized and stored into your index. To better understand what is going on, you can use the analyze API to see how text is analyzed. Specify which analyzer to use in the query-string parameters, and the text to analyze in the body:

{% highlight javascript %}
GET /_analyze?analyzer=standard 
Text to analyze
{% endhighlight %}

## _mapping
Retrieved the mapping for type tweet in index gb: 

{% highlight javascript %}
GET /gb/_mapping/tweet
{% endhighlight %}

## _search Query DSL
To use the Query DSL, pass a query in the query parameter: 
{% highlight javascript %}
GET /_search 
{
    "query": YOUR_QUERY_HERE 
}
{% endhighlight %}

Query clause typically has this structure: 
{% highlight javascript %}
{
  QUERY_NAME: {
    ARGUMENT: VALUE,
    ARGUMENT: VALUE,...
  } 
}
{% endhighlight %}

If it references one particular field, it has this structure: 
{% highlight javascript %}
{ 
    QUERY_NAME: {
      FIELD_NAME: {
         ARGUMENT: VALUE,
         ARGUMENT: VALUE,...
       }
    } 
}
{% endhighlight %}


Combining Multiple Clauses Query clauses are simple building blocks that can be combined with each other to create complex queries.

Compound clauses that are used to combine other query clauses. 
For instance, a `bool` clause allows you to combine other clauses that either must match, must_not match, or should match if possible: 

{% highlight javascript %}
{
  "bool": {
    "must": {
      "match": {
        "tweet": "elasticsearch"
      }
    },
    "must_not": {
      "match": {
        "name": "mary"
      }
    },
    "should": {
      "match": {
        "tweet": "full text"
      }
    }
  }
}
{% endhighlight %}


A `filter` asks a yes|no question of every document and is used for fields that contain exact values: 
Eg. Is the created date in the range 2013 - 2014?

A `query` is similar to a filter, but also asks the question: How well does this document match? 
A typical use for a query is to find documents Best matching the words full text search.

> As a general rule, use query clauses for full-text search or for any condition that should affect the relevance score, and use filter clauses for everything else.

The `term` filter is used to filter by exact values, be they numbers, dates, Booleans, or not_analyzed exact-value string fields: 

{% highlight javascript %}
{
  "term": {
    "age": 26
  }
}
{% endhighlight %}

The `terms` filter is the same as the term filter, but allows you to specify multiple values to match. If the field contains any of the specified values, the document matches: 

{% highlight javascript %}
{
  "terms": {
    "tag": ["search", "full_text", "nosql"]
  }
}
{% endhighlight %}


The `range` filter allows you to find numbers or dates that fall into a specified range:
{% highlight javascript %}
 {
   "range": {
     "age": {
       "gte": 20,
       "lt": 30
     }
   }
 }
{% endhighlight %}


The `exists` and `missing` filters are used to find documents in which the specified field either has one or more values (exists) or doesn’t have any values (missing).
{% highlight javascript %}
{
  "exists": {
    "field": "title"
  }
}
{% endhighlight %}


The `match_all` query simply matches all documents. It is the default query that is used if no query has been specified: 
{% highlight javascript %}
{
  "match_all": {}
}
{% endhighlight %}

The `match` query should be the standard query that you reach for whenever you want to query for a full-text or exact value in almost any field.
{% highlight javascript %}
{
  "match": {
    "tweet": "About Search"
  }
}
{% endhighlight %}


The `multi_match` query allows to run the same match query on multiple fields: 
{% highlight javascript %}
{
  "multi_match": {
    "query": "full text search",
    "fields": ["title", "body"]
  }
}
{% endhighlight %}

The search API accepts only a single query parameter, so we need to wrap the query and the filter in another query, called the `filtered` query.
We can now pass this query to the query parameter of the search API: 
{% highlight javascript %}
GET /_search
{
  "query": {
    "filtered": {
      "query": {
        "match": {
          "email": "business opportunity"
        }
      },
      "filter": {
        "term": {
          "folder": "inbox"
        }
      }
    }
  }
}{% endhighlight %}

The term filter isn’t very useful on its own, though. 
The search API expects a query, not a filter. 
To use our term filter, we need to wrap it with a filtered query. The filtered query accepts both a query and a filter.


The `bool` query, like the bool filter, is used to combine multiple query clauses. However, there are some differences. Remember that while filters give binary yes/no answers, queries calculate a relevance score instead.

`must`:These clauses must match, like and. 

`must_not`: These clauses must not match, like not.

`should` At least one of these clauses must match, like or.
should If these clauses match, they increase the _score; otherwise, they have no effect. 
They are simply used to refine the relevance score for each document.

## _validate
The validate-query API can be used to check whether a query is valid. 
{% highlight javascript %}
GET /gb/tweet/_validate/query 
{
  "query": {
    "tweet": {
      "match": "really powerful"
    }
  }
}
{% endhighlight %}

Using the explain parameter has the added advantage of returning a human-readable description of the (valid) query, which can be useful for understanding exactly how your query has been interpreted
{% highlight javascript %}
GET /_validate/query?explain
{% endhighlight %}

## sort
To sort tweets by recency, with the most recent tweets first. 
We can do this with the sort parameter: 
{% highlight javascript %}
GET /_search 
{
  "query": {
    "filtered": {
      "filter": {
        "term": {
          "user_id": 1
        }
      }
    }
  },
  "sort": {
    "date": {
      "order": "desc"
    }
  }
}
{% endhighlight %}

The _score is not calculated, when it is not being used for sorting.

As a shortcut, you can specify just the name of the field to sort on:     
{% highlight javascript %}
"sort": "number_of_children"
{% endhighlight %}

Order is important. Results are sorted by the first criterion first. 
Only results whose first sort value is identical will then be sorted by the second criterion, and so on.

{% highlight javascript %}
"sort": [{
  "date": {
    "order": "desc"
  }
}, {
  "_score": {
    "order": "desc"
  }
}]

{% endhighlight %}

Inline search with sort
{% highlight javascript %}
GET /_search?sort=date:desc&sort=_score&q=search 
{% endhighlight %}

Sorting on Multivalue Fields When sorting on fields with more than one value, remember that the values do not have any intrinsic order; a multivalue field is just a bag of values. Which one do you choose to sort on? 
For numbers and dates, you can reduce a multivalue field to a single value by using the min, max, avg, or sum sort modes. 
For instance, you could sort on the earliest date in each dates field by using the following: 

{% highlight javascript %}
"sort": {
  "dates": {
    "order": "asc",
    "mode": "min"
  }
}
{% endhighlight %} 


In order to sort on a string field, that field should contain one term only: the whole not_analyzed string. 
But of course we still need the field to be analyzed in order to be able to query it as full text.

What we really want to do is to pass in a single field but to index it in two different ways. All of the core field types (strings, numbers, Booleans, dates) accept a fields parameter that allows you to transform a simple mapping like 
{% highlight javascript %}
"tweet": {
  "type": "string",
  "analyzer": "english"
}
{% endhighlight %}

into a multifield mapping like this: 
{% highlight javascript %}
"tweet": {
  "type": "string",
  "analyzer": "english",
  "fields": {
    "raw": {
      "type": "string",
      "index": "not_analyzed"
    }
  }
}
{% endhighlight %}

The main tweet field is just the same as before: an analyzed full-text field. The new tweet.raw subfield is not_analyzed.

{% highlight javascript %}
GET /_search 
{
  "query": {
    "match": {
      "tweet": "elasticsearch"
    }
  },
  "sort": "tweet.raw"
}
{% endhighlight %}

## _explain
You can use the explain API to understand why one particular document matched or, more important, why it didn’t match. 
{% highlight javascript %}
GET /_search?explain 
{
  "query": {
    "match": {
      "tweet": "honeymoon"
    }
  }
}
{% endhighlight %}
Producing the explain output is expensive. It is a debugging tool only. Don’t leave it turned on in production.

## routing
Custom routing parameter could be provided at index time to ensure that all related documents, such as the documents belonging to a single user, are stored on a single shard. At search time, instead of searching on all the shards of an index, you can specify one or more routing values to limit the search to just those shards:
{% highlight javascript %}
GET /_search?routing=user_1,user2
{% endhighlight %}

## search_type
While query_then_fetch is the default search type, other search types can be specified for particular purposes, for example: 
{% highlight javascript %}
GET /_search?search_type=count
{% endhighlight %}

## Scan and Scroll
To use scan-and-scroll, we execute a search request setting search_type to `scan`, and passing a `scroll` parameter telling Elasticsearch how long it should keep the scroll open: 
{% highlight javascript %}
GET /old_index/_search?search_type=scan&scroll=1m
{% endhighlight %}

The response to this request doesn’t include any hits, but does include a _scroll_id, which is a long Base-64 encoded string. 

Now we can pass the `_scroll_id` to the `_search/scroll` endpoint to retrieve the first batch of results: 
{% highlight javascript %}
GET /_search/scroll?scroll=1m c2Nhbjs1OzExODpRNV9aY1VyUVM4U0NMd2pjWlJ3YWlBOzExOTpRNV9aY1VyUVM4U0
{% endhighlight %}

The scroll expiry time is refreshed every time we run a scroll request.
The scroll request also returns a new _scroll_id. 
Every time we make the next scroll request, we must pass the _scroll_id returned by the previous scroll request.

## _alias
You can check which index the alias points to: 
{% highlight javascript %}
GET /*/_alias/my_index
{% endhighlight %}

Or which aliases point to the index: 
{% highlight javascript %}
GET /my_index_v1/_alias/*
{% endhighlight %}


## cross_fields
The `cross_fields` type takes a term-centric approach, quite different from the field-centric approach taken by `best_fields` and `most_fields`.

All terms are required. 
For a document to match, both peter and smith must appear in the same field, either the first_name field or the last_name field: 
{% highlight javascript %}
(+first_name:peter +first_name:smith) (+last_name:peter  +last_name:smith) 
{% endhighlight %}


A term-centric approach would use this logic instead: 
{% highlight javascript %}
+(first_name:peter last_name:peter) +(first_name:smith last_name:smith) 
{% endhighlight %}

In other words, the term peter must appear in either field, and the term smith must appear in either field.

The cross_fields type first analyzes the query string to produce a list of terms, and then it searches for each term in any field.


If you were searching for books using the title and description fields, you might want to give more weight to the title field. 
This can be done as described before with the caret (^) syntax: 

{% highlight javascript %}
GET /books/_search 
{
  "query": {
    "multi_match": {
      "query": "peter smith",
      "type": "cross_fields",
      "fields": ["title^2", "description"]
    }
  }
}
{% endhighlight %}

## Regex Match
To find all postcodes beginning with W1, we could use a simple prefix query:
The prefix query is a low-level query that works at the term level.

{% highlight javascript %}
GET /my_index/address/_search  
{
  "query": {
    "prefix": {
      "postcode": "W1"
    }
  }
}
{% endhighlight %}

The wildcard query is a low-level, term-based query similar in nature to the prefix query, but it allows you to specify a pattern instead of just a prefix. 
It uses the standard shell wildcards: 
* `?` matches any character, and 
* `*` matches zero or more characters.

{% highlight javascript %}
GET /my_index/address/_search
{
  "query": {
    "wildcard": {
      "postcode": "W?F*HW"
    }
  }
}
{% endhighlight %}


The regexp query allows you to write these more complicated patterns: 
{% highlight javascript %}
GET /my_index/address/_search 
{
  "query": {
    "regexp": {
      "postcode": "W[0-9].+"
    }
  }
}
{% endhighlight %}


## n-grams
For search-as-you-type, we use a specialized form of n-grams called edge n-grams. Edge n-grams are anchored to the beginning of the word.

{% highlight javascript %}
{
  "filter": {
    "autocomplete_filter": {
      "type": "edge_ngram",
      "min_gram": 1,
      "max_gram": 20
    }
  }
}
{% endhighlight %}

We want to ensure that our inverted index contains edge n-grams of every word, but we want to match only the full words that the user has entered (brown and fo). We can do this by using the autocomplete analyzer at index time and the standard analyzer at search time. One way to change the search analyzer is just to specify it in the query:


{% highlight javascript %}
GET /my_index/my_type/_search 
{
  "query": {
    "match": {
      "name": {
        "query": "brown fo",
        "analyzer": "standard"
      }
    }
  }
}
{% endhighlight %}

## boosting

Sometimes, must_not can be too strict. The boosting query solves this problem. It allows us to still include results that appear to be about the fruit or the pastries, but to downgrade them—to rank them lower than they would otherwise be:

{% highlight javascript %}
GET /_search
 {
   "query": {
     "boosting": {
       "positive": {
         "match": {
           "text": "apple"
         }
       },
       "negative": {
         "match": {
           "text": "pie tart fruit crumble tree"
         }
       },
       "negative_boost": 0.5
     }
   }
 }
{% endhighlight %}

It accepts a positive query and a negative query. Only documents that match the positive query will be included in the results list, but documents that also match the negative query will be downgraded by multiplying the original _score of the document with the negative_boost.

Enter the constant_score query. 
This query can wrap either a query or a filter, and assigns a score of 1 to any documents that match, regardless of TF/IDF: 
{% highlight javascript %}
GET /_search 
{
  "query": {
    "bool": {
      "should": [{
        "constant_score": {
          "query": {
            "match": "..."
          }
        }
      }]
    }
  }
}
{% endhighlight %}

Perhaps not all features are equally important—some have more value to the user than others. If the most important feature is the pool, we could boost that clause to make it count for more: 
{% highlight javascript %}
GET /_search 
{
  "query": {
    "bool": {
      "should": [{
            "constant_score": {
              "query": {
                "match": {
                  "description": "wifi"
                }
              }
            }
          }, {
            "constant_score": {
              "query": {
                "match": {
                  "description": "garden"
                }
              }
            }
          }, {
            "constant_score": {
              "boost": 2
              ...
}
{% endhighlight %}

function_score Query The function_score query is the ultimate tool for taking control of the scoring process. It allows you to apply a function to each document that matches the main query in order to alter or completely replace the original query _score.

Where we wanted to score vacation homes by the number of features that each home possesses. We ended that section by wishing for a way to use cached filters to affect the score, and with the function_score query we can do just that.

{% highlight javascript %}
GET /_search 
{
  "query": {
    "function_score": {
      "filter": {
        "term": {
          "city": "Barcelona"
        }
      },
      "functions": [{
        "filter": {
          "term": {
            "features": "wifi"
          }
        },
        "weight": 1
      }, {
        "filter": {
          "term": {
            "features": "garden"
          }
        },
        "weight": 1
      }, {
        "filter": {
          "term": {
            "features": "pool"
          }
        },
        "weight": 2
      }],
      "score_mode": "sum",
    }
  }
}
{% endhighlight %}


## aggregation
A car dealer may want to know which color car sells the best. This is easily accomplished using a simple aggregation. 
We will do this using a terms bucket: 
{% highlight javascript %}
GET /cars/transactions/_search?search_type=count 
{
  "aggs": {
    "colors": {
      "terms": {
        "field": "color"
      }
    }
  }
}

{% endhighlight %}

{% highlight javascript %}
GET / cars / transactions / _search ? search_type = count 
{
"aggs": {
  "colors": {
  "terms": {
    "field": "color"
  },
  "aggs": {
    "avg_price": {
    "avg": {
      "field": "price"
    }
    }
  }
  }
 }
}

GET /cars / transactions / _search ? search_type = count 
{
"aggs": {
  "price": {
  "histogram": {
    "field": "price",
    "interval": 20000
  },
  "aggs": {
    "revenue": {
    "sum": {
      "field": "price"
    }
    }
  }
  }
}
}

GET /cars/transactions/_search?search_type=count 
{
    "aggs": {
      "sales": {
        "date_histogram": {
          "field": "sold",
          "interval": "month",
          "format": "yyyy-MM-dd"
        }
      }
    }

{% endhighlight %}

 We are omitting search_type=count so that search hits are returned too.

 {% highlight javascript %}
 GET /cars/transactions/_search  
 {
  "query": {
    "match": {
      "make": "ford"
    }
  },
  "aggs": {
    "colors": {
      "terms": {
        "field": "color"
      }
    }
  }
}
 {% endhighlight %}

We can use the cardinality metric to determine the number of car colors being sold at our dealership: 
{% highlight javascript %}
GET /cars/transactions/_search?search_type=count
{
  "aggs": {
    "distinct_colors": {
      "cardinality": {
        "field": "color"
      }
    }
  }
}
{% endhighlight %}


High eviction counts can indicate a serious resource issue and a reason for poor performance. Fielddata usage can be monitored: 
{% highlight javascript %}
#per-index using the indices-stats API: 
GET /_stats/fielddata?fields=* 

#per-node using the nodes-stats API: 
GET /_nodes/stats/indices/fielddata?fields=* 

#per-index per-node: 
GET /_nodes/stats/indices/fielddata?level=indices&fields=*
{% endhighlight %}


# POST

## _update
We could add a tags field and a views field to our blog post as follows: 
{% highlight javascript %}
POST /website/blog/1/_update 
{
  "doc": {
    "tags": ["testing"],
    "views": 0
  }
}
{% endhighlight %}

Update with Script
{% highlight javascript %}
POST /website/blog/1/_update 
{
  "script": "ctx._source.views+=1"
}

POST /website/blog/1/_update 
{
  "script": "ctx.op = ctx._source.views == count ? 'delete' : 'none'",
  "params": {
    "count": 1
  }
}
{% endhighlight %}

we can use the upsert parameter to specify the document that should be created if it doesn’t already exist: 
{% highlight javascript %}
POST /website/pageviews/1/_update 
{
  "script": "ctx._source.views+=1",
  "upsert": {
    "views": 1
  }
}
{% endhighlight %}

if two processes are both incrementing the page-view counter, it doesn’t matter in which order it happens; if a conflict occurs, the only thing we need to do is reattempt the update. This can be done automatically by setting the retry_on_conflict parameter to the number of times that update should retry before failing; it defaults to 0. 
{% highlight javascript %}
POST /website/pageviews/1/_update?retry_on_conflict=5 
{
  "script": "ctx._source.views+=1",
  "upsert": {
    "views": 0
  }
}
{% endhighlight %}

_bulk API is used to perform multiple indexing or delete operations in a single API call. T
his reduces overhead and can greatly increase indexing speed.
just as for the mget API, the _bulk request accepts a default /_index or /_index/_type in the URL: 
{% highlight javascript %}
POST /website/_bulk  { "index": { "_type": "log" }} { "event": "User logged in" }
{% endhighlight %}


## _alias
An alias can point to multiple indices, so we need to remove the alias from the old index at the same time as we add it to the new index. The change needs to be atomic, use the _aliases endpoint: 

{% highlight javascript %}
POST /_aliases 
{
  "actions": [{
    "remove": {
      "index": "my_index_v1",
      "alias": "my_index"
    }
  }, {
    "add": {
      "index": "my_index_v2",
      "alias": "my_index"
    }
  }]
}
{% endhighlight %}

## _refresh
In Elasticsearch, this lightweight process of writing and opening a new segment is called a refresh.
default, every shard is refreshed automatically once every second. This is why we say that Elasticsearch has near real-time search:
document changes are not visible to search immediately, but will become visible within 1 second.

Perform a manual refresh, with the refresh API: POST /_refresh POST /blogs/_refresh.

The flush API can be used to perform a manual flush: POST /blogs/_flush

Optimize API The optimize API is best described as the forced merge API. It forces a shard to be merged down to the number of segments specified in the max_num_segments parameter. The intention is to reduce the number of segments (usually to one) in order to speed up search performance.

In certain specific circumstances, the optimize API can be beneficial. The typical use case is for logging, where logs are stored in an index per day, week, or month. Older indices are essentially read-only; they are unlikely to change. In this case, it can be useful to optimize the shards of an old index down to a single segment each; it will use fewer resources and searches will be quicker:

POST /logstash-2014-10/_optimize?max_num_segments=1

  
 