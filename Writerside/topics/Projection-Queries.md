# Projection Queries

  

Most Datastore [queries](https://web.archive.org/web/20160424225927/https://cloud.google.com/appengine/docs/java/datastore/queries) return whole [entities](https://web.archive.org/web/20160424225927/https://cloud.google.com/appengine/docs/java/datastore/entities) as their results, but often an application is actually interested in only a few of the entity's properties. *Projection queries* allow you to query the Datastore for just those specific properties of an entity that you actually need, at lower latency and cost than retrieving the entire entity.

Projection queries are similar to SQL queries of the form

```
SELECT name, email, phone FROM CUSTOMER
```

You can use all of the filtering and sorting features available for standard entity queries, subject to the [limitations](#Java_Limitations_on_projections) described below. The query returns abridged results with only the specified properties (`name`, `email`, and `phone` in the example) populated with values; all other properties have no data.

## Contents

1.  [Using projection queries in Java](#Java_Using_projection_queries_in_Java)
2.  [Grouping](#Java_Grouping)
3.  [Limitations on projections](#Java_Limitations_on_projections)
4.  [Projections and multiple-valued properties](#Java_Projections_and_multiple_valued_properties)
5.  [Indexes for projections](#Java_Indexes_for_projections)

## Using projection queries in Java

Projection queries require the following imports:

```
import com.google.appengine.api.datastore.PropertyProjection;
import com.google.appengine.api.datastore.RawValue;
```

To build a projection query, you create a [`Query`](https://web.archive.org/web/20160424225927/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/Query) object and add properties to it using the [`addProjection()`](https://web.archive.org/web/20160424225927/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/Query#addProjection(com.google.appengine.api.datastore.Projection)) method:

```
Key guestbookKey = KeyFactory.createKey("Guestbook", guestbookName);

Query proj = new Query("Greeting", guestbookKey);
proj.addProjection(new PropertyProjection("user", User.class));
proj.addProjection(new PropertyProjection("date", Date.class));
```

The type you specify for each property must match the type you used when you first defined the property with [`Entity.setProperty()`](https://web.archive.org/web/20160424225927/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/Entity#setProperty(java.lang.String,%20java.lang.Object)). The following example shows how to process the query's results by iterating through the list of entities returned and casting each property value to the expected type:

```
DatastoreService datastore = DatastoreServiceFactory.getDatastoreService();
List<Entity> projTests = datastore.prepare(proj)
                                  .asList(FetchOptions.Builder.withLimit(5));

for (Entity projtest : projTests) {
  User person = (User) projtest.getProperty("user");
  System.out.println(person.toString());

  Date stamp = (Date) projtest.getProperty("date");
  System.out.println(stamp.toString());
}
```
**Note:** Instead of specifying a property's type explicitly to the `Query.addProjection()` method, you can simply pass `null` for the type. The property's value will then be returned as a [`RawValue`](https://web.archive.org/web/20160424225927/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/RawValue) object, which you can test to determine its type. (In the processing example above, you would cast the property to `RawValue` instead of a specific type.)

## Grouping <sup>(experimental)</sup>

Projection queries can use the `setDistinct()` method to ensure that only completely unique results will be returned in a result set. This will only return the first result for entities which have the same values for the properties that are being projected.

```
Query q = new Query();
q.addProjection(new PropertyProjection("A", String.class));
q.addProjection(new PropertyProjection("B", String.class));
q.setDistinct(true);
q.setFilter(Query.FilterOperator.LESS_THAN.of("B", 1));
q.addSort("B", Query.SortDirection.DESCENDING);
q.addSort("A");
```

## Limitations on projections

Projection queries are subject to the following limitations:

-   **Only indexed properties can be projected.**

    Projection is not supported for properties that are not indexed, whether explicitly or implicitly. (Long text strings ([`Text`](https://web.archive.org/web/20160424225927/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/Text)) and long byte strings ([`Blob`](https://web.archive.org/web/20160424225927/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/Blob)) are not indexed).

-   **The same property cannot be projected more than once.**

-   **Properties referenced in an equality (`EQUAL`) or membership (`IN`) filter cannot be projected.**

    For example,

    ```
    SELECT A FROM kind WHERE B = 1
    ```

    is valid (projected property not used in the equality filter), as is

    ```
    SELECT A FROM kind WHERE A > 1
    ```

    (not an equality filter), but

    ```
    SELECT A FROM kind WHERE A = 1
    ```

    (projected property used in equality filter) is not.

-   **Results returned by a projection query should not be saved back to the Datastore.**

    Because the query returns results that are only partially populated, you should not write them back to the Datastore.

## Projections and multiple-valued properties

Projecting a property with multiple values will not populate all values for that property. Instead, a separate entity will be returned for each unique combination of projected values matching the query. For example, suppose you have an entity of kind `Foo` with two multiple-valued properties, `A` and `B`:

```
entity = Foo(A=[1, 1, 2, 3], B=['x', 'y', 'x'])
```

Then the projection query

```
SELECT A, B FROM Foo WHERE A < 3
```

will return four entities with the following combinations of values:

`A` = `1`, `B` = `'x'`  
`A` = `1`, `B` = `'y'`  
`A` = `2`, `B` = `'x'`  
`A` = `2`, `B` = `'y'`

**Warning:** Including more than one multiple-valued property in a projection will result in an [exploding index](https://web.archive.org/web/20160424225927/https://cloud.google.com/appengine/docs/java/datastore/indexes#Java_Index_limits).

## Indexes for projections

Projection queries require all properties specified in the projection to be included in a Datastore [index](https://web.archive.org/web/20160424225927/https://cloud.google.com/appengine/docs/java/datastore/indexes). The App Engine development server automatically generates the needed indexes for you in the index configuration file, `datastore-indexes-auto.xml`, which is uploaded with your application.

One way to minimize the number of indexes required is to project the same properties consistently, even when not all of them are always needed. For example, these queries require two separate indexes:

```
SELECT A, B FROM Kind
SELECT A, B, C FROM Kind
```

However, if you always project properties `A`, `B`, and `C`, even when `C` is not required, only one index will be needed.

Converting an existing query into a projection query may require building a new index if the properties in the projection are not already included in another part of the query. For example, suppose you had an existing query like

```
SELECT * FROM Kind WHERE A > 1 ORDER BY A, B
```

which requires the index

```
Index(Kind, A, B)
```

Converting this to either of the projection queries

```
SELECT C FROM Kind WHERE A > 1 ORDER BY A, B
SELECT A, B, C FROM Kind WHERE A > 1 ORDER BY A, B
```

introduces a new property (`C`) and thus will require building a new index `Index(Kind,` `A,` `B,` `C)`. Note that the projection query

```
SELECT A, B FROM Kind WHERE A > 1 ORDER BY A, B
```

would *not* change the required index, since the projected properties `A` and `B` were already included in the existing query.
