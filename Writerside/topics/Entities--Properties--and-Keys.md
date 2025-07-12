# Entities, Properties, and Keys

  

Data objects in App Engine Datastore are known as *entities*. An entity has one or more named *properties*, each of which can have one or more values. Entities of the same kind need not have the same properties, and an entity's values for a given property need not all be of the same data type. (If necessary, an application can establish and enforce such restrictions in its own data model.)

Datastore supports a variety of [data types for property values](#Java_Properties_and_value_types). These include, among others:

-   Integers
-   Floating-point numbers
-   Strings
-   Dates
-   Binary data

For a full list of types, see [Properties and value types](https://web.archive.org/web/20160424230911/https://cloud.google.com/appengine/docs/java/datastore/entities#Java_Properties_and_value_types).

Each entity in Datastore has a *key* that uniquely identifies it. The key consists of the following components:

-   The *namespace* of the entity, which allows for [multitenancy](https://web.archive.org/web/20160424230911/https://cloud.google.com/appengine/docs/java/multitenancy/)
-   The [*kind*](#Java_Kinds_and_identifiers) of the entity, which categorizes it for the purpose of Datastore queries
-   An [*identifier*](#Java_Kinds_and_identifiers) for the individual entity, which can be either
    -   a *key name* string
    -   an integer *numeric ID*
-   An optional [*ancestor path*](#Java_Ancestor_paths) locating the entity within Datastore hierarchy

Don't name a property "key." This name is reserved for a special property used to store the Model key. Though it may work locally, a property named "key" will prevent deployment to App Engine.

An application has access only to entities it has created itself; it can't access data belonging to other applications. An application can fetch an individual entity from Datastore using the entity's key, or it can retrieve one or more entities by issuing a [*query*](https://web.archive.org/web/20160424230911/https://cloud.google.com/appengine/docs/java/datastore/queries) based on the entities' keys or property values.

The Java App Engine SDK includes a simple Java API, provided in the package [`com.google.appengine.api.datastore`](https://web.archive.org/web/20160424230911/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/package-summary), that supports the features of Datastore directly. All of the examples in this document are based on this low-level Java API; you can choose to use it either directly in your application or as a basis on which to build your own data management layer.

Datastore itself does not enforce any restrictions on the structure of entities, such as whether a given property has a value of a particular type; this task is left to the application.

## Contents

1.  [Kinds and identifiers](#Java_Kinds_and_identifiers)
2.  [Ancestor paths](#Java_Ancestor_paths)
3.  [Transactions and entity groups](#Java_Transactions_and_entity_groups)
4.  [Properties and value types](#Java_Properties_and_value_types)
5.  [Working with entities](#Java_Working_with_entities)
    1.  [Creating an entity](#Java_Creating_an_entity)
    2.  [Retrieving an entity](#Java_Retrieving_an_entity)
    3.  [Updating an entity](#Java_Updating_an_entity)
    4.  [Deleting an entity](#Java_Deleting_an_entity)
    5.  [Embedded entities](#Java_Embedded_entities)
    6.  [Repeated properties](#Java_Repeated_properties)
    7.  [Batch operations](#Java_Batch_operations)
    8.  [Generating keys](#Java_Generating_keys)
    9.  [Using an empty list](#Java_Using_an_empty_list)
6.  [Understanding write costs](#Java_Understanding_write_costs)

## Kinds and identifiers

Each Datastore entity is of a particular *kind,* which categorizes the entity for the purpose of queries: for instance, a human resources application might represent each employee at a company with an entity of kind `Employee`. In the Java Datastore API, you specify an entity's kind when you create it, as an argument to the [`Entity()`](https://web.archive.org/web/20160424230911/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/Entity#Entity(java.lang.String)) constructor. All kind names that begin with two underscores (`__`) are reserved and may not be used.

The following example creates an entity of kind `Employee`, populates its property values, and saves it to Datastore:

```
import java.util.Date;
import com.google.appengine.api.datastore.DatastoreService;
import com.google.appengine.api.datastore.DatastoreServiceFactory;
import com.google.appengine.api.datastore.Entity;


DatastoreService datastore = DatastoreServiceFactory.getDatastoreService();


Entity employee = new Entity("Employee");

employee.setProperty("firstName", "Antonio");
employee.setProperty("lastName", "Salieri");

Date hireDate = new Date();
employee.setProperty("hireDate", hireDate);

employee.setProperty("attendedHrTraining", true);


datastore.put(employee);
```

In addition to a kind, each entity has an *identifier*, assigned when the entity is created. Because it is part of the entity's key, the identifier is associated permanently with the entity and cannot be changed. It can be assigned in either of two ways: - Your application can specify its own *key name* string for the entity. - You can have Datastore automatically assign the entity an integer *numeric ID*.

To assign an entity a key name, provide the name as the second argument to the constructor when you create the entity:

```
Entity employee = new Entity("Employee", "asalieri");
```

To have Datastore assign a numeric ID automatically, omit this argument:

```
Entity employee = new Entity("Employee");
```

### Assigning identifiers

The Datastore can be configured to generate auto IDs using [two different auto id policies](https://web.archive.org/web/20160424230911/https://cloud.google.com/appengine/docs/java/config/appconfig#Java_appengine_web_xml_Auto_ID_policy):

-   The `default` policy generates a random sequence of unused IDs that are approximately uniformly distributed. Each ID can be up to 16 decimal digits long.
-   The `legacy` policy creates a sequence of non-consecutive smaller integer IDs.

If you want to display the entity IDs to the user, and/or depend upon their order, the best thing to do is use manual allocation.

**Note:** Instead of using key name strings or generating numeric IDs automatically, advanced applications may sometimes wish to assign their own numeric IDs manually to the entities they create. Be aware, however, that there is nothing to prevent Datastore from assigning one of your manual numeric IDs to another entity. The only way to avoid such conflicts is to have your application obtain a block of IDs with the methods [`DatastoreService.allocateIds()`](https://web.archive.org/web/20160424230911/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/DatastoreService#allocateIds(com.google.appengine.api.datastore.Key,%20java.lang.String,%20long)) or [`AsyncDatastoreService.allocateIds()`](https://web.archive.org/web/20160424230911/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/AsyncDatastoreService#allocateIds(com.google.appengine.api.datastore.Key,%20java.lang.String,%20long)). The Datastore's automatic ID generator will keep track of IDs that have been allocated with these methods and will avoid reusing them for another entity, so you can safely use such IDs without conflict.

Datastore generates a random sequence of unused IDs that are approximately uniformly distributed. Each ID can be up to 16 decimal digits long.

System-allocated ID values are guaranteed unique to the entity group. If you copy an entity from one entity group or namespace to another and wish to preserve the ID part of the key, be sure to allocate the ID first to prevent Datastore from selecting that ID for a future assignment.

## Ancestor paths

Entities in the Datastore form a hierarchically structured space similar to the directory structure of a file system. When you create an entity, you can optionally designate another entity as its *parent;* the new entity is a *child* of the parent entity (note that unlike in a file system, the parent entity need not actually exist). An entity without a parent is a *root entity.* The association between an entity and its parent is permanent, and cannot be changed once the entity is created. The Datastore will never assign the same numeric ID to two entities with the same parent, or to two root entities (those without a parent).

An entity's parent, parent's parent, and so on recursively, are its *ancestors;* its children, children's children, and so on, are its *descendants.* A root entity and all of its descendants belong to the same *entity group.* The sequence of entities beginning with a root entity and proceeding from parent to child, leading to a given entity, constitute that entity's *ancestor path.* The complete key identifying the entity consists of a sequence of kind-identifier pairs specifying its ancestor path and terminating with those of the entity itself:

```
[Person:GreatGrandpa, Person:Grandpa, Person:Dad, Person:Me]
```

For a root entity, the ancestor path is empty and the key consists solely of the entity's own kind and identifier:

```
[Person:GreatGrandpa]
```

This concept is illustrated by the following diagram:

<img src="https://web.archive.org/web/20160424230911im_/https://cloud.google.com/images/products/clouddatastore/entity-group.png" width="450" />

To designate an entity's parent, provide the parent entity's key as an argument to the [`Entity()`](https://web.archive.org/web/20160424230911/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/Entity#Entity(java.lang.String,%20com.google.appengine.api.datastore.Key)) constructor when creating the child entity. You can get the key by calling the parent entity's [`getKey()`](https://web.archive.org/web/20160424230911/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/Entity#getKey()) method:

```
Entity employee = new Entity("Employee");
datastore.put(employee);

Entity address = new Entity("Address", employee.getKey());
datastore.put(address);
```

If the new entity also has a key name, provide the key name as the second argument to the `Entity()` constructor and the key of the parent entity as the third argument:

```
Entity address = new Entity("Address", "addr1", employee.getKey());
```

## Transactions and entity groups

Every attempt to create, update, or delete an entity takes place in the context of a [*transaction*](https://web.archive.org/web/20160424230911/https://cloud.google.com/appengine/docs/java/datastore/transactions). A single transaction can include any number of such operations. To maintain the consistency of the data, the transaction ensures that all of the operations it contains are applied to Datastore as a unit or, if any of the operations fails, that none of them are applied. Furthermore, all strongly-consistent reads (ancestory queries or gets) performed within the same transaction observe a consistent snapshot of the data.

**Note:** If your application receives an exception when attempting to commit a transaction, it does not necessarily mean that the transaction has failed. It is possible to receive a [`DatastoreTimeoutException`](https://web.archive.org/web/20160424230911/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/DatastoreTimeoutException) or [`DatastoreFailureException`](https://web.archive.org/web/20160424230911/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/DatastoreFailureException) even when a transaction has been committed and will eventually be applied successfully. Whenever possible, structure your Datastore transactions so that the end result will be unaffected if the same transaction is applied more than once.

As mentioned above, an entity group is a set of entities connected through ancestry to a common root element. The organization of data into entity groups can limit what transactions can be performed:

-   All the data accessed by a transaction must be contained in at most 25 entity groups.
-   If you want to use queries within a transaction, your data must be organized into entity groups in such a way that you can specify ancestor filters that will match the right data.
-   There is a write throughput limit of about one transaction per second within a single entity group. This limitation exists because Datastore performs masterless, synchronous replication of each entity group over a wide geographic area to provide high reliability and fault tolerance.

In many applications, it is acceptable to use eventual consistency (i.e. a non-ancestor query spanning multiple entity groups, which may at times return slightly stale data) when obtaining a broad view of unrelated data, and then to use strong consistency (an ancestory query, or a `get` of a single entity) when viewing or editing a single set of highly related data. In such applications, it is usually a good approach to use a separate entity group for each set of highly related data. For more information, see [Structuring for Strong Consistency](https://web.archive.org/web/20160424230911/https://cloud.google.com/appengine/docs/java/datastore/structuring_for_strong_consistency).

**Note:** Avoid storing sensitive information in the entity group key. Entity group keys may be retained after the entity group is deleted in order to provide fast and reliable service across Datastore.

## Properties and value types

The data values associated with an entity consist of one or more *properties.* Each property has a name and one or more values. A property can have values of more than one type, and two entities can have values of different types for the same property. Properties can be indexed or unindexed (queries that order or filter on a property *P* will ignore entities where *P* is unindexed). An entity can have at most 20,000 indexed properties.

**Note:** Properties with multiple values can be useful, for instance, when performing queries with equality filters: an entity satisfies the query if any of its values for a property matches the value specified in the filter. For more details on multiple-valued properties, including issues you should be aware of, see the [Datastore Queries](https://web.archive.org/web/20160424230911/https://cloud.google.com/appengine/docs/java/datastore/queries) page.

<span id="Java_Value_type_sort_order"></span>The following value types are supported:

<table>
<colgroup>
<col style="width: 25%" />
<col style="width: 25%" />
<col style="width: 25%" />
<col style="width: 25%" />
</colgroup>
<thead>
<tr class="header">
<th>Value type</th>
<th>Java type(s)</th>
<th>Sort order</th>
<th>Notes</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Integer</td>
<td><code>short</code><br />
<code>int</code><br />
<code>long</code><br />
<code>java.lang.Short</code><br />
<code>java.lang.Integer</code><br />
<code>java.lang.Long</code></td>
<td>Numeric</td>
<td>Stored as long integer, then converted to the field type<br />
<br />
Out-of-range values overflow</td>
</tr>
<tr class="even">
<td>Floating-point number</td>
<td><code>float</code><br />
<code>double</code><br />
<code>java.lang.Float</code><br />
<code>java.lang.Double</code></td>
<td>Numeric</td>
<td>64-bit double precision,<br />
IEEE 754</td>
</tr>
<tr class="odd">
<td>Boolean</td>
<td><code>boolean</code><br />
<code>java.lang.Boolean</code></td>
<td><code>false</code>&lt;<code>true</code></td>
<td></td>
</tr>
<tr class="even">
<td>Text string (short)</td>
<td><code>java.lang.String</code></td>
<td>Unicode</td>
<td>Up to 1500 bytes<br />
<br />
Values greater than 1500 bytes throw <code>IllegalArgumentException</code></td>
</tr>
<tr class="odd">
<td>Text string (long)</td>
<td><a href="https://web.archive.org/web/20160424230911/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/Text"><code>com.google.appengine.api.datastore.Text</code></a></td>
<td>None</td>
<td>Up to 1 megabyte<br />
<br />
Not indexed</td>
</tr>
<tr class="even">
<td>Byte string (short)</td>
<td><a href="https://web.archive.org/web/20160424230911/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/ShortBlob"><code>com.google.appengine.api.datastore.ShortBlob</code></a></td>
<td>Byte order</td>
<td>Up to 1500 bytes<br />
<br />
Values longer than 1500 bytes throw <code>IllegalArgumentException</code></td>
</tr>
<tr class="odd">
<td>Byte string (long)</td>
<td><a href="https://web.archive.org/web/20160424230911/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/Blob"><code>com.google.appengine.api.datastore.Blob</code></a></td>
<td>None</td>
<td>Up to 1 megabyte<br />
<br />
Not indexed</td>
</tr>
<tr class="even">
<td>Date and time</td>
<td><code>java.util.Date</code></td>
<td>Chronological</td>
<td></td>
</tr>
<tr class="odd">
<td>Geographical point</td>
<td><a href="https://web.archive.org/web/20160424230911/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/GeoPt"><code>com.google.appengine.api.datastore.GeoPt</code></a></td>
<td>By latitude,<br />
then longitude</td>
<td></td>
</tr>
<tr class="even">
<td>Postal address</td>
<td><a href="https://web.archive.org/web/20160424230911/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/PostalAddress"><code>com.google.appengine.api.datastore.PostalAddress</code></a></td>
<td>Unicode</td>
<td></td>
</tr>
<tr class="odd">
<td>Telephone number</td>
<td><a href="https://web.archive.org/web/20160424230911/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/PhoneNumber"><code>com.google.appengine.api.datastore.PhoneNumber</code></a></td>
<td>Unicode</td>
<td></td>
</tr>
<tr class="even">
<td>Email address</td>
<td><a href="https://web.archive.org/web/20160424230911/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/Email"><code>com.google.appengine.api.datastore.Email</code></a></td>
<td>Unicode</td>
<td></td>
</tr>
<tr class="odd">
<td>Google Accounts user</td>
<td><a href="https://web.archive.org/web/20160424230911/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/users/User"><code>com.google.appengine.api.users.User</code></a></td>
<td>Email address<br />
in Unicode order</td>
<td></td>
</tr>
<tr class="even">
<td>Instant messaging handle</td>
<td><a href="https://web.archive.org/web/20160424230911/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/IMHandle"><code>com.google.appengine.api.datastore.IMHandle</code></a></td>
<td>Unicode</td>
<td></td>
</tr>
<tr class="odd">
<td>Link</td>
<td><a href="https://web.archive.org/web/20160424230911/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/Link"><code>com.google.appengine.api.datastore.Link</code></a></td>
<td>Unicode</td>
<td></td>
</tr>
<tr class="even">
<td>Category</td>
<td><a href="https://web.archive.org/web/20160424230911/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/Category"><code>com.google.appengine.api.datastore.Category</code></a></td>
<td>Unicode</td>
<td></td>
</tr>
<tr class="odd">
<td>Rating</td>
<td><a href="https://web.archive.org/web/20160424230911/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/Rating"><code>com.google.appengine.api.datastore.Rating</code></a></td>
<td>Numeric</td>
<td></td>
</tr>
<tr class="even">
<td>Datastore key</td>
<td><a href="https://web.archive.org/web/20160424230911/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/Key"><code>com.google.appengine.api.datastore.Key</code></a><br />
or the referenced object (as a child)</td>
<td>By path elements<br />
(kind, identifier,<br />
kind, identifier...)</td>
<td>Up to 1500 bytes<br />
<br />
Values longer than 1500 bytes throw <code>IllegalArgumentException</code></td>
</tr>
<tr class="odd">
<td>Blobstore key</td>
<td><a href="https://web.archive.org/web/20160424230911/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/blobstore/BlobKey"><code>com.google.appengine.api.blobstore.BlobKey</code></a></td>
<td>Byte order</td>
<td></td>
</tr>
<tr class="even">
<td><a href="#Java_Embedded_entities">Embedded entity</a></td>
<td><a href="https://web.archive.org/web/20160424230911/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/EmbeddedEntity"><code>com.google.appengine.api.datastore.EmbeddedEntity</code></a></td>
<td>None</td>
<td>Not indexed</td>
</tr>
<tr class="odd">
<td>Null</td>
<td><code>null</code></td>
<td>None</td>
<td></td>
</tr>
</tbody>
</table>

**Important:** We strongly recommend that you avoid storing a `users.User` as a property value, because this includes the email address along with the unique ID. If a user changes their email address and you compare their old, stored `user.User` to the new `user.User` value, they won't match. Instead, use the `User` *user ID value* as the user's stable unique identifier.

For text strings and unencoded binary data (byte strings), Datastore supports two value types:

-   Short strings (up to 1500 bytes) are indexed and can be used in query filter conditions and sort orders.
-   Long strings (up to 1 megabyte) are not indexed and cannot be used in query filters and sort orders.

Note: The long byte string type is named [`Blob`](https://web.archive.org/web/20160424230911/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/Blob) in the Datastore API. This type is unrelated to blobs as used in the [Blobstore API](https://web.archive.org/web/20160424230911/https://cloud.google.com/appengine/docs/java/blobstore).

<span id="Java_Value_type_ordering"></span>

When a query involves a property with values of mixed types, Datastore uses a deterministic ordering based on the internal representations:

1.  Null values
2.  Fixed-point numbers
    -   Integers
    -   Dates and times
    -   Ratings
3.  Boolean values
4.  Byte sequences
    -   Byte string
    -   Unicode string
    -   Blobstore keys
5.  Floating-point numbers
6.  Geographical points
7.  Google Accounts users
8.  Datastore keys

Because long text strings, long byte strings, and embedded entities are not indexed, they have no ordering defined.

**Note:** Integers and floating-point numbers are considered separate types in Datastore. If an entity uses a mix of integers and floats for the same property, all integers will be sorted before all floats: for example,  
  
`7` &lt; `3.2`

## Working with entities

Applications can use the Datastore API to create, retrieve, update, and delete entities. If the application knows the complete key for an entity (or can derive it from its parent key, kind, and identifier), it can use the key to operate directly on the entity. An application can also obtain an entity's key as a result of a Datastore query; see the [Datastore Queries](https://web.archive.org/web/20160424230911/https://cloud.google.com/appengine/docs/java/datastore/queries) page for more information.

The Java Datastore API uses methods of the [`DatastoreService`](https://web.archive.org/web/20160424230911/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/DatastoreService) interface to operate on entities. You obtain a `DatastoreService` object by calling the static method [`DatastoreServiceFactory.getDatastoreService()`](https://web.archive.org/web/20160424230911/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/DatastoreServiceFactory#getDatastoreService()):

```
import com.google.appengine.api.datastore.DatastoreService;
import com.google.appengine.api.datastore.DatastoreServiceFactory;

// ...
DatastoreService datastore = DatastoreServiceFactory.getDatastoreService();
```

### Creating an entity

In Java, you create a new entity by constructing an instance of class [`Entity`](https://web.archive.org/web/20160424230911/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/Entity), supplying the entity's kind as an argument to the [`Entity()`](https://web.archive.org/web/20160424230911/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/Entity#Entity(java.lang.String)) constructor. After populating the entity's properties if necessary, you save it to the datstore by passing it as an argument to the [`DatastoreService.put()`](https://web.archive.org/web/20160424230911/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/DatastoreService#put(com.google.appengine.api.datastore.Entity)) method. You can specify the entity's key name by passing it as the second argument to the constructor:

```
Entity employee = new Entity("Employee", "asalieri");
// ... set properties ...

datastore.put(employee);
```

If you don't provide a key name, Datastore will automatically generate a numeric ID for the entity's key:

```
Entity employee = new Entity("Employee");
// ... set properties ...

datastore.put(employee);
```

### Retrieving an entity

To retrieve an entity identified by a given key, pass the [`Key`](https://web.archive.org/web/20160424230911/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/Key) object to the [`DatastoreService.get()`](https://web.archive.org/web/20160424230911/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/DatastoreService#get(com.google.appengine.api.datastore.Key)) method:

```
// Key employeeKey = ...;
Entity employee = datastore.get(employeeKey);
```

### Updating an entity

To update an existing entity, modify the attributes of the Entity object, then pass it to the [`DatastoreService.put()`](https://web.archive.org/web/20160424230911/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/DatastoreService#put(com.google.appengine.api.datastore.Entity)) method. The object data overwrites the existing entity. The entire object is sent to Datastore with every call to `put()`.

**Note:** The Datastore API does not distinguish between creating a new entity and updating an existing one. If the object's key represents an entity that already exists, the put() method overwrites the existing entity. You can use a [transaction](https://web.archive.org/web/20160424230911/https://cloud.google.com/appengine/docs/java/datastore/transactions) to test whether an entity with a given key exists before creating one.

### Deleting an entity

Given an entity's key, you can delete the entity with the [`DatastoreService.delete()`](https://web.archive.org/web/20160424230911/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/DatastoreService#delete(com.google.appengine.api.datastore.Key...)) method:

```
// Key employeeKey = ...;
datastore.delete(employeeKey);
```

### Repeated properties

You can store multiple values within a single property.

```
Entity employee = new Entity("Employee");
ArrayList<String> favoriteFruit = new ArrayList<String>();
favoriteFruit.add("Pear");
favoriteFruit.add("Apple");
employee.setProperty("favoriteFruit", favoriteFruit);
datastore.put(employee);

// Sometime later
employee = datastore.get(employee.getKey());
favoriteFruit = (ArrayList<String>) employee.getProperty("favoriteFruit");
```

### Embedded entities

You may sometimes find it convenient to embed one entity as a property of another entity. This can be useful, for instance, for creating a hierarchical structure of property values within an entity. The Java class [`EmbeddedEntity`](https://web.archive.org/web/20160424230911/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/EmbeddedEntity) allows you to do this:

```
// Entity employee = /*...*/;
EmbeddedEntity embeddedContactInfo = new EmbeddedEntity();

embeddedContactInfo.setProperty("homeAddress", /*...*/);
embeddedContactInfo.setProperty("phoneNumber", /*...*/);
embeddedContactInfo.setProperty("emailAddress", /*...*/);

employee.setProperty("contactInfo", embeddedContactInfo);
```

Properties of an embedded entity are not indexed and cannot be used in queries. You can optionally associate a key with an embedded entity, but (unlike a full-fledged entity) the key is not required and, even if present, cannot be used to retrieve the entity.

Instead of populating the embedded entity's properties manually, you can use the [`setPropertiesFrom()`](https://web.archive.org/web/20160424230911/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/PropertyContainer#setPropertiesFrom(com.google.appengine.api.datastore.PropertyContainer)) method to copy them from an existing entity:

```
// Entity employee = /*...*/;
// Entity contactInfo = /*...*/;
EmbeddedEntity embeddedContactInfo = new EmbeddedEntity();
Key infoKey;

infoKey = contactInfo.getKey();
embeddedContactInfo.setKey(infoKey);
embeddedContactInfo.setPropertiesFrom(contactInfo);

employee.setProperty("contactInfo", embeddedContactInfo);
```

You can later use the same method to recover the original entity from the embedded entity:

```
// Entity employee = /*...*/;
EmbeddedEntity embeddedContactInfo = employee.getProperty("contactInfo");
Key infoKey = embeddedContactInfo.getKey();
Entity contactInfo = new Entity(infoKey);

contactInfo.setPropertiesFrom(embeddedContactInfo);
```

### Batch operations

The `DatastoreService` methods [`put()`](https://web.archive.org/web/20160424230911/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/DatastoreService#put(java.lang.Iterable)), [`get()`](https://web.archive.org/web/20160424230911/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/DatastoreService#get(java.lang.Iterable)), and [`delete()`](https://web.archive.org/web/20160424230911/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/DatastoreService#delete(java.lang.Iterable)) (and their AsyncDatastoreService counterparts) have batch versions that accept an [iterable](https://web.archive.org/web/20160424230911/http://download.oracle.com/javase/6/docs/api/index.html?java/lang/Iterable.html) object (of class [`Entity`](https://web.archive.org/web/20160424230911/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/Entity) for `put()`, [`Key`](https://web.archive.org/web/20160424230911/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/Key) for `get()` and `delete()`) and use it to operate on multiple entities in a single Datastore call:

```
import java.util.Arrays;
import java.util.List;

// ...
Entity employee1 = new Entity("Employee");
Entity employee2 = new Entity("Employee");
Entity employee3 = new Entity("Employee");
// ...

List<Entity> employees = Arrays.asList(employee1, employee2, employee3);
datastore.put(employees);
```

These batch operations group all the entities or keys by entity group and then perform the requested operation on each entity group in parallel. Such batch calls are faster than making separate calls for each individual entity, because they incur the overhead for only one service call. If multiple entity groups are involved, the work for all the groups is performed in parallel on the server side.

**Note:** A batch `put()` or `delete()` call may succeed for some entities but not others. If it is important that the call succeed completely or fail completely, use a [transaction](https://web.archive.org/web/20160424230911/https://cloud.google.com/appengine/docs/java/datastore/transactions) with all affected entities in the same entity group. Attempting a batch operation inside a transaction with entities or keys belonging to multiple entity groups will result in an `IllegalArgumentException`.

### Generating keys

Applications can use the class [`KeyFactory`](https://web.archive.org/web/20160424230911/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/KeyFactory) to create a [`Key`](https://web.archive.org/web/20160424230911/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/Key) object for an entity from known components, such as the entity's kind and identifier. For an entity with no parent, pass the kind and identifier (either a key name string or a numeric ID) to the static method [`KeyFactory.createKey()`](https://web.archive.org/web/20160424230911/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/KeyFactory#createKey(java.lang.String,%20long)) to create the key. The following examples create a key for an entity of kind `Person` with key name `"GreatGrandpa"` or numeric ID `74219`:

```
Key k = KeyFactory.createKey("Person", "GreatGrandpa");

Key k = KeyFactory.createKey("Person", 74219);
```

If the key includes a path component, you can use the helper class [`KeyFactory.Builder`](https://web.archive.org/web/20160424230911/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/KeyFactory.Builder) to build the path. This class's [`addChild`](https://web.archive.org/web/20160424230911/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/KeyFactory.Builder#addChild(java.lang.String,%20long)) method adds a single entity to the path and returns the builder itself, so you can chain together a series of calls, beginning with the root entity, to build up the path one entity at a time. After building the complete path, call [`getKey`](https://web.archive.org/web/20160424230911/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/KeyFactory.Builder#getKey()) to retrieve the resulting key:

```
Key k = new KeyFactory.Builder("Person", "GreatGrandpa")
                      .addChild("Person", "Grandpa")
                      .addChild("Person", "Dad")
                      .addChild("Person", "Me")
                      .getKey();
```

Class `KeyFactory` also includes the static methods [`keyToString`](https://web.archive.org/web/20160424230911/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/KeyFactory#keyToString(com.google.appengine.api.datastore.Key)) and [`stringToKey`](https://web.archive.org/web/20160424230911/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/KeyFactory#stringToKey(java.lang.String)) for converting between keys and their string representations:

```
String personKeyStr = KeyFactory.keyToString(personKey);
// ...

Key    personKey = KeyFactory.stringToKey(personKeyStr);
Entity person    = datastore.get(personKey);
```

The string representation of a key is "web-safe": it does not contain characters considered special in HTML or in URLs.

**Note:** The `KeyFactory.keyToString` method is different from [`Key.toString`](https://web.archive.org/web/20160424230911/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/Key#toString()), which returns a human-readable string suitable for use in debugging and logging. If you need a string value that can be converted to a usable key, use `KeyFactory.keyToString`.  
  
Note also that a key's string representation is not encrypted: a user can decode the key string to extract its components, including the kinds and identifiers of the entity and its ancestors. If it is important to conceal this information from the user, you must encrypt the key string yourself before sending it to the user.

### Using an empty list

Datastore historically did not have a representation for a property representing an empty list. The Java SDK worked around this by storing empty collections as null values, so there is no way to distinguish between null values and empty lists. To maintain backward compatibility, this remains the default behavior, synopsized as follows:

-   Null properties are written as null to Datastore
-   Empty collections are written as null to Datastore
-   A null is read as null from Datastore
-   An empty collection is read as null.

**Important:** A read modify write of an entity with an empty list will cause that list to be turned into a null value.

However, if you change the default behavior, the Appengine Datastore Java SDK will support storage of empty lists. **We recommend you consider the implications of changing the default behavior of your application and then turn on support for empty lists**.

To change default behavior so you can use empty lists, set the [DATASTORE\_EMPTY\_LIST\_SUPPORT](https://web.archive.org/web/20160424230911/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/DatastoreServiceConfig#DATASTORE_EMPTY_LIST_SUPPORT) property during your app initialization as follows:

System.setProperty(DatastoreServiceConfig.[DATASTORE\_EMPTY\_LIST\_SUPPORT](https://web.archive.org/web/20160424230911/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/DatastoreServiceConfig#DATASTORE_EMPTY_LIST_SUPPORT), Boolean.TRUE.toString());

With this property set to `true` as shown above:

-   Null properties are written as null to Datastore
-   Empty collections (#Collection.isEmpty()) are written as empty list to Datastore
-   A null is read as null from Datastore
-   When reading from Datastore an empty list is returned as an empty Collection.

**Important:** Your queries might be affected when you turn on empty list support. Null values are indexed in Datastore but empty lists are not. If you were storing empty lists as nulls and then querying for null to find them, then changing to empty list will cause these queries to not return results.

## Understanding write costs

When your application executes a Datastore `put` operation, Datastore must perform a number of writes to store the entity. Your application is charged for each of these writes. You can see how many writes will be required to store an entity by looking at the data viewer in the SDK Development Console. This section explains how these write costs are calculated.

Every entity requires a minimum of two writes to store: one for the entity itself and another for the built-in `EntitiesByKind` index, which is used by the query planner to service a variety of queries. In addition, Datastore maintains two other built-in indexes, `EntitiesByProperty` and `EntitiesByPropertyDesc`, which provide efficient scans of entities by single property values in ascending and descending order, respectively. Each of an entity's indexed property values must be written to each of these indexes.

As an example, consider an entity with properties *A*, *B*, and *C*:

```
Key: 'Foo:1' (kind = 'Foo', id = 1, no parent)
A: 1, 2
B: null
C: 'this', 'that', 'theOther'
```

Assuming there are no composite indexes (see below) for entities of this kind, this entity requires 14 writes to store:

-   1 for the entity itself
-   1 for the `EntitiesByKind` index
-   4 for property *A* (2 for each of two values)
-   2 for property *B* (a null value still needs to be written)
-   6 for property *C* (2 for each of three values)

Composite indexes (those referring to multiple properties) require additional writes to maintain. Suppose you define the following composite index:

```
Kind: 'Foo'
A ?, B ?
```

where the triangles indicate the sort order for the specified properties: ascending for property *A* and descending for property *B.* Storing the entity defined above now takes an additional write to the composite index for every combination of *A* and *B* values:

(`1`, `null`) (`2`, `null`)

This adds 2 writes for the composite index, for a total of 1 + 1 + 4 + 2 + 6 + 2 = 16. Now add property *C* to the index:

```
Kind: 'Foo'
A ?, B ?, C ?
```

Storing the same entity now requires a write to the composite index for each possible combination of *A*, *B*, and *C* values:

(`1`, `null`, `'this'`) (`1`, `null`, `'that'`) (`1`, `null`, `'theOther'`)

(`2`, `null`, `'this'`) (`2`, `null`, `'that'`) (`2`, `null`, `'theOther'`)

This brings the total number of writes to 1 + 1 + 4 + 2 + 6 + 6 = 20.

If a Datastore entity contains many multiple-valued properties, or if a single such property is referenced many times, the number of writes required to maintain the index can explode combinatorially. Such [exploding indexes](https://web.archive.org/web/20160424230911/https://cloud.google.com/appengine/docs/java/datastore/indexes#Java_Index_limits) can be very expensive to maintain. For example, consider a composite index that includes ancestors:

```
Kind: 'Foo'
A ?, B ?, C ?
Ancestor: True
```

Storing a simple entity with this index present takes the same number of writes as before. However, if the entity has ancestors, it requires a write for each possible combination of property values *and ancestors*, in addition to those for the entity itself. Thus an entity defined as

```
Key: 'GreatGrandpa:1/Grandpa:1/Dad:1/Foo:1' (kind = 'Foo', id = 1, parent = 'GreatGrandpa:1/Grandpa:1/Dad:1')
A: 1, 2
B: null
C: 'this', 'that', 'theOther'
```

would require a write to the composite index for each of the following combinations of properties and ancestors:

(`1`, `null`, `'this'`, `'GreatGrandpa'`) (`1`, `null`, `'this'`, `'Grandpa'`) (`1`, `null`, `'this'`, `'Dad'`) (`1`, `null`, `'this'`, `'Foo'`)

(`1`, `null`, `'that'`, `'GreatGrandpa'`) (`1`, `null`, `'that'`, `'Grandpa'`) (`1`, `null`, `'that'`, `'Dad'`) (`1`, `null`, `'that'`, `'Foo'`)

(`1`, `null`, `'theOther'`, `'GreatGrandpa'`) (`1`, `null`, `'theOther'`, `'Grandpa'`) (`1`, `null`, `'theOther'`, `'Dad'`) (`1`, `null`, `'theOther'`, `'Foo'`)

(`2`, `null`, `'this'`, `'GreatGrandpa'`) (`2`, `null`, `'this'`, `'Grandpa'`) (`2`, `null`, `'this'`, `'Dad'`) (`2`, `null`, `'this'`, `'Foo'`)

(`2`, `null`, `'that'`, `'GreatGrandpa'`) (`2`, `null`, `'that'`, `'Grandpa'`) (`2`, `null`, `'that'`, `'Dad'`) (`2`, `null`, `'that'`, `'Foo'`)

(`2`, `null`, `'theOther'`, `'GreatGrandpa'`) (`2`, `null`, `'theOther'`, `'Grandpa'`) (`2`, `null`, `'theOther'`, `'Dad'`) (`2`, `null`, `'theOther'`, `'Foo'`)

Storing this entity in Datastore now requires 1 + 1 + 4 + 2 + 6 + 24 = 38 writes.
