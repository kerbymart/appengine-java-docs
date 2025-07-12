# Task Queue Java API Overview

  

[Python](https://web.archive.org/web/20160424223743/https://cloud.google.com/appengine/docs/python/taskqueue/ "View this page in the Python runtime") <span style="color: #ddd; padding: 0em .5em;">|</span><span style="color: black; font-weight:bold">Java</span> <span style="color: #ddd; padding: 0em .5em;">|</span>[PHP](https://web.archive.org/web/20160424223743/https://cloud.google.com/appengine/docs/php/taskqueue/ "View this page in the PHP runtime") <span style="color: #ddd; padding: 0em .5em;">|</span>[Go](https://web.archive.org/web/20160424223743/https://cloud.google.com/appengine/docs/go/taskqueue/ "View this page in the Go runtime")

Related videos

Related topics

-   <a href="https://web.archive.org/web/20160424223743/https://developers.google.com/appengine/articles/deferred" class="gc-analytics-event" data-category="Sidebar" data-action="Related topic" data-label="Background work with the deferred library">Background work with the deferred library</a>
-   <a href="https://web.archive.org/web/20160424223743/https://dl.google.com/googleio/2010/app-engine-data-pipelines.pdf" class="gc-analytics-event" data-category="Sidebar" data-action="Related topic" data-label="Building high\u002Dthroughput data pipelines on Google App Engine">Building high-throughput data pipelines on Google App Engine</a>
-   <a href="https://web.archive.org/web/20160424223743/https://code.google.com/p/appengine-pipeline/" class="gc-analytics-event" data-category="Sidebar" data-action="Related topic" data-label="Google App Engine Pipeline API">Google App Engine Pipeline API</a>

Related samples

-   <a href="https://web.archive.org/web/20160424223743/https://github.com/GoogleCloudPlatform/appengine-last-across-the-finish-line-python" class="gc-analytics-event" data-category="Sidebar" data-action="Related sample" data-label="Last Across the Finish Line (Python)">Last Across the Finish Line (Python)</a>

[<img src="https://web.archive.org/web/20160424223743im_/https://cloud.google.com/cloud/images/stack_overflow_questions.png" title="Stack Overflow Questions" data-border="0" width="240" height="53" />](https://web.archive.org/web/20160424223743/http://stackoverflow.com/questions/tagged/google-app-engine+task-queue)

[](https://web.archive.org/web/20160424223743/http://stackoverflow.com/feeds/tag?sort=votes&tagnames=google-app-engine%2Btask-queue)

<span class="link gc-analytics-event" category="Sidebar" data-action="Stack Overflow question click"></span>

<a href="https://web.archive.org/web/20160424223743/http://stackoverflow.com/questions/tagged/google-app-engine+task-queue?sort=votes" class="gc-analytics-event" data-category="Sidebar" data-action="Stack Overflow See more link">See more...</a>

With the Task Queue API, applications can perform work outside of a user request, initiated by a user request. If an app needs to execute some background work, it can use the Task Queue API to organize that work into small, discrete units, called [tasks](#Java_Task_concepts). The app adds tasks to [task queues](#Java_Queue_concepts) to be executed later.

App Engine provides two different queue configurations:

1.  [Push queues](https://web.archive.org/web/20160424223743/https://cloud.google.com/appengine/docs/java/taskqueue/overview-push) process tasks based on the processing rate configured in the [queue definition](https://web.archive.org/web/20160424223743/https://cloud.google.com/appengine/docs/java/config/queue). App Engine automatically scales processing capacity to match your queue configuration and processing volume, and also deletes tasks after processing. Push queues are the default.
2.  [Pull queues](https://web.archive.org/web/20160424223743/https://cloud.google.com/appengine/docs/java/taskqueue/overview-pull) allow a task consumer (either your application or code external to your application) to lease tasks at a specific time for processing within a specific timeframe. Pull queues give you more control over when tasks are processed, and also allow you to integrate your application with non-App-Engine code using the experimental [Task Queue REST API](https://web.archive.org/web/20160424223743/https://cloud.google.com/appengine/docs/java/taskqueue/rest). When using pull queues, your application needs to handle scaling of instances based on processing volume, and also needs to delete tasks after processing.

This page provides the basic concepts common to both types of queues. Once you've understood the basics, you can check out the [queue configuration page](https://web.archive.org/web/20160424223743/https://cloud.google.com/appengine/docs/java/config/queue), [push queue overview](https://web.archive.org/web/20160424223743/https://cloud.google.com/appengine/docs/java/taskqueue/overview-push), and [pull queue overview](https://web.archive.org/web/20160424223743/https://cloud.google.com/appengine/docs/java/taskqueue/overview-pull) to see how to configure and use these two types of queues.

1.  [Task Queue concepts](#Java_Queue_concepts)
    -   [The default queue](#Java_The_default_queue)
    -   [Named queues](#Java_Named_queues)
    -   [Task queues in the Cloud Platform Console](#Java_Task_queues_in_the_Developers_Console)
    -   [Checking task queue statistics](#Java_Checking_task_queue_statistics)
2.  [Task concepts](#Java_Task_concepts)
    -   [Task names](#Java_Task_names)
    -   [Tasks within transactions](#Java_Tasks_within_transactions)
    -   [Deleting tasks](#Java_Deleting_tasks)
3.  [Asynchronous operations](#Java_Asynchronous_operations)

## Task Queue concepts

Tasks queues are an efficient and powerful tool for background processing; they allow your application to define tasks, add them to a queue, and then use the queue to process them in aggregate. You name queues and [configure their properties](https://web.archive.org/web/20160424223743/https://cloud.google.com/appengine/docs/java/config/queue) in a configuration file named `queue.xml`.

Push queues function only within the App Engine environment. These queues are the best choice for applications whose tasks work only with App Engine tools and services. With push queues, you simply configure a queue and add tasks to it. App Engine handles the rest. Push queues are easier to implement, but are restricted to use within App Engine. For more information about push queues and examples of how to use them, see [Using Push Queues](https://web.archive.org/web/20160424223743/https://cloud.google.com/appengine/docs/java/taskqueue/overview-push).

If you want to use a different system to consume tasks, pull queues are your best choice. In pull queues, a task consumer (either in your App Engine application, a [backend](https://web.archive.org/web/20160424223743/https://cloud.google.com/appengine/docs/java/backends), or code outside of App Engine) leases a specific number of tasks from a specific queue for a specific timeframe. After leasing tasks from a pull queue, the task consumer is responsible for deleting them. If you are consuming tasks from within App Engine, you can use calls from the [com.google.appengine.api.taskqueue package](https://web.archive.org/web/20160424223743/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/taskqueue/package-summary). If you are consuming tasks from outside of App Engine, you need to use the [Task Queue REST API](https://web.archive.org/web/20160424223743/https://cloud.google.com/appengine/docs/java/taskqueue/rest). Pull queues give you more power and flexibility over when and where tasks are processed, but they require you to handle scaling of workers based on processing volume. Your task consumer also needs to delete tasks after processing.

In summary, push queues allow you to process tasks within App Engine at a steady rate and App Engine scales computing resources according to the number of tasks in your queue. Pull queues allow an alternate task consumer to process tasks at a specific time, either in or outside App Engine, but your application needs to scale workers based on processing volume, as well as delete tasks after processing.

You can read more about these two types of queues in the [Using Push Queues](https://web.archive.org/web/20160424223743/https://cloud.google.com/appengine/docs/java/taskqueue/overview-push) and [Using Pull Queues](https://web.archive.org/web/20160424223743/https://cloud.google.com/appengine/docs/java/taskqueue/overview-pull).

### The default queue

For convenience, App Engine provides a default push queue for each application (there is no default pull queue). If you do not name a queue for a task, App Engine automatically inserts it into the default queue. You can use this queue immediately without any additional configuration. All modules and versions of the same application share the same default task queue.

The default queue is preconfigured with a throughput rate of 5 task invocations per second. If you want to change the preconfigured settings, simply define a queue named `default` in `queue.xml`. Code may always insert new tasks into the default queue, but if you wish to disable execution of these tasks, you may do so by clicking the Pause Queue button in the Cloud Platform Console [Task queues page](https://web.archive.org/web/20160424223743/https://cloud.google.com/appengine/docs/java/console/#queues).

### Named queues

While the default queue makes it easy to enqueue tasks with no configuration, you can also create custom queues by defining them in `queue.xml`. Custom queues allow you to more effectively handle task processing by grouping similar types of tasks. You can control the processing rate—and a number of other settings—based specifically on the type of task in each queue.

All versions of an application share the same named task queues. A queue name can contain uppercase and lowercase letters, numbers, and hyphens. The maximum length for a queue name is 100 characters.

For more information about configuring queues in `queue.xml`, please see [Task Queue Configuration](https://web.archive.org/web/20160424223743/https://cloud.google.com/appengine/docs/java/config/queue).

### Task queues in the Cloud Platform Console

**Important**: The default queue appears in the Console only after the app has enqueued a task to it for the first time.

You can manage task queues for an application using the Cloud Platform Console [Task queues page](https://web.archive.org/web/20160424223743/https://cloud.google.com/appengine/docs/java/console/#queues). The **Task Queue** tab lists all of the queues in the application. Clicking on a queue name brings up the **Task Queue Details** page where you can see all of the tasks scheduled to run in a queue and you can manually delete individual tasks or purge every task from a queue. This is useful if a task in a push queue cannot be completed successfully and is stuck waiting to be retried. You can also pause and resume a queue on this page.

You can view details of individual tasks by clicking the task name from the list of tasks on the **Task Queue Details** page. This page allows you to debug why a task did not run successfully. You can also see information about the previous run of the task as well as the task's body.

### Checking task queue statistics

You can view queue statistics in the Console to determine performance and status. However, you can also access queue statistics programmatically using the [QueueStatistics](https://web.archive.org/web/20160424223743/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/taskqueue/QueueStatistics.html) class.

## Task concepts

A *task* is a unit of work to be performed by the application. Each task is an object of the [`TaskOptions`](https://web.archive.org/web/20160424223743/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/taskqueue/TaskOptions) class. Each Task object contains an endpoint (with a request handler for the task and an optional data payload that parameterizes the task). You can enqueue push tasks to a queue defined in `queue.xml`. Push tasks and pull tasks are defined differently; see the [Using Push Queues](https://web.archive.org/web/20160424223743/https://cloud.google.com/appengine/docs/java/taskqueue/overview-push) and [Using Pull Queues](https://web.archive.org/web/20160424223743/https://cloud.google.com/appengine/docs/java/taskqueue/overview-pull) for specific usage details.

### Task names

You may optionally assign a name to a task. Task names must be unique within a queue. Once a named task is added to a queue, any subsequent attempt to insert another task with the same name into the same queue will fail. If a task successfully executes or is deleted, you must wait for a period of 10 days before you can insert a new task with that same name.

Note that task names do not provide an absolute guarantee of once-only semantics. In extremely rare cases, multiple calls to create a task of the same name may succeed, but in this event, only one of the tasks would be executed. It's also possible in exceptional cases for a task to run more than once.

A task name can contain uppercase and lowercase letters, numbers, underscores, and hyphens. The maximum length for a task name is 500 characters.

A task is deleted immediately upon successful execution or deletion, or after a maximum number of failures. The task name can then be re-used after 10 days. Attempting to create another task with that same name during this 10-day period will result in an "item exists" error. To avoid issues with task name re-use, we recommend that you let App Engine generate the task name automatically.

### Tasks within transactions

You can enqueue a task as part of a datastore transaction, such that the task is only enqueued—and guaranteed to be enqueued—if the transaction is committed successfully. Tasks added within a [transaction](https://web.archive.org/web/20160424223743/https://cloud.google.com/appengine/docs/java/datastore/transactions) are considered to be a part of it and have the same level of [isolation and consistency](https://web.archive.org/web/20160424223743/https://cloud.google.com/appengine/docs/java/datastore/transactions#Java_Isolation_and_consistency).

An application cannot insert more than five [transactional tasks](https://web.archive.org/web/20160424223743/https://cloud.google.com/appengine/docs/java/taskqueue/#Java_Tasks_within_transactions) into [task queues](https://web.archive.org/web/20160424223743/https://cloud.google.com/appengine/docs/java/taskqueue/#Java_Queue_concepts) during a single transaction. Transactional tasks must not have user-specified names.

The following code sample demonstrates how to insert transactional tasks into a push queue as part of a datastore transaction:

```
DatastoreService ds = DatastoreServiceFactory.getDatastoreService();
Queue queue = QueueFactory.getDefaultQueue();
try {
    Transaction txn = ds.beginTransaction();

    // ...

    queue.add(TaskOptions.Builder.withUrl("/path/to/my/worker"));

    // ...
    txn.commit();
} catch (DatastoreFailureException e) {
}
```

## Deleting tasks

**Warning**: Do not create new tasks in the same second as the purge call. Tasks created in close temporal proximity to a purge call will also be purged.

You can delete tasks through the Cloud Platform Console or programatically. It may take several hours to reclaim the quota used by purged queues. It takes the system about 20 seconds to notice that the queue has been purged. In push queues, tasks continue to execute during this time.

You have two ways to delete tasks:

1.  Using the [Cloud Platform Console](https://web.archive.org/web/20160424223743/https://cloud.google.com/appengine/docs/java/taskqueue/#Java_Task_queues_in_the_Developers_Console).
2.  Programmatically, using [purge()](https://web.archive.org/web/20160424223743/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/taskqueue/Queue#purge()) to delete all tasks from the specified queue, or using [deleteTask()](https://web.archive.org/web/20160424223743/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/taskqueue/Queue#deleteTask(java.lang.String)) to delete an individual task:

```
// Purge entire queue...
Queue queue = QueueFactory.getQueue("foo");
queue.purge();
    
// Delete an individual task...
Queue q = QueueFactory.getQueue("queue1");
q.deleteTask("foo")
```

## Asynchronous operations

The taskqueue library also provides an asynchronous interface for latency-sensitive applications.

However, in most cases, you can just use the synchronous interface because adding a task is usually a fast operation; the median time to add a task is 5ms and 1 out of every 1000 tasks can take up to 300ms. Periodic incidents, such as back-end upgrades, can cause spikes to 1 out of every 1000 tasks taking up to 1s. Also keep in mind that asynchronous code will become more complicated than using the synchronous one.

If you are building applications which are super-sensitive to the latency and you have multiple RPCs that are independent of one another, it's worthwhile to consider using the asynchronous interface in order to minimize the latency.

Consider the case where you need to add 10 tasks to 10 different queues (thus you cannot batch them). In the worst case, calling `queue.add()` 10 times in a for loop could block up to 10 seconds although it's very rare. If you use the asynchronous interface instead, you can reduce the worst-case latency down to 1 second by adding the tasks to their respective queues in parallel.

If you want to make asynchronous calls to a task queue, you use the asynchronous methods provided by the [Queue](https://web.archive.org/web/20160424223743/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/taskqueue/Queue) class. Call `get` on the returned Future to force the request to complete. When asynchronously adding tasks in a transaction, you should call `get()` on the Future before committing the transaction to ensure that the request has finished.
