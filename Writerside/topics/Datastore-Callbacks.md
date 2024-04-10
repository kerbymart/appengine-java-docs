# Datastore Callbacks

  

A callback allows you to execute code at various points in the persistence process. You can use these callbacks to easily implement cross-functional logic like logging, sampling, decoration, auditing, and validation (among other things). The Datastore API currently supports callbacks that execute before and after `put()` and `delete()` operations.

1.  [PrePut](#PrePut)
2.  [PostPut](#PostPut)
3.  [PreDelete](#PreDelete)
4.  [PostDelete](#PostDelete)
5.  [PreGet](#PreGet)
6.  [PreQuery](#PreQuery)
7.  [PostLoad](#PostLoad)
8.  [Batch Operations](#Batch_Operations)
9.  [Async Operations](#Async_Operations)
10. [Using Callbacks With App Engine Services](#Using_Callbacks_With_App_Engine_Services)
11. [Common Errors To Avoid](#Common_Errors_To_Avoid)
12. [Using Callbacks With Eclipse](#Using_Callbacks_With_Eclipse)

## PrePut

When you register a [`PrePut`](https://web.archive.org/web/20160424230753/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/PrePut) callback with a specific kind, the callback will be invoked before any entity of that kind is put (or, if no kind is provided, before any entity is put):

```
import com.google.appengine.api.datastore.PrePut;
import com.google.appengine.api.datastore.PutContext;
import java.util.Date;
import java.util.logging.Logger;

class PrePutCallbacks {
    static Logger logger = Logger.getLogger("callbacks");

    @PrePut(kinds = {"Customer", "Order"})
    void log(PutContext context) {
        logger.fine("Putting " + context.getCurrentElement().getKey());
    }

    @PrePut // Applies to all kinds
    void updateTimestamp(PutContext context) {
        context.getCurrentElement().setProperty("last_updated", new Date());
    }
}
```

A method annotated with `PrePut` must be an instance method that returns `void`, accepts a [`PutContext`](https://web.archive.org/web/20160424230753/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/PutContext) as its only argument, does not throw any checked exceptions, and belongs to a class with a no-arg constructor. When a `PrePut` callback throws an exception, no further callbacks are executed, and the `put()` operation throws the exception before any RPCs are issued to the datastore.

## PostPut

When you register a [`PostPut`](https://web.archive.org/web/20160424230753/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/PostPut) callback with a specific kind, the callback will be invoked after any entity of that kind is put (or, if no kind is provided, after any entity is put).

```
import com.google.appengine.api.datastore.PostPut;
import com.google.appengine.api.datastore.PutContext;
import java.util.logging.Logger;

class PostPutCallbacks {
    static Logger logger = Logger.getLogger("logger");

    @PostPut(kinds = {"Customer", "Order"}) // Only applies to Customers and Orders
    void log(PutContext context) {
        logger.fine("Finished putting " + context.getCurrentElement().getKey());
    }

    @PostPut // Applies to all kinds
    void collectSample(PutContext context) {
        Sampler.getSampler().collectSample(
            "put", context.getCurrentElement().getKey());
    }
}
```

A method annotated with `PostPut` must be an instance method that returns `void`, accepts a [`PutContext`](https://web.archive.org/web/20160424230753/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/PutContext) as its only argument, does not throw any checked exceptions, and belongs to a class with a no-arg constructor. When a `PostPut` callback throws an exception, no further callbacks are executed but the result of the `put()` operation is unaffected. Note that if the `put()` operation itself fails, `PostPut` callbacks will not be invoked at all. Also, `PostPut` callbacks that are associated with transactional `put()` operations will not run until the transaction successfully commits.

## PreDelete

When you register a [`PreDelete`](https://web.archive.org/web/20160424230753/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/PreDelete) callback with a specific kind, the callback will be invoked before any entity of that kind is deleted (or, if no kind is provided, before any entity is deleted):

```
import com.google.appengine.api.datastore.DeleteContext;
import com.google.appengine.api.datastore.PreDelete;
import java.util.logging.Logger;

class PreDeleteCallbacks {
    static Logger logger = Logger.getLogger("logger");

    @PreDelete(kinds = {"Customer", "Order"})
    void checkAccess(DeleteContext context) {
        if (!Auth.canDelete(context.getCurrentElement()) {
            throw new SecurityException();
        }
    }

    @PreDelete // Applies to all kinds
    void log(DeleteContext context) {
        logger.fine("Deleting " + context.getCurrentElement().getKey());
    }
}
```

A method annotated with `PreDelete` must be an instance method that returns `void`, accepts a [`DeleteContext`](https://web.archive.org/web/20160424230753/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/DeleteContext) as its only argument, does not throw any checked exceptions, and belongs to a class with a no-arg constructor. When a `PreDelete` callback throws an exception, no further callbacks are executed and the `delete()` operation throws the exception before any RPCs are issued to the datastore.

## PostDelete

When you register a [`PostDelete`](https://web.archive.org/web/20160424230753/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/PostDelete) callback with a specific kind, the callback will be invoked after any entity of that kind is deleted (or, if no kind is provided, after any entity is deleted):

```
import com.google.appengine.api.datastore.DeleteContext;
import com.google.appengine.api.datastore.PostDelete;
import java.util.logging.Logger;

class PostDeleteCallbacks {
    static Logger logger = Logger.getLogger("logger");

    @PostDelete(kinds = {"Customer", "Order"})
    void log(DeleteContext context) {
        logger.fine(
            "Finished deleting " + context.getCurrentElement().getKey());
    }

    @PostDelete // Applies to all kinds
    void collectSample(DeleteContext context) {
        Sampler.getSampler().collectSample(
            "delete", context.getCurrentElement().getKey());
    }
}
```

A method annotated with `PostDelete` must be an instance method that returns `void`, accepts a [`DeleteContext`](https://web.archive.org/web/20160424230753/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/DeleteContext) as its only argument, does not throw any checked exceptions, and belongs to a class with a no-arg constructor. When a `PostDelete` callback throws an exception, no further callbacks are executed but the result of the `delete()` operation is unaffected. Note that if the `delete()` operation itself fails, `PostDelete` callbacks will not be invoked at all. Also, `PostDelete` callbacks that are associated with transactional `delete()` operations will not run until the transaction successfully commits.

## PreGet

You can register a ` `[`PreGet`](https://web.archive.org/web/20160424230753/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/PreGet) callback to be called before getting entities of some specific kinds (or all kinds). You might use this, for example, to intercept some get requests and fetch the data from a cache instead of from the datastore.

```
import com.google.appengine.api.datastore.Entity;
import com.google.appengine.api.datastore.PreGetContext;
import com.google.appengine.api.datastore.PreGet;

class PreGetCallbacks {
  @PreGet(kinds = {"Customer", "Order"})
  public void preGet(PreGetContext context) {
    Entity e = MyCache.get(context.getCurrentElement());
    if (e != null) {
      context.setResultForCurrentElement(e);
    }
    // if not in cache, don't set result; let the normal datastore-fetch happen
  }
}
```

## PreQuery

You can register a ` `[`PreQuery`](https://web.archive.org/web/20160424230753/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/PreQuery) callback to be called before executing queries for specific kinds (or all kinds). You might use this, for example, to add an equality filter based on the logged in user. The callback is passed the query; by modifying it, the callback can cause a different query to be executed instead. Or the callback can throw an unchecked exception to prevent the query from being executed.

```
import com.google.appengine.api.datastore.PreQueryContext;
import com.google.appengine.api.datastore.PreQuery;
import com.google.appengine.api.datastore.Query;
import com.google.appengine.api.users.UserService;
import com.google.appengine.api.users.UserServiceFactory;

class PreQueryCallbacks {
  @PreQuery(kinds = {"Customer"})
  // Queries should only return data owned by the logged in user.
  public void preQuery(PreQueryContext context) {
    UserService users = UserServiceFactory.getUserService();
    context.getCurrentElement().setFilter(
        new FilterPredicate("owner", Query.FilterOperator.EQUAL, users.getCurrentUser()));
    }
}
```

## PostLoad

You can register a ` `[`PostLoad`](https://web.archive.org/web/20160424230753/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/PostLoad) callback to be called after loading entities of specific kinds (or all kinds). Here, "loading" might mean the result of a get or a query. This is useful for decorating loaded entities.

```
import com.google.appengine.api.datastore.PostLoadContext;
import com.google.appengine.api.datastore.PostLoad;

class PostLoadCallbacks {
   @PostLoad(kinds = {"Order"})
   public void postLoad(PostLoadContext context) {
     context.getCurrentElement().setProperty("read_timestamp",
                                             System.currentTimeMillis());
   }
}
```

## Batch Operations

When you execute a batch operation (a `put()` with multiple entities for example), your callbacks are invoked once per entity. You can access the entire batch of objects by calling [`CallbackContext.getElements()`](https://web.archive.org/web/20160424230753/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/CallbackContext#getElements()) on the argument to your callback method. This allows you to implement callbacks that operate on the entire batch instead of one element at a time.

```
import com.google.appengine.api.datastore.PrePut;
import com.google.appengine.api.datastore.PutContext;

class Validation {
    @PrePut(kinds = "TicketOrder")
    void checkBatchSize(PutContext context) {
        if (context.getElements().size() > 5) {
            throw new IllegalArgumentException(
                "Cannot purchase more than 5 tickets at once.");
        }
    }
}
```

If you need your callback to only execute once per batch, use [`CallbackContext.getCurrentIndex()`](https://web.archive.org/web/20160424230753/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/CallbackContext#getCurrentIndex()) to determine if you're looking at the first element of the batch.

```
import com.google.appengine.api.datastore.PrePut;
import com.google.appengine.api.datastore.PutContext;
import java.util.logging.Logger;

class Validation {
    static Logger logger = Logger.getLogger("logger");

    @PrePut
    void log(PutContext context) {
        if (context.getCurrentIndex() == 0) {
            logger.fine("Putting batch of size " + getElements().size());
        }
    }
}
```

## Async Operations

There are a few important things to know about how callbacks interact with async datastore operations. When you execute a `put()` or a `delete()` using the [async datastore API](https://web.archive.org/web/20160424230753/https://cloud.google.com/appengine/docs/java/datastore/async), any `Pre*` callbacks that you've registered will execute synchronously. Your `Post*` callbacks will execute synchronously as well, but not until you call any of the [`Future.get()`](https://web.archive.org/web/20160424230753/http://download.oracle.com/javase/6/docs/api/java/util/concurrent/Future.html#get()) methods to retrieve the result of the operation.

## Using Callbacks With App Engine Services

Callbacks, like any other code in your application, have access to the full range of App Engine backend services, and you can define as many as you want in a single class.

```
import com.google.appengine.api.datastore.DatastoreService;
import com.google.appengine.api.datastore.DatastoreServiceFactory;
import com.google.appengine.api.datastore.DeleteContext;
import com.google.appengine.api.datastore.PrePut;
import com.google.appengine.api.datastore.PostPut;
import com.google.appengine.api.datastore.PostDelete;
import com.google.appengine.api.datastore.PutContext;
import com.google.appengine.api.memcache.MemcacheService;
import com.google.appengine.api.memcache.MemcacheServiceFactory;
import com.google.appengine.api.taskqueue.Queue;
import com.google.appengine.api.taskqueue.QueueFactory;
import com.google.appengine.api.urlfetch.URLFetchService;
import com.google.appengine.api.urlfetch.URLFetchServiceFactory;

class ManyCallbacks {

    @PrePut(kinds = {"kind1", "kind2"})
    void foo(PutContext context) {
      MemcacheService ms = MemcacheServiceFactory.getMemcacheService();
      // ...
    }

    @PrePut
    void bar(PutContext context) {
      DatastoreService ds = DatastoreServiceFactory.getDatastoreService();
      // ...
    }

    @PostPut(kinds = {"kind1", "kind2"})
    void baz(PutContext context) {
        Queue q = QueueFactory.getDefaultQueue();
        // ...
    }

    @PostDelete(kinds = {"kind2"})
    void yam(DeleteContext context) {
        URLFetchService us = URLFetchServiceFactory.getURLFetchService();
        // ...
    }
}
```

## Common Errors To Avoid

There are a number of common errors to be aware of when implementing callbacks.

-   [Do Not Maintain Non-static State](#Do_Not_Maintain_Non_Static_State)
-   [Do Not Make Assumptions About Callback Execution Order](#Do_Not_Make_Assumptions_About_Callback_Execution_Order)
-   [One Callback Per Method](#One_Callback_Per_Method)
-   [Do Not Forget To Retrieve Async Results](#Do_Not_Forget_To_Retrieve_Async_Results)
-   [Avoid Infinite Loops](#Avoid_Infinite_Loops)

### Do Not Maintain Non-static State

```
import java.util.logging.Logger;
import com.google.appengine.api.datastore.PrePut;

class MaintainsNonStaticState {
    static Logger logger = Logger.getLogger("logger");
    // ERROR!
    // should be static to avoid assumptions about lifecycle of the instance
    boolean alreadyLogged = false;

    @PrePut
    void log(PutContext context) {
        if (!alreadyLogged) {
            alreadyLogged = true;
            logger.fine(
                "Finished deleting " + context.getCurrentElement().getKey());
        }
    }
}
```

### Do Not Make Assumptions About Callback Execution Order

`Pre*` callbacks will always execute before `Post*` callbacks, but it is not safe to make any assumptions about the order in which a `Pre*` callback executes relative to other `Pre*` callbacks, nor is it safe to make any assumptions about the order in which a `Post*` callback executes relative to other `Post*` callbacks.

```
import com.google.appengine.api.datastore.Entity;
import com.google.appengine.api.datastore.PrePut;

import java.util.HashSet;
import java.util.Set;

class MakesAssumptionsAboutOrderOfCallbackExecution {
    static Set<Key> paymentKeys = new HashSet<Key>();

    @PrePut(kinds = "Order")
    void prePutOrder(PutContext context) {
        Entity order = context.getCurrentElement();
        paymentKeys.addAll((Collection<Key>) order.getProperty("payment_keys"));
    }

    @PrePut(kinds = "Payment")
    void prePutPayment(PutContext context) {
        // ERROR!
        // Not safe to assume prePutOrder() has already run!
        if (!paymentKeys.contains(context.getCurrentElement().getKey()) {
            throw new IllegalArgumentException("Unattached payment!");
        }
    }
}
```

### One Callback Per Method

Even though a class can have an unlimited number of callback methods, a single method can only be associated with a single callback.

```
import com.google.appengine.api.datastore.PrePut;
import com.google.appengine.api.datastore.PostPut;

class MultipleCallbacksOnAMethod {
    @PrePut
    @PostPut // Compiler error!
    void foo(PutContext context) { }
}
```

### Do Not Forget To Retrieve Async Results

`Post*` callbacks do not run until you call [`Future.get()`](https://web.archive.org/web/20160424230753/http://download.oracle.com/javase/6/docs/api/java/util/concurrent/Future.html#get()) to retrieve the result of the operation. If you forget to call `Future.get()` before you finish servicing the HTTP request, your `Post*` callbacks will not run.

```
import com.google.appengine.api.datastore.AsyncDatastoreService;
import com.google.appengine.api.datastore.DatastoreServiceFactory;
import com.google.appengine.api.datastore.Entity;
import com.google.appengine.api.datastore.PostPut;
import com.google.appengine.api.datastore.PutContext;

import java.util.concurrent.Future;

class IgnoresAsyncResult {
    AsyncDatastoreService ds = DatastoreServiceFactory.getAsyncDatastoreService();

    @PostPut
    void collectSample(PutContext context) {
        Sampler.getSampler().collectSample(
            "put", context.getCurrentElement().getKey());
    }

    void addOrder(Entity order) {
        Future result = ds.put(order);
        // ERROR! Never calls result.get() so collectSample() will not run.
    }
}
```

### Avoid Infinite Loops

If you perform datastore operations in your callbacks, be careful not to fall into an infinite loop.

```
import com.google.appengine.api.datastore.DatastoreService;
import com.google.appengine.api.datastore.DatastoreServiceFactory;
import com.google.appengine.api.datastore.Entity;
import com.google.appengine.api.datastore.PrePut;
import com.google.appengine.api.datastore.PutContext;

class InfiniteLoop {
    DatastoreService ds = DatastoreServiceFactory.getDatastoreService();

    @PrePut
    void audit(PutContext context) {
        Entity original = ds.get(context.getCurrentElement().getKey());
        Entity auditEntity = new Entity(original.getKind() + "_audit");
        auditEntity.setPropertiesFrom(original);
        // INFINITE LOOP!
        // Restricting the callback to specific kinds would solve this.
        ds.put(auditEntity);
    }
}
```

## Using Callbacks With Eclipse

If you are developing your app with Eclipse you will need to perform a small number of configuration steps to use datastore callbacks. These steps are for Eclipse 3.7. We expect to make these steps unnecessary in a future release of the Google Plugin For Eclipse.

-   Open the Properties dialog for your Project (Project &gt; Properties)
-   Open the Annotation Processing dialog (Java Compiler &gt; Annotation Processing)
-   Check "Enable annotation processing"
-   Check "Enable processing in editor"
-   Open the Factory Path dialog (Java Compiler &gt; Annotation Processing &gt; Factory Path)
-   Click "Add External JARs"
-   Select &lt;SDK\_ROOT&gt;/lib/impl/appengine-api.jar (where SDK\_ROOT is the top level directory of your sdk installation)
-   Click "OK"

You can verify that callbacks are properly configured by implementing a method with multiple callbacks (see the code snippet under [One Callback Per Method](#One_Callback_Per_Method)). This should generate a compiler error.
