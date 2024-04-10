# Implementing Multitenancy Using Namespaces

  

The Namespaces API allows you to easily enable [multitenancy](https://web.archive.org/web/20160424225338/https://cloud.google.com/appengine/docs/java/multitenancy/index) in your application, simply by selecting a namespace string for each tenant in `web.xml` using the `NamespaceManager` package.

1.  [Setting the current namespace](#Java_Setting_the_current_namespace)
2.  [Avoiding data leaks](#Java_Avoiding_data_leaks)
3.  [Deploying namespaces](#Java_Deploying_namespaces)
    1.  [Creating namespaces on a per user basis](#Java_Creating_namespaces_on_a_per_user_basis)
    2.  [Using namespaces with the Datastore](#Java_Using_namespaces_with_the_Datastore)
    3.  [Using namespaces with the Memcache](#Java_Using_namespaces_with_the_Memcache)
    4.  [Using namespaces with the Task Queue](#Java_Using_namespaces_with_the_Task_Queue)
    5.  [Using namespaces with the Blobstore](#Java_Using_namespaces_with_the_Blobstore)
    6.  [Setting namespaces for Datastore Queries](#Java_Setting_namespaces_in_the_Datastore_Viewer)
    7.  [Using namespaces with the Bulkloader](#Java_Using_namespaces_with_the_Bulkloader)
    8.  [Using namespaces with Search](#Java_Using_namespaces_with_Search)

## Setting the current namespace

You can get, set, and validate namespaces using `NamespaceManager`. The namespace manager allows you to set a current namespace for [namespace-enabled APIs](https://web.archive.org/web/20160424225338/https://cloud.google.com/appengine/docs/java/multitenancy/#Java_App_Engine_APIs_that_use_namespaces). You set a current namespace up-front using `web.xml`, and the datastore and memcache automatically use that namespace.

Most App Engine developers will use their Google Apps domain as the current namespace. Google Apps lets you deploy your app to any domain that you own, so you can easily use this mechanism to configure different namespaces for different domains. Then, you can use those separate namespaces to segregate data across the domains. For more information about setting multiple domains in the Google Apps dashboard, see [Deploying Your Application on Your Google Apps URL](https://web.archive.org/web/20160424225338/https://cloud.google.com/appengine/articles/domains.html).

The following code sample shows you how to set the current namespace to the Google Apps domain that was used to map the URL. Notably, this string will be the same for all URLs mapped via the same Google Apps domain.

You can set namespaces in Java using the servlet Filter interface before invoking servlet methods. The following simple example demonstrates how to use your Google Apps domain as the current namespace:

```
// Filter to set the Google Apps domain as the namespace.
public class NamespaceFilter implements javax.servlet.Filter {
  @Override
  public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain)
      throws IOException, ServletException {
    // Make sure set() is only called if the current namespace is not already set.
    if (NamespaceManager.get() == null) {
      NamespaceManager.set(NamespaceManager.getGoogleAppsNamespace());
    }
  }
... remaining Filter methods init() and destroy()
}
```

The namespace filter must be configured in the `web.xml` file. Note that, if there are multiple filter entries, the first namespace to be set is the one that will be used.

The following code sample demonstrates how to configure the namespace filter in `web.xml`:

```
<!-- Configure the namespace filter. -->
<filter>
  <filter-name>NamespaceFilter</filter-name>
  <filter-class>(put the class path here).NamespaceFilter</filter-class>
</filter>
<filter-mapping>
  <filter-name>NamespaceFilter</filter-name> 
  <url-pattern>/*</url-pattern>
</filter-mapping>
```

You can also set a new namespace for a temporary operation, resetting the original namespace once the operation is complete, using the `try`/`finally` pattern shown below:

```
// Set the namepace temporarily to "abc"
String oldNamespace = NamespaceManager.get();
NamespaceManager.set("abc");
try {
  ... perform operation using current namespace ...
} finally {
  NamespaceManager.set(oldNamespace);
}
```

If you do not specify a value for `namespace`, the namespace is set to an empty string. The `namespace` string is arbitrary, but also limited to a maximum of 100 alphanumeric characters, periods, underscores, and hyphens. More explicitly, namespace strings must match the regular expression `[0-9A-Za-z._-]{0,100}`.

By convention, all namespaces starting with "`_`" (underscore) are reserved for system use. This system namespace rule is not enforced, but you could easily encounter undefined negative consequences if you do not follow it.

## Avoiding data leaks

One of the risks commonly associated with multitenant apps is the danger that data will leak across namespaces. Unintended data leaks can arise from many sources, including:

-   Using namespaces with App Engine APIs that do not yet support namespaces. For example, Blobstore does not support namespaces. If you use [Namespaces with Blobstore](#Java_Using_namespaces_with_the_Blobstore), you need to avoid using Blobstore queries for end user requests, or Blobstore keys from untrusted sources.
-   Using an external storage medium (instead of memcache and datastore), via `URL Fetch` or some other mechanism, without providing a compartmentalization scheme for namespaces.
-   Setting a namespace based on a user's email domain. In most cases, you don't want all email addresses of a domain to access a namespace. Using the email domain also prevents your application from using a namespace until the user is logged in.

## Deploying namespaces

The following sections describe how to deploy namespaces with other App Engine tools and APIs.

### Creating namespaces on a per user basis

Some applications need to create namespaces on a per-user basis. If you want to compartmentalize data at the user level for logged-in users, consider using [`User.getUserId()`](https://web.archive.org/web/20160424225338/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/users/User#getUserId()), which returns a unique, permanent ID for the user. The following code sample demonstrates how to use the Users API for this purpose:

```
if (NamespaceManager.get() == null) {
  // Assuming there is a logged in user.
  namespace = UserServiceFactory.getUserService().getCurrentUser().getUserId();
  NamespaceManager.set(namespace);
}
```

Typically, apps that create namespaces on a per-user basis also provide specific landing pages to different users. In these cases, the application needs to provide a URL scheme dictating which landing page to display to a user.

### Using namespaces with the Datastore

By default, the datastore uses the current namespace setting in the namespace manager for datastore requests. The API applies this current namespace to [`Key`](https://web.archive.org/web/20160424225338/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/Key) or [`Query`](https://web.archive.org/web/20160424225338/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/Query) objects when they are created. Therefore, you need to be careful if an application stores `Key` or `Query` objects in serialized forms, since the namespace is preserved in those serializations.

If you are using deserialized `Key` and `Query` objects, make sure that they behave as intended. Most simple applications that use datastore (`put`/`query`/`get`) without using other storage mechanisms will work as expected by setting the current namespace before calling any datastore API.

**Note:** An application that reads Keys, or other namespace-aware objects, from untrusted sources (like the web browser client) introduces security vulnerabilities. Applications that rely on keys from untrusted sources must incorporate a security layer verifying that the current user is authorized to access the requested namespace.

`Query` and `Key` objects demonstrate the following, unique behaviors with regard to namespaces:

-   `Query` and `Key` objects inherit the current namespace when constructed, unless you set an explicit namespace.
-   When an application creates a new `Key` from an ancestor, the new `Key` inherits the namespace of the ancestor.
-   There is no Java API to explicitly set the namespace of a `Key` or `Query`.

The following code example shows the `SomeRequest` request handler for incrementing the count for the current namespace and the arbitrarily named `-global-` namespace in a `Counter` datastore entity.
```
import com.google.appengine.api.NamespaceManager;

import java.io.IOException;

import javax.jdo.JDOCanRetryException;
import javax.jdo.JDOObjectNotFoundException;
import javax.jdo.PersistenceManager;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class UpdateCountsServlet {
  private static final int NUM_RETRIES = 10;

  // Increment the count in a Counter datastore entity.
  public static long updateCount(String countName) {
    long theCount = 0;
    PersistenceManager pm = PMF.get().getPersistenceManager();
    for (int i = 0; i < NUM_RETRIES; i++) {
      pm.currentTransaction().begin();

     // Avoid calling NamespaceManager.set() within
     // transactions or between creating JDO objects
     // and their makePersistent calls.
      Counter counter;
      try {
        counter = pm.getObjectById(Counter.class, countName);
        theCount = counter.incrementCount();
      } catch (JDOObjectNotFoundException e) {
        counter = new Counter(countName);
        theCount = counter.incrementCount();
        pm.makePersistent(counter);
      }

      try {
          pm.currentTransaction().commit();
          break;

      } catch (JDOCanRetryException ex) {
          if (i == (NUM_RETRIES - 1)) {
              throw ex;
          }
      }
    }
    return theCount;
  }

  @Override
  protected void doGet(HttpServletRequest req,
  HttpServletResponse resp)
    throws IOException {

    // Update the count for the current namespace.
    updateCount("request");

    // Update the count for the "-global-" namespace.
    String namespace = NamespaceManager.get();
    try {
      // "-global-" is namespace reserved by the application.
      NamespaceManager.set("-global-");
      updateCount("request");
    } finally {
      NamespaceManager.set(namespace);
    }
    resp.setContentType("text/plain");
    resp.getWriter().println("Counts are now updated.");
  }
}
```

### Using namespaces with the Memcache

By default, memcache uses the current namespace from the namespace manager for memcache requests. In most cases, you do not need to explicitly set a namespace in the memcache, and doing so could introduce unexpected bugs.

However, there are some unique instances where it is appropriate to explicitly set a namespace in the memcache. For example, your application might have common data shared across all namespaces (such as a table containing country codes).

**Warning:** If you explicitly set a namespace in the memcache, it will ignore the current settings from the namespace manager.

The following code snippet demonstrates how to explicitly set the namespace in the memcache:

By default, the Java API to memcache queries the namespace manager for the current namespace from `MemcacheService`. You can also explicitly state a namespace when you construct the memcache using `getMemcacheService(Namespace)`. For most applications, you don't need to explicitly specify a namespace.

The following code sample demonstrates how to create a memcache that uses the current namespace in the namespace manager.

```
// Create a MemcacheService that uses the current namespace by
// calling NamespaceManager.get() for every access.
MemcacheService current = MemcacheServiceFactory.getMemcacheService();

// stores value in namespace "abc"
String oldNamespace = NamespaceManager.get();
NamespaceManager.set("abc");
try {
   current.put("key", value);  // stores value in namespace abc
} finally {
   NamespaceManager.set(oldNamespace);
}
```

This code sample explicitly specifies a namespace when creating a memcache service:

```
// Create a MemcacheService that uses the namespace "abc".
MemcacheService explicit = MemcacheServiceFactory.getMemcacheService("abc");
explicit.put("key", value);  // stores value in namespace "abc"
```

### Using namespaces with the Task Queue

By default, [push queues](https://web.archive.org/web/20160424225338/https://cloud.google.com/appengine/docs/java/taskqueue/overview-push) use the current namespace as set in the namespace manager at the time the task was created. In most cases, you do not need to explicitly set a namespace in the task queue, and doing so could introduce unexpected bugs.

**Warning:** Tasks in [pull queues](https://web.archive.org/web/20160424225338/https://cloud.google.com/appengine/docs/java/taskqueue/) do not provide any namespace functionality. If you use namespaces with pull queues, you need to ensure that namespaces are saved in the payload and restored as needed by the application.

Task names are shared across all namespaces. You cannot create two tasks of the same name, even if they use different namespaces. If you wish to use the same task name for many namespaces, you can simply append each namespace to the task name.

When a new task calls the task queue [`add()`](https://web.archive.org/web/20160424225338/https://cloud.google.com/appengine/docs/python/taskqueue/functions#add) method, the task queue copies the current namespace and (if applicable) the Google Apps domain from the namespace manager. When the task is executed, the current namespace and Google Apps namespace are restored.

If the current namespace is not set in the originating request (in other words, if `get()` returns `null`), then the task queue sets the namespace to `""` in the executed tasks.

There are some unique instances where it is appropriate to explicitly set a namespace for a task that works across all namespaces. For example, you might create a task that aggregates usage statistics across all namespaces. You could then explicitly set the namespace of the task. The following code sample demonstrates how to explicitly set namespaces with the task queue.

First, create a task queue handler that increments the count in a `Counter` datastore entity:

```
// Update count task request handler
public class UpdateCount extends HttpServlet {
  private static final int NUM_RETRIES = 10;

  // Increment the count in a Counter datastore entity.
  public static long updateCount(String countName) {
    long theCount = 0;
    PersistenceManager pm = PMF.get().getPersistenceManager();
    for (int i = 0; i < NUM_RETRIES; i++) {
      pm.currentTransaction().begin();
  
      Counter counter;
      try {
        counter = pm.getObjectById(Counter.class, countName);
        theCount = counter.incrementCount();
      } catch (JDOObjectNotFoundException e) {
        counter = new Counter(countName);
        theCount = counter.incrementCount();
        pm.makePersistent(counter);
      }

      try {
          pm.currentTransaction().commit();
          break;

      } catch (JDOCanRetryException ex) {
          if (i == (NUM_RETRIES - 1)) {
              throw ex;
          }
      }
    }
    return theCount;
  }

  @Override
  protected void doPost(HttpServletRequest req,
                        HttpServletResponse resp) {
    String countName[] = req.getParameterValues("countName");
    if (countName.length != 1) {
      resp.setStatus(HttpServletResponse.SC_BAD_REQUEST);
      return;
    }
    updateCount(countName[0]);
  }
}
```

Then, create tasks with a servlet:

```
import com.google.appengine.api.NamespaceManager;
import com.google.appengine.api.taskqueue.QueueFactory;
import com.google.appengine.api.taskqueue.TaskOptions;

import java.io.IOException;

import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class SomeRequest extends HttpServlet {
  // Handler for URL get requests.
  @Override
  protected void doGet(HttpServletRequest req, 
                       HttpServletResponse resp) 
      throws IOException {

    // Increment the count for the current namespace asynchronously.
    QueueFactory.getDefaultQueue().add(
        TaskOptions.Builder.url("/_ah/update_count")
                           .param("countName", "SomeRequest"));
    // Increment the global count and set the
    // namespace locally.  The namespace is
    // transferred to the invoked request and 
    // executed asynchronously.
    String namespace = NamespaceManager.get();
    try {
      NamespaceManager.set("-global-");
      QueueFactory.getDefaultQueue().add(
          TaskOptions.Builder.url("/_ah/update_count")
                             .param("countName", "SomeRequest"));
    } finally {
      NamespaceManager.set(namespace);
    }
    resp.setContentType("text/plain");
    resp.getWriter().println("Counts are being updated.");
  }
}
```

### Using namespaces with the Blobstore

The Blobstore is not segmented by namespace. To preserve a namespace in Blobstore, you need to access Blobstore via a storage medium that is aware of the namespace (currently only memcache, datastore, and task queue). For example, if a blob's `Key` is stored in a datastore entity, you can access it with a datastore `Key` or `Query` that is aware of the namespace.

If the application is accessing Blobstore via keys stored in namespace-aware storage, the Blobstore itself does not need to be segmented by namespace. Applications must avoid blob leaks between namespaces by:

-   Not using `com.google.appengine.api.blobstore.BlobInfoFactory` for end-user requests. You can use BlobInfo queries for administrative requests (such as generating reports about all the applications blobs), but using it for end-user requests may result in data leaks because all BlobInfo records are not compartmentalized by namespace.
-   Not using Blobstore keys from untrusted sources.

### Setting namespaces for Datastore Queries

In the Google Cloud Platform Console, you can <a href="https://web.archive.org/web/20160424225338/https://console.cloud.google.com/project/_/datastore/query" data-track-metadata-end-goal="queryDatastore" data-track-metadata-position="body" data-track-name="consoleLink" data-track-type="tasks">set the namespace for Datastore queries</a>.

If you don't want to use the default, select the namespace you want to use from the drop-down.

### Using namespaces with the Bulk Loader

The bulk loader supports a `--namespace=NAMESPACE` flag that allows you to specify the namespace to use. Each namespace is handled separately and, if you want to access all namespaces, you will need to iterate through them.

### Using namespaces with Search

A new instance of `Index` inherits the namespace of the `SearchService` used to create it. Once you've created a reference to an index, its namespace cannot be changed. There are two ways to set the namespace for a `SearchService` before using it to create an index:

-   By default, a new `SearchService` takes the current namespace. You can set the current namespace before creating the service:

    ```
    // Set the current namespace to "aSpace"
    NamespaceManager.set("aSpace");
    // Create a SearchService with the namespace "aSpace"
    SearchService searchService = SearchServiceFactory.getSearchService();
    // Create an IndexSpec
    IndexSpec indexSpec = IndexSpec.newBuilder().setName("myIndex").build();
    // Create an Index with the namespace "aSpace"
    Index index = searchService.getIndex(indexSpec);
    ```

-   You can specify a namespace in the `SearchServiceConfig` when creating a service:

    ```
    // Create a SearchServiceConfig, specifying the namespace "anotherSpace"
    SearchServiceConfig config =  SearchServiceConfig.newBuilder().setNamespace("anotherSpace").build(); 
    // Create a SearchService with the namespace "anotherSpace"
    SearchService searchService = SearchServiceFactory.getSearchService(config); 
    // Create an IndexSpec
    IndexSpec indexSpec = IndexSpec.newBuilder().setName("myindex").build(); 
    // Create an Index with the namespace "anotherSpace"
    Index index = searchService.getIndex(indexSpec);
    ```
