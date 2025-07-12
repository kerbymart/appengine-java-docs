# Async Datastore API

  

The Async Datastore API allows you to make parallel, non-blocking calls to the datastore and to retrieve the results of these calls at a later point in the handling of the request. This documentation describes the following aspects of the Async Datastore API:

1.  [Working with the Async Datastore Service](#Working_with_the_AsyncDatastoreService)
2.  [Working with Async Transactions](#Working_with_Async_Transactions)
3.  [Working with Futures](#Working_with_Futures)
4.  [Async Queries](#Async_Queries)
5.  [When To Use Async Datastore Calls](#When_To_Use_Async_Datastore_Calls)

## Working with the Async Datastore Service

With the async datastore API, you make datastore calls using methods of the [AsyncDatastoreService](https://web.archive.org/web/20160424225420/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/AsyncDatastoreService) interface. You get this object by calling the [getAsyncDatastoreService()](https://web.archive.org/web/20160424225420/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/DatastoreServiceFactory#getAsyncDatastoreService()) class method of the [DatastoreServiceFactory](https://web.archive.org/web/20160424225420/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/DatastoreServiceFactory) class.

```
import com.google.appengine.api.datastore.AsyncDatastoreService;
import com.google.appengine.api.datastore.DatastoreServiceFactory;

// ...
AsyncDatastoreService datastore = DatastoreServiceFactory.getAsyncDatastoreService();
```

`AsyncDatastoreService` supports the same operations as [DatastoreService](https://web.archive.org/web/20160424225420/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/DatastoreService), except most methods immediately return a [Future](https://web.archive.org/web/20160424225420/http://download.oracle.com/javase/6/docs/api/java/util/concurrent/Future.html) whose result you can block on at some later point. For example, [DatastoreService.get()](https://web.archive.org/web/20160424225420/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/DatastoreService#get(com.google.appengine.api.datastore.Key)) returns an [Entity](https://web.archive.org/web/20160424225420/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/Entity) but [AsyncDatastoreService.get()](https://web.archive.org/web/20160424225420/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/AsyncDatastoreService#get(com.google.appengine.api.datastore.Key)) returns a `Future<Entity>`.

```
// ...

Key key = KeyFactory.createKey("Employee", "Max");
// Async call returns immediately
Future<Entity> entityFuture = datastore.get(key);

// Do other stuff while the get operation runs in the background...

// Blocks if the get operation has not finished, otherwise returns instantly
Entity entity = entityFuture.get();
```

**Note:** Exceptions are not thrown until you call the get() method. Calling this method allows you to verify that the asynchronous operation succeeded.

If you have an `AsyncDatastoreService` but need to execute an operation synchronously, invoke the appropriate `AsyncDatastoreService` method and then immediately block on the result:

```
// ...

Entity entity = new Employee("Employee", "Alfred");
// ... populate entity properties

// Make a sync call via the async interface
Key key = datastore.put(key).get();
```

## Working with Async Transactions

Async datastore API calls can participate in transactions just like synchronous calls. Here's a function that adjusts the salary of an `Employee` and writes an additional `SalaryAdjustment` entity in the same entity group as the `Employee`, all within a single transaction.

```
void giveRaise(AsyncDatastoreService datastore, Key employeeKey, long raiseAmount)
        throws Exception {
    Future<Transaction> txn = datastore.beginTransaction();

    // Async call to lookup the Employee entity
    Future<Entity> employeeEntityFuture = datastore.get(employeeKey);

    // Create and put a SalaryAdjustment entity in parallel with the lookup
    Entity adjustmentEntity = new Entity("SalaryAdjustment", employeeKey);
    adjustmentEntity.setProperty("adjustment", raiseAmount);
    adjustmentEntity.setProperty("adjustmentDate", new Date());
    datastore.put(adjustmentEntity);

    // Fetch the result of our lookup to make the salary adjustment
    Entity employeeEntity = employeeEntityFuture.get();
    long salary = (Long) employeeEntity.getProperty("salary");
    employeeEntity.setProperty("salary", salary + raiseAmount);

    // Re-put the Employee entity with the adjusted salary.
    datastore.put(employeeEntity);
    txn.get().commit(); // could also call txn.get().commitAsync() here
}
```

This sample illustrates an important difference between async calls without transactions and async calls with transactions. When you are not using a transaction, the only way to ensure that an individual async call has completed is to fetch the return value of the `Future` that was returned when the call was made. When you are using a transaction, calling [Transaction.commit()](https://web.archive.org/web/20160424225420/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/Transaction#commit()) blocks on the result of all async calls made since the transaction started before committing it.

So, in our example above, even though our async call to insert the `SalaryAdjustment` entity may still be outstanding when we call `commit()`, the commit will not happen until the insert completes. Similarly, if you elect to call `commitAsync()` instead of `commit()`, invoking `get()` on the `Future` returned by `commitAsync()` blocks until all oustanding async calls have completed.

**Note:** Transactions are associated with a specific thread, not a specific instance of `DatastoreService` or `AsyncDatastoreService`. This means that if you initiate a transaction with a `DatastoreService` and perform an async call with an `AsyncDatastoreService`, the async call participates in the transaction. Or, to put it more succinctly, `DatastoreService.getCurrentTransaction()` and `AsyncDatastoreService.getCurrentTransaction()` always returns the same `Transaction`.

## Working with Futures

The [Future Javadoc](https://web.archive.org/web/20160424225420/http://download.oracle.com/javase/6/docs/api/java/util/concurrent/Future.html) explains most of what you need to know to successfully work with a `Future` returned by the Async Datastore API, but there are a few App Engine-specific things you need to be aware of:

-   When you call [Future.get(long timeout, TimeUnit unit)](https://web.archive.org/web/20160424225420/http://download.oracle.com/javase/6/docs/api/java/util/concurrent/Future.html#get(long,%20java.util.concurrent.TimeUnit)), the timeout is separate from any RPC deadline set when you created the `AsyncDatastoreService`. For more information, see the [Datastore Queries](https://web.archive.org/web/20160424225420/https://cloud.google.com/appengine/docs/java/datastore/queries#Java_Data_consistency) page.
-   When you call [Future.cancel(boolean mayInterruptIfRunning)](https://web.archive.org/web/20160424225420/http://download.oracle.com/javase/6/docs/api/java/util/concurrent/Future.html#cancel(boolean)) and that call returns `true`, that does not necessarily mean that the state of your datastore is unchanged. In other words, cancelling a `Future` is not the same as rolling back a transaction.

## Async Queries

We do not currently expose an explicitly async API for queries. However, when you invoke [PreparedQuery.asIterable()](https://web.archive.org/web/20160424225420/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/PreparedQuery#asIterable()), [PreparedQuery.asIterator()](https://web.archive.org/web/20160424225420/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/PreparedQuery#asIterator()) or [PreparedQuery.asList(FetchOptions fetchOptions)](https://web.archive.org/web/20160424225420/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/PreparedQuery#asList(com.google.appengine.api.datastore.FetchOptions)), both DatastoreService and AsyncDatastoreService immediately return and asynchronously prefetch results. This allows your application to perform work in parallel while query results are fetched.

```
// ...

Query q1 = new Query("Salesperson");
q1.setFilter(new FilterPredicate("dateOfHire", FilterOperator.LESS_THAN, oneMonthAgo));

// Returns instantly, query is executing in the background.
Iterable<Entity> recentHires = datastore.prepare(q1).asIterable();

Query q2 = new Query("Customer");
q2.setFilter(new FilterPredicate("lastContact", FilterOperator.GREATER_THAN, oneYearAgo));

// Also returns instantly, query is executing in the background.
Iterable<Entity> needsFollowup = datastore.prepare(q2).asIterable();

schedulePhoneCall(recentHires, needsFollowUp);
```

## When To Use Async Datastore Calls

The operations exposed by the [DatastoreService](https://web.archive.org/web/20160424225420/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/DatastoreService) interface are synchronous. For example, when you call `DatastoreService.get()`, your code blocks until the call to the datastore completes. If the only thing your application needs to do is render the result of the `get()` in HTML, blocking until the call is complete is a perfectly reasonable thing to do. However, if your application needs the result of the `get()` plus the result of a `Query` to render the response, and if the `get()` and the `Query` don't have any data dependencies, then waiting until the `get()` completes to initiate the `Query` is a waste of time. Here is an example of some code that can be improved by using the async API:

```
DatastoreService datastore = DatastoreServiceFactory.getDatastoreService();
Key empKey = KeyFactory.createKey("Employee", "Max");

// Read employee data from the Datastore
Entity employee = datastore.get(empKey); // Blocking for no good reason!

// Fetch payment history
Query query = new Query("PaymentHistory");
PreparedQuery pq = datastore.prepare(query);
List<Entity> result = pq.asList(FetchOptions.Builder.withLimit(10));
renderHtml(employee, result);
```

Instead of waiting for the `get()` to complete, use an instance of [AsyncDatastoreService](https://web.archive.org/web/20160424225420/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/AsyncDatastoreService) to execute the call asynchronously:

```
AsyncDatastoreService datastore = DatastoreServiceFactory.getAsyncDatastoreService();
Key empKey = KeyFactory.createKey("Employee", "Max");

// Read employee data from the Datastore
Future<Entity> employeeFuture = datastore.get(empKey); // Returns immediately!

// Fetch payment history for the employee
Query query = new Query("PaymentHistory", empKey);
PreparedQuery pq = datastore.prepare(query);

// Run the query while the employee is being fetched
List<Entity> result = pq.asList(FetchOptions.Builder.withLimit(10));
// Implicitly performs query asynchronously
Entity employee = employeeFuture.get(); // Blocking!
renderHtml(employee, result); 
```

The synchronous and asynchronous versions of this code use similar amounts of CPU (after all, they both perform the same amount of work), but since the asynchronous version allows the two datastore operations to execute in parallel, the asynchronous version has lower latency. In general, if you need to perform multiple datastore operations that don't have any data dependencies, the `AsyncDatastoreService` can significantly improve latency.
