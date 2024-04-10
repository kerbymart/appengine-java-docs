# Handling Search Results

  

When a query call completes normally, it returns the result as a [`Results`](https://web.archive.org/web/20160424230742/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/search/Results) object. The Results object tells you how many matching documents were found in the index, and how many matched documents were returned. It also includes a collection of matching [`ScoredDocuments`](https://web.archive.org/web/20160424230742/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/search/ScoredDocument). The collection usually contains a portion of all the matching documents found, since search returns a limited number of documents each time it's called. By using an offset or a cursor you can retrieve all the matching documents, a subset at a time.

1.  [Results](#Java_Results)
2.  [Scored documents](#Java_Scored_documents)
3.  [Using offsets](#Java_Using_offsets)
4.  [Using cursors](#Java_Using_cursors)

## Results

Your calling code should be prepared to handle exceptions which might be thrown if the query is invalid or there were problems processing it:

```
// index and query have already been defined ... 
try {
    Results<ScoredDocument> result = index.search(query);
    long totalMatches = result.getNumberFound();
    int numberOfDocsReturned = result.getNumberReturned();
    Collection<ScoredDocument> listOfDocs = result.getResults();
} catch (SearchException e) {
    // handle exception...
}
```

Depending on the value of the `limit` [query option](https://web.archive.org/web/20160424230742/https://cloud.google.com/appengine/docs/java/search/options#Java_QueryOptions), the number of matching documents returned in the result may be less than the number found. Remember that the number found will be an estimate if the number found accuracy is less than the number found. No matter how you configure the search options, a `search()` call will find no more than 10,000 matching documents.

If more documents were found than returned, and you want to retrieve all of them, you need to repeat the search using either an offset or a cursor, as explained below.

## Scored documents

The search results will include a Collection of [`ScoredDocuments`](https://web.archive.org/web/20160424230742/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/search/ScoredDocument) that match the query. You can retrieve the collection using the method `getResults()` or iterate over its members directly from the search results itself. Both methods are shown here:

```
// index and query have already been defined ...
Results<ScoredDocument> result = index.search(query);

// Grab the collection for later use:
Collection<ScoredDocument> theDocs = result.getResults();

// Alternatively, iterate over the results directly:
for (ScoredDocument doc : result) {
    // do work
}
```

By default, a scored document contains all the fields of the original document that was indexed. If your [query options](https://web.archive.org/web/20160424230742/https://cloud.google.com/appengine/docs/java/search/options#Java_QueryOptions) specified `setFieldsToReturn`, only those fields will appear in the results when you call [`getFields()`](https://web.archive.org/web/20160424230742/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/search/Document#getFields()) on the document. If you used `addExpressionToReturn` or `setFieldsToSnippet` to create computed fields, retrieve them separately by calling [`getExpressions()`](https://web.archive.org/web/20160424230742/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/search/ScoredDocument#getExpressions()) on the document.

## Using offsets

If your search finds more documents than you can return at once, use an offset to index into the list of matching documents. For example, the default query limit is 20 documents. After you've executed a search the first time (with offset 0) and retrieved the first 20 documents, retrieve the next 20 documents by setting the offset to 20 and running the same search again. Keep repeating the search, incrementing the offset each time by the number of documents returned:

```
// index and queryString have already been defined

try {
    int numberRetrieved = 0;
    int offset = 0;
    do {
        // build options and query
        QueryOptions options = QueryOptions.newBuilder()
            .setOffset(offset)
            .build();
        Query query = Query.newBuilder().setOptions(options).build(queryString);
        
        // search at least once
        Results<ScoredDocument> result = index.search(query);
        numberRetrieved = result.getNumberReturned();
        if (numberRetrieved > 0) {
            offset += numberRetrieved;
            // process the matched docs
        }
        
    }
    while (numberRetrieved > 0);
} catch (SearchException e) {
    // handle exception...
}
```

Offsets can be inefficient when iterating over a very large result set.

## Using cursors

You can also use cursors to retrieve a subrange of results. Cursors are useful when you intend to present your search results in consecutive pages and you want to be sure you do not skip any documents in the case where an index could be modified between queries. Cursors are also more efficient when iterating across a very large result set.

In order to use cursors, you must create an initial cursor and include it in the query options. There are two kinds of cursors, *per-query* and *per-result*. A per-query cursor causes a separate cursor to be associated with the results object returned by the search call. A per-result cursor causes a cursor to be associated with every scored document in the results.

### Using a per-query cursor

By default, a newly constructed cursor is a per-query cursor. This cursor holds the position of the last document returned in the search's results. It is updated with each search. To enumerate all matching documents in an index, execute the same search until the result returns a null cursor:

```
// index and queryString have already been defined
    
try {
    // create the initial cursor
    Cursor cursor = Cursor.newBuilder().build();

    do {
        // build options and query
        QueryOptions options = QueryOptions.newBuilder()
            .setCursor(cursor)
            .build();
        Query query = Query.newBuilder().setOptions(options).build(queryString);
    
        // search at least once
        Results<ScoredDocument> result = index.search(query);
        int numberRetrieved = result.getNumberReturned();
        cursor = result.getCursor();

        if (numberRetrieved > 0) {
            // process the matched docs
        }
    
    }
    while (cursor != null);
    // all done!
} catch (SearchException e) {
    // handle exception...
}
```

### Using a per-result cursor

To create per-result cursors, you must set the cursor perResult property to true when you create the initial cursor. When the search returns, every document will have a cursor associated with it. You can use that cursor to specify a new search with results that begin with a specific document. Note that when you pass a per-result cursor to search, there will be no per-query cursor associated with the result itself; result.getCursor() will return null so you can't use this to test whether you've retrieved all the matches.

```
// index and queryString have already been defined
    
try {
    // create an initial per-result cursor
    Cursor cursor = Cursor.newBuilder().setPerResult(true).build();
    // build options and query
    QueryOptions options = QueryOptions.newBuilder()
        .setCursor(cursor)
        .build();
    Query query = Query.newBuilder().setOptions(options).build(queryString);
    Results<ScoredDocument> result = index.search(query);

    // process the matched docs
    cursor = null;
    for (ScoredDocument doc : result) {
        // discover some document of interest and grab its cursor
        if (...)
            cursor = doc.getCursor();
     }
    
    // Start the next search from the document of interest
    if (cursor != null) {
        options = QueryOptions.newBuilder()
            .setCursor(cursor)
            .build();
        query = Query.newBuilder().setOptions(options).build(queryString);
        result = index.search(query);
    }
} catch (SearchException e) {
    // handle exception
}
```

### Saving and restoring cursors

A cursor can be serialized as a web-safe string, saved, and then restored for later use:

```
String cursorString = cursor.toWebSafeString();
// Save the string ... and restore:
Cursor cursor = Cursor.newBuilder().build(cursorString));
```
