# Using Pull Queues in Java

  

Pull queues allow you to design your own system to consume App Engine tasks. The task consumer can be part of your App Engine app (such as a [module](https://web.archive.org/web/20160424231004/https://cloud.google.com/appengine/docs/java/modules/)) or a system outside of App Engine (using the [Task Queue REST API](https://web.archive.org/web/20160424231004/https://cloud.google.com/appengine/docs/java/taskqueue/rest)). The task consumer leases a specific number of tasks for a specific duration, then processes and deletes them before the lease ends.

Using pull queues requires your application to handle some functions that are automated in push queues:

-   Your application needs to scale the number of workers based on processing volume. If your application does not handle scaling, you risk wasting computing resources if there are no tasks to process; you also risk latency if you have too many tasks to process.
-   Your application also needs to explicitly delete tasks after processing. In push queues, App Engine deletes the tasks for you. If your application does not delete pull queue tasks after processing, another worker might re-process the task. This wastes computing resources and risks errors if tasks are not [idempotent](https://web.archive.org/web/20160424231004/https://en.wikipedia.org/wiki/idempotent).

Pull queues require a specific configuration in `queue.xml`. For more information, please see [Defining Pull Queues](https://web.archive.org/web/20160424231004/https://cloud.google.com/appengine/docs/java/config/queue#Java_Defining_pull_queues) on the Task Queue configuration page.

The following sections describe the process of enqueuing, leasing, and deleting tasks using pull queues.

1.  [Pull queue overview](#Java_Pull_queue_overview)
2.  [Pulling tasks within App Engine](#Java_Pulling_tasks_within_App_Engine)
    -   [Adding tasks to a pull queue](#Java_Adding_tasks_to_a_pull_queue)
    -   [Leasing tasks](#Java_Leasing_tasks)
    -   [Deleting tasks](#Java_Deleting_tasks)
3.  [Pulling tasks to a module](#Java_Pulling_tasks_to_a_module)
4.  [Pulling tasks from outside App Engine](#Java_Pulling_tasks_from_outside_App_Engine)
    -   [Prerequisites](#Java_Prerequisites)
    -   [Using the task queue REST API with the Java Google API library](#Java_Using_the_task_queue_REST_API_with_the_Java_Google_API_library)
5.  [Quotas and limits for pull queues](#Java_Quotas_and_limits_for_pull_queues)

## Pull queue overview

Pull queues allow a task consumer to process tasks outside of App Engine's default task processing system. If the task consumer is a part of your App Engine app, you can manipulate tasks using simple API calls from the [`com.google.appengine.api.taskqueue`](https://web.archive.org/web/20160424231004/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/taskqueue/package-summary) package. Task consumers outside of App Engine can pull tasks using the [Task Queue REST API](https://web.archive.org/web/20160424231004/https://cloud.google.com/appengine/docs/java/taskqueue/rest).

The process works like this:

1.  The task consumer leases tasks, either via the [Task Queue API](https://web.archive.org/web/20160424231004/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/taskqueue/package-summary.html) (if the consumer is internal to App Engine) or the [Task Queue REST API](https://web.archive.org/web/20160424231004/https://cloud.google.com/appengine/docs/java/taskqueue/rest) (if the consumer is external to App Engine).
2.  App Engine sends task data to the consumer.
3.  The consumer processes the tasks. If the task fails to execute before the lease expires, the consumer can lease it again. This counts as a retry attempt, and you can [configure the maximum number of retry attempts](https://web.archive.org/web/20160424231004/https://cloud.google.com/appengine/docs/java/config/queue#Java_Configuring_retry_attempts_for_failed_tasks) before the system deletes the task.
4.  Once a task executes successfully, the task consumer must delete it.
5.  The task consumer is responsible for scaling instances based on processing volume.

## Pulling tasks within App Engine

You can use pull queues within the App Engine environment using simple API calls to add tasks to a pull queue, lease them, and delete them after processing.

Before you begin, make sure to [configure the pull queue](https://web.archive.org/web/20160424231004/https://cloud.google.com/appengine/docs/java/config/queue#Java_Defining_pull_queues) in `queue.xml`.

### Adding tasks to a pull queue

To add tasks to a pull queue, simply get the queue using the [queue name](https://web.archive.org/web/20160424231004/https://cloud.google.com/appengine/docs/java/config/queue.html#name) defined in `queue.xml`, and use [`TaskOptions.Method.PULL`](https://web.archive.org/web/20160424231004/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/taskqueue/TaskOptions.Method#PULL). The following example enqueues tasks in a pull queue named `pull-queue`:

```
Queue q = QueueFactory.getQueue("pull-queue");
q.add(TaskOptions.Builder.withMethod(TaskOptions.Method.PULL)
                                     .payload("hello world"));
```

### Leasing tasks

**Important:** Your polling loops should detect whether they are attempting to lease tasks faster than the queue can supply them. If this failure occurs, the following exceptions from [leaseTasks()](https://web.archive.org/web/20160424231004/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/taskqueue/Queue#leaseTasks(long,%20java.util.concurrent.TimeUnit,%20long)) can be generated: [`TransientFailureException`](https://web.archive.org/web/20160424231004/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/taskqueue/TransientFailureException) and [`ApiDeadlineExceededException`](https://web.archive.org/web/20160424231004/https://cloud.google.com/appengine/docs/java/javadoc/com/google/apphosting/api/ApiProxy.ApiDeadlineExceededException).  
  
Your code must catch these exceptions, backoff from calling [`leaseTasks()`](https://web.archive.org/web/20160424231004/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/taskqueue/Queue#leaseTasks(long,%20java.util.concurrent.TimeUnit,%20long)), and then try again later. To avoid this problem, consider setting a higher RPC deadline when calling [`leaseTasks()`](https://web.archive.org/web/20160424231004/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/taskqueue/Queue#leaseTasks(long,%20java.util.concurrent.TimeUnit,%20long)). You should also backoff when a lease request returns an empty list of tasks.  
  
If you generate more than 10 LeaseTasks requests per queue per second, only the first 10 requests will return results. The others will return no results.

Once you have added tasks to a pull queue, you can lease one or more tasks using [`leaseTasks()`](https://web.archive.org/web/20160424231004/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/taskqueue/Queue#leaseTasks(long,%20java.util.concurrent.TimeUnit,%20long)). There may be a short delay before tasks recently added using [`add()`](https://web.archive.org/web/20160424231004/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/taskqueue/Queue#add()) become available via [`leaseTasks()`](https://web.archive.org/web/20160424231004/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/taskqueue/Queue#leaseTasks(long,%20java.util.concurrent.TimeUnit,%20long)). When you request a lease, you specify the number of tasks to lease (up to a maximum of 1,000 tasks) and the duration of the lease in seconds (up to a maximum of one week). The lease duration needs to be long enough to ensure that the slowest task will have time to finish before the lease period expires. You can extend a task lease using [`modifyTaskLease()`](https://web.archive.org/web/20160424231004/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/taskqueue/Queue#modifyTaskLease(com.google.appengine.api.taskqueue.TaskHandle,%20long,%20java.util.concurrent.TimeUnit)).

Leasing a task makes it unavailable for processing by another worker, and it remains unavailable until the lease expires. If you lease an individual task, the API selects the task from the front of the queue. If no such task is available, an empty list is returned.

**Note:** `leaseTasks()` operates only on pull queues. If you attempt to lease tasks added in a push queue, App Engine throws an exception. You can change a push queue to a pull queue by changing its definition in `queue.xml`. Please see [Defining Pull Queues](https://web.archive.org/web/20160424231004/https://cloud.google.com/appengine/docs/java/config/queue#Java_Defining_pull_queues) for more information. The following code sample leases 100 tasks from the queue `pull-queue` for one hour:
```
Queue q = QueueFactory.getQueue("pull-queue");

List<TaskHandle> tasks = q.leaseTasks(3600, TimeUnit.SECONDS, 100);
```
**Beta**

This is a Beta release of Task Tagging. This feature is not covered by any SLA or deprecation policy and may be subject to backward-incompatible changes.

Not all tasks are alike; your code can "tag" tasks and then choose tasks to lease by tag. The tag acts as a filter.

```
Queue q = QueueFactory.getQueue("pull-queue");

q.add(TaskOptions.Builder.withMethod(TaskOptions.Method.PULL)
                                     .payload("parse1")
                                     .tag("parse".getBytes()));
q.add(TaskOptions.Builder.withMethod(TaskOptions.Method.PULL)
                                     .payload("parse2")
                                     .tag("parse".getBytes()));
q.add(TaskOptions.Builder.withMethod(TaskOptions.Method.PULL)
                                     .payload("render1")
                                     .tag("render".getBytes()));
q.add(TaskOptions.Builder.withMethod(TaskOptions.Method.PULL)
                                     .payload("render2")
                                     .tag("render".getBytes()));

// Lease render tasks, but not parse
q.leaseTasksByTag(3600, TimeUnit.SECONDS, 100, "render");

// You can also specify a tag to lease via LeaseOptions passed to leaseTasks.
```

### Deleting tasks

In general, once a worker completes a task, it needs to delete the task from the queue. If you see tasks remaining in a queue after a worker finishes processing them, it is likely that the worker failed; in this case, the tasks need to be processed by another worker.

You can delete an individual task or a list of tasks using [`deleteTask()`](https://web.archive.org/web/20160424231004/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/taskqueue/Queue#deleteTask(java.lang.String)). You must know the name of a task in order to delete it. If you are deleting tasks from a pull queue, you can find task names in the [`Task object`](https://web.archive.org/web/20160424231004/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/taskqueue/TaskOptions) returned by `leaseTasks()`. If you are deleting tasks from a push queue, you need to know the task name through some other means (for example, if you created the task explicitly).

The following code sample demonstrates how to deletes a task named `foo` from the queue named `pull-queue`:

```
Queue q = QueueFactory.getQueue("pull-queue");
q.deleteTask("foo");
```

## Pulling tasks to a module

You can use [App Engine Modules](https://web.archive.org/web/20160424231004/https://cloud.google.com/appengine/docs/java/modules) as workers to lease and process pull queue tasks. Modules allow you to process more work without having to worry about request deadlines and other restrictions normally imposed by App Engine. Using modules with pull queues gives you processing efficiencies by allowing you to batch task processing using leases.

For more information about using modules, check out the [Modules documentation](https://web.archive.org/web/20160424231004/https://cloud.google.com/appengine/docs/java/modules).

## Pulling tasks from outside App Engine

If you need to use pull queues from outside App Engine, you must use the [Task Queue REST API](https://web.archive.org/web/20160424231004/https://cloud.google.com/appengine/docs/java/taskqueue/rest). The REST API is a Google web service accessible at a globally-unique URI of the form:

```
https://www.googleapis.com/taskqueue/v1beta2/projects/taskqueues
```

Google provides the following client libraries that you can use to call the Task Queue methods remotely:

In the tables below, the first column shows each library's stage of development (note that some are in early stages), and links to documentation for the library. The second column links to available samples for each library.

<table data-border="1">
<thead>
<tr class="header">
<th>Documentation</th>
<th>Samples</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><a href="https://web.archive.org/web/20160424231004/https://developers.google.com/api-client-library/java/">Google API Client Library for Java</a></td>
<td><a href="https://web.archive.org/web/20160424231004/https://cloud.google.com/api-client-library/java/apis">Java samples</a></td>
</tr>
<tr class="even">
<td><a href="https://web.archive.org/web/20160424231004/https://cloud.google.com/api-client-library/javascript/start/start-js">Google API Client Library for JavaScript (beta)</a></td>
<td><a href="https://web.archive.org/web/20160424231004/https://cloud.google.com/api-client-library/javascript/samples/samples">JavaScript samples</a></td>
</tr>
<tr class="odd">
<td><a href="https://web.archive.org/web/20160424231004/https://cloud.google.com/api-client-library/dotnet/get_started">Google API Client Library for .NET</a></td>
<td><a href="https://web.archive.org/web/20160424231004/https://cloud.google.com/api-client-library/dotnet/apis">.NET samples</a></td>
</tr>
<tr class="even">
<td><a href="https://web.archive.org/web/20160424231004/http://code.google.com/p/google-api-objectivec-client/">Google API Client Library for Objective-C</a></td>
<td><a href="https://web.archive.org/web/20160424231004/http://code.google.com/p/google-api-objectivec-client/wiki/Samples">Objective-C samples</a></td>
</tr>
<tr class="odd">
<td><a href="https://web.archive.org/web/20160424231004/https://cloud.google.com/api-client-library/php/">Google API Client Library for PHP (beta)</a></td>
<td><a href="https://web.archive.org/web/20160424231004/https://github.com/google/google-api-php-client/tree/master/examples">PHP samples</a></td>
</tr>
<tr class="even">
<td><a href="https://web.archive.org/web/20160424231004/https://cloud.google.com/api-client-library/python/">Google API Client Library for Python</a></td>
<td><a href="https://web.archive.org/web/20160424231004/https://github.com/google/google-api-python-client/tree/master/samples">Python samples</a></td>
</tr>
</tbody>
</table>

These early-stage libraries are also available:

<table data-border="1">
<thead>
<tr class="header">
<th>Documentation</th>
<th>Samples</th>
</tr>
<tr class="odd">
<th><a href="https://web.archive.org/web/20160424231004/https://www.dartlang.org/googleapis/">Google APIs Client Libraries for Dart (beta)</a></th>
<th><a href="https://web.archive.org/web/20160424231004/https://github.com/dart-lang/googleapis_examples">Dart samples</a></th>
</tr>
<tr class="header">
<th><a href="https://web.archive.org/web/20160424231004/https://github.com/google/google-api-go-client">Google API Client Library for Go (alpha)</a></th>
<th><a href="https://web.archive.org/web/20160424231004/https://github.com/google/google-api-go-client/tree/master/examples">Go samples</a></th>
</tr>
<tr class="odd">
<th><a href="https://web.archive.org/web/20160424231004/https://github.com/google/google-api-nodejs-client/">Google API Client Library for Node.js (alpha)</a></th>
<th><a href="https://web.archive.org/web/20160424231004/https://github.com/google/google-api-nodejs-client/tree/master/examples">Node.js samples</a></th>
</tr>
<tr class="header">
<th><a href="https://web.archive.org/web/20160424231004/https://cloud.google.com/api-client-library/ruby/start/get_started">Google API Client Library for Ruby (alpha)</a></th>
<th><a href="https://web.archive.org/web/20160424231004/https://github.com/google/google-api-ruby-client-samples">Ruby samples</a></th>
</tr>
</thead>
&#10;</table>

### Prerequisites

The REST API uses OAuth as the authorization mechanism. When you configure your pull queue, make sure that your `queue.xml` file supplies the email addresses of the users that can access the queue using the REST API. The OAuth scope for all methods is `https://www.googleapis.com/auth/taskqueue`.

### Using the task queue REST API with the Java Google API library

This section demonstrates the use of the REST API in an application. The sample allows you to interact with the REST API via the command line and also provides an example of how to lease and delete tasks. The sections below show the basics of using the Client Library with relevant code snippets.

#### Importing the Client Library for Java

Most of the functionality of the Client Library is encapsulated in an `HttpTransport` object. This object stores your authentication header and the default headers to be used with every request. Once the authentication is complete (see `Auth.java` in the sample code for how OAuth is handled), we set up an `HttpTransport` instance that will be used for all requests. From `Util.java`:

```
import com.google.api.client.googleapis.GoogleHeaders;
import com.google.api.client.googleapis.GoogleUtils;
import com.google.api.client.http.HttpTransport;
import com.google.api.client.http.javanet.NetHttpTransport;
...
public static final HttpTransport TRANSPORT = newTransport();
...
static HttpTransport newTransport() {
    HttpTransport result = new NetHttpTransport();
    GoogleUtils.useMethodOverride(result);
    GoogleHeaders headers = new GoogleHeaders();
    headers.setApplicationName("Google-TaskQueueSample/1.0");
    result.defaultHeaders = headers;
    return result;
}
...
```

The `HttpTransport` instance can now be used to make requests to the REST API. For example, here is the code from `TaskQueue.java` that calls [`TaskQueue.Get`](https://web.archive.org/web/20160424231004/https://cloud.google.com/appengine/docs/java/taskqueue/rest#method_taskqueue_taskqueues_get) and displays information about the queue:

```
public void get(String projectName, String taskQueueName, boolean getStats) throws IOException {
  HttpRequest request = Util.TRANSPORT.buildGetRequest();
  request.url = TaskQueueUrl.forTaskQueueServiceQueues(projectName,
                                                       taskQueueName,
                                                       getStats);
  return request.execute().parseAs(TaskQueue.class);
}
```

The following sections describe the two most common functions used with the Task Queue API, allowing you to lease and delete tasks.

#### Leasing Tasks

The Java library provides methods that invoke the [`Tasks.lease`](https://web.archive.org/web/20160424231004/https://cloud.google.com/appengine/docs/java/taskqueue/rest/tasks/lease) method in the REST API. When you create a lease, you need to specify the number of tasks to lease (up to a maximum of 1,000 tasks); the API returns the specified number of tasks in order of the oldest task ETA.

You also need to specify the duration of the lease in seconds (up to a maximum of one week). The lease must be long enough so you can finish all the leased tasks, yet short enough so that tasks can be made available for leasing if your consumer crashes. Similarly, if you lease too many tasks at once and your client crashes, a large number of tasks will become unavailable until the lease expires.

The following code shows how to lease tasks:

```
private static void leaseAndDeleteTask() throws IOException {
  Task task = new Task();
  List<Task> leasedTasks = task.lease(projectName, taskQueueName, leaseSecs,
      numTasks);
  if (leasedTasks.size() == 0) {
    System.out.println("No tasks to lease and hence exiting");
  }
  for (Task leasedTask : leasedTasks) {
    leasedTask.executeTask();
    leasedTask.delete();
  }
}
```

This code enables the command-line tool to lease a specified number of tasks for a set duration:

```
mvn -q exec:java -Dexec.args="appengtaskpuller 30 100"
```

When run, this command-line tool constructs the following URI call to the REST API:

```
POST
https://www.googleapis.com/taskqueue/v1beta2/projects/taskqueues/appengtaskpuller/tasks/lease?alt=json&lease_secs=30&numTasks=100
```

This request returns an array of 100 tasks with the following JSON structure:

```
{
  "kind": "taskqueues#tasks",
  "items": [
    {
      "kind": "taskqueues#task",
      "id": string,
      "queueName": string,
      "payloadBase64": string,
      "enqueueTimestamp": number,
      "leaseTimestamp": number
    }
    ...
  ]
}
```

After processing each task, you need to delete it, as described in the following section.

#### Deleting Tasks

In general, once a worker completes a task, it needs to delete the task from the queue. If you see tasks remaining in a queue after a worker finishes processing, it is likely that the worker failed; in this case, the tasks need to be processed by another worker.

You can delete an individual task or a list of tasks using the REST method [`Task.delete`](https://web.archive.org/web/20160424231004/https://cloud.google.com/appengine/docs/java/taskqueue/rest/tasks/delete). You must know the name of a task in order to delete it. You can get the task name from the id field of the [`Task object`](https://web.archive.org/web/20160424231004/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/taskqueue/TaskOptions) returned by [`Task.lease`](https://web.archive.org/web/20160424231004/https://cloud.google.com/appengine/docs/java/taskqueue/rest/tasks/lease).

Call delete if you have finished a task, even if you have exceeded the lease time. Tasks should be [idempotent](https://web.archive.org/web/20160424231004/https://en.wikipedia.org/wiki/Idempotence), so even if a task lease expires and another client leases the task, performing the same task twice should not cause an error.

**Note:** When you delete a task, it immediately becomes invisible to queries, but its name remains in the system for up to seven days. During this time, attempting to create another task with the same name will result in an "item exists" error. The system offers no method to determine if deleted task names are still in the system. To avoid these issues, we recommend that you let App Engine generate the task name automatically.

The following code snippet deletes tasks from a queue:

```
private static void leaseAndDeleteTask() throws IOException {
  Task task = new Task();
  List<Task> leasedTasks = task.lease(projectName, taskQueueName, leaseSecs,
      numTasks);
  if (leasedTasks.size() == 0) {
    System.out.println("No tasks to lease and hence exiting");
  }
  for (Task leasedTask : leasedTasks) {
    leasedTask.executeTask();
    leasedTask.delete();
  }
}
```

The delete method implementation can be seen here:

```
public void delete() throws HttpResponseException, IOException {
  HttpRequest request = Util.TRANSPORT.buildDeleteRequest();
  String projectName = getProjectFromQueueName();
  String queueName = getQueueFromQueueName();
  if (projectName.isEmpty() || queueName.isEmpty()) {
    System.out.println("Error parsing full queue name:" + this.queueName +
        " Hence unable to delete task" + this.id);
    return;
  }
  request.url = TaskQueueUrl.forTaskQueueServiceTasks(projectName, queueName);
  request.url.pathParts.add(this.id);
  try {
    request.execute();
  } catch (HttpResponseException hre) {
    System.out.println("Error deleting task: " + this.id);
    throw hre;
  }
}
```

If the delete command is successful, the REST API returns an HTTP 200 response. If deletion fails, the API returns an HTTP failure code.

## Quotas and limits for pull queues

Enqueuing a task counts counts toward the following quotas:

-   Task Queue Stored Task Count
-   Task Queue API Calls
-   Task Queue Stored Task Bytes

Leasing a task counts toward the following quotas:

-   Task Queue API Calls
-   Outgoing Bandwidth (if using the REST API)

The Task Queue Stored Task Bytes quota is configurable in `queue.xml` by setting [`<total-storage-limit>`](https://web.archive.org/web/20160424231004/https://cloud.google.com/appengine/docs/java/config/queue#Setting_the_Storage_Limit_for_All_Queues). This quota counts towards your Stored Data (billable) quota.

The following limits apply to the use of pull queues:

<table>
<thead>
<tr class="header">
<th colspan="2">Pull Queue Limits</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Maximum task size</td>
<td>1MB</td>
</tr>
<tr class="even">
<td>Maximum countdown/ETA for a task</td>
<td>30 days from the current date and time</td>
</tr>
<tr class="odd">
<td>Maximum number of tasks that can be added in a batch</td>
<td>100 tasks</td>
</tr>
<tr class="even">
<td>Maximum number of tasks that can be added in a transaction</td>
<td>5 tasks</td>
</tr>
<tr class="odd">
<td>Maximum number of tasks that you can lease in a single operation</td>
<td>1000 tasks</td>
</tr>
<tr class="even">
<td>Maximum payload size when leasing a batch of tasks</td>
<td>32MB (1MB when using the REST API)</td>
</tr>
</tbody>
</table>

<table>
<thead>
<tr class="header">
<th colspan="2">Combined Limits (Push and Pull Queues)</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Maximum number of active queues (not including the default queue)</td>
<td>Free apps: 10 queues, Billed apps: 100 queues</td>
</tr>
</tbody>
</table>
