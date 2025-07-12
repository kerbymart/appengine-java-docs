# Datastore Indexes

  

Every Datastore [query](https://web.archive.org/web/20160424230615/https://cloud.google.com/appengine/docs/java/datastore/queries) computes its results using one or more *indexes,* which contain entity keys in a sequence specified by the index's properties and, optionally, the entity's ancestors. The indexes are updated incrementally to reflect any changes the application makes to its entities, so that the correct results of all queries are available with no further computation needed.

**Important:** For an in-depth discussion of indexes and querying, see the article [Index Selection and Advanced Search](https://web.archive.org/web/20160424230615/https://cloud.google.com/appengine/articles/indexselection).

App Engine predefines a simple index on each property of an entity. An App Engine application can define further custom indexes in an *[index configuration file](https://web.archive.org/web/20160424230615/https://cloud.google.com/appengine/docs/java/config/indexconfig)* named `datastore-indexes.xml`, which is generated in your application's `/war/WEB-INF/appengine-generated` directory. The development server automatically adds suggestions to this file as it encounters queries that cannot be executed with the existing indexes. You can tune indexes manually by editing the file before uploading the application.

**Note:** The index-based query mechanism supports a wide range of queries and is suitable for most applications. However, it does not support some kinds of query common in other database technologies: in particular, joins and aggregate queries aren't supported within the Datastore query engine. See the [Datastore Queries](https://web.archive.org/web/20160424230615/https://cloud.google.com/appengine/docs/java/datastore/queries#Java_Restrictions_on_queries) page for limitations on Datastore queries.

## Contents

1.  [Index definition and structure](#Java_Index_definition_and_structure)
2.  [Index configuration](#Java_Index_configuration)
3.  [Indexes and properties](#Java_Indexes_and_properties)
    1.  [Properties with mixed value types](#Java_Properties_with_mixed_value_types)
    2.  [Unindexed properties](#Java_Unindexed_properties)
4.  [Index limits](#Java_Index_limits)

## Index definition and structure

An index is defined on a list of properties of a given [entity kind](https://web.archive.org/web/20160424230615/https://cloud.google.com/appengine/docs/java/datastore/entities#Java_Kinds_and_identifiers), with a corresponding order (ascending or descending) for each property. For use with ancestor queries, the index may also optionally include an entity's ancestors.

An index table contains a column for every property named in the index's definition. Each row of the table represents an entity in the Datastore that is a potential result for queries based on the index. An entity is included in the index only if it has an indexed value set for every property used in the index; if the index definition refers to a property for which the entity has no value, that entity will not appear in the index and hence will never be returned as a result for any query based on the index.

**Note:** The Datastore distinguishes between an entity that does not possess a property and one that possesses the property with a null value (`null`). If you explicitly assign a null value to an entity's property, that entity may be included in the results of a query referring to that property.

**Note:** Indexes composed of multiple properties require that each individual property must not be set to [unindexed](#Java_Unindexed_properties).

The rows of an index table are sorted first by ancestor and then by property values, in the order specified in the index definition. The *perfect index* for a query, which allows the query to be executed most efficiently, is defined on the following properties, in order:

1.  Properties used in equality filters
2.  Property used in an inequality filter (of which there can be [no more than one](https://web.archive.org/web/20160424230615/https://cloud.google.com/appengine/docs/java/datastore/queries#Java_Inequality_filters_are_limited_to_at_most_one_property))
3.  Properties used in sort orders

This ensures that all results for every possible execution of the query appear in consecutive rows of the table. The Datastore executes a query using a perfect index by the following steps:

1.  Identifies the index corresponding to the query's kind, filter properties, filter operators, and sort orders.
2.  Scans from the beginning of the index to the first entity that meets all of the query's filter conditions.
3.  Continues scanning the index, returning each entity in turn, until it
    -   encounters an entity that does not meet the filter conditions, or
    -   reaches the end of the index, or
    -   has collected the maximum number of results requested by the query.

For example, consider the following query:

```
Query q = new Query("Person")
                .addFilter("lastName", Query.FilterOperator.EQUAL, "Smith")
                .addFilter("height", Query.FilterOperator.LESS_THAN, 72)
                .addSort("height", Query.SortDirection.DESCENDING);
```

The perfect index for this query is a table of keys for entities of kind `Person`, with columns for the values of the `lastName` and `height` properties. The index is sorted first in ascending order by `lastName` and then in descending order by `height`.

Two queries of the same form but with different filter values use the same index. For example, the following query uses the same index as the one above:

```
Query q = new Query("Person")
                .addFilter("lastName", Query.FilterOperator.EQUAL, "Jones")
                .addFilter("height", Query.FilterOperator.LESS_THAN, 63)
                .addSort("height", Query.SortDirection.DESCENDING);
```

The following two queries also use the same index, despite their different forms:

```
Query q = new Query("Person")
                .addFilter("lastName", Query.FilterOperator.EQUAL, "Friedkin")
                .addFilter("firstName", Query.FilterOperator.EQUAL, "Damian")
                .addSort("height", Query.SortDirection.ASCENDING);
```

and

```
Query q = new Query("Person")
                .addFilter("lastName", Query.FilterOperator.EQUAL, "Blair")
                .addSort("firstName", Query.SortDirection.ASCENDING)
                .addSort("height", Query.SortDirection.ASCENDING);
```
**Note:** You can get a list of your application's indexes at runtime by calling the methods [`DatastoreService.getIndexes()`](https://web.archive.org/web/20160424230615/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/DatastoreService#getIndexes()) or [`AsyncDatastoreService.getIndexes()`](https://web.archive.org/web/20160424230615/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/AsyncDatastoreService#getIndexes()).

**Caution:** Using a perfect index can significantly increase the cost of writing entities, especially in the case of [exploding indexes](#Java_Index_limits). The article [Index Selection and Advanced Search](https://web.archive.org/web/20160424230615/https://cloud.google.com/appengine/articles/indexselection) describes how you can manually select indexes to avoid these problems.

## Index configuration

By default, the Datastore automatically predefines an index for each property of each entity kind. These predefined indexes are sufficient to perform many simple queries, such as equality-only queries and simple inequality queries. For all other queries, the application must define the indexes it needs in an [*index configuration file*](https://web.archive.org/web/20160424230615/https://cloud.google.com/appengine/docs/java/config/indexconfig) named `datastore-indexes.xml`. If the application tries to perform a query that cannot be executed with the available indexes (either predefined or specified in the index configuration file), the query will fail with a [`DatastoreNeedIndexException`](https://web.archive.org/web/20160424230615/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/DatastoreNeedIndexException).

The Datastore builds automatic indexes for queries of the following forms:

-   Kindless queries using only ancestor and key filters
-   Queries using only ancestor and equality filters
-   Queries using only inequality filters (which are [limited to a single property](https://web.archive.org/web/20160424230615/https://cloud.google.com/appengine/docs/java/datastore/queries#Java_Inequality_filters_are_limited_to_at_most_one_property))
-   Queries using only ancestor filters, equality filters on properties, and inequality filters on keys
-   Queries with no filters and only one sort order on a property, either ascending or descending

Other forms of query require their indexes to be specified in the index configuration file, including:

-   Queries with ancestor and inequality filters
-   Queries with one or more inequality filters on a property and one or more equality filters on other properties
-   Queries with a sort order on keys in descending order
-   Queries with multiple sort orders

**Note:** The App Engine SDK automatically suggests indexes that are appropriate for most applications. Depending on your application's use of the Datastore and the size and shape of your data, manual adjustments to your indexes may be warranted. For example, writing entities with multiple property values may result in an [exploding index](#Java_Index_limits) with high write costs.

**Note:** The development web server can help make it easier to manage your index configuration file. Instead of failing to execute a query that requires an index and does not have one, the development web server can generate an index configuration that would allow the query to succeed. If your local testing of an application exercises every possible query the application will issue, using every combination of filter and sort order, the generated entries will represent a complete set of indexes. If your testing does not exercise every possible query form, you can review and adjust the index configuration file before uploading the application.

## Indexes and properties

Here are a few special considerations to keep in mind about indexes and how they relate to the properties of entities in the Datastore:

### Properties with mixed value types

When two entities have properties of the same name but different value types, an index of the property sorts the entities first by [value type](https://web.archive.org/web/20160424230615/https://cloud.google.com/appengine/docs/java/datastore/entities#Java_Value_type_sort_order) and then by a [secondary ordering](https://web.archive.org/web/20160424230615/https://cloud.google.com/appengine/docs/java/datastore/entities#Java_Value_type_sort_order) appropriate to each type. For example, if two entities each have a property named `age`, one with an integer value and one with a string value, the entity with the integer value always precedes the one with the string value when sorted by the `age` property, regardless of the property values themselves.

This is especially worth noting in the case of integers and floating-point numbers, which are treated as separate types by the Datastore. Because all integers are sorted before all floats, a property with the integer value `38` is sorted before one with the floating-point value `37.5`.

### Unindexed properties

If you know you will never have to filter or sort on a particular property, you can tell the Datastore not to maintain index entries for that property by declaring the property *unindexed*. This lowers the cost of running your application by decreasing the number of Datastore writes it has to perform. An entity with an unindexed property behaves as if the property were not set: queries with a filter or sort order on the unindexed property will never match that entity.

**Note:** If a property appears in an index composed of multiple properties, then setting it to unindexed will prevent it from being indexed in the composed index.  
  
For example, suppose that an entity has properties **a** and **b** and that you want to create an index able to satisfy queries like `WHERE a ="bike" and b="red"`. Also suppose that you don't care about the queries `WHERE a="bike"` and `WHERE b="red"`. If you set **a** to unindexed and create an index for **a** and **b** Datastore will not create index entries for the **a** and **b** index and so the `WHERE a="bike" and b="red"` query won't work. For Datastore to create entries for the **a** and **b** indexes, both **a** and **b** must be indexed.

**Note:** In addition to any unindexed properties you declare explicitly, those typed as long text strings ([`Text`](https://web.archive.org/web/20160424230615/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/Text)), long byte strings ([`Blob`](https://web.archive.org/web/20160424230615/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/Blob)), and embedded entities ([`EmbeddedEntity`](https://web.archive.org/web/20160424230615/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/EmbeddedEntity)) are automatically treated as unindexed.

In the low-level Java Datastore API, properties are defined as indexed or unindexed on a per-entity basis, depending on the method you use to set them ([`setProperty()`](https://web.archive.org/web/20160424230615/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/PropertyContainer#setProperty(java.lang.String,%20java.lang.Object)) or [`setUnindexedProperty()`](https://web.archive.org/web/20160424230615/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/PropertyContainer#setUnindexedProperty(java.lang.String,%20java.lang.Object))):

```
DatastoreService datastore = DatastoreServiceFactory.getDatastoreService();

Key acmeKey = KeyFactory.createKey("Company", "Acme");

Entity tom = new Entity("Person", "Tom", acmeKey);
tom.setProperty("name", "Tom");
tom.setProperty("age", 32);
datastore.put(tom);

Entity lucy = new Entity("Person", "Lucy", acmeKey);
lucy.setProperty("name", "Lucy");
lucy.setUnindexedProperty("age", 29);
datastore.put(lucy);

Filter ageFilter = new FilterPredicate("age", FilterOperator.GREATER_THAN, 25);

Query q = new Query("Person")
                .setAncestor(acmeKey)
                .setFilter(ageFilter);

// Returns tom but not lucy, because her age is unindexed
List<Entity> results = datastore.prepare(q)
                                .asList(FetchOptions.Builder.withDefaults());
```

You can change an indexed property to unindexed by resetting its value with `setUnindexedProperty()`, or from unindexed to indexed by resetting it with `setProperty()`.

Note, however, that changing a property from unindexed to indexed does not affect any existing entities that may have been created before the change. Queries filtering on the property will not return such existing entities, because the entities weren't written to the query's index when they were created. To make the entities accessible by future queries, you must rewrite them to the Datastore so that they will be entered in the appropriate indexes. That is, you must do the following for each such existing entity:

1.  Retrieve (get) the entity from the Datastore.
2.  Write (put) the entity back to the Datastore.

Similarly, changing a property from indexed to unindexed only affects entities subsequently written to the Datastore. The index entries for any existing entities with that property will continue to exist until the entities are updated or deleted. To avoid unwanted results, you must purge your code of all queries that filter or sort by the (now unindexed) property.

## Index limits

The Datastore imposes [limits](https://web.archive.org/web/20160424230615/https://cloud.google.com/appengine/docs/java/datastore/#Java_Quotas_and_limits) on the number and overall size of index entries that can be associated with a single entity. These limits are large, and most applications are not affected. However, there are circumstances in which you might encounter the limits.

As described [above](#Java_Index_configuration), the Datastore creates an entry in a predefined index for every property of every entity except long text strings ([`Text`](https://web.archive.org/web/20160424230615/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/Text)), long byte strings ([`Blob`](https://web.archive.org/web/20160424230615/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/Blob)), and embedded entities ([`EmbeddedEntity`](https://web.archive.org/web/20160424230615/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/EmbeddedEntity)) and those you have explicitly declared as [unindexed](#Java_Unindexed_properties). The property may also be included in additional, custom indexes declared in your [index configuration file](https://web.archive.org/web/20160424230615/https://cloud.google.com/appengine/docs/java/config/indexconfig) (`datastore-indexes.xml`). Provided that an entity has no list properties, it will have at most one entry in each such custom index (for non-ancestor indexes) or one for each of the entity's ancestors (for ancestor indexes). Each of these index entries must be updated every time the value of the property changes.

For a property that has a single value for each entity, each possible value needs to be stored just once per entity in the property's predefined index. Even so, it is possible for an entity with a large number of such single-valued properties to exceed the index entry or size limit. Similarly, an entity that can have multiple values for the same property requires a separate index entry for each value; again, if the number of possible values is large, such an entity can exceed the entry limit.

The situation becomes worse in the case of entities with multiple properties, each of which can take on multiple values. To accommodate such an entity, the index must include an entry for every possible *combination* of property values. Custom indexes that refer to multiple properties, each with multiple values, can "explode" combinatorially, requiring large numbers of entries for an entity with only a relatively small number of possible property values. Such *exploding indexes* can dramatically increase the cost of writing an entity to the Datastore, because of the large number of index entries that must be updated, and also can easily cause the entity to exceed the index entry or size limit.

Consider the query

```
Query q = new Query("Widget")
                .addFilter("x", Query.FilterOperator.EQUAL, 1)
                .addFilter("y", Query.FilterOperator.EQUAL, 2)
                .addSort("date", Query.SortDirection.ASCENDING);
```

which causes the SDK to suggest the following index:

```
<?xml version="1.0" encoding="utf-8"?>
<datastore-indexes>
  <datastore-index kind="Widget">
    <property name="x" direction="asc" />
    <property name="y" direction="asc" />
    <property name="date" direction="asc" />
  </datastore-index>
</datastore-indexes>
```
This index will require a total of `|x|` `*` `|y|` `*` `|date|` entries for each entity (where `|x|` denotes the number of values associated with the entity for property `x`). For example, the following code
```
Entity widget = new Entity("Widget");
widget.setProperty("x", Arrays.asList(1, 2, 3, 4));
widget.setProperty("y", Arrays.asList("red", "green", "blue"));
widget.setProperty("date", new Date());
ds.put(widget);
```

creates an entity with four values for property `x`, three values for property `y`, and `date` set to the current date. This will require 12 index entries, one for each possible combination of property values:

(`1`, `"red"`, `<now>`) (`1`, `"green"`, `<now>`) (`1`, `"blue"`, `<now>`)

(`2`, `"red"`, `<now>`) (`2`, `"green"`, `<now>`) (`2`, `"blue"`, `<now>`)

(`3`, `"red"`, `<now>`) (`3`, `"green"`, `<now>`) (`3`, `"blue"`, `<now>`)

(`4`, `"red"`, `<now>`) (`4`, `"green"`, `<now>`) (`4`, `"blue"`, `<now>`)

When the same property is repeated multiple times, the Datastore can detect exploding indexes and suggest an alternative index. However, in all other circumstances (such as the query defined in this example), the Datastore will generate an exploding index. In this case, you can circumvent the exploding index by manually configuring an index in your index configuration file:

```
<?xml version="1.0" encoding="utf-8"?>
<datastore-indexes>
  <datastore-index kind="Widget">
    <property name="x" direction="asc" />
    <property name="date" direction="asc" />
  </datastore-index>
  <datastore-index kind="Widget">
    <property name="y" direction="asc" />
    <property name="date" direction="asc" />
  </datastore-index>
</datastore-indexes>
```
This reduces the number of entries needed to only `(|x|` `*` `|date|` `+` `|y|` `*` `|date|)`, or 7 entries instead of 12:

(`1`, `<now>`) (`2`, `<now>`) (`3`, `<now>`) (`4`, `<now>`)

(`"red"`, `<now>`) (`"green"`, `<now>`) (`"blue"`, `<now>`)

Any put operation that would cause an index to exceed the index entry or size limit will fail with an [`IllegalArgumentException`](https://web.archive.org/web/20160424230615/http://docs.oracle.com/javase/6/docs/api/java/lang/IllegalArgumentException.html). The text of the exception describes which limit was exceeded (`"Too many indexed properties"` or `"Index entries too large"`) and which custom index was the cause. If you create a new index that would exceed the limits for any entity when built, queries against the index will fail and the index will appear in the `Error` state in the App Engine Administration Console. To handle such `Error` indexes,

1.  Remove the index from your index configuration file (`datastore-indexes.xml`).
2.  Run the application configuration command `AppCfg` `vacuum_indexes`.
3.  Either
    -   reformulate the index definition and corresponding queries, or
    -   remove the entities that are causing the index to explode.
4.  Add the index back to `datastore-indexes.xml`.
5.  Run `AppCfg` `update_indexes`.

You can avoid exploding indexes by avoiding queries that would require a custom index using a list property. As described above, this includes queries with multiple sort orders or queries with a mix of equality and inequality filters.

**Important:** For more information on avoiding exploding indexes, see the article [Index Selection and Advanced Search](https://web.archive.org/web/20160424230615/https://cloud.google.com/appengine/articles/indexselection).
