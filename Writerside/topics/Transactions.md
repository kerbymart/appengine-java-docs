# Transactions

  

The App Engine Datastore supports *transactions*. A transaction is an operation or set of operations that is atomic—either all of the operations in the transaction occur, or none of them occur. An application can perform multiple operations and calculations in a single transaction.

## Contents

1.  [Using transactions](#Java_Using_transactions)
2.  [What can be done in a transaction](#Java_What_can_be_done_in_a_transaction)
3.  [Isolation and consistency](#Java_Isolation_and_consistency)
4.  [Uses for transactions](#Java_Uses_for_transactions)
5.  [Transactional task enqueuing](#Java_Transactional_task_enqueuing)

## Using transactions

A *transaction* is a set of Datastore operations on one or more entities. Each transaction is guaranteed to be atomic, which means that transactions are never partially applied. Either all of the operations in the transaction are applied, or none of them are applied. Transactions have a maximum duration of 60 seconds with a 10 second idle expiration time after 30 seconds.

An operation may fail when:

-   Too many concurrent modifications are attempted on the same entity group.
-   The transaction exceeds a resource limit.
-   The Datastore encounters an internal error.

In all these cases, the Datastore API raises an exception.

**Note:** If your application receives an exception when committing a transaction, it does not always mean that the transaction failed. You can receive [`DatastoreTimeoutException`](https://web.archive.org/web/20160424230814/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/DatastoreTimeoutException) or [`DatastoreFailureException`](https://web.archive.org/web/20160424230814/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/DatastoreFailureException) exceptions in cases where transactions have been committed and eventually will be applied successfully. Whenever possible, make your Datastore transactions idempotent so that if you repeat a transaction, the end result will be the same.

Transactions are an optional feature of the Datastore; you're not required to use transactions to perform Datastore operations.

Here is an example of updating field named `vacationDays` in an entity of kind `Employee` named `Joe`:

```
DatastoreService datastore = DatastoreServiceFactory.getDatastoreService()
Transaction txn = datastore.beginTransaction();
try {
    Key employeeKey = KeyFactory.createKey("Employee", "Joe");
    Entity employee = datastore.get(employeeKey);
    employee.setProperty("vacationDays", 10);

    datastore.put(employee);

    txn.commit();
} finally {
    if (txn.isActive()) {
        txn.rollback();
    }
}
```

Note that in order to keep our examples more succinct we sometimes omit the `finally` block that performs a rollback if the transaction is still active. In production code it is important to ensure that every transaction is either explicitly committed or rolled back.

### Entity groups

Every entity belongs to an *entity group*, a set of one or more entities that can be manipulated in a single transaction. Entity group relationships tell App Engine to store several entities in the same part of the distributed network. A transaction sets up Datastore operations for an entity group, and all of the operations are applied as a group, or not at all if the transaction fails.

When the application creates an entity, it can assign another entity as the *parent* of the new entity. Assigning a parent to a new entity puts the new entity in the same entity group as the parent entity.

An entity without a parent is a *root* entity. An entity that is a parent for another entity can also have a parent. A chain of parent entities from an entity up to the root is the *path* for the entity, and members of the path are the entity's *ancestors*. The parent of an entity is defined when the entity is created, and cannot be changed later.

Every entity with a given root entity as an ancestor is in the same entity group. All entities in a group are stored in the same Datastore node. A single transaction can modify multiple entities in a single group, or add new entities to the group by making the new entity's parent an existing entity in the group. The following code snippet demonstrates transactions on various types of entities:

```
DatastoreService datastore = DatastoreServiceFactory.getDatastoreService();
Entity person = new Entity("Person", "tom");
datastore.put(person);

// Transactions on root entities
Transaction tx = datastore.beginTransaction();

Entity tom = datastore.get(person.getKey());
tom.setProperty("age", 40);
datastore.put(tom);
tx.commit();

// Transactions on child entities
tx = datastore.beginTransaction();
tom = datastore.get(person.getKey());
Entity photo = new Entity("Photo", tom.getKey());

// Create a Photo that is a child of the Person entity named "tom"
photo.setProperty("photoUrl", "http://domain.com/path/to/photo.jpg");
datastore.put(photo);
tx.commit();

// Transactions on entities in different entity groups
tx = datastore.beginTransaction();
tom = datastore.get(person.getKey());
Entity photoNotAChild = new Entity("Photo");
photoNotAChild.setProperty("photoUrl", "http://domain.com/path/to/photo.jpg");
datastore.put(photoNotAChild);

// Throws IllegalArgumentException because the Person entity
// and the Photo entity belong to different entity groups.
tx.commit();
```

### Creating an entity in a specific entity group

When your application constructs a new entity, you can assign it to an entity group by supplying the key of another entity. The example below constructs the key of a MessageBoard entity, then uses that key to create and persist a Message entity that resides in the same entity group as the MessageBoard:

```
DatastoreService datastore = DatastoreServiceFactory.getDatastoreService();

String messageTitle = req.getParameter("title");
String messageText = req.getParameter("body");
Date postDate = new Date();

Transaction txn = datastore.beginTransaction();

Key messageBoardKey = KeyFactory.createKey("MessageBoard", boardName);

Entity message = new Entity("Message", messageBoardKey);
message.setProperty("message_title", messageTitle);
message.setProperty("message_text", messageText);
message.setProperty("post_date", postDate);
datastore.put(message);

txn.commit();
```

### Using cross-group transactions

Using a [cross-group (XG) transaction](https://web.archive.org/web/20160424230814/https://cloud.google.com/appengine/docs/java/datastore/#Java_Cross_group_transactions) is similar to using a single group transaction, except that you need to specify that you want the transaction to be XG when you begin the transaction, using [`TransactionOptions`](https://web.archive.org/web/20160424230814/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/TransactionOptions):

```
import com.google.appengine.api.datastore.DatastoreService;
import com.google.appengine.api.datastore.DatastoreServiceFactory;
import com.google.appengine.api.datastore.Entity;
import com.google.appengine.api.datastore.Transaction;
import com.google.appengine.api.datastore.TransactionOptions;

void myTxn() {
  DatastoreService datastore = DatastoreServiceFactory.getDatastoreService();
  TransactionOptions options = TransactionOptions.Builder.withXG(true);
  Transaction txn = datastore.beginTransaction(options);

  Entity a = new Entity("A");
  a.setProperty("a", 22);
  datastore.put(txn, a);

  Entity b = new Entity("B");
  b.setProperty("b", 11);
  datastore.put(txn, b);

  txn.commit();
}
```

## What can be done in a transaction

The Datastore imposes restrictions on what can be done inside a single transaction.

All Datastore operations in a transaction must operate on entities in the same entity group if the transaction is a single group transaction, or on entities in a maximum of twenty-five entity groups if the transaction is a [cross-group (XG) transaction](https://web.archive.org/web/20160424230814/https://cloud.google.com/appengine/docs/java/datastore/#Java_Cross_group_transactions). This includes querying for entities by ancestor, retrieving entities by key, updating entities, and deleting entities. Notice that each root entity belongs to a separate entity group, so a single transaction cannot create or operate on more than one root entity unless it is an XG transaction.

When two or more transactions simultaneously attempt to modify entities in one or more common entity groups, only the first transaction to commit its changes can succeed; all the others will fail on commit. Because of this design, using entity groups limits the number of concurrent writes you can do on any entity in the groups. When a transaction starts, App Engine uses [optimistic concurrency control](https://web.archive.org/web/20160424230814/http://en.wikipedia.org/wiki/Optimistic_concurrency_control) by checking the last update time for the entity groups used in the transaction. Upon commiting a transaction for the entity groups, App Engine again checks the last update time for the entity groups used in the transaction. If it has changed since our initial check, an exception is thrown. For an explanation of entity groups, see the [Datastore Overview](https://web.archive.org/web/20160424230814/https://cloud.google.com/appengine/docs/java/datastore/#Java_Ancestor_paths) page.

An app can perform a [query](https://web.archive.org/web/20160424230814/https://cloud.google.com/appengine/docs/java/datastore/queries) during a transaction, but only if it includes an ancestor filter. An app can also get Datastore entities by key during a transaction. You can prepare keys prior to the transaction, or you can build keys inside the transaction with key names or IDs.

## Isolation and consistency

Outside of transactions, the Datastore's isolation level is closest to read committed. Inside of transactions, serializable isolation is enforced. This means that another transaction cannot concurrently modify the data that is **read or modified** by this transaction. Read the [serializable isolation](https://web.archive.org/web/20160424230814/http://en.wikipedia.org/wiki/Serializability) wiki and the [Transaction Isolation](https://web.archive.org/web/20160424230814/https://cloud.google.com/appengine/articles/transaction_isolation) article for more information on isolation levels.

In a transaction, all reads reflect the current, consistent state of the Datastore at the time the transaction started. Queries and gets inside a transaction are guaranteed to see a single, consistent snapshot of the Datastore as of the beginning of the transaction. Entities and index rows in the transaction's entity group are fully updated so that queries return the complete, correct set of result entities, without the false positives or false negatives described in [Transaction Isolation](https://web.archive.org/web/20160424230814/https://cloud.google.com/appengine/articles/transaction_isolation) that can occur in queries outside of transactions.

This consistent snapshot view also extends to reads after writes inside transactions. Unlike with most databases, queries and gets inside a Datastore transaction do *not* see the results of previous writes inside that transaction. Specifically, if an entity is modified or deleted within a transaction, a query or get returns the *original* version of the entity as of the beginning of the transaction, or nothing if the entity did not exist then.

## Uses for transactions

This example demonstrates one use of transactions: updating an entity with a new property value relative to its current value. Since the Datastore API does not retry transactions, we can add logic for the transaction to be retried in case another request updates the same MessageBoard or any of its Messages at the same time.

```
int retries = 3;
while (true) {
    Transaction txn = datastore.beginTransaction();
    try {
        Key boardKey = KeyFactory.createKey("MessageBoard", boardName);
        Entity messageBoard = datastore.get(boardKey);

        long count = (Long) messageBoard.getProperty("count");
        ++count;
        messageBoard.setProperty("count", count);
        datastore.put(messageBoard);

        txn.commit();
        break;
    } catch (ConcurrentModificationException e) {
        if (retries == 0) {
            throw e;
        }
        // Allow retry to occur
        --retries;
    } finally {
        if (txn.isActive()) {
            txn.rollback();
        }
    }
}
```
**Warning:** The above sample depicts transactionally incrementing a counter only for the sake of simplicity. If your app has counters that are updated frequently, you should not increment them transactionally, or even within a single entity. A best practice for working with counters is to use a technique known as [counter-sharding](https://web.archive.org/web/20160424230814/https://cloud.google.com/appengine/articles/sharding_counters).

This requires a transaction because the value may be updated by another user after this code fetches the object, but before it saves the modified object. Without a transaction, the user's request uses the value of `count` prior to the other user's update, and the save overwrites the new value. With a transaction, the application is told about the other user's update. If the entity is updated during the transaction, then the transaction fails with a `ConcurrentModificationException`. The application can repeat the transaction to use the new data.

**Note:** In extremely rare cases, the transaction is fully committed even if a transaction returns a timeout or internal error exception. For this reason, it's best to make transactions idempotent whenever possible.

Another common use for transactions is to fetch an entity with a named key, or create it if it doesn't yet exist:

```
Transaction txn = datastore.beginTransaction();
try {
    Key boardKey = KeyFactory.createKey("MessageBoard", "Foo");
    Entity messageBoard = datastore.get(boardKey);
} catch (EntityNotFoundException e) {
    messageBoard = new Entity("MessageBoard", boardName);
    messageBoard.setProperty("count", 0L);
    boardKey = datastore.put(messageBoard);
}
txn.commit();
```

As before, a transaction is necessary to handle the case where another user is attempting to create or update an entity with the same string ID. Without a transaction, if the entity does not exist and two users attempt to create it, the second overwrites the first without knowing that it happened. With a transaction, the second attempt fails atomically. If it makes sense to do, the application can try again to fetch the entity and update it.

When a transaction fails, you can have your app retry the transaction until it succeeds, or you can let your users deal with the error by propagating it to your app's user interface level. You do not have to create a retry loop around every transaction.

**Note:** A transaction should happen as quickly as possible to reduce the likelihood that the entities used by the transaction will change, causing the transaction to fail. As much as possible, prepare data outside of the transaction, then execute the transaction to perform Datastore operations that depend on a consistent state. The application should prepare keys for objects used outside the transaction, then fetch the entities inside the transaction.

Finally, you can use a transaction to read a consistent snapshot of the Datastore. This can be useful when multiple reads are needed to render a page or export data that must be consistent. These kinds of transactions are often called *read-only* transactions, since they perform no writes. Read-only single-group transactions never fail due to concurrent modifications, so you don't have to implement retries upon failure. However, XG transactions can fail due to concurrent modifications, so these should have retries. Committing and rolling back a read-only transaction are both no-ops.

```
DatastoreService ds = DatastoreServiceFactory.getDatastoreService();

// Display information about a message board and its first 10 messages.
Key boardKey = KeyFactory.createKey("MessageBoard", boardName);

Transaction txn = datastore.beginTransaction();

Entity messageBoard = datastore.get(boardKey);
long count = (Long) messageBoard.getProperty("count");

Query q = new Query("Message", boardKey);

// This is an ancestor query.
PreparedQuery pq = datastore.prepare(txn, q);
List<Entity> messages = pq.asList(FetchOptions.Builder.withLimit(10)));

txn.commit();
```

## Transactional task enqueuing

You can enqueue a task as part of a Datastore transaction, such that the task is only enqueued—and guaranteed to be enqueued—if the transaction is committed successfully. If the transaction does get committed, the task is guaranteed to be enqueued. Once enqueued, the task is not guaranteed to execute immediately and any operations performed within the task execute independent of the original transaction. The task retries until it succeeds. This applies to any task enqueued in the context of a transaction.

Transactional tasks are useful because they allow you to enlist non-Datastore actions in a Datastore transaction (such as sending an email to confirm a purchase). You can also tie Datastore actions to the transaction, such as committing changes to additional entity groups outside of the transaction if and only if the transaction succeeds.

An application cannot insert more than five [transactional tasks](https://web.archive.org/web/20160424230814/https://cloud.google.com/appengine/docs/java/taskqueue/#Java_Tasks_within_transactions) into [task queues](https://web.archive.org/web/20160424230814/https://cloud.google.com/appengine/docs/java/taskqueue/#Java_Queue_concepts) during a single transaction. Transactional tasks must not have user-specified names.

```
DatastoreService datastore = DatastoreServiceFactory.getDatastoreService();
Queue queue = QueueFactory.getDefaultQueue();
Transaction txn = datastore.beginTransaction();
// ...

queue.add(TaskOptions.Builder.withUrl("/path/to/handler"));

// ...

txn.commit();
```
