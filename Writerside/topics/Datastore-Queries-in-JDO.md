# Datastore Queries in JDO

  

This document focuses on the use of the [Java Data Objects (JDO)](https://web.archive.org/web/20160424230825/http://db.apache.org/jdo/index) persistence framework for App Engine Datastore queries. For more general information about queries, see the main [Datastore Queries](https://web.archive.org/web/20160424230825/https://cloud.google.com/appengine/docs/java/datastore/queries) page.

A Datastore *query* retrieves *[entities](https://web.archive.org/web/20160424230825/https://cloud.google.com/appengine/docs/java/datastore/entities)* from the App Engine Datastore that meet a specified set of conditions. The query operates on entities of a given *[kind](https://web.archive.org/web/20160424230825/https://cloud.google.com/appengine/docs/java/datastore/entities#Java_Kinds_and_identifiers)*; it can specify *[filters](#Filters)* on the entities' property values, keys, and ancestors, and can return zero or more entities as *results.* A query can also specify *[sort orders](#Sort_Orders)* to sequence the results by their property values. The results include all entities that have at least one (possibly null) value for every property named in the filters and sort orders, and whose property values meet all the specified filter criteria. The query can return entire entities, [projected entities](https://web.archive.org/web/20160424230825/https://cloud.google.com/appengine/docs/java/datastore/projectionqueries), or just entity [keys](https://web.archive.org/web/20160424230825/https://cloud.google.com/appengine/docs/java/datastore/#Java_Kinds_keys_and_identifiers).

A typical query includes the following:

-   An *[entity kind](https://web.archive.org/web/20160424230825/https://cloud.google.com/appengine/docs/java/datastore/entities#Java_Kinds_and_identifiers)* to which the query applies
-   Zero or more *[filters](#Filters)* based on the entities' property values, keys, and ancestors
-   Zero or more *[sort orders](#Sort_Orders)* to sequence the results

When executed, the query retrieves all entities of the given kind that satisfy all of the given filters, sorted in the specified order. Queries execute as read-only.

**Note:** To conserve memory and improve performance, a query should, whenever possible, specify a limit on the number of results returned.

Every Datastore query computes its results using one or more *[indexes](https://web.archive.org/web/20160424230825/https://cloud.google.com/appengine/docs/java/datastore/indexes),* which contain entity keys in a sequence specified by the index's properties and, optionally, the entity's ancestors. The indexes are updated incrementally to reflect any changes the application makes to its entities, so that the correct results of all queries are available with no further computation needed.

**Note:** The index-based query mechanism supports a wide range of queries and is suitable for most applications. However, it does not support some kinds of query common in other database technologies: in particular, joins and aggregate queries aren't supported within the Datastore query engine. See the [Datastore Queries](https://web.archive.org/web/20160424230825/https://cloud.google.com/appengine/docs/java/datastore/queries#Java_Restrictions_on_queries) page for limitations on Datastore queries.

## Contents

1.  [Queries with JDOQL](#Queries_with_JDOQL)
    1.  [Filters](#Filters)
    2.  [Sort Orders](#Sort_Orders)
    3.  [Ranges](#Ranges)
2.  [Key-Based Queries](#Key_Based_Queries)
3.  [Extents](#Extents)
4.  [Deleting Entities by Query](#Deleting_Entities_by_Query)
5.  [Query Cursors](#Query_Cursors)
6.  [Datastore Read Policy and Call Deadline](#Datastore_Read_Policy_and_Call_Deadline)

## Queries with JDOQL

JDO includes a query language for retrieving objects that meet a set of criteria. This language, called [JDOQL](https://web.archive.org/web/20160424230825/http://db.apache.org/jdo/jdoql), refers to JDO data classes and fields directly, and includes type checking for query parameters and results. JDOQL is similar to SQL, but is more appropriate for object-oriented databases like the App Engine Datastore. (App Engine's implementation of the JDO API doesn't support SQL queries directly.)

The JDO [`Query`](https://web.archive.org/web/20160424230825/http://db.apache.org/jdo/api30/apidocs/javax/jdo/Query) interface supports several calling styles: you can specify a complete query in a string, using the JDOQL string syntax, or you can specify some or all parts of the query by calling methods on the `Query` object. The following example shows the method style of calling, with one filter and one sort order, using parameter substitution for the value used in the filter. The argument values passed to the `Query` object's [`execute()`](https://web.archive.org/web/20160424230825/http://db.apache.org/jdo/api30/apidocs/javax/jdo/Query#execute()) method are substituted into the query in the order specified:

```
import java.util.List;
import javax.jdo.Query;

// ...

Query q = pm.newQuery(Person.class);
q.setFilter("lastName == lastNameParam");
q.setOrdering("height desc");
q.declareParameters("String lastNameParam");

try {
  List<Person> results = (List<Person>) q.execute("Smith");
  if (!results.isEmpty()) {
    for (Person p : results) {
      // Process result p
    }
  } else {
    // Handle "no results" case
  }
} finally {
  q.closeAll();
}
```

Here is the same query using the string syntax:

```
Query q = pm.newQuery("select from Person " +
                      "where lastName == lastNameParam " +
                      "parameters String lastNameParam " +
                      "order by height desc");

List<Person> results = (List<Person>) q.execute("Smith");
```

You can mix these styles of defining the query. For example:

```
Query q = pm.newQuery(Person.class,
                      "lastName == lastNameParam order by height desc");
q.declareParameters("String lastNameParam");

List<Person> results = (List<Person>) q.execute("Smith");
```

You can reuse a single `Query` instance with different values substituted for the parameters by calling the `execute()` method multiple times. Each call performs the query and returns the results as a collection.

The JDOQL string syntax supports literal specification of string and numeric values; all other value types must use parameter substitution. Literals within the query string can be enclosed in either single (`'`) or double (`"`) quotation marks. Here is an example using a string literal:

```
Query q = pm.newQuery(Person.class,
                      "lastName == 'Smith' order by height desc");
```

### Filters

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
```
Query q = pm.newQuery(Person.class);
q.setFilter("height <= maxHeight");
```

The property value must be supplied by the application; it cannot refer to or be calculated in terms of other properties. An entity satisfies the filter if it has a property of the given name whose value compares to the value specified in the filter in the manner described by the comparison operator.

The comparison operator can be any of the following:

<table>
<thead>
<tr class="header">
<th>Operator</th>
<th>Meaning</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>==</code></td>
<td>Equal to</td>
</tr>
<tr class="even">
<td><code>&lt;</code></td>
<td>Less than</td>
</tr>
<tr class="odd">
<td><code>&lt;=</code></td>
<td>Less than or equal to</td>
</tr>
<tr class="even">
<td><code>&gt;</code></td>
<td>Greater than</td>
</tr>
<tr class="odd">
<td><code>&gt;=</code></td>
<td>Greater than or equal to</td>
</tr>
<tr class="even">
<td><code>!=</code></td>
<td>Not equal to</td>
</tr>
</tbody>
</table>

As described on the main [Queries](https://web.archive.org/web/20160424230825/https://cloud.google.com/appengine/docs/java/datastore/queries#Restrictions_on_Queries) page, a single query cannot use inequality filters (`<`, `<=`, `>`, `>=`, `!=`) on more than one property. (Multiple inequality filters on the same property, such as querying for a range of values, are permitted.) `contains()` filters, corresponding to `IN` filters in SQL, are supported using the following syntax:

```
// Query for all persons with lastName equal to Smith or Jones
Query q = pm.newQuery(Person.class, ":p.contains(lastName)");
q.execute(Arrays.asList("Smith", "Jones"));
```

The not-equal (`!=`) operator actually performs two queries: one in which all other filters are unchanged and the not-equal filter is replaced with a less-than (`<`) filter, and one where it is replaced with a greater-than (`>`) filter. The results are then merged, in order. A query can have no more than one not-equal filter, and a query that has one cannot have any other inequality filters.

The `contains()` operator also performs multiple queries: one for each item in the specified list, with all other filters unchanged and the `contains()` filter replaced with an equality (`==`) filter. The results are merged in order of the items in the list. If a query has more than one `contains()` filter, it is performed as multiple queries, one for each possible *combination* of values in the `contains()` lists.

A single query containing not-equal (`!=`) or `contains()` operators is limited to no more than 30 subqueries.

For more information about how `!=` and `contains()` queries translate to multiple queries in a JDO/JPA framework, see the article [Queries with != and IN filters](https://web.archive.org/web/20160424230825/http://gae-java-persistence.blogspot.com/2009/12/queries-with-and-in-filters.html).

In the JDOQL string syntax, you can separate multiple filters with the `&&` (logical "and") and `||` (logical "or") operators:

```
q.setFilter("lastName == 'Smith' && height < maxHeight");
```

Negation (logical "not") is not supported. Keep in mind, also, that the `||` operator can be employed only when the filters it separates all have the same property name (that is, when they can be combined into a single `contains()` filter):

```
// Legal: all filters separated by || are on the same property
Query q = pm.newQuery(Person.class,
                      "(lastName == 'Smith' || lastName == 'Jones')" +
                      " && firstName == 'Harold'");

// Not legal: filters separated by || are on different properties
Query q = pm.newQuery(Person.class,
                      "lastName == 'Smith' || firstName == 'Harold'");
```

### Sort Orders

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

For example:

```
// Order alphabetically by last name:
Query q = pm.newQuery(Person.class);
q.setOrdering("lastName asc");

// Order by height, tallest to shortest:
Query q = pm.newQuery(Person.class);
q.setOrdering("height desc");
```

If a query includes multiple sort orders, they are applied in the sequence specified. The following example sorts first by ascending last name and then by descending height:

```
Query q = new Query("Person")
                .addSort("lastName", SortDirection.ASCENDING)
                .addSort("height", SortDirection.DESCENDING);
```
```
Query q = pm.newQuery(Person.class);
q.setOrdering("lastName asc, height desc");
```

If no sort orders are specified, the results are returned in the order they are retrieved from the Datastore.

**Note:** Because of the way the App Engine Datastore executes queries, if a query specifies inequality filters on a property and sort orders on other properties, the property used in the inequality filters must be ordered before the other properties.

### Ranges

A query can specify a *range* of results to be returned to the application. The range indicates which results in the complete result set should be the first and last returned. Results are identified by their numeric indices, with `0` denoting the first result in the set. For example, a range of `5,` `10` returns the 6th through 10th results:

```
q.setRange(5, 10);
```

**Note:** The use of ranges can affect performance, since the Datastore must retrieve and then discard all results preceding the starting offset. For example, a query with a range of `5,` `10` retrieves ten results from the Datastore, discards the first five, and returns the remaining five to the application.

## Key-Based Queries

Entity keys can be the subject of a query filter or sort order. The Datastore considers the complete key value for such queries, including the entity's ancestor path, the kind, and the application-assigned key name string or system-assigned numeric ID. Because the key is unique across all entities in the system, key queries make it easy to retrieve entities of a given kind in batches, such as for a batch dump of the contents of the Datastore. Unlike JDOQL ranges, this works efficiently for any number of entities.

When comparing for inequality, keys are ordered by the following criteria, in order:

1.  Ancestor path
2.  Entity kind
3.  Identifier (key name or numeric ID)

Elements of the ancestor path are compared similarly: by kind (string), then by key name or numeric ID. Kinds and key names are strings and are ordered by byte value; numeric IDs are integers and are ordered numerically. If entities with the same parent and kind use a mix of key name strings and numeric IDs, those with numeric IDs precede those with key names.

In JDO, you refer to the entity key in the query using the primary key field of the object. To use a key as a query filter, you specify the parameter type `Key` to the ` `[`declareParameters()`](https://web.archive.org/web/20160424230825/http://db.apache.org/jdo/api30/apidocs/javax/jdo/Query#declareParameters(java.lang.String)) method. The following finds all `Person` entities with a given favorite food, assuming an [unowned one-to-one relationship](https://web.archive.org/web/20160424230825/https://cloud.google.com/appengine/docs/java/datastore/jdo/relationships#Unowned_Relationships) between `Person` and `Food`:

```
Food chocolate = /*...*/;

Query q = pm.newQuery(Person.class);
q.setFilter("favoriteFood == favoriteFoodParam");
q.declareParameters(Key.class.getName() + " favoriteFoodParam");

List<Person> chocolateLovers = (List<Person>) q.execute(chocolate.getKey());
```

A *keys-only query* returns just the keys of the result entities instead of the entities themselves, at lower latency and cost than retrieving entire entities:

```
Query q = new Query("Person")
                .setKeysOnly();
```
```
Query q = pm.newQuery("select id from " + Person.class.getName());
List<String> ids = (List<String>) q.execute();
```

It is often more economical to do a keys-only query first, and then fetch a subset of entities from the results, rather than executing a general query which may fetch more entities than you actually need.

## Extents

A JDO *extent* represents every object in the Datastore of a particular class. You create it by passing the desired class to the Persistence Manager's ` `[`getExtent()`](https://web.archive.org/web/20160424230825/http://db.apache.org/jdo/api30/apidocs/javax/jdo/PersistenceManager#getExtent(java.lang.Class)) method. The [`Extent`](https://web.archive.org/web/20160424230825/http://db.apache.org/jdo/api30/apidocs/javax/jdo/Extent) interface extends the ` `[`Iterable`](//web.archive.org/web/20160424230825/https://docs.oracle.com/javase/6/docs/api/java/lang/Iterable.html) interface for accessing results, retrieving them in batches as needed. When you're finished accessing the results, you call the extent's [`closeAll()`](https://web.archive.org/web/20160424230825/http://db.apache.org/jdo/api30/apidocs/javax/jdo/Extent#closeAll()) method.

The following example iterates over every `Person` object in the Datastore:

```
import java.util.Iterator;
import javax.jdo.Extent;

// ...

Extent<Person> extent = pm.getExtent(Person.class, false);
for (Person p : extent) {
  // ...
}
extent.closeAll();
```

## Deleting Entities by Query

If you are issuing a query with the goal of deleting all the entities that match the query filter, you can save yourself a bit of coding by using JDO's "delete by query" feature. The following deletes all persons over a given height:

```
Query q = pm.newQuery(Person.class);
q.setFilter("height > maxHeightParam");
q.declareParameters("int maxHeightParam");
q.deletePersistentAll(maxHeight);
```

You'll notice that the only difference here is that we are calling ` `[`q.deletePersistentAll()`](https://web.archive.org/web/20160424230825/http://db.apache.org/jdo/api30/apidocs/javax/jdo/Query#deletePersistentAll()) instead of [`q.execute()`](https://web.archive.org/web/20160424230825/http://db.apache.org/jdo/api30/apidocs/javax/jdo/Query#execute()). All of the rules and restrictions described above for filters, sort orders, and indexes apply to queries whether you are selecting or deleting the result set. Note, however, that just as if you had deleted these `Person` entities with ` `[`pm.deletePersistent()`](https://web.archive.org/web/20160424230825/http://db.apache.org/jdo/api30/apidocs/javax/jdo/PersistenceManager#deletePersistent(java.lang.Object)), any dependent children of the entities deleted by the query will also be deleted. For more information on dependent children, see the [Entity Relationships in JDO](https://web.archive.org/web/20160424230825/https://cloud.google.com/appengine/docs/java/datastore/jdo/relationships#Dependent_Children_and_Cascading_Deletes) page.

## Query Cursors

In JDO, you can use an extension and the `JDOCursorHelper` class to use cursors with JDO queries. Cursors work when fetching results as a list or using an iterator. To get a cursor, you pass either the result list or iterator to the static method `JDOCursorHelper.getCursor()`:

```
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import javax.jdo.Query;
import com.google.appengine.api.datastore.Cursor;
import org.datanucleus.store.appengine.query.JDOCursorHelper;


Query q = pm.newQuery(Person.class);
q.setRange(0, 20);

List<Person> results = (List<Person>) q.execute();
// Use the first 20 results

Cursor cursor = JDOCursorHelper.getCursor(results);
String cursorString = cursor.toWebSafeString();
// Store the cursorString

// ...

// Query q = the same query that produced the cursor
// String cursorString = the string from storage
Cursor cursor = Cursor.fromWebSafeString(cursorString);
Map<String, Object> extensionMap = new HashMap<String, Object>();
extensionMap.put(JDOCursorHelper.CURSOR_EXTENSION, cursor);
q.setExtensions(extensionMap);
q.setRange(0, 20);

List<Person> results = (List<Person>) q.execute();
// Use the next 20 results
```

For more information on query cursors, see the [Datastore Queries](https://web.archive.org/web/20160424230825/https://cloud.google.com/appengine/docs/java/datastore/queries#Query_Cursors) page.

## Datastore Read Policy and Call Deadline

You can set the read policy (strong consistency vs. eventual consistency) and the Datastore call deadline for all calls made by a `PersistenceManager` instance using configuration. You can also override these options for an individual `Query` object. (Note, however, that there is no way to override the configuration for these options when you fetch entities by key.)

When eventual consistency is selected for a Datastore query, the indexes the query uses to gather results are also accessed with eventual consistency. Queries occasionally return entities that don't match the query criteria—though this is also true with a strongly consistent read policy. (If the query uses an ancestor filter, you can use [transactions](https://web.archive.org/web/20160424230825/https://cloud.google.com/appengine/docs/java/datastore/transactions) to ensure a consistent result set.) See the article [Transaction Isolation in App Engine](https://web.archive.org/web/20160424230825/https://cloud.google.com/appengine/articles/transaction_isolation) for more information on how entities and indexes are updated.

To override the read policy for a single query, call its ` `[`addExtension()`](https://web.archive.org/web/20160424230825/http://db.apache.org/jdo/api30/apidocs/javax/jdo/Query#addExtension(java.lang.String,%20java.lang.Object)) method:

```
Query q = pm.newQuery(Person.class);
q.addExtension("datanucleus.appengine.datastoreReadConsistency", "EVENTUAL");
```

The possible values are `"EVENTUAL"` and `"STRONG"`; the default is `"STRONG"`, unless set otherwise in the configuration file `jdoconfig.xml`.

To override the Datastore call deadline for a single query, call its ` `[`setDatastoreReadTimeoutMillis()`](https://web.archive.org/web/20160424230825/http://db.apache.org/jdo/api30/apidocs/javax/jdo/Query#setDatastoreReadTimeoutMillis(java.lang.Integer)) method:

```
q.setDatastoreReadTimeoutMillis(3000);
```

The value is a time interval expressed in milliseconds.
