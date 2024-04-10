# Datastore Statistics in Java

  

Datastore maintains statistics about the data stored for an application, such as how many entities there are of a given kind, or how much space is used by property values of a given type. You can view these statistics in the Cloud Platform Console, in the <a href="https://web.archive.org/web/20160424230854/https://console.cloud.google.com/project/_/datastore/stats" target="console" data-track-metadata-end-goal="viewStats" data-track-metadata-position="body" data-track-name="consoleLink" data-track-type="concepts">Dashboard</a> page.

You can also access these values programmatically within the application by querying for specially named entites using the Datastore API. Each statistic is accessible as an entity whose kind name begins and ends with two underscores. For example, each app has exactly one entity of the kind `__Stat_Total__` that represents statistics about all of the entities in Datastore in total. Each statistic entity has the following properties:

-   `count`, the number of items considered by the statistic (a long integer)
-   `bytes`, the total size of the items for this statistic (a long integer)
-   `timestamp`, the time of the most recent update to the statistic (a date-time value)

Some statistic kinds also have additional properties, listed below.

A Java application can access statistic entities using the low-level Datastore API. For example:

```
import com.google.appengine.api.datastore.DatastoreService;
import com.google.appengine.api.datastore.DatastoreServiceFactory;
import com.google.appengine.api.datastore.Entity;
import com.google.appengine.api.datastore.Query;

// ...
DatastoreService datastore = DatastoreServiceFactory.getDatastoreService();
Entity globalStat = datastore.prepare(new Query("__Stat_Total__")).asSingleEntity();
Long totalBytes = (Long) globalStat.getProperty("bytes");
Long totalEntities = (Long) globalStat.getProperty("count");
```

When the statistics system creates new statistic entities, it does not delete the old ones right away. The best way to get a consistent view of the statistics is to query for the `__Stat_Total__` entity with the most recent `timestamp`, then use that timestamp value as a filter when fetching other statistic entities.

The statistic entities are included in the calculated statistic values. Statistic entities take up space relative to the number of unique kinds and property names used by the application.

