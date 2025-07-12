# Storing Data in Java

  

[Python](https://web.archive.org/web/20160424230136/https://cloud.google.com/appengine/docs/python/storage "View this page in the Python runtime") <span style="color: #ddd; padding: 0em .5em;">|</span><span style="color: black; font-weight:bold">Java</span> <span style="color: #ddd; padding: 0em .5em;">|</span>[PHP](https://web.archive.org/web/20160424230136/https://cloud.google.com/appengine/docs/php/storage "View this page in the PHP runtime") <span style="color: #ddd; padding: 0em .5em;">|</span>[Go](https://web.archive.org/web/20160424230136/https://cloud.google.com/appengine/docs/go/storage "View this page in the Go runtime")

The App Engine environment provides multiple options to store data for your application:

-   [App Engine Datastore:](#app_engine_datastore) A schemaless object datastore with automatic caching, a sophisticated query engine, and atomic transactions.
-   [Google Cloud SQL:](#google_cloud_sql) A relational SQL database for your App Engine application, based on the familiar MySQL database.
-   [Google Cloud Storage:](#google_cloud_storage) A storage service for objects and files up to terabytes in size, and accessible to App Engine apps via the Google Cloud Storage client library.

  

<table width="75%">
<colgroup>
<col style="width: 25%" />
<col style="width: 25%" />
<col style="width: 25%" />
<col style="width: 25%" />
</colgroup>
<thead>
<tr class="header">
<th width="16%">Name</th>
<th width="13%">Structure</th>
<th width="16%">Consistency</th>
<th width="55%">Cost</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><strong>App Engine Datastore</strong></td>
<td>Schemaless</td>
<td>Strongly consistent except when performing global queries.</td>
<td>The App Engine Datastore offers a free quota with daily limits. Paid accounts offer unlimited storage, read, and write operations. More information is available on the <a href="https://web.archive.org/web/20160424230136/https://cloud.google.com/appengine/docs/quotas#Datastore">Datastore Quotas page</a>.</td>
</tr>
<tr class="even">
<td><strong>Google Cloud SQL</strong></td>
<td>Relational</td>
<td>Strongly consistent</td>
<td>Google offers two billing plans for Google Cloud SQL: Packages and Per Use. More information is available in the <a href="https://web.archive.org/web/20160424230136/https://cloud.google.com/cloud-sql/docs/billing">Cloud SQL price sheet</a>.</td>
</tr>
<tr class="odd">
<td><strong>Google Cloud Storage</strong></td>
<td>Files and their associated metadata</td>
<td>Strongly consistent except when performing list operations that get a list of buckets or objects.</td>
<td>There are no charges associated with making calls to Google Cloud Storage. However, any data stored in Google Cloud Storage is charged the usual Google Cloud Storage data storage fees.<br />
<br />
Cloud Storage prices are available on the <a href="https://web.archive.org/web/20160424230136/https://cloud.google.com/storage/docs/pricing-and-terms#pricing">Cloud Storage price sheet</a>.</td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
</tbody>
</table>

[Alternative storage solutions](#alternative_storage_solutions) are also available, but they have been superseded by the options listed above.

## App Engine Datastore

[App Engine Datastore](https://web.archive.org/web/20160424230136/https://cloud.google.com/appengine/docs/java/datastore) is a schemaless object datastore providing robust, scalable storage for your application, with the following features:

-   Highly reliable and covered by the [App Engine SLA](https://web.archive.org/web/20160424230136/https://cloud.google.com/appengine/sla?hl=en).
-   [ACID](https://web.archive.org/web/20160424230136/https://en.wikipedia.org/wiki/ACID) transactions.
-   Advanced [querying features](https://web.archive.org/web/20160424230136/https://cloud.google.com/appengine/docs/java/datastore/queries).
-   High availability of reads and writes.
-   Strong consistency for reads and ancestor queries.
-   Eventual consistency for all other queries.

You can access Datastore using the low-level API described throughout the Datastore documentation, which provides direct access to all of Datastore's features, or you can use one of the higher-level open-source APIs for Datastore that provide ORM-like features and a more abstract experience, such as [Objectify](https://web.archive.org/web/20160424230136/https://github.com/objectify/objectify).

Read more about App Engine Datastore on the [App Engine Datastore API documentation](https://web.archive.org/web/20160424230136/https://cloud.google.com/appengine/docs/java/datastore/) page.

## Google Cloud SQL

[Google Cloud SQL](https://web.archive.org/web/20160424230136/https://cloud.google.com/sql/docs/introduction) is a web service that allows you to create, configure, and use relational databases that live in Google's cloud. It is a fully-managed service offering the capabilities of a MySQL database, allowing you to focus on application development.

To get started using Google Cloud SQL, read the [Google Cloud SQL with App Engine SDK documentation](https://web.archive.org/web/20160424230136/https://cloud.google.com/appengine/docs/java/cloud-sql/).

## Google Cloud Storage

[Google Cloud Storage](https://web.archive.org/web/20160424230136/https://cloud.google.com/storage/docs/overview) is useful for storing and serving large files. Additionally, Cloud Storage offers the use of access control lists (ACLs), the ability to resume upload operations if they're interrupted, and many other features. The Google Cloud Storage client library makes use of this resume capability automatically for your app, providing you with a robust way to stream data into Google Cloud Storage.

There are no charges associated with making calls to Google Cloud Storage. However, any data stored at Google Cloud Storage is charged the usual data storage fees, which are listed on the [Cloud Storage price sheet.](https://web.archive.org/web/20160424230136/https://cloud.google.com/storage/docs/pricing-and-terms#pricing)

Applications connect to Google Cloud Storage using the Google Cloud Storage Client Library. To get started, read the [Google Cloud Storage Client Library Getting Started Guide for Java](https://web.archive.org/web/20160424230136/https://cloud.google.com/appengine/docs/java/googlecloudstorageclient/getstarted).

## Alternative Storage Solutions

The following solutions are supported by App Engine, but they have been superseded by the options listed above.

**[Blobstore API:](https://web.archive.org/web/20160424230136/https://cloud.google.com/appengine/docs/java/blobstore/)** Google Cloud Storage is recommended over using the Blobstore API.
