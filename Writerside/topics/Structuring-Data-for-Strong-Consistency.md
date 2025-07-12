# Structuring Data for Strong Consistency

  

The Google App Engine High Replication Datastore (HRD) provides high availability, scalability and durability by distributing data over many machines and using masterless, synchronous replication over a wide geographic area. However, there is a tradeoff in this design, which is that the write throughput for any single [*entity group*](https://web.archive.org/web/20160424231001/https://cloud.google.com/appengine/docs/java/datastore/entities#Java_Ancestor_paths) is limited to about one commit per second, and there are limitations on queries or transactions that span multiple entity groups. This page describes these limitations in more detail and discusses best practices for structuring your data to support strong consistency while still meeting your application's write throughput requirements.

Strongly-consistent reads always return current data, and, if performed within a transaction, will appear to come from a single, consistent snapshot. However, queries must specify an ancestor filter in order to be strongly-consistent or participate in a transaction, and transactions can involve at most 25 entity groups. Eventually-consistent reads do not have those limitations, and are adequate in many cases. Using eventually-consistent reads may allow you to distribute your data among a larger number of entity groups, enabling you to obtain greater write throughput by executing commits in parallel on the different entity groups. But, you need to understand the characteristics of eventually-consistent reads in order to determine whether they are suitable for your application:

-   The results from these reads may not reflect the latest transactions. This can occur because these reads do not ensure that the replica they are running on is up-to-date. Instead, they use whatever data is available on that replica at the time of query execution. Replication latency is almost always less than a few seconds.
-   A committed transaction that spanned multiple entities may appear to have been applied to some of the entities and not others. Note, though, that a transaction will never appear to have been partially applied within a single entity.
-   The query results may include entities that should not have been included according to the filter criteria, and may exclude entities that should have been included. This can occur because indexes may be read at a different version than the entity itself is read at.

To understand how to structure your data for strong consistency, compare two different approaches for the [`guestbook` example application](https://web.archive.org/web/20160424231001/http://code.google.com/p/googleappengine/source/browse/trunk/java/demos/guestbook/src/guestbook/SignGuestbookServlet.java) from the App Engine [Getting Started](https://web.archive.org/web/20160424231001/https://cloud.google.com/appengine/docs/java/gettingstarted/) exercise. The first approach creates a new root entity for each greeting:

```
import com.google.appengine.api.datastore.Entity;

Entity greeting = new Entity("Greeting");
// No parent key specified, so Greeting is a root entity.

greeting.setProperty("user", user);
greeting.setProperty("date", date);
greeting.setProperty("content", content);
```

It then queries on the entity kind `Greeting` for the ten most recent greetings.

```
import com.google.appengine.api.datastore.DatastoreService;
import com.google.appengine.api.datastore.DatastoreServiceFactory;
import com.google.appengine.api.datastore.Entity;

DatastoreService datastore = DatastoreServiceFactory.getDatastoreService();

Query query = new Query("Greeting")
                    .addSort("date", Query.SortDirection.DESCENDING);

List<Entity> greetings = datastore.prepare(query)
                                  .asList(FetchOptions.Builder.withLimit(10));
```

However, because we are using a non-ancestor query, the replica used to perform the query in this scheme may not have seen the new greeting by the time the query is executed. Nonetheless, nearly all writes will be available for non-ancestor queries within a few seconds of commit. For many applications, a solution that provides the results of a non-ancestor query in the context of the current user's own changes will usually be sufficient to make such replication latencies completely acceptable.

If strong consistency is important to your application, an alternate approach is to write entities with an ancestor path that identifies the same root entity across all entities that must be read in a single, strongly-consistent ancestor query:

```
import com.google.appengine.api.datastore.DatastoreService;
import com.google.appengine.api.datastore.DatastoreServiceFactory;
import com.google.appengine.api.datastore.Entity;

String guestbookName = req.getParameter("guestbookName");
Key guestbookKey = KeyFactory.createKey("Guestbook", guestbookName);
String content = req.getParameter("content");
Date date = new Date();

// Place greeting in same entity group as guestbook
Entity greeting = new Entity("Greeting", guestbookKey);
greeting.setProperty("user", user);
greeting.setProperty("date", date);
greeting.setProperty("content", content);
```

You will then be able to perform a strongly-consistent ancestor query within the entity group identified by the common root entity:

```
import com.google.appengine.api.datastore.DatastoreService;
import com.google.appengine.api.datastore.DatastoreServiceFactory;
import com.google.appengine.api.datastore.Entity;

DatastoreService datastore = DatastoreServiceFactory.getDatastoreService();

Key guestbookKey = KeyFactory.createKey("Guestbook", guestbookName);
Query query = new Query("Greeting", guestbookKey)
                    .setAncestor(guestbookKey)
                    .addSort("date", Query.SortDirection.DESCENDING);

List<Entity> greetings = datastore.prepare(query)
                                  .asList(FetchOptions.Builder.withLimit(10));
```

This approach achieves strong consistency by writing to a single entity group per guestbook, but it also limits changes to the guestbook to no more than 1 write per second (the supported limit for entity groups). If your application is likely to encounter heavier write usage, you may need to consider using other means: for example, you might put recent posts in a [memcache](https://web.archive.org/web/20160424231001/https://cloud.google.com/appengine/docs/java/memcache/overview) with an expiration and display a mix of recent posts from the memcache and the Datastore, or you might cache them in a cookie, put some state in the URL, or something else entirely. The goal is to find a caching solution that provides the data for the current user for the period of time in which the user is posting to your application. Remember, if you do a get, an ancestor query, or any operation within a transaction, you will always see the most recently written data.
