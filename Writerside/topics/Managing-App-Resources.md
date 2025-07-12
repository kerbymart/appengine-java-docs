# Managing App Resources

  

[Python](https://web.archive.org/web/20160424230521/https://cloud.google.com/appengine/docs/python/console/managing-resources "View this page in the Python runtime") <span style="color: #ddd; padding: 0em .5em;">|</span><span style="color: black; font-weight:bold">Java</span> <span style="color: #ddd; padding: 0em .5em;">|</span>[PHP](https://web.archive.org/web/20160424230521/https://cloud.google.com/appengine/docs/php/console/managing-resources "View this page in the PHP runtime") <span style="color: #ddd; padding: 0em .5em;">|</span>[Go](https://web.archive.org/web/20160424230521/https://cloud.google.com/appengine/docs/go/console/managing-resources "View this page in the Go runtime")

App Engine generates usage reports to help you understand your application's performance and the resources your application is using. Based on these reports, you can employ the strategies listed below to manage your resources. After making any changes, you will see that information reflected in subsequent usage reports. For more information, please see our [pricing](https://web.archive.org/web/20160424230521/https://cloud.google.com/appengine/pricing.html) page.

1.  [Viewing usage reports](#viewing_usage_reports)
2.  [Managing dynamic instance scaling](#managing_dynamic_instance_scaling)
    1.  [Decreasing latency](#decreasing_latency)
    2.  [Change auto scaling performance settings](#change_auto_scaling_performance_settings)
    3.  [Enable concurrent requests in Java](#enable_concurrent_requests_in_java)
    4.  [Enable concurrent requests in Python](#enable_concurrent_requests_in_python)
    5.  [Configuring TaskQueue settings](#configuring_taskqueue_settings)
    6.  [Serve static content where possible](#serve_static_content_where_possible)
3.  [Managing application storage](#managing_application_storage)
4.  [Managing datastore usage](#managing_datastore_usage)
5.  [Managing bandwidth](#managing_bandwidth)
6.  [Managing other resources](#managing_other_resources)

## Viewing usage reports

When you evaluate the performance of your application, you will be interested in how many [instances](https://web.archive.org/web/20160424230521/https://cloud.google.com/appengine/docs/scaling) it is running, and how it is consuming resources.

<a href="https://web.archive.org/web/20160424230521/https://console.cloud.google.com/appengine" class="button button-primary" data-track-metadata-end-goal="viewDashboard" data-track-metadata-position="body" data-track-name="consoleLink" data-track-type="tasks">View the dashboard usage reports</a>

[Learn more about the dashboard](https://web.archive.org/web/20160424230521/https://cloud.google.com/appengine/docs/java/console/#dashboard) that summarizes resource usage.

<a href="https://web.archive.org/web/20160424230521/https://console.cloud.google.com/appengine/instances" class="button button-primary" data-track-metadata-end-goal="viewAppEngineInstances" data-track-metadata-position="body" data-track-name="consoleLink" data-track-type="tasks">View the Instances page</a>

[Learn more about viewing instances](https://web.archive.org/web/20160424230521/https://cloud.google.com/appengine/docs/java/console/#instances) from the Cloud Platform Console.

The following sections suggest some strategies you can use to manage resources, and explain what these strategies could mean for your application's performance.

## Managing dynamic instance scaling

### Decreasing latency

Application latency has an impact on how many instances are required to handle your traffic. By decreasing latency, you can reduce the number of instances we use to serve your application. [Google Cloud Trace](https://web.archive.org/web/20160424230521/https://cloud.google.com/trace/) is a useful tool that you can use to view data about latency and understand what changes you can make to decrease it.

After you have used Cloud Trace to view your latency, you can to try some of the following strategies for reducing it:

-   **Increase caching of frequently accessed shared data** - That’s another way of saying - use Memcache. Also, if you set your application’s cache-control headers, this can have a big impact on how efficiently your data is cached by servers and browsers. Even caching things for a few seconds can have a big impact on how efficiently your application serves traffic. Python applications should also make use of [caching in the runtime](https://web.archive.org/web/20160424230521/https://cloud.google.com/appengine/docs/python/#Python_App_caching) as well.
-   **Use Memcache more efficiently** - Use batch calls for get, set, delete, etc instead of a series of individual calls. Where appropriate, consider using the Memcache Async API ([Java](https://web.archive.org/web/20160424230521/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/memcache/AsyncMemcacheService.html), [Python](https://web.archive.org/web/20160424230521/https://cloud.google.com/appengine/docs/python/refdocs/google.appengine.api.memcache#google.appengine.api.memcache.Client)).
-   **Use Tasks for non-request bound functionality**- If your application performs work that can be done outside of the scope of a user-facing request, put it in a task! Sending this work to the Task Queue instead of waiting for it to complete before returning a response can significantly reduce user-facing latency. The Task Queue can then give you much more control over execution rates and help smooth out your load.
-   **Use the datastore more efficiently** - We go in to more detail for this below.
-   **Parallelize multiple URL Fetch calls**
    -   Use the async URL Fetch API ([Java](https://web.archive.org/web/20160424230521/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/urlfetch/URLFetchService#fetchAsync(com.google.appengine.api.urlfetch.HTTPRequest)), [Python](https://web.archive.org/web/20160424230521/https://cloud.google.com/appengine/docs/python/urlfetch/asynchronousrequests)).
    -   Use [goroutines](https://web.archive.org/web/20160424230521/http://golang.org/doc/effective_go.html#concurrency) (Go).
    -   Batch together multiple URL Fetch calls (which you might handle individually inside individual user facing requests) and handle them in an offline task in parallel via async URL Fetch.
-   **For Java HTTP sessions, write asynchronously** - HTTP sessions ([Java](https://web.archive.org/web/20160424230521/https://cloud.google.com/appengine/docs/java/config/appconfig#Java_appengine_web_xml_Enabling_sessions)) lets you configure your application to asynchronously write HTTP session data to the datastore by adding `<async-session-persistence enabled="true"/>` to your `appengine-web.xml`. Session data is always written synchronously to memcache, and if a request tries to read the session data when memcache is not available it will fail over to the datastore, which might not yet have the most recent update. This means there is a small risk your application will see stale session data, but for most applications the latency benefit far outweighs the risk.

### Change auto scaling performance settings

The module configuration file contains two settings you can use to adjust the tradeoff between performance and resource load:

-   **Max Idle Instances** - Setting [Max Idle Instances](https://web.archive.org/web/20160424230521/https://cloud.google.com/appengine/docs/java/config/appconfig#max_idle_instances) allows App Engine to shut down idle instances above the specified limit so they won't consume additional quota or create charges. However, fewer idle instances also means that the App Engine Scheduler may have to spin up new instances if you experience a spike in traffic -- potentially increasing user-visible latency for your app.
-   **Min Pending Latency** - Raising [Min Pending Latency](https://web.archive.org/web/20160424230521/https://cloud.google.com/appengine/docs/java/config/appconfig#min_pending_latency) instructs App Engine’s scheduler to not start a new instance unless a request has been pending for more than the specified time. If all instances are busy, user-facing requests may have to wait in the pending queue until this threshold is reached. Setting a high value for this setting will require fewer instances to be started, but may result in high user-visible latency during increased load.

### Enable concurrent requests in Java

In our [1.4.3 release](https://web.archive.org/web/20160424230521/http://code.google.com/p/googleappengine/wiki/SdkForJavaReleaseNotes), we introduced the ability for your application's instances to serve multiple requests concurrently in Java. Enabling this setting will decrease the number of instances needed to serve traffic for your application, but your application must be threadsafe in order for this to work correctly. Read about how to enable concurrent requests in our [Java documentation](https://web.archive.org/web/20160424230521/https://cloud.google.com/appengine/docs/java/config/appconfig#Java_appengine_web_xml_Using_concurrent_requests).

### Enable concurrent requests in Python

In [Python 2.7](https://web.archive.org/web/20160424230521/https://cloud.google.com/appengine/docs/python/python25/diff27), we introduced the ability for your application's instances to serve multiple requests concurrently in Python. Enabling this setting will decrease the number of instances needed to serve traffic for your application, but your application must be threadsafe in order for this to work correctly. Read about how to enable concurrent requests in our [Python documentation](https://web.archive.org/web/20160424230521/https://cloud.google.com/appengine/docs/python/config/appconfig#Python_app_yaml_Using_concurrent_requests). Concurrent requests are not available in the Python 2.5 runtime.

\*\*Note:\*\* All Go instances have concurrent requests enabled automatically.

### Configuring TaskQueue settings

The default settings for the Task Queue are tuned for performance. With these defaults, when you put several tasks into a queue simultaneously, they will likely cause new Frontend Instances to spin up. Here are some suggestions for how to tune the Task Queue to conserve Instance Hours:

-   Set the X-AppEngine-FailFast header on tasks that are not latency sensitive. This header instructs the Scheduler to immediately fail the request if an existing instance is not available. The Task Queue will retry and back-off until an existing instance becomes available to service the request. However, it is important to note that when requests with X-AppEngine-FailFast set occupy existing instances, requests without that header set may still cause new instances to be started.
-   Configure your Task Queue's settings([Java](https://web.archive.org/web/20160424230521/https://cloud.google.com/appengine/docs/java/config/queue#Queue_Definitions), [Python](https://web.archive.org/web/20160424230521/https://cloud.google.com/appengine/docs/python/config/queue#Python_Queue_definitions)).
    -   If you set the "rate" parameter to a lower value, Task Queue will execute your tasks at a slower rate.
    -   If you set the "max\_concurrent\_requests" parameter to a lower value, fewer tasks will be executed simultaneously.
-   Use backends([Java](https://web.archive.org/web/20160424230521/https://cloud.google.com/appengine/docs/java/backends), [Python](https://web.archive.org/web/20160424230521/https://cloud.google.com/appengine/docs/python/backends)) in order to completely control the number of instances used for task execution. You can use push queues with dynamic backends, or pull queues with resident backends.

### Serve static content where possible

Static content serving ([Java](https://web.archive.org/web/20160424230521/https://cloud.google.com/appengine/docs/java/gettingstarted/staticfiles.html), [Python](https://web.archive.org/web/20160424230521/https://cloud.google.com/appengine/docs/python/gettingstartedpython27/staticfiles.html)) is handled by specialized App Engine infrastructure, which does not consume Instance Hours. If you need to set custom headers, use the Blobstore API ([Java](https://web.archive.org/web/20160424230521/https://cloud.google.com/appengine/docs/java/blobstore), [Python](https://web.archive.org/web/20160424230521/https://cloud.google.com/appengine/docs/python/blobstore/), [Go](https://web.archive.org/web/20160424230521/https://cloud.google.com/appengine/docs/go/blobstore/)). The actual serving of the Blob response does not consume Instance Hours.

## Managing application storage

App Engine calculates storage costs based on the size of entities in the datastore, the size of datastore indexes, the size of tasks in the task queue, and the amount of data stored in Blobstore. Here are some things you can do to make sure you don't store more data than necessary:

-   Delete any entities or blobs your application no longer needs.
-   Remove any unnecessary indexes, as discussed in the *Managing Datastore Usage* section below, to reduce index storage costs.

## Managing datastore usage

App Engine accounts for the number of operations performed in the Datastore. Here are a few strategies that can result in reduced Datastore resource consumption, as well as lower latency for requests to the datastore:

-   The Cloud Platform Console dataviewer displays the number of write ops that were required to create every entity in your local datastore. You can use this information to understand the cost of writing each entity. See [Understanding Write Costs](https://web.archive.org/web/20160424230521/https://cloud.google.com/appengine/docs/java/datastore/entities.html#Understanding_Write_Costs) for information on how to interpret this data.
-   Remove any unnecessary indexes, which will reduce storage and entity write costs. Use the "Get Indexes" functionality ([Java](https://web.archive.org/web/20160424230521/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/AsyncDatastoreService#getIndexes()), [Python](https://web.archive.org/web/20160424230521/https://cloud.google.com/appengine/docs/python/datastore/functions)) to see what indexes are defined on your application. You can see what indexes are currently serving for your application in the Cloud Platform Console [Search page](https://web.archive.org/web/20160424230521/https://console.cloud.google.com/project/_/appengine/search).
-   When designing your data model, you might be able to write your queries in such a way so as to avoid custom indexes altogether. Read our [Queries and Indexes](https://web.archive.org/web/20160424230521/https://cloud.google.com/appengine/docs/python/datastore/queries) documentation for more information on how App Engine generates indexes.
-   Whenever possible, replace indexed properties (which are the default) with unindexed properties ([Java](https://web.archive.org/web/20160424230521/https://cloud.google.com/appengine/docs/java/datastore/queries.html#Introduction_to_Indexes), [Python](https://web.archive.org/web/20160424230521/https://cloud.google.com/appengine/docs/python/datastore/queries.html#Introduction_to_Indexes)), which reduces the number of datastore write operations when you put an entity. Caution, if you later decide that you do need to be able to query on the unindexed property, you will need to not only modify your code to again use indexed properties, but you will have to run a [map reduce](https://web.archive.org/web/20160424230521/http://code.google.com/p/appengine-mapreduce/) over all entities to reput them.
-   Due to the datastore query planner improvements in the App Engine 1.5.2 and 1.5.3 releases ([Java](https://web.archive.org/web/20160424230521/http://code.google.com/p/googleappengine/wiki/SdkForJavaReleaseNotes), [Python](https://web.archive.org/web/20160424230521/http://code.google.com/p/googleappengine/wiki/SdkReleaseNotes)), your queries may now require fewer indexes than they did previously. While you may still choose to keep certain custom indexes for performance reasons, you may be able to delete others, reducing storage and entity write costs.
-   Reconfigure your data model so that you can replace queries with fetch by key ([Java](https://web.archive.org/web/20160424230521/https://cloud.google.com/appengine/docs/java/datastore/entities.html#Saving_Getting_and_Deleting_Entities), [Python](https://web.archive.org/web/20160424230521/https://cloud.google.com/appengine/docs/python/datastore/modelclass.html#Model_get), [Go](https://web.archive.org/web/20160424230521/https://cloud.google.com/appengine/docs/go/datastore/reference.html#Get)), which is cheaper and more efficient.
-   Use keys-only queries instead of entity queries when possible.
-   To decrease latency, replace multiple entity `get()`s with a batch `get()`.
-   Use datastore cursors for pagination rather than offset.
-   Parallelize multiple datastore RPCs via the async datastore API ([Java](https://web.archive.org/web/20160424230521/https://cloud.google.com/appengine/docs/java/datastore/async.html), [Python](https://web.archive.org/web/20160424230521/https://cloud.google.com/appengine/docs/python/datastore/async.html)), or [goroutines](https://web.archive.org/web/20160424230521/http://golang.org/doc/effective_go.html#concurrency) (Go).

**Note:** Small datastore operations include calls to allocate datastore ids or keys-only queries. See the [pricing](https://web.archive.org/web/20160424230521/https://cloud.google.com/appengine/pricing#cost_resource) page for more information on costs.

## Managing bandwidth

For Outgoing Bandwidth, one way to reduce usage is to, whenever possible, set the appropriate `Cache-Control` header on your responses and set reasonable expiration times ([Java](https://web.archive.org/web/20160424230521/https://cloud.google.com/appengine/docs/java/config/appconfig#Java_appengine_web_xml_Static_cache_expiration), [Python](https://web.archive.org/web/20160424230521/https://cloud.google.com/appengine/docs/python/config/appconfig#Python_app_yaml_Static_file_handlers)) for static files. Using public `Cache-Control` headers in this way will allow proxy servers and your clients' browser to cache responses for the designated period of time.

Incoming Bandwidth is more difficult to control, since that's the amount of data your users are sending to your app. However, this is a good opportunity to mention our DoS Protection Service for [Python](https://web.archive.org/web/20160424230521/https://cloud.google.com/appengine/docs/python/config/dos.html) and [Java](https://web.archive.org/web/20160424230521/https://cloud.google.com/appengine/docs/java/config/dos.html), which allows you block traffic from IPs that you consider abusive.

## Managing other resources

The last items on the report are the usages for the Email, XMPP, and Channel APIs. For these APIs, your best bet is to make sure you are using them effectively. One of the best strategies for auditing your usage of these APIs is to use Appstats ([Python](https://web.archive.org/web/20160424230521/https://cloud.google.com/appengine/docs/python/tools/appstats.html), [Java](https://web.archive.org/web/20160424230521/https://cloud.google.com/appengine/docs/java/tools/appstats.html)) to make sure you're not making more calls than are necessary. Also, it's always a good idea to make sure you are checking your error rates and looking out for any invalid calls you might be making. In some cases it might be possible to catch those calls early.
