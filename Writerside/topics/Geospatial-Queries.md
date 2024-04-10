# Geospatial Queries

  

This page describes how to use geospatial queries in Datastore. A geospatial query allows you to retrieve entities containing a `GeoPt` property whose latitude, longitude coordinates lie within a given geographic region.

**Alpha**

This is an Alpha release of Geospatial search for datastore. It is not covered by any SLA or deprecation policy and may be subject to backward-incompatible changes. It is not recommended for production use. Access to this feature is by invitation only.

1.  [Adding a GeoPt to an entity](#adding_a_geopt_to_an_entity)
2.  [Geospatial queries](#geospatial_queries_1)
3.  [Building an Index file](#building_an_index_file)
4.  [Known issues](#known_issues)

## Adding a GeoPt to an entity

To add a `GeoPt` property to an [entity](https://web.archive.org/web/20160424225630/https://cloud.google.com/appengine/docs/java/datastore/entities#Java_Creating_an_entity), use the [GeoPt](https://web.archive.org/web/20160424225630/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/GeoPt) class, specifying latitude and longitude:

```
Entity station = new Entity("GasStation");
station.setProperty("brand", "Ocean Ave Shell");
station.setProperty("location", new GeoPt(37.7913156f,-122.3926051f));
ds.put(station);
```

## Geospatial queries

Use the datastore query filter [StContainsFilter](https://web.archive.org/web/20160424225630/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/Query.StContainsFilter) to test if a `GeoPt` is contained in a given [GeoRegion](https://web.archive.org/web/20160424225630/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/Query.GeoRegion), for example:

```
// Testing for containment within a circle
GeoPt center = new GeoPt(latitude, longitude);
double radius = valueInMeters;
Filter f = new StContainsFilter("location", new Circle(center, radius));
Query q = new Query("Kind").setFilter(f);

// Testing for containment within a rectangle
GeoPt southwest = ...
GeoPt northeast = ...
Filter f = new StContainsFilter("location", new Rectangle(southwest, northeast));
Query q = new Query("Kind").setFilter(f);
```

You can combine geospatial containment terms with equality tests on any type of property:

```
Filter f = CompositeFilter.and(
    new StContainsFilter("location", new Circle(center, radius)),
    new FilterPredicate("brand", FilterOperator.EQUAL, value));
```

Geospatial queries must follow these rules:

-   The query must include at least one containment term.
-   The query may contain any number of non-geospatial terms, but these other terms may only test for EQUAL (not "less than", "greater than", etc.).
-   The query cannot use an [ancestor filter](https://web.archive.org/web/20160424225630/https://cloud.google.com/appengine/docs/java/datastore/queries#Java_Ancestor_queries).

## Building an Index file

As with other Datastore queries, the recommended development workflow is to rely on the automatic index configuration provided by the dev appserver. You implement queries in your code, and then execute all of them while running tests on the dev appserver. This causes the dev appserver to write an index configuration file defining the necessary indexes.

The form of an index definition that supports geospatial queries is a bit different from traditional composite indexes. For example:

```
<?xml version="1.0" encoding="utf-8"?>
<datastore-indexes>
  <datastore-index kind="GasStation">
    <property name="brand" />
    <property name="location" mode="geospatial" />
  </datastore-index>
</datastore-indexes>
```

If you build your own search index files to handle geospatial searches, be sure to follow these rules:

-   Add the `mode="geospatial"` attribute to properties that appear in geo-containment query terms.
-   Do not assign the `direction` attribute to *any* property in an index designed to handle geospatial searches.
-   Non-geospatial properties should be listed first, and each of the two groups of properties should appear in increasing alpha order by property name.
-   Do not assign the `ancestor` attribute to an index that contains a `mode="geospatial"` property.

One index configuration file can include multiple indexes of any kind.

Recall that "ordinary" (non-geospatial) queries require that an index exists that contains all the properties in the query and no others. This constraint is relaxed for geospatial queries. An index with more than one `mode="geospatial"` property can be used to perform a geospatial query that only uses a subset of the geospatial properties in that index. If a geospatial query includes other non-geospatial properties, the index must also contain those non-geo properties and no others, as the case with non-geospatial queries.

## Known issues

This is an alpha release of geospatial search. It has some limitations that will be removed in future releases:

-   Geospatial search is only available in the Java runtime, Python support is coming soon.
-   Geospatial search is only accessible using App Engine's Datastore API, there is no GQL support at this time.
-   [Cursors](https://web.archive.org/web/20160424225630/https://cloud.google.com/appengine/docs/java/datastore/queries#Java_Query_cursors) will not appear in results from geospatial queries. Calls to `QueryResultIterator.getCusor()` return `NULL`.
-   In the Google Cloud Platform Console, you can <a href="https://web.archive.org/web/20160424225630/https://console.cloud.google.com/project/_/datastore/indexes" data-track-metadata-end-goal="viewIndexes" data-track-metadata-position="body" data-track-name="consoleLink" data-track-type="tasks">view indexes that support geospatial queries</a>. However, property names with the `mode="geospatial"` attribute are not labeled as such, and they are (incorrectly) assigned a default descending sort direction.
-   It is not possible to run a geospatial query from the Google Cloud Platform Console.
