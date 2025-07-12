# Datastore Queries

  

A Datastore *query* retrieves *[entities](https://web.archive.org/web/20160424230618/https://cloud.google.com/appengine/docs/java/datastore/entities)* from the App Engine Datastore that meet a specified set of conditions. The query operates on entities of a given *[kind](https://web.archive.org/web/20160424230618/https://cloud.google.com/appengine/docs/java/datastore/entities#Java_Kinds_and_identifiers)*; it can specify *[filters](#Java_Filters)* on the entities' property values, keys, and ancestors, and can return zero or more entities as *results.* A query can also specify *[sort orders](#Java_Sort_orders)* to sequence the results by their property values. The results include all entities that have at least one (possibly null) value for every property named in the filters and sort orders, and whose property values meet all the specified filter criteria. The query can return entire entities, [projected entities](https://web.archive.org/web/20160424230618/https://cloud.google.com/appengine/docs/java/datastore/projectionqueries), or just entity [keys](https://web.archive.org/web/20160424230618/https://cloud.google.com/appengine/docs/java/datastore/#Java_Kinds_keys_and_identifiers).

A typical query includes the following:

-   An *[entity kind](https://web.archive.org/web/20160424230618/https://cloud.google.com/appengine/docs/java/datastore/entities#Java_Kinds_and_identifiers)* to which the query applies
-   Zero or more *[filters](#Java_Filters)* based on the entities' property values, keys, and ancestors
-   Zero or more *[sort orders](#Java_Sort_orders)* to sequence the results

When executed, the query retrieves all entities of the given kind that satisfy all of the given filters, sorted in the specified order. Queries execute as read-only.

**Note:** To conserve memory and improve performance, a query should, whenever possible, specify a limit on the number of results returned.

Every Datastore query computes its results using one or more *[indexes](https://web.archive.org/web/20160424230618/https://cloud.google.com/appengine/docs/java/datastore/indexes),* which contain entity keys in a sequence specified by the index's properties and, optionally, the entity's ancestors. The indexes are updated incrementally to reflect any changes the application makes to its entities, so that the correct results of all queries are available with no further computation needed.

**Note:** The index-based query mechanism supports a wide range of queries and is suitable for most applications. However, it does not support some kinds of query common in other database technologies: in particular, joins and aggregate queries aren't supported within the Datastore query engine. See [Restrictions on Queries](#Java_Restrictions_on_queries), below, for limitations on Datastore queries.

## Contents

1.  [Java query interface](#Java_query_interface)
2.  [Query structure](#Java_Query_structure)
    1.  [Filters](#Java_Filters)
        1.  [Property filters](#Java_Property_filters)
        2.  [Key filters](#Java_Key_filters)
        3.  [Ancestor filters](#Java_Ancestor_filters)
    2.  [Sort orders](#Java_Sort_orders)
    3.  [Special query types](#Java_Special_query_types)
        1.  [Kindless queries](#Java_Kindless_queries)
        2.  [Ancestor queries](#Java_Ancestor_queries)
        3.  [Kindless ancestor queries](#Java_Kindless_ancestor_queries)
        4.  [Keys-only queries](#Java_Keys_only_queries)
        5.  [Projection queries](#Java_Projection_queries)
3.  [Restrictions on queries](#Java_Restrictions_on_queries)
4.  [Retrieving results](#Java_Retrieving_results)
5.  [Offsets versus cursors](#Java_Offsets_versus_cursors)
6.  [Query cursors](#Java_Query_cursors)
    1.  [Limitations of cursors](#Java_Limitations_of_cursors)
    2.  [Cursors and data updates](#Java_Cursors_and_data_updates)
7.  [Data consistency](#Java_Data_consistency)

## Java query interface

The low-level Java Datastore API provides class [`Query`](https://web.archive.org/web/20160424230618/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/Query.html) for constructing queries and the [`PreparedQuery`](https://web.archive.org/web/20160424230618/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/PreparedQuery.html) interface for retrieving entities from the Datastore:

```
import com.google.appengine.api.datastore.DatastoreServiceFactory;
import com.google.appengine.api.datastore.DatastoreService;
import com.google.appengine.api.datastore.Query.Filter;
import com.google.appengine.api.datastore.Query.FilterPredicate;
import com.google.appengine.api.datastore.Query.FilterOperator;
import com.google.appengine.api.datastore.Query.CompositeFilter;
import com.google.appengine.api.datastore.Query.CompositeFilterOperator;
import com.google.appengine.api.datastore.Query;
import com.google.appengine.api.datastore.PreparedQuery;
import com.google.appengine.api.datastore.Entity;

// Get the Datastore Service
DatastoreService datastore = DatastoreServiceFactory.getDatastoreService();

Filter heightMinFilter =
  new FilterPredicate("height",
                      FilterOperator.GREATER_THAN_OR_EQUAL,
                      minHeight);

Filter heightMaxFilter =
  new FilterPredicate("height",
                      FilterOperator.LESS_THAN_OR_EQUAL,
                      maxHeight);

//Use CompositeFilter to combine multiple filters
Filter heightRangeFilter =
  CompositeFilterOperator.and(heightMinFilter, heightMaxFilter);


// Use class Query to assemble a query
Query q = new Query("Person").setFilter(heightRangeFilter);

// Use PreparedQuery interface to retrieve results
PreparedQuery pq = datastore.prepare(q);


for (Entity result : pq.asIterable()) {
  String firstName = (String) result.getProperty("firstName");
  String lastName = (String) result.getProperty("lastName");
  Long height = (Long) result.getProperty("height");

  System.out.println(firstName + " " + lastName + ", " + height + " inches tall");
}
```

Notice the use of [`FilterPredicate`](https://web.archive.org/web/20160424230618/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/Query.FilterPredicate.html) and [`CompositeFilter`](https://web.archive.org/web/20160424230618/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/Query.CompositeFilter.html) to construct filters. If you're setting only one filter on a query, you can just use `FilterPredicate` by itself:

```
Filter heightMinFilter =
  new FilterPredicate("height",
                      FilterOperator.GREATER_THAN_OR_EQUAL,
                      minHeight);

Query q = new Query("Person").setFilter(heightMinFilter);
```

However, if you want to set more than one filter on a query, you must use `CompositeFilter`, which requires at least two filters. The example above uses the shortcut helper `CompositeFilterOperator.and`; the following example shows one way of constructing a composite OR filter:

```
Filter tooShortFilter =
  new FilterPredicate("height", FilterOperator.LESS_THAN, minHeight);

Filter tooTallFilter =
  new FilterPredicate("height", FilterOperator.GREATER_THAN, maxHeight);

Filter heightOutOfRangeFilter =
  CompositeFilterOperator.or(tooShortFilter, tooTallFilter);

Query q = new Query("Person").setFilter(heightOutOfRangeFilter);
```
**Caution:** Note that using an OR filter will cause multiple queries to be executed against the underlying Datastore.

**Note:** The properties being filtered on must have a corresponding predefined index which can be defined in your [index configuration file](https://web.archive.org/web/20160424230618/https://cloud.google.com/appengine/docs/java/config/indexconfig) (`datastore-indexes.xml`).

## Query structure

A query can specify an [entity kind](https://web.archive.org/web/20160424230618/https://cloud.google.com/appengine/docs/java/datastore/entities#Java_Kinds_and_identifiers), zero or more [filters](#Java_Filters), and zero or more [sort orders](#Java_Sort_orders).

### Filters

A query's *filters* set constraints on the [properties](#Java_Property_filters), [keys](#Java_Key_filters), and [ancestors](#Java_Ancestor_filters) of the entities to be retrieved.

#### Property filters

A *property filter* specifies

-   A property name
-   A comparison operator
-   A property value

For example:
```
Filter propertyFilter =
  new FilterPredicate("height",
                      FilterOperator.GREATER_THAN_OR_EQUAL,
                      minHeight);
Query q = new Query("Person").setFilter(propertyFilter);
```

The property value must be supplied by the application; it cannot refer to or be calculated in terms of other properties. An entity satisfies the filter if it has a property of the given name whose value compares to the value specified in the filter in the manner described by the comparison operator.

The comparison operator can be any of the following (defined as enumerated constants in the nested class ` `[`Query.FilterOperator`](https://web.archive.org/web/20160424230618/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/Query.FilterOperator)):

<table>
<thead>
<tr class="header">
<th>Operator</th>
<th>Meaning</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>EQUAL</code></td>
<td>Equal to</td>
</tr>
<tr class="even">
<td><code>LESS_THAN</code></td>
<td>Less than</td>
</tr>
<tr class="odd">
<td><code>LESS_THAN_OR_EQUAL</code></td>
<td>Less than or equal to</td>
</tr>
<tr class="even">
<td><code>GREATER_THAN</code></td>
<td>Greater than</td>
</tr>
<tr class="odd">
<td><code>GREATER_THAN_OR_EQUAL</code></td>
<td>Greater than or equal to</td>
</tr>
<tr class="even">
<td><code>NOT_EQUAL</code></td>
<td>Not equal to</td>
</tr>
<tr class="odd">
<td><code>IN</code></td>
<td>Member of (equal to any of the values in a specified list)</td>
</tr>
</tbody>
</table>

The `NOT_EQUAL` operator actually performs two queries: one in which all other filters are unchanged and the `NOT_EQUAL` filter is replaced with a `LESS_THAN` filter, and one where it is replaced with a `GREATER_THAN` filter. The results are then merged, in order. A query can have no more than one `NOT_EQUAL` filter, and a query that has one cannot have any other inequality filters.

The `IN` operator also performs multiple queries: one for each item in the specified list, with all other filters unchanged and the `IN` filter replaced with an `EQUAL` filter. The results are merged in order of the items in the list. If a query has more than one `IN` filter, it is performed as multiple queries, one for each possible *combination* of values in the `IN` lists.

A single query containing `NOT_EQUAL` or `IN` operators is limited to no more than 30 subqueries.

For more information about how `NOT_EQUAL` and `IN` queries translate to multiple queries in a JDO/JPA framework, see the article [Queries with != and IN filters](https://web.archive.org/web/20160424230618/http://gae-java-persistence.blogspot.com/2009/12/queries-with-and-in-filters.html).

#### Key filters

To filter on the value of an entity's key, use the special property [`Entity.KEY_RESERVED_PROPERTY`](https://web.archive.org/web/20160424230618/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/Entity#KEY_RESERVED_PROPERTY):

```
Filter keyFilter =
  new FilterPredicate(Entity.KEY_RESERVED_PROPERTY,
                      FilterOperator.GREATER_THAN,
                      lastSeenKey)
Query q =  new Query("Person").setFilter(keyFilter);
```

Ascending sorts on `Entity.KEY_RESERVED_PROPERTY` are also supported.

When comparing for inequality, keys are ordered by the following criteria, in order:

1.  Ancestor path
2.  Entity kind
3.  Identifier (key name or numeric ID)

Elements of the ancestor path are compared similarly: by kind (string), then by key name or numeric ID. Kinds and key names are strings and are ordered by byte value; numeric IDs are integers and are ordered numerically. If entities with the same parent and kind use a mix of key name strings and numeric IDs, those with numeric IDs precede those with key names.

Queries on keys use indexes just like queries on properties and require custom indexes in the same cases, with a couple of exceptions: inequality filters or an ascending sort order on the key do not require a custom index, but a descending sort order on the key does. As with all queries, the development web server creates appropriate entries in the [index configuration file](https://web.archive.org/web/20160424230618/https://cloud.google.com/appengine/docs/java/datastore/indexes#Java_Index_configuration) when a query that needs a custom index is tested.

#### Ancestor filters

You can filter your Datastore queries to a specified [*ancestor*](https://web.archive.org/web/20160424230618/https://cloud.google.com/appengine/docs/java/datastore/entities#Java_Ancestor_paths), so that the results returned will include only entities descended from that ancestor:

```
Query q = new Query("Person")
                .setAncestor(ancestorKey);
```
**Caution:** Passing `null` as the ancestor key does *not* query for root entities (those without ancestors). This type of query is not currently supported and will return an error.

### Sort orders

A query *sort order* specifies

-   A property name
-   A sort direction (ascending or descending)

For example:

```
// Order alphabetically by last name:
Query q = new Query("Person")
                .addSort("lastName", SortDirection.ASCENDING);

// Order by height, tallest to shortest:
Query q = new Query("Person")
                .addSort("height", SortDirection.DESCENDING);
```

If a query includes multiple sort orders, they are applied in the sequence specified. The following example sorts first by ascending last name and then by descending height:

```
Query q = new Query("Person")
                .addSort("lastName", SortDirection.ASCENDING)
                .addSort("height", SortDirection.DESCENDING);
```

If no sort orders are specified, the results are returned in the order they are retrieved from the Datastore.

**Note:** Because of the way the App Engine Datastore executes queries, if a query specifies inequality filters on a property and sort orders on other properties, the property used in the inequality filters must be ordered before the other properties.

### Special query types

Some specific types of query deserve special mention:

#### Kindless queries

A query with no kind and no ancestor filter retrieves all of the entities of an application from the Datastore. This includes entities created and managed by other App Engine features, such as [statistics entities](https://web.archive.org/web/20160424230618/https://cloud.google.com/appengine/docs/java/datastore/stats) and [Blobstore metadata entities](https://web.archive.org/web/20160424230618/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/blobstore/package-summary) (if any). Such *kindless queries* cannot include filters or sort orders on property values. They can, however, filter on entity keys by specifying [`Entity.KEY_RESERVED_PROPERTY`](https://web.archive.org/web/20160424230618/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/Entity#KEY_RESERVED_PROPERTY) as the property name:

```
Filter keyFilter =
  new FilterPredicate(Entity.KEY_RESERVED_PROPERTY,
                      FilterOperator.GREATER_THAN,
                      lastSeenKey)
Query q =  new Query().setFilter(keyFilter);
```

#### Ancestor queries

A query with an [ancestor filter](#Java_Ancestor_filters) limits its results to the specified entity and its descendants:

```
DatastoreService datastore = DatastoreServiceFactory.getDatastoreService();

Entity tom = new Entity("Person", "Tom");
Key tomKey = tom.getKey();
datastore.put(tom);


Entity weddingPhoto = new Entity("Photo", tomKey);
weddingPhoto.setProperty("imageURL",
                         "http://domain.com/some/path/to/wedding_photo.jpg");

Entity babyPhoto = new Entity("Photo", tomKey);
babyPhoto.setProperty("imageURL",
                      "http://domain.com/some/path/to/baby_photo.jpg");

Entity dancePhoto = new Entity("Photo", tomKey);
dancePhoto.setProperty("imageURL",
                       "http://domain.com/some/path/to/dance_photo.jpg");

Entity campingPhoto = new Entity("Photo");
campingPhoto.setProperty("imageURL",
                         "http://domain.com/some/path/to/camping_photo.jpg");

List<Entity> photoList = Arrays.asList(weddingPhoto, babyPhoto,
                                       dancePhoto, campingPhoto);
datastore.put(photoList);


Query photoQuery = new Query("Photo")
                         .setAncestor(tomKey);  


// This returns weddingPhoto, babyPhoto, and dancePhoto,
// but not campingPhoto, because tom is not an ancestor
List<Entity> results = datastore.prepare(photoQuery)
                                .asList(FetchOptions.Builder.withDefaults());
```
**Note:** Setting an [ancestor filter](#Java_Ancestor_filters) allows for strongly consistent queries. Queries without an ancestor filter only return eventually consistent results.

#### Kindless ancestor queries

A kindless query that includes an ancestor filter will retrieve the specified ancestor and all of its descendants, regardless of kind. This type of query does not require custom indexes. Like all kindless queries, it cannot include filters or sort orders on property values, but can filter on the entity's key:

```
Filter keyFilter =
  new FilterPredicate(Entity.KEY_RESERVED_PROPERTY,
                      FilterOperator.GREATER_THAN,
                      lastSeenKey)
Query q =  new Query()
  .setAncestor(ancestorKey)
  .setFilter(keyFilter);
```

The following example illustrates how to retrieve all entities descended from a given ancestor:

```
DatastoreService datastore = DatastoreServiceFactory.getDatastoreService();

Entity tom = new Entity("Person", "Tom");
Key tomKey = tom.getKey();
datastore.put(tom);


Entity weddingPhoto = new Entity("Photo", tomKey);
weddingPhoto.setProperty("imageURL",
                         "http://domain.com/some/path/to/wedding_photo.jpg");

Entity weddingVideo = new Entity("Video", tomKey);
weddingVideo.setProperty("videoURL",
                         "http://domain.com/some/path/to/wedding_video.avi");

List<Entity> mediaList = Arrays.asList(weddingPhoto, weddingVideo);
datastore.put(mediaList);


// By default, ancestor queries include the specified ancestor itself.
// The following filter excludes the ancestor from the query results.
Filter keyFilter
  = new FilterPredicate(Entity.KEY_RESERVED_PROPERTY,
                        FilterOperator.GREATER_THAN,
                        tomKey)

Query mediaQuery = new Query()
  .setAncestor(tomKey)
  .setFilter(keyFilter);


// Returns both weddingPhoto and weddingVideo,
// even though they are of different entity kinds
List<Entity> results = datastore.prepare(mediaQuery)
  .asList(FetchOptions.Builder.withDefaults());
```

#### Keys-only queries

A *keys-only query* returns just the keys of the result entities instead of the entities themselves, at lower latency and cost than retrieving entire entities:

```
Query q = new Query("Person")
                .setKeysOnly();
```

It is often more economical to do a keys-only query first, and then fetch a subset of entities from the results, rather than executing a general query which may fetch more entities than you actually need.

#### Projection queries

Sometimes all you really need from the results of a query are the values of a few specific properties. In such cases, you can use a *projection query* to retrieve just the properties you're actually interested in, at lower latency and cost than retrieving the entire entity; see the [Projection Queries](https://web.archive.org/web/20160424230618/https://cloud.google.com/appengine/docs/java/datastore/projectionqueries) page for details.

## Restrictions on queries

The nature of the index query mechanism imposes certain restrictions on what a query can do:

### Entities lacking a property named in the query are ignored

Entities of the same kind need not have the same properties. To be eligible as a query result, an entity must possess a value (possibly null) for every property named in the query's filters and sort orders. If not, the entity is omitted from the indexes used to execute the query and consequently will not be included in the query's results.

**Note:** It is not possible to query for entities that are specifically *lacking* a given property. One alternative is to use `null` for the default property value, then filter for entities with `null` as the value of that property.

### Filtering on unindexed properties returns no results

A query can't find property values that aren't indexed, nor can it sort on such properties. See the [Datastore Indexes](https://web.archive.org/web/20160424230618/https://cloud.google.com/appengine/docs/java/datastore/indexes#Java_Unindexed_properties) page for a detailed discussion of unindexed properties.

### Inequality filters are limited to at most one property

To avoid having to scan the entire index, the query mechanism relies on all of a query's potential results being adjacent to one another in the index. To satisfy this constraint, a single query may not use inequality comparisons (`LESS_THAN`, `LESS_THAN_OR_EQUAL`, `GREATER_THAN`, `GREATER_THAN_OR_EQUAL`, `NOT_EQUAL`) on more than one property across all of its filters. For example, the following query is valid, because both inequality filters apply to the same property:

```
import com.google.appengine.api.datastore.Query;
import com.google.appengine.api.datastore.Query.Filter;
import com.google.appengine.api.datastore.Query.FilterPredicate;
import com.google.appengine.api.datastore.Query.FilterOperator;
import com.google.appengine.api.datastore.Query.CompositeFilter;
import com.google.appengine.api.datastore.Query.CompositeFilterOperator;

Filter birthYearMinFilter =
  new FilterPredicate("birthYear",
                      FilterOperator.GREATER_THAN_OR_EQUAL,
                      minBirthYear);

Filter birthYearMaxFilter =
  new FilterPredicate("birthYear",
                      FilterOperator.LESS_THAN_OR_EQUAL,
                      maxBirthYear);
 
Filter birthYearRangeFilter =
  CompositeFilterOperator.and(birthYearMinFilter, birthYearMaxFilter);

Query q = new Query("Person").setFilter(birthYearRangeFilter);
```

However, this query is *not* valid, because it uses inequality filters on two different properties:

```
Filter birthYearMinFilter =
  new FilterPredicate("birthYear",
                      FilterOperator.GREATER_THAN_OR_EQUAL,
                      minBirthYear);

Filter heightMaxFilter =
  new FilterPredicate("height",
                      FilterOperator.LESS_THAN_OR_EQUAL,
                      maxHeight);
  
Filter invalidFilter =
  CompositeFilterOperator.and(birthYearMinFilter, heightMaxFilter);

Query q = new Query("Person").setFilter(invalidFilter);
```

Note that a query *can* combine equality (`EQUAL`) filters for different properties, along with one or more inequality filters on a single property. Thus the following *is* a valid query:

```
Filter lastNameFilter =
  new FilterPredicate("lastName",
                      FilterOperator.EQUAL,
                      targetLastName);

Filter cityFilter =
  new FilterPredicate("city",
                      FilterOperator.EQUAL,
                      targetCity);

Filter birthYearMinFilter =
  new FilterPredicate("birthYear",
                      FilterOperator.GREATER_THAN_OR_EQUAL,
                      minBirthYear);

Filter birthYearMaxFilter =
  new FilterPredicate("birthYear",
                      FilterOperator.LESS_THAN_OR_EQUAL,
                      maxBirthYear);

Filter validFilter = CompositeFilterOperator.and(lastNameFilter,
                                                 cityFilter,
                                                 birthYearMinFilter,
                                                 birthYearMaxFilter);

Query q = new Query("Person").setFilter(validFilter);
```

### Ordering of query results is undefined when no sort order is specified

When a query does not specify a sort order, the results are returned in the order they are retrieved. As the Datastore implementation evolves (or if an application's indexes change), this order may change. Therefore, if your application requires its query results in a particular order, be sure to specify that sort order explicitly in the query.

### Sort orders are ignored on properties with equality filters

Queries that include an equality filter for a given property ignore any sort order specified for that property. This is a simple optimization to save needless processing for single-valued properties, since all results have the same value for the property and so no further sorting is needed. Multiple-valued properties, however, may have additional values besides the one matched by the equality filter. Because this use case is rare and applying the sort order would be expensive and require extra indexes, the Datastore query planner simply ignores the sort order even in the multiple-valued case. This may cause query results to be returned in a different order than the sort order appears to imply.

### Properties used in inequality filters must be sorted first

To retrieve all results that match an inequality filter, a query scans the index for the first row matching the filter, then scans forward until it encounters a nonmatching row. For the consecutive rows to encompass the complete result set, they must be ordered by the property used in the inequality filter before any other properties. Thus if a query specifies one or more inequality filters along with one or more sort orders, the first sort order must refer to the same property named in the inequality filters. The following is a valid query:

```
import com.google.appengine.api.datastore.Query;
import com.google.appengine.api.datastore.Query.Filter;
import com.google.appengine.api.datastore.Query.FilterPredicate;
import com.google.appengine.api.datastore.Query.FilterOperator;
import com.google.appengine.api.datastore.Query.CompositeFilter;
import com.google.appengine.api.datastore.Query.CompositeFilterOperator;
import com.google.appengine.api.datastore.Query.SortDirection;

Filter birthYearMinFilter =
  new FilterPredicate("birthYear",
                      FilterOperator.GREATER_THAN_OR_EQUAL,
                      minBirthYear);

Query q = new Query("Person")
                .setFilter(birthYearMinFilter)
                .addSort("birthYear", SortDirection.ASCENDING)
                .addSort("lastName", SortDirection.ASCENDING);
```

This query is *not* valid, because it doesn't sort on the property used in the inequality filter:

```
Filter birthYearMinFilter =
  new FilterPredicate("birthYear",
                      FilterOperator.GREATER_THAN_OR_EQUAL,
                      minBirthYear);

Query q = new Query("Person")
                .setFilter(birthYearMinFilter)
                .addSort("lastName", SortDirection.ASCENDING);
```

Similarly, this query is not valid because the property used in the inequality filter is not the first one sorted:

```
Filter birthYearMinFilter =
  new FilterPredicate("birthYear",
                      FilterOperator.GREATER_THAN_OR_EQUAL,
                      minBirthYear);

Query q = new Query("Person")
                .setFilter(birthYearMinFilter)
                .addSort("lastName", SortDirection.ASCENDING)
                .addSort("birthYear", SortDirection.ASCENDING);
```

### Properties with multiple values can behave in surprising ways

Because of the way they're indexed, entities with multiple values for the same property can sometimes interact with query filters and sort orders in unexpected and surprising ways.

If a query has multiple inequality filters on a given property, an entity will match the query only if at least one of its individual values for the property satisfies *all* of the filters. For example, if an entity of kind `Widget` has values `1` and `2` for property `x`, it will *not* match the query:

```
Query q = new Query("Widget")
  .setFilter(new FilterPredicate("x", FilterOperator.GREATER_THAN, 1))
  .setFilter(new FilterPredicate("x", FilterOperator.LESS_THAN, 2));
```

Each of the entity's `x` values satisfies one of the filters, but neither single value satisfies both. Note that this does not apply to equality filters. For example, the same entity *will* satisfy the query

```
Query q = new Query("Widget")
  .setFilter(new FilterPredicate("x", FilterOperator.EQUAL, 1))
  .setFilter(new FilterPredicate("x", FilterOperator.EQUAL, 2));
```

even though neither of the entity's individual `x` values satisfies both filter conditions.

The `NOT_EQUAL` operator works as a "value is other than" test. So, for example, the query

```
Query q = new Query("Widget")
  .setFilter(new FilterPredicate("x", FilterOperator.NOT_EQUAL, 1));
```

matches any `Widget` entity with an `x` value other than `1`.

In Java, you can also use a query like

```
Query q = new Query("Widget")
  .setFilter(new FilterPredicate("x", FilterOperator.NOT_EQUAL, 1))
  .setFilter(new FilterPredicate("x", FilterOperator.NOT_EQUAL, 2));
```

which acts like

```
x < 1 OR (x > 1 AND x < 2) OR x > 2
```

so a `Widget` entity with `x` values `1`, `2`, and `3` matches, but one with values `1` and `2` does not.

Similarly, the sort order for multiple-valued properties is unusual. Because such properties appear once in the index for each unique value, the first value seen in the index determines an entity's sort order:

-   If the query results are sorted in ascending order, the smallest value of the property is used for ordering.
-   If the results are sorted in descending order, the greatest value is used for ordering.
-   Other values do not affect the sort order, nor does the number of values.

This has the unusual consequence that an entity with property values `1` and `9` precedes one with values `4`, `5`, `6`, and `7` in both ascending *and* descending order.

### Queries inside transactions must include ancestor filters

Datastore [transactions](https://web.archive.org/web/20160424230618/https://cloud.google.com/appengine/docs/java/datastore/transactions) operate only on entities belonging to the same [entity group](https://web.archive.org/web/20160424230618/https://cloud.google.com/appengine/docs/java/datastore/#Java_Ancestor_paths) (descended from a common ancestor). To preserve this restriction, all queries performed within a transaction must include an [ancestor filter](#Java_Ancestor_filters) specifying an ancestor in the same entity group as the other operations in the transaction.

## Retrieving results

After constructing a query, you can specify a number of retrieval options to further control the results it returns.

To retrieve just a single entity matching your query, use the method [`PreparedQuery.asSingleEntity()`](https://web.archive.org/web/20160424230618/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/PreparedQuery#asSingleEntity()):

```
Query q = new Query("Person")
  .setFilter(new FilterPredicate("lastName",
                                 FilterOperator.EQUAL,
                                 targetLastName));

PreparedQuery pq = datastore.prepare(q);
Entity result = pq.asSingleEntity();
```

This returns the first result found in the index that matches the query. (If there is more than one matching result, it throws a [`TooManyResultsException`](https://web.archive.org/web/20160424230618/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/PreparedQuery.TooManyResultsException).

To retrieve only selected properties of an entity rather than the entire entity, use a [*projection query*](https://web.archive.org/web/20160424230618/https://cloud.google.com/appengine/docs/java/datastore/projectionqueries). This type of query runs faster and costs less than one that returns complete entities.

Similarly, a [*keys-only query*](#Java_Keys_only_queries) saves time and resources by returning just the keys to the entities it matches, rather than the full entities themselves. To create this type of query, use the [`Query.setKeysOnly()`](https://web.archive.org/web/20160424230618/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/Query#setKeysOnly()) method:

```
Query q = new Query("Person")
                .setKeysOnly();
```

You can specify a *limit* for your query to control the maximum number of results returned in one batch. The following example retrieves the five tallest people from the Datastore:

```
List<Entity> getTallestPeople() {
  DatastoreService datastore = DatastoreServiceFactory.getDatastoreService();

  Query q = new Query("Person")
                  .addSort("height", SortDirection.DESCENDING);

  PreparedQuery pq = datastore.prepare(q);
  return pq.asList(FetchOptions.Builder.withLimit(5));
}
```

When iterating through the results of a query using the [`PreparedQuery.asIterable()`](https://web.archive.org/web/20160424230618/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/PreparedQuery#asIterable()) and [`PreparedQuery.asIterator()`](https://web.archive.org/web/20160424230618/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/PreparedQuery#asIterator()) methods, the Datastore retrieves the results in batches. By default each batch contains 20 results, but you can change this value using [`FetchOptions.chunkSize()`](https://web.archive.org/web/20160424230618/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/FetchOptions#chunkSize(int)). You can continue iterating through query results until all are returned or the request times out.

## Offsets versus cursors

**Although Datastore supports integer offsets, you should avoid using them.** Instead, use cursors. Using an offset only avoids returning the skipped entities to your application, but these entities are still retrieved internally. The skipped entities do affect the latency of the query, and your application is billed for the read operations required to retrieve them. Using cursors instead of offsets lets you avoid all these costs.

## Query cursors

*Query cursors* allow an application to retrieve a query's results in convenient batches without incurring the overhead of a query offset. After performing a [retrieval operation](#Java_Retrieving_results), the application can obtain a cursor, which is an opaque base64-encoded string marking the index position of the last result retrieved. The application can save this string (for instance in the Datastore, in Memcache, in a Task Queue task payload, or embedded in a web page as an HTTP `GET` or `POST` parameter), and can then use the cursor as the starting point for a subsequent retrieval operation to obtain the next batch of results from the point where the previous retrieval ended. A retrieval can also specify an end cursor, to limit the extent of the result set returned.

In the low-level Java API, the application can use cursors via the [`QueryResultList`](https://web.archive.org/web/20160424230618/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/QueryResultList), [`QueryResultIterable`](https://web.archive.org/web/20160424230618/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/QueryResultIterable), and [`QueryResultIterator`](https://web.archive.org/web/20160424230618/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/QueryResultIterator) interfaces, which are returned by the [`PreparedQuery`](https://web.archive.org/web/20160424230618/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/PreparedQuery) methods [`asQueryResultList()`](https://web.archive.org/web/20160424230618/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/PreparedQuery#asQueryResultList(com.google.appengine.api.datastore.FetchOptions)), [`asQueryResultIterable()`](https://web.archive.org/web/20160424230618/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/PreparedQuery#asQueryResultIterable()), and [`asQueryResultIterator()`](https://web.archive.org/web/20160424230618/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/PreparedQuery#asQueryResultIterator()), respectively. Each of these result objects provides a [`getCursor()`](https://web.archive.org/web/20160424230618/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/QueryResultIterator#getCursor()) method, which in turn returns a [`Cursor`](https://web.archive.org/web/20160424230618/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/Cursor) object. The application can get a web-safe string representing the cursor by calling the `Cursor` object's [`toWebSafeString()`](https://web.archive.org/web/20160424230618/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/Cursor#toWebSafeString()) method, and can later use the static method [`Cursor.fromWebSafeString()`](https://web.archive.org/web/20160424230618/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/Cursor#fromWebSafeString(java.lang.String)) to reconstitute the cursor from the string.

The following example demonstrates the use of cursors for pagination:

```
import java.io.IOException;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import com.google.appengine.api.datastore.Cursor;
import com.google.appengine.api.datastore.DatastoreService;
import com.google.appengine.api.datastore.DatastoreServiceFactory;
import com.google.appengine.api.datastore.Entity;
import com.google.appengine.api.datastore.FetchOptions;
import com.google.appengine.api.datastore.PreparedQuery;
import com.google.appengine.api.datastore.Query;
import com.google.appengine.api.datastore.QueryResultList;


public class ListPeopleServlet extends HttpServlet {

  @Override
  protected void doGet(HttpServletRequest req, HttpServletResponse resp)
                   throws ServletException, IOException {

    DatastoreService datastore = DatastoreServiceFactory.getDatastoreService();
    
    resp.setContentType("text/html");
    resp.getWriter().println("<ul>");

    int pageSize = 15;
    FetchOptions fetchOptions = FetchOptions.Builder.withLimit(pageSize);
    String startCursor = req.getParameter("cursor");
    
    // If this servlet is passed a cursor parameter, let's use it
    if (startCursor != null) {
      fetchOptions.startCursor(Cursor.fromWebSafeString(startCursor));
    }
    
    Query q = new Query("Person");
    PreparedQuery pq = datastore.prepare(q);
    
    QueryResultList<Entity> results = pq.asQueryResultList(fetchOptions);
    for (Entity entity : results) {
      resp.getWriter().println("<li>" + entity.getProperty("name") + "</li>");
    }
    resp.getWriter().println("</ul>");
   
    String cursorString = results.getCursor().toWebSafeString();
    
    // Assuming this servlet lives at '/people'
    resp.getWriter().println(
      "<a href='/people?cursor=" + cursorString + "'>Next page</a>");
  }
}
```
**Caution:** Be careful when passing a Datastore cursor to a client, such as in a web form. Although the client cannot change the cursor value to access results outside of the original query, it is possible for it to decode the cursor to expose information about result entities, such as the application ID, entity kind, key name or numeric ID, ancestor keys, and properties used in the query's filters and sort orders. If you don't want users to have access to that information, you can encrypt the cursor, or store it and provide the user with an opaque key.

### Limitations of cursors

Cursors are subject to the following limitations:

-   A cursor can be used only by the same application that performed the original query, and only to continue the same query. To use the cursor in a subsequent retrieval operation, you must reconstitute the original query exactly, including the same entity kind, ancestor filter, property filters, and sort orders. It is not possible to retrieve results using a cursor without setting up the same query from which it was originally generated.
-   Because the `NOT_EQUAL` and `IN` operators are implemented with multiple queries, queries that use them do not support cursors, nor do composite queries constructed with the `CompositeFilterOperator.or` method.
-   Cursors don't always work as expected with a query that uses an inequality filter or a sort order on a property with multiple values. The de-duplication logic for such multiple-valued properties does not persist between retrievals, possibly causing the same result to be returned more than once.
-   New App Engine releases may change internal implementation details, invalidating cursors that depend on them. If an application attempts to use a cursor that is no longer valid, the Datastore raises an [`IllegalArgumentException`](https://web.archive.org/web/20160424230618/http://docs.oracle.com/javase/6/docs/api/java/lang/IllegalArgumentException.html) (low-level API), [`JDOFatalUserException`](//web.archive.org/web/20160424230618/https://db.apache.org/jdo/api20/apidocs/javax/jdo/JDOFatalUserException.html) (JDO), or [`PersistenceException`](https://web.archive.org/web/20160424230618/http://docs.oracle.com/javaee/6/api/javax/persistence/PersistenceException.html) (JPA).

### Cursors and data updates

The cursor's position is defined as the location in the result list after the last result returned. A cursor is not a relative position in the list (it's not an offset); it's a marker to which the Datastore can jump when starting an index scan for results. If the results for a query change between uses of a cursor, the query notices only changes that occur in results after the cursor. If a new result appears before the cursor's position for the query, it will not be returned when the results after the cursor are fetched. Similarly, if an entity is no longer a result for a query but had appeared before the cursor, the results that appear after the cursor do not change. If the last result returned is removed from the result set, the cursor still knows how to locate the next result.

When retrieving query results, you can use both a start cursor and an end cursor to return a continuous group of results from the Datastore. When using a start and end cursor to retrieve the results, you are not guaranteed that the size of the results will be the same as when you generated the cursors. Entities may be added or deleted from the Datastore between the time the cursors are generated and when they are used in a query.

## Data consistency

Datastore queries can deliver their results at either of two consistency levels:

-   [*Strongly consistent*](https://web.archive.org/web/20160424230618/https://en.wikipedia.org/wiki/Strong_consistency) queries guarantee the freshest results, but may take longer to complete.
-   [*Eventually consistent*](https://web.archive.org/web/20160424230618/https://en.wikipedia.org/wiki/Eventual_consistency) queries generally run faster, but may occasionally return stale results.

In an eventually consistent query, the indexes used to gather the results are also accessed with eventual consistency. Consequently, such queries may sometimes return entities that no longer match the original query criteria, while strongly consistent queries are always transactionally consistent. See the article [Transaction Isolation in App Engine](https://web.archive.org/web/20160424230618/https://cloud.google.com/appengine/articles/transaction_isolation) for more information on how entities and indexes are updated.

Queries return their results with different levels of consistency guarantee, depending on the nature of the query:

-   [Ancestor queries](#Java_Ancestor_queries) (those within an [entity group](https://web.archive.org/web/20160424230618/https://cloud.google.com/appengine/docs/java/datastore/entities#Java_Ancestor_paths)) are strongly consistent by default, but can instead be made eventually consistent by setting the Datastore read policy (see below).
-   Non-ancestor queries are always eventually consistent.

To improve performance, you can set the Datastore *read policy* so that all reads and queries are eventually consistent. (The API also allows you to explicitly set a strong consistency policy, but this setting will have no practical effect, since non-ancestor queries are always eventually consistent regardless of policy.)

You can also set the Datastore *call deadline:* the maximum time, in seconds, that the application will wait for the Datastore to return a result before aborting with an error. The default deadline is 60 seconds; it is not currently possible to set it higher, but you can adjust it downward to ensure that a particular operation fails quickly (for instance, to return a faster response to the user).

To set the Datastore read policy in Java, you construct a *Datastore service configuration* ([`DatastoreServiceConfig`](https://web.archive.org/web/20160424230618/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/DatastoreServiceConfig)), using the nested helper class [`DatastoreServiceConfig.Builder`](https://web.archive.org/web/20160424230618/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/DatastoreServiceConfig.Builder), and pass it an instance of class [`ReadPolicy`](https://web.archive.org/web/20160424230618/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/ReadPolicy). The following example shows how to set the read policy, the call deadline, or both:

```
import com.google.appengine.api.datastore.DatastoreService;
import com.google.appengine.api.datastore.DatastoreServiceConfig;
import com.google.appengine.api.datastore.DatastoreServiceFactory;
import com.google.appengine.api.datastore.ReadPolicy.Consistency;


double deadline = 5.0;

// Construct a read policy for eventual consistency
ReadPolicy policy = new ReadPolicy(ReadPolicy.Consistency.EVENTUAL);
    
// Set the read policy
DatastoreServiceConfig eventuallyConsistentConfig =
                         DatastoreServiceConfig.Builder.withReadPolicy(policy);
    
// Set the call deadline
DatastoreServiceConfig deadlineConfig =
                         DatastoreServiceConfig.Builder.withDeadline(deadline);
    
// Set both the read policy and the call deadline
DatastoreServiceConfig datastoreConfig =
      DatastoreServiceConfig.Builder.withReadPolicy(policy).deadline(deadline);
    
// Get Datastore service with the given configuration
DatastoreService datastore =
                   DatastoreServiceFactory.getDatastoreService(datastoreConfig);
```
