# Java Datastore API

  

[Python](https://web.archive.org/web/20160424225851/https://cloud.google.com/appengine/docs/python/datastore/ "View this page in the Python runtime") <span style="color: #ddd; padding: 0em .5em;">|</span><span style="color: black; font-weight:bold">Java</span> <span style="color: #ddd; padding: 0em .5em;">|</span><span style="color:lightgray" title="This page is not available in the PHP runtime">PHP</span> <span style="color: #ddd; padding: 0em .5em;">|</span>[Go](https://web.archive.org/web/20160424225851/https://cloud.google.com/appengine/docs/go/datastore/ "View this page in the Go runtime")

Related videos

[<img src="https://web.archive.org/web/20160424225851im_/https://cloud.google.com/cloud/images/stack_overflow_questions.png" title="Stack Overflow Questions" data-border="0" width="240" height="53" />](https://web.archive.org/web/20160424225851/http://stackoverflow.com/questions/tagged/google-app-engine+gae-datastore)

[](https://web.archive.org/web/20160424225851/http://stackoverflow.com/feeds/tag?sort=votes&tagnames=google-app-engine%2Bgae-datastore)

<span class="link gc-analytics-event" category="Sidebar" data-action="Stack Overflow question click"></span>

<a href="https://web.archive.org/web/20160424225851/http://stackoverflow.com/questions/tagged/google-app-engine+gae-datastore?sort=votes" class="gc-analytics-event" data-category="Sidebar" data-action="Stack Overflow See more link">See more...</a>

App Engine Datastore is a schemaless [NoSQL](https://web.archive.org/web/20160424225851/https://en.wikipedia.org/wiki/NoSQL) datastore providing robust, scalable storage for your web application, with the following features:

-   No planned downtime
-   Atomic transactions
-   High availability of reads and writes
-   Strong consistency for reads and ancestor queries
-   Eventual consistency for all other queries

You can access the Datastore using the low-level API described throughout the Datastore documentation, which provides direct access to all of Datastore's features, or you can use one of the higher-level open-source APIs for Datastore that provide ORM-like features and a more abstract experience, such as [Objectify](https://web.archive.org/web/20160424225851/https://github.com/objectify/objectify). Note that while the low-level Datastore API is provided and supported by Google, Objectify is provided by a third-party, and Google does not provide support for it.

The Datastore replicates data across multiple datacenters. This provides a high level of availability for reads and writes. Most queries are [eventually consistent](https://web.archive.org/web/20160424225851/https://en.wikipedia.org/wiki/Eventual_consistency).

The Datastore holds data objects known as *entities*. An entity has one or more *properties*, named values of one of several supported data types: for instance, a property can be a string, an integer, or a reference to another entity. Each entity is identified by its *kind*, which categorizes the entity for the purpose of queries, and a *key* that uniquely identifies it within its kind. The Datastore can execute multiple operations in a single *transaction*. By definition, a transaction cannot succeed unless every one of its operations succeeds; if any of the operations fails, the transaction is automatically rolled back. This is especially useful for distributed web applications, where multiple users may be accessing or manipulating the same data at the same time.

## Contents

1.  [Comparison with traditional databases](#Java_Comparison_with_traditional_databases)
2.  [Entities](#Java_Entities)
    1.  [Kinds, keys, and identifiers](#Java_Kinds_keys_and_identifiers)
    2.  [Ancestor paths](#Java_Ancestor_paths)
3.  [Queries and indexes](#Java_Queries_and_indexes)
4.  [Transactions](#Java_Transactions)
    1.  [Transactions and entity groups](#Java_Transactions_and_entity_groups)
    2.  [Cross-group transactions](#Java_Cross_group_transactions)
5.  [Datastore writes and data visibility](#Java_Datastore_writes_and_data_visibility)
6.  [Datastore statistics](#Java_Datastore_statistics)
7.  [Quotas and limits](#Java_Quotas_and_limits)

## Comparison with traditional databases

Unlike traditional relational databases, the Datastore uses a distributed architecture to automatically manage scaling to very large data sets. While the Datastore interface has many of the same features as traditional databases, it differs from them in the way it describes relationships between data objects. Entities of the same kind can have different properties, and different entities can have properties with the same name but different value types.

These unique characteristics imply a different way of designing and managing data to take advantage of the ability to scale automatically. In particular, the Datastore differs from a traditional [relational database](https://web.archive.org/web/20160424225851/https://en.wikipedia.org/wiki/Relational_database) in the following important ways:

-   The Datastore is designed to scale, allowing applications to maintain high performance as they receive more traffic:
    -   Datastore writes scale by automatically distributing data as necessary.
    -   Datastore reads scale because the only queries supported are those whose performance scales with the size of the result set (as opposed to the data set). This means that a query whose result set contains 100 entities performs the same whether it searches over a hundred entities or a million. This property is the key reason some types of queries are not supported.
-   Because all queries are served by pre-built [indexes](#Java_Queries_and_indexes), the types of queries that can be executed are more restrictive than those allowed on a relational database with SQL. In particular, the following are not supported:
    -   Join operations
    -   Inequality filtering on multiple properties
    -   Filtering of data based on results of a subquery
-   Unlike traditional relational databases, the Datastore doesn't require entities of the same kind to have a consistent property set (although you can choose to enforce such a requirement in your own application code).

## Entities

Objects in the Datastore are known as *entities*. An entity has one or more named *properties*, each of which can have one or more values. Property values can belong to [a variety of data types](https://web.archive.org/web/20160424225851/https://cloud.google.com/appengine/docs/java/datastore/entities#Java_Properties_and_value_types), including integers, floating-point numbers, strings, dates, and binary data, among others. A query on a property with multiple values tests whether any of the values meets the query criteria. This makes such properties useful for membership testing.

**Note:** Datastore entities are *schemaless*: unlike traditional relational databases, the Datastore does not require that all entities of a given kind have the same properties or that all of an entity's values for a given property be of the same data type. If a formal schema is needed, the application itself is responsible for ensuring that entities conform to it.

### Kinds, keys, and identifiers

Each Datastore entity is of a particular *kind,* which categorizes the entity for the purpose of queries; for instance, a human resources application might represent each employee at a company with an entity of kind `Employee`. In addition, each entity has its own *key*, which uniquely identifies it. The key consists of the following components:

-   The entity's kind
-   An *identifier*, which can be either
    -   a *key name* string
    -   an integer *ID*
-   An optional [*ancestor path*](#Java_Ancestor_paths) locating the entity within the Datastore hierarchy

The identifier is assigned when the entity is created. Because it is part of the entity's key, it is associated permanently with the entity and cannot be changed. It can be assigned in either of two ways:

-   Your application can specify its own key name string for the entity.
-   You can have the Datastore automatically assign the entity an integer numeric ID.

**Note:** Instead of using key name strings or generating numeric IDs automatically, advanced applications may sometimes wish to assign their own numeric IDs manually to the entities they create. Be aware, however, that if you choose this option you must take special steps to prevent your manually assigned numeric IDs from conflicting with those assigned automatically by the Datastore; see the [Entities, Properties, and Keys](https://web.archive.org/web/20160424225851/https://cloud.google.com/appengine/docs/java/datastore/entities#Java_Kinds_and_identifiers) page for further details.

### Ancestor paths

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

<img src="https://web.archive.org/web/20160424225851im_/https://cloud.google.com/images/products/clouddatastore/entity-group.png" width="450" />

## Queries and indexes

In addition to retrieving entities from the Datastore directly by their keys, an application can perform a *[query](https://web.archive.org/web/20160424225851/https://cloud.google.com/appengine/docs/java/datastore/queries)* to retrieve them by the values of their properties. The query operates on entities of a given *[kind](#Java_Kinds_keys_and_identifiers)*; it can specify *[filters](https://web.archive.org/web/20160424225851/https://cloud.google.com/appengine/docs/java/datastore/queries#Java_Filters)* on the entities' property values, keys, and ancestors, and can return zero or more entities as *results.* A query can also specify *[sort orders](https://web.archive.org/web/20160424225851/https://cloud.google.com/appengine/docs/java/datastore/queries#Java_Sort_orders)* to sequence the results by their property values. The results include all entities that have at least one (possibly null) value for every property named in the filters and sort orders, and whose property values meet all the specified filter criteria. The query can return entire entities, [projected entities](https://web.archive.org/web/20160424225851/https://cloud.google.com/appengine/docs/java/datastore/projectionqueries), or just entity [keys](https://web.archive.org/web/20160424225851/https://cloud.google.com/appengine/docs/java/datastore/#Java_Kinds_keys_and_identifiers).

A typical query includes the following:

-   An *[entity kind](#Java_Kinds_keys_and_identifiers)* to which the query applies
-   Zero or more *[filters](https://web.archive.org/web/20160424225851/https://cloud.google.com/appengine/docs/java/datastore/queries#Java_Filters)* based on the entities' property values, keys, and ancestors
-   Zero or more *[sort orders](https://web.archive.org/web/20160424225851/https://cloud.google.com/appengine/docs/java/datastore/queries#Java_Sort_orders)* to sequence the results

When executed, the query retrieves all entities of the given kind that satisfy all of the given filters, sorted in the specified order. Queries execute as read-only.

**Note:** To conserve memory and improve performance, a query should, whenever possible, specify a limit on the number of results returned.

A query can also include an [*ancestor filter*](https://web.archive.org/web/20160424225851/https://cloud.google.com/appengine/docs/java/datastore/queries#Java_Ancestor_filters) limiting the results to just the entity group descended from a specified ancestor. Such a query is known as an [*ancestor query*](https://web.archive.org/web/20160424225851/https://cloud.google.com/appengine/docs/java/datastore/queries#Java_Ancestor_queries). By default, ancestor queries return [*strongly consistent*](https://web.archive.org/web/20160424225851/https://en.wikipedia.org/wiki/Strong_consistency) results, which are guaranteed to be up to date with the latest changes to the data. Non-ancestor queries, by contrast, can span the entire Datastore rather than just a single entity group, but are only [*eventually consistent*](https://web.archive.org/web/20160424225851/https://en.wikipedia.org/wiki/Eventual_consistency) and may return stale results. If strong consistency is important to your application, you may need to take this into account when structuring your data, placing related entities in the same entity group so they can be retrieved with an ancestor rather than a non-ancestor query; see [Structuring Data for Strong Consistency](https://web.archive.org/web/20160424225851/https://cloud.google.com/appengine/docs/java/datastore/structuring_for_strong_consistency) for more information.

Every Datastore query computes its results using one or more *[indexes](https://web.archive.org/web/20160424225851/https://cloud.google.com/appengine/docs/java/datastore/indexes),* which contain entity keys in a sequence specified by the index's properties and, optionally, the entity's ancestors. The indexes are updated incrementally to reflect any changes the application makes to its entities, so that the correct results of all queries are available with no further computation needed.

App Engine predefines a simple index on each property of an entity. An App Engine application can define further custom indexes in an *[index configuration file](https://web.archive.org/web/20160424225851/https://cloud.google.com/appengine/docs/java/config/indexconfig)* named `datastore-indexes.xml`, which is generated in your application's `/war/WEB-INF/appengine-generated` directory. The development server automatically adds suggestions to this file as it encounters queries that cannot be executed with the existing indexes. You can tune indexes manually by editing the file before uploading the application.

**Note:** The index-based query mechanism supports a wide range of queries and is suitable for most applications. However, it does not support some kinds of query common in other database technologies: in particular, joins and aggregate queries aren't supported within the Datastore query engine. See the [Datastore Queries](https://web.archive.org/web/20160424225851/https://cloud.google.com/appengine/docs/java/datastore/queries#Java_Restrictions_on_queries) page for limitations on Datastore queries.

## Transactions

Every attempt to insert, update, or delete an entity takes place in the context of a [*transaction*](https://web.archive.org/web/20160424225851/https://cloud.google.com/appengine/docs/java/datastore/transactions). A single transaction can include any number of such operations. To maintain the consistency of the data, the transaction ensures that all of the operations it contains are applied to the Datastore as a unit or, if any of the operations fails, that none of them are applied.

You can perform multiple actions on an entity within a single transaction. For example, to increment a counter field in an object, you need to read the value of the counter, calculate the new value, and then store it back. Without a transaction, it is possible for another process to increment the counter between the time you read the value and the time you update it, causing your application to overwrite the updated value. Doing the read, calculation, and write in a single transaction ensures that no other process can interfere with the increment.

### Transactions and entity groups

Only ancestor queries are allowed within a transaction: that is, each transactional query must be limited to a single entity group. The transaction itself can apply to multiple entities, which can belong either to a single entity group or (in the case of a [*cross-group transaction*](#Java_Cross_group_transactions)) to as many as twenty-five different entity groups.

The Datastore uses [*optimistic concurrency*](https://web.archive.org/web/20160424225851/https://en.wikipedia.org/wiki/Optimistic_concurrency) to manage transactions. When two or more transactions try to change the same entity group at the same time (either updating existing entities or creating new ones), the first transaction to commit will succeed and all others will fail on commit. These other transactions can then be retried on the updated data. Note that this limits the number of concurrent writes you can do to any entity in a given entity group.

### Cross-group transactions

A transaction on entities belonging to different entity groups is called a *cross-group (XG) transaction*. The transaction can be applied across a maximum of twenty-five entity groups, and will succeed as long as no concurrent transaction touches any of the entity groups to which it applies. This gives you more flexibility in organizing your data, because you aren't forced to put disparate pieces of data under the same ancestor just to perform atomic writes on them.

As in a single-group transaction, you cannot perform a non-ancestor query in an XG transaction. You can, however, perform ancestor queries on separate entity groups. Nontransactional (non-ancestor) queries may see all, some, or none of the results of a previously committed transaction. (For background on this issue, see [Datastore Writes and Data Visibility](#Java_Datastore_writes_and_data_visibility).) However, such nontransactional queries are more likely to see the results of a partially committed XG transaction than those of a partially commited single-group transaction.

**Note:** The first read of an entity group in an XG transaction may throw a [`ConcurrentModificationException`](https://web.archive.org/web/20160424225851/http://download.oracle.com/javase/6/docs/api/java/util/ConcurrentModificationException.html) if there is a conflict with other transactions accessing that same entity group. This means that even an XG transaction that performs only reads can fail with a concurrency exception.

An XG transaction that touches only a single entity group has exactly the same performance and cost as a single-group, non-XG transaction. In an XG transaction that touches multiple entity groups, operations cost the same as if they were performed in a non-XG transaction, but may experience higher latency.

## Datastore writes and data visibility

Data is written to the Datastore in two phases:

1.  In the Commit phase, the entity data is recorded in the transaction logs of a majority of replicas, and any replicas in which it was not recorded are marked as not having up-to-date logs.
2.  The Apply phase occurs independently in each replica, and consists of two actions performed in parallel:
    -   The entity data is written in that replica.
    -   The index rows for the entity are written in that replica. (Note that this can take longer than writing the data itself.)

The write operation returns immediately after the Commit phase and the Apply phase then takes place asynchronously, possibly at different times in each replica, and possibly with delays of a few hundred milliseconds or more from the completion of the Commit phase. If a failure occurs during the Commit phase, there are automatic retries; but if failures continue, the Datastore returns an error message that your application receives as an exception. If the Commit phase succeeds but the Apply fails in a particular replica, the Apply is rolled forward to completion in that replica when one of the following occurs:

-   Periodic Datastore sweeps check for uncompleted Commit jobs and apply them.
-   Certain operations (`get`, `put`, `delete`, and ancestor queries) that use the affected entity group cause any changes that have been committed but not yet applied to be completed in the replica in which they are executing before proceeding with the new operation.

This write behavior can have several implications on how and when data is visible to your application at different parts of the Commit and Apply phases:

-   If a write operation reports a timeout error, it cannot be determined (without attempting to read the data) whether the operation succeeded or failed.
-   Because Datastore gets and ancestor queries apply any outstanding modifications to the replica on which they are executing, these operations always see a consistent view of all previous successful transactions. This means that a `get` operation (looking up an updated entity by its key) is guaranteed to see the latest version of that entity.
-   Non-ancestor queries may return stale results because they may be executing on a replica on which the latest transactions have not yet been applied. This can occur even if an operation was performed that is guaranteed to apply outstanding transactions because the query may execute on a different replica than the previous operation.
-   The timing of concurrent changes may affect the results of non-ancestor queries. If an entity initially satisfies a query but is later changed so that it no longer does, the entity may still be included in the query's result set if the changes had not yet been applied to the indexes in the replica on which the query was executed.

## Datastore statistics

The Datastore maintains statistics about the data stored for an application, such as how many entities there are of a given kind or how much space is used by property values of a given type. You can view these statistics in the Cloud Platform Console [Cloud Datastore dashboard](https://web.archive.org/web/20160424225851/https://console.cloud.google.com/project/_/datastore/stats) page. You can also use the Datastore API to access these values programmatically from within the application by querying for specially named entities; see [Datastore Statistics in Java](https://web.archive.org/web/20160424225851/https://cloud.google.com/appengine/docs/java/datastore/stats) for more information.

## Quotas and limits

Various aspects of your application's Datastore usage are counted toward your resource quotas:

-   Data sent to the Datastore by the application counts toward the **Data Sent to Datastore API** quota.
-   Data received by the application from the Datastore counts toward the **Data Received from Datastore API** quota.
-   The total amount of data currently stored in the Datastore for the application cannot exceed the **[Stored Data (billable)](https://web.archive.org/web/20160424225851/https://cloud.google.com/appengine/docs/quotas#Storage)** quota. This includes all entity properties and keys, as well as the indexes needed to support querying those entities. See the article [How Entities and Indexes Are Stored](https://web.archive.org/web/20160424225851/https://cloud.google.com/appengine/articles/storage_breakdown) for a complete breakdown of the metadata required to store entities and indexes at the Bigtable level.

For information on systemwide safety limits, see the [Quotas and Limits](https://web.archive.org/web/20160424225851/https://cloud.google.com/appengine/docs/quotas#Safety_Quotas_and_Billable_Quotas) page and the quota details page in the [Cloud Platform Console](https://web.archive.org/web/20160424225851/https://cloud.google.com/appengine/docs/developers-console/#quotas). In addition to such system-wide limits, the following limits apply specifically to the use of the Datastore:

<table>
<thead>
<tr class="header">
<th>Limit</th>
<th>Amount</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Maximum entity size</td>
<td>1 megabyte</td>
</tr>
<tr class="even">
<td>Maximum transaction size</td>
<td>10 megabytes</td>
</tr>
<tr class="odd">
<td>Maximum number of <a href="https://web.archive.org/web/20160424225851/https://cloud.google.com/appengine/docs/python/datastore/indexes#Python_Index_limits">index entries</a> for an entity</td>
<td>20000</td>
</tr>
<tr class="even">
<td>Maximum <a href="https://web.archive.org/web/20160424225851/https://cloud.google.com/appengine/docs/python/datastore/indexes#Python_Index_limits">number of bytes in composite indexes</a> for an entity</td>
<td>2 megabytes</td>
</tr>
</tbody>
</table>
