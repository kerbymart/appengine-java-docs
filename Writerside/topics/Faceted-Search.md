# Faceted Search

  

Faceted search gives you the ability to attach categorical information to your documents. A facet is an attribute/value pair. For example, the facet named "size" might have values "small", "medium", and "large."

By using facets with search, you can retrieve summary information to help you refine a query and "drill down" into your results in a series of steps.

The aggregated data for a facet shows you how a facet's values are distributed. For instance, the facet "size" may appear in many of the documents in your result set. The aggregated data for that facet might show that the value "small" appeared 100 times, "medium" 300 times, and "large" 250 times. Each facet/value pair represents a subset of documents in the query result. A key, called a *refinement*, is associated with each pair. You can include refinements in a query to retrieve documents that match the query string and that have the facet values corresponding to one or more refinements.

When you perform a search, you can choose which facets to collect and show with the results, or you can enable facet discovery to automatically select the facets that appear most often in your documents.

1.  [Adding facets to a document](#adding_facets_to_a_document)
2.  [Using a faceted search to retrieve facet information](#using_a_faceted_search_to_retrieve_facet_information)
    1.  [Automatic facet discovery](#automatic_facet_discovery)
    2.  [Selecting facets by name](#selecting_facets_by_name)
    3.  [Selecting facets by name and value](#selecting_facets_by_name_and_value)
    4.  [Options](#options)
3.  [Retrieving facet results](#retrieving_facet_results)
4.  [Using facets to refine/filter a query](#using_facets_to_refinefilter_a_query)

## Adding facets to a document

Add facets to a document before you add the document to an index. Do this at the same time you specify the document's fields:

```
package com.google.test.facet;

import java.io.IOException;
import javax.servlet.http.*;
import com.google.appengine.api.search.*;

public class FacetsearchjavaServlet extends HttpServlet {
  public void doPut(HttpServletRequest req, HttpServletResponse resp) throws IOException {
    Document doc1 = Document.newBuilder()
      .setId("doc1")
      .addField(Field.newBuilder().setName("name").setAtom("x86"))
      .addFacet(Facet.withAtom("type", "computer"))
      .addFacet(Facet.withNumber("ram_size_gb", 8.0))
      .build();
    IndexSpec indexSpec = IndexSpec.newBuilder().setName("products").build();
      Index index = SearchServiceFactory.getSearchService().getIndex(indexSpec);
    index.put(doc1);
  }
}
```

A facet is similar to a document field; it has a name, and takes one value.

Facet names follow the same rules as document fields: Names are case sensitive and can only contain ASCII characters. They must start with a letter and can contain letters, digits, or underscore. A name cannot be longer than 500 characters.

The value of a facet can be either an atomic string (no longer than 500 characters) or a number (a double precision floating point value between -2,147,483,647 and 2,147,483,647).

You can assign multiple values to a facet on one document by adding a facet with the same name and type many times, using a different value each time.

There is no limit to the number of values a facet can have. There is also no limit to the number of facets that you can add to a document or the number of uniquely-named facets in an index.

Note that each time you use a facet, it can take either an atomic or numeric value. A facet with the name "size" can be attached to one document with the string value "small" and another document with the numeric value 8. In fact, the same facet can appear multiple times on the same document with both kinds of values. We do not recommend using both atom and number values for the same facet, even though it is allowed.

While a facet has a specific type when you add it to a document, the search results gather *all* of its values together. For example, the results for facet "size" might show that there were 100 instances of the value "small", 150 instances of "medium", and 135 instances of numeric values in the range \[4, 8). The exact numeric values and their frequency distribution are not shown.

When you retrieve a document using a query, you cannot directly access its facets and values. You must request that facet information be returned with your query, as explained in the next section.

## Using a faceted search to retrieve facet information

You can ask the search backend to discover the most frequently used facets for you, this is called automatic discovery. You can also retrieve facet information explicitly by selecting a facet by name, or by name and value. You can mix and match all three kinds of facet retrieval in a single query.

Asking for facet information will not affect the documents your query returns. It can affect performance. Performing a faceted search with the default depth of 1000 has the same effect as setting the sort options scorer limit to 1000.

### Automatic facet discovery

Automatic facet discovery looks for the facets that appear most often *in the aggregate* in your documents. For example, suppose the documents matching your query include a "color" facet that appears 5 times with the value "red", 5 times with the value "white", and 5 times with the color "blue". This facet has a total count of 15. For the purposes of discovery, it would be ranked higher than another facet "shade" that appears in the same matching documents 6 times with the value "dark" and 7 times with the value "light".

You must enable facet discovery by setting it in your Query:

```
package com.google.test.facet;

import java.io.IOException;
import javax.servlet.http.*;
import com.google.appengine.api.search.*;

public class FacetsearchjavaServlet extends HttpServlet {
  public void doGet(HttpServletRequest req, HttpServletResponse resp) throws IOException {
    IndexSpec indexSpec = IndexSpec.newBuilder().setName("products").build();
      Index index = SearchServiceFactory.getSearchService().getIndex(indexSpec);
      Results<ScoredDocument> result = index.search(
          Query.newBuilder().setEnableFacetDiscovery(true) // enable discovery
          .build("name:x86"));
      for(FacetResult facetResult : result.getFacets()) {
        resp.getWriter().printf("Facet %s:\n", facetResult.getName());
        for (FacetResultValue facetValue : facetResult.getValues()) {
          resp.getWriter().printf("   %s: Count=%s, RefinementKey=%s\n",
              facetValue.getLabel(),
              facetValue.getCount(),
              facetValue.getRefinementToken());
        }
      }
  }
}
```

When you retrieve facets by discovery, by default only the 10 most often occuring values for a facet will be returned. You can increase this limit up to 100 using `FacetOptions.Builder.setDiscoveryValueLimit()`.

String values will be returned individually. The numeric values of a discovered facet are returned in a single range \[min max). You can examine this range and create a smaller subrange for a later query.

### Selecting facets by name

To retrieve information about a facet by its name only, add a `ReturnFacet` object to your query, specifying the facet name:
```
package com.google.test.facet;

import java.io.IOException;
import javax.servlet.http.*;
import com.google.appengine.api.search.*;

public class FacetsearchjavaServlet extends HttpServlet {
  public void doGet(HttpServletRequest req, HttpServletResponse resp) throws IOException {
    IndexSpec indexSpec = IndexSpec.newBuilder().setName("products").build();
      Index index = SearchServiceFactory.getSearchService().getIndex(indexSpec);
      Results<ScoredDocument> result = index.search(Query.newBuilder()
        .addReturnFacet("type")
        .addReturnFacet("ram_size_gb")
        .build("name:x86"));
      for(FacetResult facetResult : result.getFacets()) {
        resp.getWriter().printf("Facet %s:\n", facetResult.getName());
        for (FacetResultValue facetValue : facetResult.getValues()) {
          resp.getWriter().printf("   %s: Count=%s, RefinementKey=%s\n",
              facetValue.getLabel(),
              facetValue.getCount(),
              facetValue.getRefinementToken());
        }
      }
  }
}
```

When you retrieve facets by name, by default only the 10 most often occuring values for a facet will be returned. You can increase this limit up to 20 using `FacetOptions.Builder.setDiscoveryValueLimit()`.

### Selecting facets by name and value

To retrieve a facet with a particular value, add a `ReturnFacet` object that includes a `FacetRequest`:
```
package com.google.test.facet;

import java.io.IOException;
import javax.servlet.http.*;
import com.google.appengine.api.search.*;

public class FacetsearchjavaServlet extends HttpServlet {
    public void doGet(HttpServletRequest req, HttpServletResponse resp) throws IOException {
        IndexSpec indexSpec = IndexSpec.newBuilder().setName("products").build();
        Index index = SearchServiceFactory.getSearchService().getIndex(indexSpec);
        // Fetch the "type" facet with values "computer and "printer"
        // along with the "ram_size_gb" facet with values in the ranges [0,4), [4, 8), and [8, max]
        Results<ScoredDocument> result = index.search(Query.newBuilder()
            .addReturnFacet(FacetRequest.newBuilder()
                .setName("type")
                .addValueConstraint("computer")
                .addValueConstraint("printer"))
            .addReturnFacet(FacetRequest.newBuilder()
                .setName("ram_size_gb")
                .addRange(FacetRange.withEnd(4.0))
                .addRange(FacetRange.withStartEnd(4.0, 8.0))
                .addRange(FacetRange.withStart(8.0)))
            .build("name:x86"));
        for(FacetResult facetResult : result.getFacets()) {
            resp.getWriter().printf("Facet %s:\n", facetResult.getName());
            for (FacetResultValue facetValue : facetResult.getValues()) {
                resp.getWriter().printf("   %s: Count=%s, RefinementKey=%s\n",
                        facetValue.getLabel(),
                        facetValue.getCount(),
                        facetValue.getRefinementToken());
            }
        }
    }
}
```

The values in a single `FacetRequest` must all be the same type, either a list of string values or, for numbers, a list of `FacetRanges`, which are intervals that are closed on the left (start), and open on the right (end). If your facet has a mix of string and number values, add separate FacetRequests for each.

### Options

You can control faceted search by adding a `FacetOptions` parameter to a Query call. This parameter takes a single instance of `FacetOptions`. Use this parameter to override the default behavior of faceted search.
```
Results<ScoredDocument> results = index.search(Query.newBuilder()
  .addReturnFacet(FacetRequest.newBuilder()
    .setName("type")
    .addValueConstraint("computer")
    .addValueConstraint("printer"))
  .addReturnFacet(FacetRequest.newBuilder()
    .setName("ram_size_gb")
    .addRange(FacetRange.withEnd(4.0))
    .addRange(FacetRange.withStartEnd(4.0, 8.0))
    .addRange(FacetRange.withStart(8.0)))
  .setFacetOptions(FacetOptions.newBuilder()
    .setDiscoveryLimit(5)
    .setDiscoveryValueLimit(10)
    .setDepth(6000).build());
  .build(some_query);
```

<table>
<thead>
<tr class="header">
<th>Parameter</th>
<th>Description</th>
<th>Default</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>DiscoveryLimit</code></td>
<td>Number of facets to discover if facet discovery is turned on. If 0, discover facets will be disabled.</td>
<td>10</td>
</tr>
<tr class="even">
<td><code>DiscoveryValueLimit</code></td>
<td>Number of values to be returned for each of the top discovered facets.</td>
<td>10</td>
</tr>
<tr class="odd">
<td><code>Depth</code></td>
<td>The minimum number of documents in query results to evaluate to gather facet information.</td>
<td>1000</td>
</tr>
</tbody>
</table>

The `Depth` option applies to all three kinds of facet aggregation: by name, name and value, and auto-discovery. The other options are for auto-discovery only.

Note that facet depth is usually much greater than the query limit. Facet results are computed to at least the depth number of documents. If you have set the sort options scoring limit higher than depth, than the scoring limit will be used instead.

## Retrieving facet results

When you use faceted search parameters in a query, the aggregated facet information comes with the query result itself.

A query will have a list of `FacetResult`. There will be one result in the list for each facet that appeared in a document that matched your query. For each result, you'll get:

-   The facet name
-   A list of the most frequent values for the facet, for each value there is a count of how many times it appeared, and a refinement key that can be used to retrieve the documents that match this query and facet value.

Note that the values list will include a facet's string *and* numeric values. If the facet was auto-discovered, its numeric values are returned as a single interval \[min max). If you explicitly asked for a numeric facet with one or more ranges in your query, the list will contain one closed-open interval \[start end) for each range.

The list of facet values might not include all of the values found in your documents, since query options determine how many documents to examine and how many values to return.

The aggregated information for each facet can be read from the search results:

```
Results<ScoredDocument> results = index.search(...);
for (FacetResult facetInfo : results.getFacets()) {
  ...
}
```

For example, a query may have found documents that included a "size" facet with the string values and numeric values. The FacetResult for this facet will be constructed like this:

```
FacetResult.newBuilder()
        .setName("size")
        .addValue(FacetResultValue.create("[8, 10)", 22, refinement_key)
        .addValue(FacetResultValue.create("small", 100, refinement_key)
        .addValue(FacetResultValue.create("medium", 300, refinement_key)
        .addValue(FacetResultValue.create("large", 250, refinement_key).build());
```

FacetResultValue.label is constructed from a facet value. Numeric values are returned as a "stringified" representation of a range.

The `refinement_key` is a web/url safe string that can be used in a later query to retrieve the documents matching that result's facet name and value.

## Using facets to refine/filter a query

```
Query query = Query.newBuilder()
  .addFacetRefinementFromToken(refinement_key1)
  .addFacetRefinementFromToken(refinement_key2)
  .addFacetRefinementFromToken(refinement_key3)
  .build(some_query);
```

You can combine refinements for one or more different facets in the same request. All the refinements belonging to the same facet are joined with an OR. Refinements for different facets are combined with AND.

It is also possible to create a custom `FacetRefinement` key by hand. Please see the class documentation for more information.
