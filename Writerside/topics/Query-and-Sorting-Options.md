# Query and Sorting Options

  

When you call the `search()` method using a query string alone, the results are returned according to the default query options:

-   Documents are returned sorted in order of descending rank
-   Documents are returned in groups of 20 at a time
-   Retrieved documents contain all of their original fields

You can use an instance of the [Query](https://web.archive.org/web/20160424230739/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/search/Query) class as the argument to `search()` to change these options. The Query class allows you to specify how many documents to return at a time. It also lets you customize the contents of the retrieved documents. You can ask for document identifiers only, or request that documents contain only a subset of their fields. You can also create custom fields in the retrieved documents: [snippets](#Java_Snippets) (fragments of text fields showing the text surrounding a matched string), and [field expressions](#Java_FieldExpressions) (fields with values derived from other fields in the document).

Apart from the query options, the Query class can also include an instance of the `SortOptions` class. Using sort options you can change the sort order, and sort the results on multiple keys.

1.  [Searching with the Query class](#Java_Searching_with_the_Query_class)
2.  [QueryOptions](#Java_QueryOptions)
3.  [SortOptions](#Java_SortOptions)
4.  [Writing expressions](#Java_Writing_expressions)

## Searching with the Query class

When you search with an instance of the Query class, you need to construct an instance of the class in several steps. This is the general order:

1.  Create a query string.
2.  Create `SortOptions` if needed.
3.  Create `QueryOptions`.
4.  Create a Query object that includes the query string and the (optional) `QueryOptions`.
5.  Call the search method on the Query object.

The various query and sort options are specified by calling setter methods on instances of the `QueryOptions.Builder` and `SortOptions.Builder` classes, as in this example:

```
import com.google.appengine.api.search.Index;
import com.google.appengine.api.search.IndexSpec;
import com.google.appengine.api.search.SearchServiceFactory;
import com.google.appengine.api.search.Query;
import com.google.appengine.api.search.QueryOptions;
import com.google.appengine.api.search.Results;
import com.google.appengine.api.search.SearchException;
import com.google.appengine.api.search.SortExpression;
import com.google.appengine.api.search.SortOptions;
import com.google.appengine.api.search.ScoredDocument;

try {
    // Build the SortOptions with 2 sort keys
    SortOptions sortOptions = SortOptions.newBuilder()
        .addSortExpression(SortExpression.newBuilder()
            .setExpression("price")
            .setDirection(SortExpression.SortDirection.DESCENDING)
            .setDefaultValueNumeric(0))
        .addSortExpression(SortExpression.newBuilder()
            .setExpression("brand")
            .setDirection(SortExpression.SortDirection.DESCENDING)
            .setDefaultValue(""))
        .setLimit(1000)
        .build();

    // Build the QueryOptions
    QueryOptions options = QueryOptions.newBuilder()
        .setLimit(25)
        .setFieldsToReturn("model", "price", "description")
        .setSortOptions(sortOptions)
        .build();

    // A query string
    String queryString = "product: piano AND price < 5000";

    //  Build the Query and run the search
    Query query = Query.newBuilder().setOptions(options).build(queryString);
    IndexSpec indexSpec = IndexSpec.newBuilder().setName(indexName).build();
    Index index = SearchServiceFactory.getSearchService().getIndex(indexSpec);
    Results<ScoredDocument> result =  index.search(query);
} catch (SearchException e) {
    // handle exception...
}
```

## QueryOptions

You must use the [`QueryOptions.Builder`](https://web.archive.org/web/20160424230739/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/search/QueryOptions.Builder) to set query options. You do not have access to these properties directly.

These properties control how many results are returned and in what order. The offset and cursor options, which are mutually exclusive, support pagination. They specify which selected documents to return in the results.

<table>
<colgroup>
<col style="width: 25%" />
<col style="width: 25%" />
<col style="width: 25%" />
<col style="width: 25%" />
</colgroup>
<thead>
<tr class="header">
<th>Property</th>
<th>Description</th>
<th>Default</th>
<th>Maximum</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>Limit</code></td>
<td>The maximum number of documents to return in the results.</td>
<td>20</td>
<td>1000</td>
</tr>
<tr class="even">
<td><code>NumberFoundAccuracy</code></td>
<td>This property determines the accuracy of the result returned by <a href="https://web.archive.org/web/20160424230739/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/search/Results#getNumberFound()"><code>Results.getNumberFound()</code></a> . It sets a limit for how many matches are actually counted, stopping the search when the limit is reached.<br />
<br />
If the number of matches in the index is less than or equal to the limit, the count returned is exact. Otherwise, the count is an estimate based on the matches that were found and the size and structure of the index. Note that setting a high value for this property can affect the complexity of the search operation and may cause timeouts.</td>
<td>If unspecified accuracy is set to the same value as <code>Limit</code></td>
<td>25000</td>
</tr>
<tr class="odd">
<td><code>Offset</code></td>
<td>The offset of the first document in the results to return.</td>
<td>0. Results will contain all matching documents (up to limit).</td>
<td>1,000</td>
</tr>
<tr class="even">
<td><code>Cursor</code></td>
<td>A cursor can be used in lieu of an offset to retrieve groups of documents in sorted order. A cursor is updated as it is passed into and out of consecutive queries, allowing each new search to be continued from the end of the previous one. Cursor and offset are discussed on the <a href="https://web.archive.org/web/20160424230739/https://cloud.google.com/appengine/docs/java/search/results">Handling Results</a> page.</td>
<td>Null. Results will contain all matching documents (up to limit).</td>
<td>-</td>
</tr>
<tr class="odd">
<td><code>SortOptions</code></td>
<td>Set a <code>SortOptions</code> object to control the ordering of the search results. An instance of <code>SortOptions</code> has its own set of properties which are described below.</td>
<td>Null. Sort by decreasing document rank.</td>
<td>-</td>
</tr>
</tbody>
</table>

These properties control what document fields appear in the results.

<table>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<thead>
<tr class="header">
<th>Property</th>
<th>Description</th>
<th>Default</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>ReturningIdsOnly</code></td>
<td>Set to <code>True</code> or <code>False</code>. When <code>True</code>, the documents returned in the results will contain IDs only, no fields.</td>
<td><code>False</code> (return all fields).</td>
</tr>
<tr class="even">
<td><code>FieldsToReturn</code></td>
<td>Specifies which document fields to include in the results. No more than 100 fields can be specified.</td>
<td>Return all document fields (up to 100 fields).</td>
</tr>
<tr class="odd">
<td><span id="Java_FieldExpressions"></span><code>ExpressionsToReturn</code></td>
<td>Field expressions describing computed fields that are added to each document returned in the search results. These fields are added to the expressions property of the document. The field value is specified by <a href="#Java_Writing_expressions">writing an expression</a> which may include one or more document fields.</td>
<td>None</td>
</tr>
<tr class="even">
<td><code>FieldsToSnippet</code></td>
<td>A list of text field names. A <a href="#Java_Snippets">snippet</a> is generated for each field. This is a computed field that is added to the expressions property of the documents in the search results. The snippet field has the same name as its source field.<br />
<br />
This option implicitly uses the snippet function with only two arguments, creating a snippet with at most one matching string, based on the same query string that the search used to retrieve the results: <code>snippet("query-string", field-name)</code>.<br />
<br />
You can also create customized snippets with the <code>ExpressionsToReturn</code> option by adding a field expression that explicitly calls the <a href="#Java_Snippet_function">snippet function</a>.</td>
<td>None</td>
</tr>
</tbody>
</table>

## SortOptions

The properties of `SortOptions` control the ordering and scoring of the search results. You must use the [`SortOptions.Builder`](https://web.archive.org/web/20160424230739/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/search/SortOptions.Builder) to set the sort options. You do not have access to these properties directly.

The properties of `SortOptions` control the ordering and scoring of the search results.

<table>
<thead>
<tr class="header">
<th>Property</th>
<th>Description</th>
<th>Default</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>SortExpressions</code></td>
<td>A list of <a href="#Java_Sorting_on_multiple_keys"><code>SortExpressions</code></a> representing a multi-dimensional sort of Documents.</td>
<td>None</td>
</tr>
<tr class="even">
<td><code>MatchScorer</code></td>
<td>An optional <a href="https://web.archive.org/web/20160424230739/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/search/MatchScorer"><code>MatchScorer</code></a> object. When present this will cause the documents to be scored according to search term frequency. The score will be available as the <code>_score</code> field. Scoring documents can be expensive (in both billable operations and execution time) and can slow down your searches. Use scoring sparingly.</td>
<td>None</td>
</tr>
<tr class="odd">
<td><code>Limit</code></td>
<td>Maximum number of objects to score and/or sort. Cannot be more than 10,000.</td>
<td>1,000</td>
</tr>
</tbody>
</table>

### Sorting on multiple keys

You can order the search results on multiple sort keys. Each key can be a simple field name, or a value that is computed from several fields. Note that the term 'expression' is used with multiple meanings when speaking about sort options: The `SortOption` itself has an *expressions attribute*. This attribute is a list of `SortExpression` objects which correspond to sort keys. Finally, each `SortExpression` object contains an *expression attribute* which specifies how to calculate the value of the sort key. This expression is constructed according to the rules in the next section.

A `SortExpression` also defines the direction of the sort and a default key value to use if the expression cannot be calculated for a document. Here is the complete list of properties:

<table>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<thead>
<tr class="header">
<th>Property</th>
<th>Description</th>
<th>Default</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>Expression</code></td>
<td>An expression to be evaluated when sorting results for each matching document.</td>
<td>None</td>
</tr>
<tr class="even">
<td><code>Direction</code></td>
<td>The direction to sort the search results, either <code>ASCENDING</code> or <code>DESCENDING</code>.</td>
<td><code>DESCENDING</code></td>
</tr>
<tr class="odd">
<td><code>DefaultValue</code><br />
<code>DefaultValueDate</code><br />
<code>DefaultValueNumber</code></td>
<td>The default value of the expression, if no field is present and cannot be calculated for a document. A text value must be specified for text sorts. A numeric value must be specified for numeric sorts.</td>
<td>None</td>
</tr>
</tbody>
</table>

### Sorting on multi-valued fields

When you sort on a multi-valued field of a particular type, only the first value assigned to the field is used. For example, consider two documents, DocA and DocB that both have a text field named “color”. Two values are assigned to the DocA “color” field in the order (red, blue), and two values to DocB in the order (green, red). When you perform a sort specifying the text field “color”, DocA is sorted on the value “red” and DocB on the value “green”. The other field values are not used in the sort.

### To sort or not to sort

If you do not specify any sort options, your search results are automatically returned sorted by descending rank. There is no limit to the number of documents that are returned in this case. If you specify any sorting options, the sort is performed after all the matching documents have been selected. There is an explicit property, `SortOptions.limit`, that controls the size of the sort. You can never sort more than 10,000 docs, the default is 1,000. If there are more matching documents than the number specified by `SortOptions.limit`, search only retrieves, sorts, and returns that limited number. It selects the documents to sort from the list of all matching documents, which is in descending rank order. It is possible that a query may select more matching documents than you can sort. If you are using sort options and it is important to retrieve every matching document, you should try to insure that your query will return no more documents than you can sort.

## Writing expressions

Expressions are used to define field expressions (which are set in the `QueryOptions`) and sort expressions (which are set in the `SortOptions`). They are written as strings:

```
"price * quantity"
"(men + women)/2"
"min(daily_use, 10) * rate"
"snippet('rose', flower, 120)"
```

Expressions involving Number fields can use the arithmetical operators (+, -, \*, /) and the built-in numeric functions listed below. Expressions involving geopoint fields can use the geopoint and distance functions. Expressions for Text and HTML fields can use the snippet function.

Expressions can also include these special terms:

<table>
<thead>
<tr class="header">
<th>Term</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>_rank</code></td>
<td>A document's <a href="https://web.archive.org/web/20160424230739/https://cloud.google.com/appengine/docs/java/search/#other_document_properties">rank</a> property. It can be used in field expressions and sort expressions.</td>
</tr>
<tr class="even">
<td><code>_score</code></td>
<td>The score assigned to a document when you include a <a href="https://web.archive.org/web/20160424230739/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/search/MatchScorer"><code>MatchScorer</code></a> in the <code>SortOptions</code>. This term can only appear in sort expressions; it cannot be used in field expressions.</td>
</tr>
</tbody>
</table>

### Numeric functions

The expressions to define numeric values for `FieldExpressions` and `SortExpressions` can use these built-in functions. The arguments must be numbers, field names, or expressions using numbers and field names.

<table>
<thead>
<tr class="header">
<th>Function</th>
<th>Description</th>
<th>Example</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>max</code></td>
<td>Returns the largest of its arguments.</td>
<td><code>max(recommended_retail_price, discount_price, wholesale_price)</code></td>
</tr>
<tr class="even">
<td><code>min</code></td>
<td>Returns the smallest of its arguments.</td>
<td><code>min(height, width, length)</code></td>
</tr>
<tr class="odd">
<td><code>log</code></td>
<td>Returns the natural logarithm.</td>
<td><code>log(x)</code></td>
</tr>
<tr class="even">
<td><code>abs</code></td>
<td>Returns the absolute value.</td>
<td><code>abs(x)</code></td>
</tr>
<tr class="odd">
<td><code>pow</code></td>
<td>Takes two numeric arguments. The call pow(x, y) computes the value of x raised to the y power.</td>
<td><code>pow(x, 2)</code></td>
</tr>
<tr class="even">
<td><code>count</code></td>
<td>Takes a field name as its argument. Returns the number of fields in the document with that name. Remember that a document can contain multiple fields of different types with the same name. Note: <code>count</code> can only be used in <code>FieldExpressions</code>. It cannot appear in <code>SortExpressions</code>.</td>
<td><code>count(user)</code></td>
</tr>
</tbody>
</table>

### Geopoint functions

These functions can be used for expressions involving geopoint fields.

<table>
<thead>
<tr class="header">
<th>Function</th>
<th>Description</th>
<th>Example</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>geopoint</code></td>
<td>Defines a geopoint given a latitude and longitude.</td>
<td><code>geopoint(-31.3, 151.4)</code></td>
</tr>
<tr class="even">
<td><code>distance</code></td>
<td>Computes the distance in meters between two geopoints. Note that either of the two arguments can be the name of a geopoint field or an invocation of the geopoint function. However, only one argument can be a field name.</td>
<td><code>distance(geopoint(23, 134), store_location)</code></td>
</tr>
</tbody>
</table>

### Snippets

A snippet is a fragment of a text field that matches a query string and includes the surrounding text. Snippets are created by calling the `snippet` function:

<span id="Java_Snippet_function"></span> `snippet(query, body, [max_chars])`

###### `query`

A quoted query string specifying the text to find in the field.

###### `body`

The name of a text, HTML, or atom field.

###### `max_chars`

The maximum number of characters to return in the snippet. This argument is optional; it defaults to 160 characters.

The function returns an HTML string. The string contains a snippet of the body field's value, with the text that matched the query in boldface.
