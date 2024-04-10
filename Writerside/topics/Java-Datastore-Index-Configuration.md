# Java Datastore Index Configuration

  

The App Engine Datastore uses [indexes](https://web.archive.org/web/20160424230308/https://cloud.google.com/appengine/docs/java/datastore/indexes) for every query your application makes. These indexes are updated whenever an entity changes, so the results can be returned quickly when the app makes a query. To do this, the datastore needs to know in advance which queries the application will make. You specify which indexes your app needs in a configuration file. The development server can generate the datastore index configuration automatically as you test your app.

1.  [About datastore-indexes.xml](#Java_About_datastore-indexes_xml)
2.  [Using automatic index configuration](#Java_Using_automatic_index_configuration)

## About datastore-indexes.xml

You specify configuration for datastore indexes in `WEB-INF/datastore-indexes.xml`, in your app's `war/` directory. This is an XML file whose root element is `<datastore-indexes>`. It contains zero or more `<datastore-index>` elements, one for each index that the Datastore should maintain.

As described on the [Datastore Indexes](https://web.archive.org/web/20160424230308/https://cloud.google.com/appengine/docs/java/datastore/indexes) page, an index is a table of values for a set of given properties for entities of a given kind. Each column of property values is sorted either in ascending or descending order. Configuration for an index specifies the kind of the entities, and the names of the properties and their sort orders.

Here is an example that specifies two indexes:

```
<?xml version="1.0" encoding="utf-8"?>
<datastore-indexes
  autoGenerate="true">
    <datastore-index kind="Employee" ancestor="false">
        <property name="lastName" direction="asc" />
        <property name="hireDate" direction="desc" />
    </datastore-index>
    <datastore-index kind="Project" ancestor="false">
        <property name="dueDate" direction="asc" />
        <property name="cost" direction="desc" />
    </datastore-index>
</datastore-indexes>
```

The `<datastore-indexes>` element has an `autoGenerate` attribute that controls whether this file should be considered along with automatically generated index configuration. See [Using Automatic Index Configuration](#Java_Using_automatic_index_configuration) below.

Each `<datastore-index>` element represents an index. The `kind` attribute specifies the kind of the entities to index. The `ancestor` attribute is `true` if the index supports queries that filter by ancestor-key to constrain results to a single entity group, `false` otherwise.

The `<property>` elements in a `<datastore-index>` represent the entity properties to index. The `name` attribute is the property name, and the `direction` attribute is the sort order, either `asc` for ascending or `desc` for descending. The order of the property elements specifies the order in the index: rows are sorted by the first property, then the second property, and so on.

## Using automatic index configuration

Determining the indexes required by your application's queries manually can be tedious and error-prone. Thankfully, the development server can determine the index configuration for you. To use automatic index configuration, add the attribute `autoGenerate="true"` to your `WEB-INF/datastore-indexes.xml` file's `<datastore-indexes>` element. Automatic index configuration is also used if your app does not have a `datastore-indexes.xml` file.

With automatic index configuration enabled, the development server maintains a file named `WEB-INF/appengine-generated/datastore-indexes-auto.xml` in your app's `war/` directory. When your app, running in the development server, attempts a datastore query for which there is no corresponding index in either `datastore-indexes.xml` or `datastore-indexes-auto.xml`, the server adds the appropriate configuration to `datastore-indexes-auto.xml`.

If automatic index configuration is enabled when you upload your application, AppCfg uses both `datastore-indexes.xml` and `datastore-indexes-auto.xml` to determine which indexes need to be built for your app in production.

If `autoGenerate="false"` is in your `datastore-indexes.xml`, the development server and `AppCfg` ignore the contents of `datastore-indexes-auto.xml`. If the app running locally performs a query whose index is not specified in `datastore-indexes.xml`, the development server throws an exception, just as the production Datastore would.

It's a good idea to occasionally move index configuration from `datastore-indexes-auto.xml` to `datastore-indexes.xml`, then disable automatic index configuration and test your app in the development server. This makes it easy to maintain indexes without having to manage two files, and ensures that your testing will reproduce errors caused by missing index configuration.
