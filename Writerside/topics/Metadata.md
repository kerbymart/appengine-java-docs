# Metadata

  

The App Engine Datastore provides programmatic access to some of its metadata to support metaprogramming, implementing backend administrative functions, simplify consistent caching, and similar purposes; you can use it, for instance, to build a custom datastore viewer for your application. The metadata available includes information about the entity groups, namespaces, entity kinds, and properties your application uses, as well as the [property representations](#Java_Property_queries_non_keys_only_property_representation) for each property.

The [Cloud Datastore Dashboard](https://web.archive.org/web/20160424230438/https://console.cloud.google.com/project/_/datastore/stats) in the Cloud Platform Console also provides some metadata about your application, but the data displayed there differs in some important respects from that returned by these functions.

-   **Freshness.** Reading metadata using the API gets current data, whereas data in the dashboard is updated only once daily.
-   **Contents.** Some metadata in the dashboard is not available via the APIs; the reverse is also true.
-   **Speed.** Metadata gets and queries are [billed in the same way as datastore gets and queries](https://web.archive.org/web/20160424230438/https://cloud.google.com/appengine/docs/pricing#costs-for-datastore-calls). Metadata queries that fetch information on namespaces, kinds, and properties are generally slow to execute. As a rule of thumb, expect a metadata query that returns *N* entities to take about the same time as *N* ordinary queries each returning a single entity. Furthermore, [property representation queries](#Java_Property_queries_non_keys_only_property_representation) (non-keys-only property queries) are slower than [keys-only property queries](#Java_Property_queries_keys_only). Metadata gets of entity group metadata are somewhat faster than getting a regular entity.

## Contents

1.  [Entity group metadata](#Java_Entity_group_metadata)
2.  [Metadata queries](#Java_Metadata_queries)
    -   [Namespace queries](#Java_Namespace_queries)
    -   [Kind queries](#Java_Kind_queries)
    -   [Property queries](#Java_Property_queries)
        -   [Property queries: keys-only](#Java_Property_queries_keys_only)
        -   [Property queries: non-keys-only (property representation)](#Java_Property_queries_non_keys_only_property_representation)

## Entity group metadata

The High-Replication Datastore provides access to the "version" of an entity group, a strictly positive number that is guaranteed to increase on every change to the entity group. Entity group versions can be used, e.g., to easily write code to keep a consistent cache of a complex ancestor query on an entity group.

Entity group versions are obtained by calling `get()` on a special pseudo-entity that contains a strictly positive `__version__` property. The pseudo-entity's key can be created using the [`Entities.createEntityGroupKey()`](https://web.archive.org/web/20160424230438/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/Entities#createEntityGroupKey()) method:

```
import com.google.appengine.api.datastore.DatastoreService;
import com.google.appengine.api.datastore.Entities;
import com.google.appengine.api.datastore.Entity;
import com.google.appengine.api.datastore.EntityNotFoundException;
import com.google.appengine.api.datastore.Key;

import java.io.PrintWriter;

long getEntityGroupVersion(DatastoreService ds, Transaction tx, Key entityKey) {
  try {
    return Entities.getVersionProperty(
        ds.get(tx, Entities.createEntityGroupKey(entityKey)));
  } catch (EntityNotFoundException e) {
    // No entity group information, return a value strictly smaller than any
    // possible version
    return 0;
  }
}

void testEntityGroupVersions(DatastoreService ds, PrintWriter writer) {
  Entity entity1 = new Entity("Simple");
  Key key1 = ds.put(entity1);
  Key entityGroupKey = Entities.createEntityGroupKey(key1);

  // Print entity1's entity group version
  writer.println("version " + getEntityGroupVersion(ds, null, key1));

  // Write to a different entity group
  Entity entity2 = new Entity("Simple");
  ds.put(entity2);

  // Will print the same version, as entity1's entity group has not changed
  writer.println("version " + getEntityGroupVersion(ds, null, key1));

  // Change entity1's entity group by adding a new child entity
  Entity entity3 = new Entity("Simple", entity1.getKey());
  ds.put(entity3);

  // Will print a higher version, as entity1's entity group has changed
  writer.println("version " + getEntityGroupVersion(ds, null, key1));
}
```

This example caches query results (a count of matching results) and uses entity group versions to ensure that it only uses the cached value if current:

```
import com.google.appengine.api.datastore.DatastoreService;
import com.google.appengine.api.datastore.Entities;
import com.google.appengine.api.datastore.FetchOptions;
import com.google.appengine.api.datastore.Key;
import com.google.appengine.api.datastore.PreparedQuery;
import com.google.appengine.api.datastore.Query;
import com.google.appengine.api.datastore.Transaction;
import com.google.appengine.api.memcache.MemcacheService;

import java.io.PrintWriter;
import java.io.Serializable;

// Reuses getEntityGroupVersion method from the previous example.

// A simple class for tracking consistent entity group counts
class EntityGroupCount implements Serializable {
  long version; // Version of the entity group whose count we are tracking
  int count;
  EntityGroupCount(long version, int count) {
    this.version = version;
    this.count = count;
  }
}

// Display count of entities in an entity group, with consistent caching
void showEntityGroupCount(DatastoreService ds, MemcacheService cache, PrintWriter writer,
                          Key entityGroupKey) {
  EntityGroupCount egCount = (EntityGroupCount) cache.get(entityGroupKey);
  if (egCount != null && egCount.version == getEntityGroupVersion(ds, null, entityGroupKey)) {
    // Cached value matched current entity group version, use that
    writer.println(egCount.count + " entities (cached)");
  } else {
    // Need to actually count entities. Using a transaction to get a consistent count
    // and entity group version.
    Transaction tx = ds.beginTransaction();
    PreparedQuery pq = ds.prepare(tx, new Query(entityGroupKey));
    int count = pq.countEntities(FetchOptions.Builder.withLimit(5000));
    cache.put(entityGroupKey,
              new EntityGroupCount(getEntityGroupVersion(ds, tx, entityGroupKey), count));
    tx.rollback();
    writer.println(count + " entities");
  }
}
```

`__entity_group__` entities may not exist for entity groups which have never been written to.

The entity group version is guaranteed to increase on any change to the entity group; it may also (rarely) increase in the absence of user-visible changes.

## Metadata queries

The Java class [`Entities`](https://web.archive.org/web/20160424230438/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/Entities), defined in the [`com.google.appengine.api.datastore`](https://web.archive.org/web/20160424230438/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/package-summary) package, provides three special entity kinds that are reserved for metadata queries. They are denoted by static constants of the `Entities` class:

<table>
<thead>
<tr class="header">
<th>Static constant</th>
<th>Entity kind</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><a href="https://web.archive.org/web/20160424230438/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/Entities#NAMESPACE_METADATA_KIND"><code>Entities.NAMESPACE_METADATA_KIND</code></a></td>
<td><code>__namespace__</code></td>
</tr>
<tr class="even">
<td><a href="https://web.archive.org/web/20160424230438/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/Entities#KIND_METADATA_KIND"><code>Entities.KIND_METADATA_KIND</code></a></td>
<td><code>__kind__</code></td>
</tr>
<tr class="odd">
<td><a href="https://web.archive.org/web/20160424230438/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/Entities#PROPERTY_METADATA_KIND"><code>Entities.PROPERTY_METADATA_KIND</code></a></td>
<td><code>__property__</code></td>
</tr>
</tbody>
</table>

These kinds will not conflict with others of the same names that may already exist in your application. By querying on these special kinds, you can retrieve entities containing the desired metadata.

The entities returned by metadata queries are generated dynamically, based on the current state of the Datastore. While you can create local \`Entity\` objects of kinds \`\_\_namespace\_\_\`, \`\_\_kind\_\_\`, or `__property__`, any attempt to store them in the Datastore will fail with an [`IllegalArgumentException`](https://web.archive.org/web/20160424230438/http://docs.oracle.com/javase/6/docs/api/java/lang/IllegalArgumentException.html) .

The easiest way to issue metadata queries is with the [low-level Datastore API](https://web.archive.org/web/20160424230438/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/package-summary). The following example prints the names of all namespaces in an application:

```
import com.google.appengine.api.datastore.DatastoreService;
import com.google.appengine.api.datastore.Entities;
import com.google.appengine.api.datastore.Entity;
import com.google.appengine.api.datastore.Query;

void printAllNamespaces(DatastoreService ds, PrintWriter writer) {

  Query q = new Query(Entities.NAMESPACE_METADATA_KIND);

  for (Entity e : ds.prepare(q).asIterable()) {
    // A nonzero numeric id denotes the default namespace;
    // see <a href="#Namespace_Queries">Namespace Queries</a>, below
    if (e.getKey().getId() != 0) {
      writer.println("<default>");
    } else {
      writer.println(e.getKey().getName());
    }
  }
```

### Namespace queries

If your application uses the [Namespaces API](https://web.archive.org/web/20160424230438/https://cloud.google.com/appengine/docs/java/multitenancy), you can use a *namespace query* to find all namespaces used in the application's entities. This allows you to perform activities such as administrative functions across multiple namespaces.

Namespace queries return entities of the special kind `__namespace__` whose key name is the name of a namespace. (An exception is the default namespace designated by the empty string `""`: since the empty string is not a valid key name, this namespace is keyed with the numeric ID `1` instead.) Queries of this type support filtering only for ranges over the special pseudoproperty `__key__`, whose value is the entity's key. The results can be sorted by ascending (but not descending) `__key__` value. Because `__namespace__` entities have no properties, both keys-only and non-keys-only queries return the same information.

The following example returns a list of an application's namespaces in the range between two specified names, `start` and `end`:

```
import com.google.appengine.api.datastore.DatastoreService;
import com.google.appengine.api.datastore.Entities;
import com.google.appengine.api.datastore.Entity;
import com.google.appengine.api.datastore.Key;
import com.google.appengine.api.datastore.KeyFactory;
import com.google.appengine.api.datastore.Query;
import com.google.appengine.api.datastore.Query.Filter;
import com.google.appengine.api.datastore.Query.FilterOperator;
import com.google.appengine.api.datastore.Query.CompositeFilter;
import com.google.appengine.api.datastore.Query.CompositeFilterOperator;
import com.google.appengine.api.datastore.Query.FilterPredicate;

import java.util.ArrayList;
import java.util.List;

List<String> getNamespaces(DatastoreService ds, String start, String end) {

  // Start with unrestricted namespace query
  Query q = new Query(Entities.NAMESPACE_METADATA_KIND);
  List<Filter> subFilters = new ArrayList();
  // Limit to specified range, if any
  if (start != null) {
    subFilters.add(new FilterPredicate(Entity.KEY_RESERVED_PROPERTY,
                Query.FilterOperator.GREATER_THAN_OR_EQUAL,
                Entities.createNamespaceKey(start)));
  }
  if (end != null) {
    subFilters.add(new FilterPredicate(Entity.KEY_RESERVED_PROPERTY,
                Query.FilterOperator.LESS_THAN_OR_EQUAL,
                Entities.createNamespaceKey(end)));
  }

  q.setFilter(CompositeFilterOperator.and(subFilters));

  // Initialize result list
  List<String> results = new ArrayList<String>();

  // Build list of query results
  for (Entity e : ds.prepare(q).asIterable()) {
    results.add(Entities.getNamespaceFromNamespaceKey(e.getKey()));
  }

  // Return result list
  return results;
}
```

### Kind queries

*Kind queries* return entities of kind `__kind__` whose key name is the name of an entity kind. Queries of this type are implicitly restricted to the current namespace and support filtering only for ranges over the `__key__` pseudoproperty. The results can be sorted by ascending (but not descending) `__key__` value. Because `__kind__` entities have no properties, both keys-only and non-keys-only queries return the same information.

The following example prints all kinds whose names start with a lowercase letter:

```
import com.google.appengine.api.datastore.DatastoreService;
import com.google.appengine.api.datastore.Entities;
import com.google.appengine.api.datastore.Entity;
import com.google.appengine.api.datastore.Key;
import com.google.appengine.api.datastore.KeyFactory;
import com.google.appengine.api.datastore.Query;
import com.google.appengine.api.datastore.Query.Filter;
import com.google.appengine.api.datastore.Query.FilterOperator;
import com.google.appengine.api.datastore.Query.CompositeFilter;
import com.google.appengine.api.datastore.Query.CompositeFilterOperator;
import com.google.appengine.api.datastore.Query.FilterPredicate;

import java.io.PrintWriter;

void printLowercaseKinds(DatastoreService ds, PrintWriter writer) {

  // Start with unrestricted kind query
  Query q = new Query(Entities.KIND_METADATA_KIND);

  List<Filter> subFils = new ArrayList();

  // Limit to lowercase initial letters
  subFils.add(new FilterPredicate(Entity.KEY_RESERVED_PROPERTY,
              Query.FilterOperator.GREATER_THAN_OR_EQUAL,
              Entities.createKindKey("a")));

  String endChar = Character.toString((char)('z' + 1));   // Character after 'z'

  subFils.add(new FilterPredicate(Entity.KEY_RESERVED_PROPERTY,
              Query.FilterOperator.LESS_THAN,
              Entities.createKindKey(endChar)));

  q.setFilter(CompositeFilterOperator.and(subFils));

  // Print heading
  writer.println("Lowercase kinds:");

  // Print query results
  for (Entity e : ds.prepare(q).asIterable()) {
    writer.println("  " + e.getKey().getName());
  }
}
```

### Property queries

*Property queries* return entities of kind `__property__` denoting the properties associated with an entity kind. The entity representing property *P* of kind *K* is built as follows:

-   The entity's key has kind `__property__` and key name *P*.
-   The parent entity's key has kind `__kind__` and key name *K*.

The behavior of a property query depends on whether it is a [keys-only](#Java_Property_queries_keys_only) or a [non-keys-only (property representation)](#Java_Property_queries_non_keys_only_property_representation) query, as detailed in the subsections below.

#### Property queries: keys-only

*Keys-only property queries* return a key for each indexed property of a specified entity kind. (Unindexed properties are not included.) The following example prints the names of all of an application's entity kinds and the properties associated with each:

```
import com.google.appengine.api.datastore.DatastoreService;
import com.google.appengine.api.datastore.Entities;
import com.google.appengine.api.datastore.Entity;
import com.google.appengine.api.datastore.Key;
import com.google.appengine.api.datastore.KeyFactory;
import com.google.appengine.api.datastore.Query;

import java.io.PrintWriter;

void printProperties(DatastoreService ds, PrintWriter writer) {

  // Create unrestricted keys-only property query
  q = new Query(Entities.PROPERTY_METADATA_KIND).setKeysOnly();

  // Print query results
  for (Entity e : ds.prepare(q).asIterable()) {
    writer.println(e.getKey().getParent().getName()
                     + ": "
                     + e.getKey().getName());
  }
}
```

Queries of this type are implicitly restricted to the current namespace and support filtering only for ranges over the pseudoproperty `__key__`, where the keys denote either `__kind__` or `__property__` entities. The results can be sorted by ascending (but not descending) `__key__` value. Filtering is applied to kind-property pairs, ordered first by kind and second by property: for instance, suppose you have an entity with these properties:

-   kind `Account` with properties
    -   `balance`
    -   `company`
-   kind `Employee` with properties
    -   `name`
    -   `ssn`
-   kind `Invoice` with properties
    -   `date`
    -   `amount`
-   kind `Manager` with properties
    -   `name`
    -   `title`
-   kind `Product` with properties
    -   `description`
    -   `price`

The query to return the property data would look like this:

```
import com.google.appengine.api.datastore.DatastoreService;
import com.google.appengine.api.datastore.Entities;
import com.google.appengine.api.datastore.Entity;
import com.google.appengine.api.datastore.Key;
import com.google.appengine.api.datastore.KeyFactory;
import com.google.appengine.api.datastore.Query;
import com.google.appengine.api.datastore.Query.Filter;
import com.google.appengine.api.datastore.Query.FilterOperator;
import com.google.appengine.api.datastore.Query.CompositeFilter;
import com.google.appengine.api.datastore.Query.CompositeFilterOperator;
import com.google.appengine.api.datastore.Query.FilterPredicate;

import java.io.PrintWriter;


void printPropertyRange(DatastoreService ds, PrintWriter writer) {

  // Start with unrestricted keys-only property query
  q = new Query(Entities.PROPERTY_METADATA_KIND).setKeysOnly();

  // Limit range
  q.setFilter(CompositeFilterOperator.and(
              new FilterPredicate(Entity.KEY_RESERVED_PROPERTY,
              Query.FilterOperator.GREATER_THAN_OR_EQUAL,
              Entities.createPropertyKey("Employee", "salary")),
              new FilterPredicate(Entity.KEY_RESERVED_PROPERTY,
              Query.FilterOperator.LESS_THAN_OR_EQUAL,
              Entities.createPropertyKey("Manager", "salary"))));


  // Print query results
  for (Entity e : ds.prepare(q).asIterable()) {
    writer.println(e.getKey().getParent().getName()
                     + ": "
                     + e.getKey().getName());
  }
}
```

The above query would return the following:

```
Employee: ssn
Invoice: date
Invoice: amount
Manager: name
```

Notice that the results do not include the `name` property of kind `Employee` and the `title` property of kind `Manager`, nor any properties of kinds `Account` and `Product`, because they fall outside the range specified for the query.

**Note:** Since the `__key__` kind filters on kind-property pairs, you can't use a single query for the same property name in different kinds. You can query for all properties in all kinds (or in a range of kinds), but you cannot construct a single query that retrieves, for example, all properties with names between *m* and *q* in all kinds (or in kinds *A* and *B*).

Property queries also support ancestor filtering on a `__kind__` or `__property__` key, to limit the query results to a single kind or property. You can use this, for instance, to get the properties associated with a given entity kind, as in the following example:

```
import com.google.appengine.api.datastore.DatastoreService;
import com.google.appengine.api.datastore.Entities;
import com.google.appengine.api.datastore.Entity;
import com.google.appengine.api.datastore.Query;

import java.util.ArrayList;
import java.util.List;

List<String> propertiesOfKind(DatastoreService ds, String kind) {

  // Start with unrestricted keys-only property query
  Query q = new Query(Entities.PROPERTY_METADATA_KIND).setKeysOnly();

  // Limit to specified kind
  q.setAncestor(Entities.createKindKey(kind));

  // Initialize result list
  ArrayList<String> results = new ArrayList<String>();

  //Build list of query results
  for (Entity e : ds.prepare(q).asIterable()) {
    results.add(e.getKey().getName());
  }

  // Return result list
  return results;
}
```

#### Property queries: non-keys-only (property representation)

Non-keys-only property queries, known as *property representation queries*, return additional information on the representations used by each kind-property pair. (Unindexed properties are not included.) The entity returned for property *P* of kind *K* has the same key as for a corresponding [keys-only query](#Java_Property_queries_keys_only), along with an additional `property_representation` property returning the property's representations. The value of this property is an instance of class [`java.util.Collection<String>`](https://web.archive.org/web/20160424230438/http://docs.oracle.com/javase/6/docs/api/java/util/Collection.html) containing one string for each representation of property *P* found in any entity of kind *K*.

Note that representations are not the same as property classes; multiple property classes can map to the same representation. (For example, [`java.lang.String`](https://web.archive.org/web/20160424230438/http://docs.oracle.com/javase/6/docs/api/java/lang/String.html) and [`com.google.appengine.api.datastore.PhoneNumber`](https://web.archive.org/web/20160424230438/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/PhoneNumber) both use the `STRING` representation.)

The following table maps from property classes to their representations:

<table>
<thead>
<tr class="header">
<th>Property class</th>
<th>Representation</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>java.lang.Byte</code></td>
<td><code>INT64</code></td>
</tr>
<tr class="even">
<td><code>java.lang.Short</code></td>
<td><code>INT64</code></td>
</tr>
<tr class="odd">
<td><code>java.lang.Integer</code></td>
<td><code>INT64</code></td>
</tr>
<tr class="even">
<td><code>java.lang.Long</code></td>
<td><code>INT64</code></td>
</tr>
<tr class="odd">
<td><code>java.lang.Float</code></td>
<td><code>DOUBLE</code></td>
</tr>
<tr class="even">
<td><code>java.lang.Double</code></td>
<td><code>DOUBLE</code></td>
</tr>
<tr class="odd">
<td><code>java.lang.Boolean</code></td>
<td><code>BOOLEAN</code></td>
</tr>
<tr class="even">
<td><code>java.lang.String</code></td>
<td><code>STRING</code></td>
</tr>
<tr class="odd">
<td><a href="https://web.archive.org/web/20160424230438/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/ShortBlob"><code>com.google.appengine.api.datastore.ShortBlob</code></a></td>
<td><code>STRING</code></td>
</tr>
<tr class="even">
<td><code>java.util.Date</code></td>
<td><code>INT64</code></td>
</tr>
<tr class="odd">
<td><a href="https://web.archive.org/web/20160424230438/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/GeoPt"><code>com.google.appengine.api.datastore.GeoPt</code></a></td>
<td><code>POINT</code></td>
</tr>
<tr class="even">
<td><a href="https://web.archive.org/web/20160424230438/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/PostalAddress"><code>com.google.appengine.api.datastore.PostalAddress</code></a></td>
<td><code>STRING</code></td>
</tr>
<tr class="odd">
<td><a href="https://web.archive.org/web/20160424230438/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/PhoneNumber"><code>com.google.appengine.api.datastore.PhoneNumber</code></a></td>
<td><code>STRING</code></td>
</tr>
<tr class="even">
<td><a href="https://web.archive.org/web/20160424230438/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/Email"><code>com.google.appengine.api.datastore.Email</code></a></td>
<td><code>STRING</code></td>
</tr>
<tr class="odd">
<td><a href="https://web.archive.org/web/20160424230438/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/users/User"><code>com.google.appengine.api.users.User</code></a></td>
<td><code>USER</code></td>
</tr>
<tr class="even">
<td><a href="https://web.archive.org/web/20160424230438/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/IMHandle"><code>com.google.appengine.api.datastore.IMHandle</code></a></td>
<td><code>STRING</code></td>
</tr>
<tr class="odd">
<td><a href="https://web.archive.org/web/20160424230438/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/Link"><code>com.google.appengine.api.datastore.Link</code></a></td>
<td><code>STRING</code></td>
</tr>
<tr class="even">
<td><a href="https://web.archive.org/web/20160424230438/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/Category"><code>com.google.appengine.api.datastore.Category</code></a></td>
<td><code>STRING</code></td>
</tr>
<tr class="odd">
<td><a href="https://web.archive.org/web/20160424230438/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/Rating"><code>com.google.appengine.api.datastore.Rating</code></a></td>
<td><code>INT64</code></td>
</tr>
<tr class="even">
<td><a href="https://web.archive.org/web/20160424230438/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/Key"><code>com.google.appengine.api.datastore.Key</code></a></td>
<td><code>REFERENCE</code></td>
</tr>
<tr class="odd">
<td><a href="https://web.archive.org/web/20160424230438/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/blobstore/BlobKey"><code>com.google.appengine.api.blobstore.BlobKey</code></a></td>
<td><code>STRING</code></td>
</tr>
<tr class="even">
<td><code>java.util.Collection&lt;T&gt;</code></td>
<td>Representation of <code>T</code></td>
</tr>
</tbody>
</table>

The following example finds all representations of a specified property for a given entity kind:

```
import com.google.appengine.api.datastore.DatastoreService;
import com.google.appengine.api.datastore.Entities;
import com.google.appengine.api.datastore.Entity;
import com.google.appengine.api.datastore.Query;

import java.util.Collection;

Collection<String> representationsOfProperty(DatastoreService ds,
                                             String kind,
                                             String property) {

  // Start with unrestricted non-keys-only property query
  Query q = new Query(Entities.PROPERTY_METADATA_KIND);

  // Limit to specified kind and property
  q.setFilter(new FilterPredicate('__key__',  Query.FilterOperator.EQUAL, Entities.createPropertyKey(kind, property)));

  // Get query result
  Entity propInfo = ds.prepare(q).asSingleEntity();

  // Return collection of property representations
  return (Collection<String>) propInfo.getProperty("property_representation");
}
```
