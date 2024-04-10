# Java Index Configuration Using YAML

  

The App Engine datastore uses [indexes](https://web.archive.org/web/20160424225359/https://cloud.google.com/appengine/docs/java/datastore/indexes) for every query your application makes. These indexes are updated whenever an entity changes, so the results can be returned quickly when the app makes a query. To do this, the datastore needs to know in advance which queries the application will make. You specify which indexes your app needs in a configuration file. The development server can generate the datastore index configuration automatically as you test your app.

1.  [About index.yaml](#About_index_yaml)
2.  [Index Definitions](#Index_Definitions)
3.  [Automatic Indexes](#Automatic_and_Manual_Indexes)

## About index.yaml

Every datastore query made by an application needs a corresponding index. Indexes for simple queries, such as queries over a single property, are created automatically. Indexes for complex queries must be defined in a configuration file named `index.yaml`. This file is stored in the application's `WEB-INF` directory to create indexes in the datastore.

If a query needs an index that does not have an appropriate entry in the configuration file, the [development web server](https://web.archive.org/web/20160424225359/https://cloud.google.com/appengine/docs/java/tools/devserver) automatically adds items to this file when the application tries to execute the query. You can adjust indexes or create new ones manually by editing the file.

**Tip:** If, during testing, the app exercises every query it will make using the development web server, then the generated entries in `index.yaml` will be complete. You only need to edit the file manually to delete indexes that are no longer used, or to define indexes not created by the development web server.

For more information about indexes, see the [Datastore Indexes](https://web.archive.org/web/20160424225359/https://cloud.google.com/appengine/docs/java/datastore/indexes) page.

The following is an example of an `index.yaml` file:

```
indexes:

- kind: Cat
  ancestor: no
  properties:
  - name: name
  - name: age
    direction: desc

- kind: Cat
  properties:
  - name: name
    direction: asc
  - name: whiskers
    direction: desc

- kind: Store
  ancestor: yes
  properties:
  - name: business
    direction: asc
  - name: owner
    direction: asc
```

The syntax of `index.yaml` is the YAML format. For more information about this syntax, see [the YAML website](https://web.archive.org/web/20160424225359/http://www.yaml.org/) for more information.

**Tip:** The YAML format supports comments. A line that begins with a pound (`#`) character is ignored:  
`# This is a comment.`

### Index Definitions

`index.yaml` has a single list element called `indexes`. Each element in the list represents an index for the application.

An index element can have the following elements:

`kind`  
The kind of the entity for the query. The kind attribute specifies the kind of the entities to index.

`properties`  
A list of properties to include as columns of the index, in the order to be sorted: properties used in equality filters first, followed by the property used in inequality filters, then the sort orders and their directions.

Each element in this list has the following elements:

`name`  
The datastore name of the property.

`direction`  
The direction to sort, either `asc` for ascending or `desc` for descending. This is only required for properties used in sort orders of the query, and must match the direction used by the query. The default is `asc`.

`ancestor`  
The ancestor attribute is `true` if the index supports a query that filters entities by the entity group parent, `false` otherwise.

### Automatic Indexes

Determining the indexes required by your application's queries manually can be tedious and error-prone. Thankfully, when you use `index.yaml`, the development server automatically determines the index configuration for you.

The development server maintains the auto-generated index in a file named `datastore-indexes-auto.xml` in the directory `WEB-INF/appengine-generated/` in your application's WAR. AppCfg uses both `index.yaml` and `datastore-indexes-auto.xml` to determine which indexes need to be built for your app on App Engine.

If you have an app running in the development server, and it attempts a datastore query for which there is no corresponding index in either `index.yaml` or `datastore-indexes-auto.xml`, then the server adds the appropriate configuration to `datastore-indexes-auto.xml`.
