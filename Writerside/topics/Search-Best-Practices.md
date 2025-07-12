# Search Best Practices

  

This document recommends best practices for the Search API. We use single quotes (‘’) throughout to delimit query strings. This way a query that contains multi-word phrases surrounded by double quotes can be delimited without confusion: `‘field:"some text" some-value’`.

### Batch Index.put() and Index.delete() calls

You can pass up to 200 documents at a time when adding or deleting them from an index. This is much more efficient than handling them one at a time.

### Use document rank to pre-sort documents

By default, search returns its results by descending rank. Also by default, the Search API sets the rank of each document to seconds since Jan 1st 2011. This results in the freshest documents being returned first. However, if you don’t need documents to be sorted by the time they were added, you can use rank for other purposes. Suppose you have a real estate application. What customers want most is sorting by price. For an efficient default sort, you could set the rank to the house price.

If you need multiple sort orders such as price low-to-high and price high-to-low, you can create a separate index for each order. One index would have rank = price and the other rank = MAXINT-price (since rank must be positive).

Using rank as the sort key will improve search performance. To specify other sort keys, you must use sort options, which limits the number of search results to 10,000 documents. In this case, the sort order determined by rank will determine which documents will be included in the sort. Read about [sort options](https://web.archive.org/web/20160424230344/https://cloud.google.com/appengine/docs/java/search/options#Java_SortOptions) to learn more.

### Use atom fields for boolean data

Storing boolean data in number fields is very inefficient. Use atom fields instead, and assign your favorite constants (True/False, yes/no, 0/1).

### Turn negatives into positives

Suppose you have a special term to identify restaurants whose cuisine is undefined. If you want to exclude those restaurants you could use `‘NOT cuisine:undefined’` as your query. This is, however, more expensive to evaluate (in both billable operations and computation time) than having the opposite, finding restaurants whose cuisine is known. Rather than having one field, cuisine, you can use two, `cuisine`, and `cuisine_known`, with the latter being an atom field. For restaurants for which cuisine is defined, you set the first field to the actual cuisine and the second field to `"yes"`. For restaurants for which you do not know the cuisine, you set cuisine to `""` (an empty string) and `cuisine_known` to `"no"`. Now to find restaurants for which cuisine is known you issue a query `‘cuisine_known:yes’`, which is much faster than the negation.

### Turn disjunctions into conjunctions

The "OR" disjunction is an expensive operation in both billable operations and computation time. Suppose you want to search for `‘cuisine:Japanese OR cuisine:Korean’`. An alternative is to index documents with more general categories of cuisine. In this case, the query may be simplified to `‘cuisine:Asian’`.

### Eliminate tautologies from your queries

Suppose you want to find all restaurants in Toronto. Assuming that your documents have only a single field named "city", if you use the query `‘city:toronto AND NOT city:montreal’` you get the same results as `‘city:toronto’`, because if city is set to `"toronto"` it cannot be set to `"montreal"`. The second query runs much faster since it involves only one term. The first query performs three steps: first, it finds a list of documents where city is set to "toronto", then it finds a list of all cities for where city is not set to "montreal", and finally it computes the intersection of the two lists.

### Narrow the range before sorting

Suppose your application stores information about restaurants around the world, and you would like to show the restaurants closest to the current user. One way of doing this is to sort matching documents by the distance from the user’s location. But if you have 1,000,000 restaurants, running a query like `‘cuisine:japanese’` with the sort expression distance(geopoint(x, y), restaurant\_loc) will take a long time. It's a good idea to add filters to a query so that you start with a more salient set of selected documents to sort. One solution is to create geographical categories, such as country, state and city - you could infer city and state from the user’s location. Then your query becomes `‘cuisine:japanese AND city:<user-city>’`. Chances are very good that you'll no longer need to sort 1,000,000 documents.

### Use narrow categories to avoid or minimize sorting

If you use rank to sort restaurants by price, you could create a `price_range` field that contains price categories: `price_0_10`, `price_11_20`, `price_21_30`, `price_31_40`, `price_41_lots`. You could then find all restaurants that cost between $21 and $40 with no sorting at all using the query `‘price_range:price_21_30 OR price_range:price_31_40’`. In many cases the appropriate categories are not as clear-cut, but with this technique you can reject a large number of documents before winnowing down the search with expensive queries such as `‘... AND price>25 AND price<35’`.

### Do not score matches unless you need to

Scoring is used to indicate how well a given document matched a query. However, unless you intend to sort by score, do not request scoring. It will only slow down your search.