The statistics system will also create statistics specific to each [namespace](https://web.archive.org/web/20160424230854/https://cloud.google.com/appengine/docs/java/multitenancy/overview). Note that if an application does not use Datastore namespaces then namespace specific statistics will not be created. Namespace specific stats are found in the namespace that they're specific to. The kind names for namespace specific stats are prefixed with `__Stat_Ns_` and have the same corresponding suffix as application wide statistics kinds.

Applications with thousands of namespaces, kinds, or property names require a very large number of statistics entities. To keep the overhead of storing and updating the statistics reasonable, Datastore progressively drops statistics entities, in the following order:

-   per-namespace, per-kind, and per-property statistics: `__Stat_Ns_PropertyName_Kind__`, `__Stat_Ns_PropertyType_PropertyName_Kind__`
-   per-kind and per-property statistics: `__Stat_PropertyName_Kind__`, `__Stat_PropertyType_PropertyName_Kind__`
-   per-namespace and per-kind statistics: `__Stat_Ns_Kind__`, `__Stat_Ns_Kind_IsRootEntity__`, `__Stat_Ns_Kind_NotRootEntity__`, `__Stat_Ns_PropertyType_Kind__`
-   per-kind statistics: `__Stat_Kind__`, `__Stat_Kind_IsRootEntity__`, `__Stat_Kind_NotRootEntity__`, `__Stat_PropertyType_Kind__`
-   per-namespace statistics: `__Stat_Namespace__`, `__Stat_Ns_Kind_CompositeIndex__`, `__Stat_Ns_PropertyType__`, `__Stat_Ns_Total__`

The summary statistics entities (`__Stat_Kind_CompositeIndex__`, `__Stat_PropertyType__`, `__Stat_Total__`) are never dropped.

The complete list of available statistics is as follows:

<table>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<thead>
<tr class="header">
<th>Statistic</th>
<th>Stat Entity Kind</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>all entities</td>
<td><code>__Stat_Total__</code><br />
Namespace specific entry:<br />
<code>__Stat_Ns_Total__</code></td>
<td>All entities. Additional properties:<br />
<br />
• <code>entity_bytes</code>: The storage in the entities table measured in bytes.<br />
• <code>builtin_index_bytes</code>: The storage in built-in index entries measured in bytes.<br />
• <code>builtin_index_count</code>: the count of built-in index entries.<br />
• <code>composite_index_bytes</code>: The storage in composite index entries measured in bytes.<br />
• <code>composite_index_count</code>: The count of composite index entries.</td>
</tr>
<tr class="even">
<td>all entities in a namespace</td>
<td><code>__Stat_Namespace__</code><br />
Note that <code>__Stat_Namespace__</code> entities are created for each namespace encountered and are only found in the empty string namespace.</td>
<td>All entities in a namespace.<br />
<br />
• <code>subject_namespace</code>, the namespace represented (a string)<br />
• <code>entity_bytes</code>: The storage in the entities table measured in bytes.<br />
• <code>builtin_index_bytes</code>: The storage in built-in index entries measured in bytes.<br />
• <code>builtin_index_count</code>: the count of built-in index entries.<br />
• <code>composite_index_bytes</code>: The storage in composite index entries measured in bytes.<br />
• <code>composite_index_count</code>: The count of composite index entries.</td>
</tr>
<tr class="odd">
<td>all entries in application defined indexes</td>
<td><code>__Stat_Kind_CompositeIndex__</code><br />
Namespace specific entry: <code>__Stat_Ns_Kind_CompositeIndex__</code><br />
</td>
<td>Entries in the composite index table; one stat entity for each kind of entity stored. Additional properties:<br />
<br />
• <code>index_id</code>, the index id.<br />
• <code>kind_name</code>, the name of the kind represented (a string)</td>
</tr>
<tr class="even">
<td>entities of a kind</td>
<td><code>__Stat_Kind__</code><br />
Namespace specific entry:<br />
<code>__Stat_Ns_Kind__</code></td>
<td>Entities of a kind; one stat entity for each kind of entity stored. Additional properties:<br />
<br />
• <code>kind_name</code>, the name of the kind represented (a string)<br />
• <code>entity_bytes</code>: The storage in the entities table measured in bytes.<br />
• <code>builtin_index_bytes</code>: The storage in built-in index entries measured in bytes.<br />
• <code>builtin_index_count</code>: the count of built-in index entries.<br />
• <code>composite_index_bytes</code>: The storage in composite index entries measured in bytes.<br />
• <code>composite_index_count</code>: The count of composite index entries.</td>
</tr>
<tr class="odd">
<td>root entities of a kind</td>
<td><code>__Stat_Kind_IsRootEntity__</code><br />
Namespace specific entry:<br />
<code>__Stat_Ns_Kind_IsRootEntity__</code></td>
<td>Entities of a kind that are entity group root entities (have no ancestor parent); one stat entity for each kind of entity stored. Additional properties:<br />
<br />
• <code>kind_name</code>, the name of the kind represented (a string)<br />
• <code>entity_bytes</code>: The storage in the entities table measured in bytes.</td>
</tr>
<tr class="even">
<td>non-root entities of a kind</td>
<td><code>__Stat_Kind_NotRootEntity__</code><br />
Namespace specific entry:<br />
<code>__Stat_Ns_Kind_NotRootEntity__</code></td>
<td>Entities of a kind that are not entity group root entities (have an ancestor parent); one stat entity for each kind of entity stored. Additional properties:<br />
<br />
• <code>kind_name</code>, the name of the kind represented (a string)<br />
• <code>entity_bytes</code>: The storage in the entities table measured in bytes.</td>
</tr>
<tr class="odd">
<td>properties of a type</td>
<td><code>__Stat_PropertyType__</code><br />
Namespace specific entry:<br />
<code>__Stat_Ns_PropertyType__</code></td>
<td>Properties of a value type across all entities; one stat entity per value type. Additional properties:<br />
<br />
• <code>property_type</code>, the name of the value type (a string)<br />
• <code>entity_bytes</code>: The storage in the entities table measured in bytes.<br />
• <code>builtin_index_bytes</code>: The storage in built-in index entries measured in bytes.<br />
• <code>builtin_index_count</code>: the count of built-in index entries.</td>
</tr>
<tr class="even">
<td>properties of a type per kind</td>
<td><code>__Stat_PropertyType_Kind__</code><br />
Namespace specific entry:<br />
<code>__Stat_Ns_PropertyType_Kind__</code></td>
<td>Properties of a value type across entities of a given kind; one stat entity per combination of property type and kind. Additional properties:<br />
<br />
• <code>property_type</code>, the name of the value type (a string)<br />
• <code>kind_name</code>, the name of the kind represented (a string)<br />
• <code>entity_bytes</code>: The storage in the entities table measured in bytes.<br />
• <code>builtin_index_bytes</code>: The storage in the built-in index measured in bytes.<br />
• <code>builtin_index_count</code>: the count of built-in index entries.</td>
</tr>
<tr class="odd">
<td>properties with a name</td>
<td><code>__Stat_PropertyName_Kind__</code><br />
Namespace specific entry:<br />
<code>__Stat_Ns_PropertyName_Kind__</code></td>
<td>Properties with a given name across entities of a given kind; one stat entity per combination of unique property name and kind. Additional properties:<br />
<br />
• <code>property_name</code>, the name of the property (a string)<br />
• <code>kind_name</code>, the name of the kind represented (a string)<br />
• <code>entity_bytes</code>: The storage in the entities table measured in bytes.<br />
• <code>builtin_index_bytes</code>: The storage in built-in index entries measured in bytes.<br />
• <code>builtin_index_count</code>: the count of built-in index entries.</td>
</tr>
<tr class="even">
<td>properties of a type and with a name</td>
<td><code>__Stat_PropertyType_PropertyName_Kind__</code><br />
Namespace specific entry:<br />
<code>__Stat_Ns_PropertyType_PropertyName_Kind__</code></td>
<td>Properties with a given name and of a given value type across entities of a given kind; one stat entity per combination of property name, value type and kind that exists in Datastore. Additional properties:<br />
<br />
• <code>property_type</code>, the name of the value type (a string)<br />
• <code>property_name</code>, the name of the property (a string)<br />
• <code>kind_name</code>, the name of the kind represented (a string)<br />
• <code>entity_bytes</code>: The storage in the entities table measured in bytes.<br />
• <code>builtin_index_bytes</code>: The storage in built-in index entries measured in bytes.<br />
• <code>builtin_index_count</code>: the count of built-in index entries.</td>
</tr>
</tbody>
</table>

Some statistics refer to Datastore property value types by name, as strings. These names are as follows:

-   `"Blob"`
-   `"BlobKey"`
-   `"Boolean"`
-   `"Category"`
-   `"Date/Time"`
-   `"Email"`
-   `"Float"`
-   `"GeoPt"`
-   `"IM"`
-   `"Integer"`
-   `"Key"`
-   `"Link"`
-   `"NULL"`
-   `"PhoneNumber"`
-   `"PostalAddress"`
-   `"Rating"`
-   `"ShortBlob"`
-   `"String"`
-   `"Text"`
-   `"User"`

**Note:** `__Stat_Namespace__` entities contain the same information found in `__Stat_Ns_Total__` records. `__Stat_Namespace__` entities are stored in the empty namespace and contain a `subject_namespace` field describing the namespace to which they belong. `__Stat_Ns_Total__` records are stored in the namespace to which they refer, and thus do not contain a `subject_namespace` field. Hence, a query on kind `__Stat_Namespace__` (from the empty string namespace) ordered descending by `bytes` will list the namespaces that consume the largest storage first. Since queries across namespaces are not possible, any query for `__Stat_Ns_Total__` entities will only ever produce at most a single record.
