# Using Push Queues in Java

  

In App Engine push queues, a *task* is a unit of work to be performed by the application. Each task is an object of the [TaskOptions class](https://web.archive.org/web/20160424230340/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/taskqueue/TaskOptions). Each Task object contains an application-specific URL with a request handler for the task, and an optional data payload that parameterizes the task.

For example, consider a calendaring application that needs to notify an invitee, via email, that an event has been updated. The data payload for this task consists of the email address and name of the invitee, along with a description of the event. The webhook might live at `/app_worker/send_email` and contain a function that adds the relevant strings to an email template and sends the email. The app can create a separate task for each email it needs to send.

You can use push queues only within the App Engine environment; if you need to access App Engine tasks from outside of App Engine, use [pull queues](https://web.archive.org/web/20160424230340/https://cloud.google.com/appengine/docs/java/taskqueue/overview-pull).

1.  [Using push queues](#Java_Using_push_queues)
2.  [Push task execution](#Java_Task_execution)
    1.  [Task request headers](#task_request_headers)
    2.  [Task deadlines](#task_deadlines)
    3.  [Task retries](#task_retries)
    4.  [The rate of task execution](#the_rate_of_task_execution)
    5.  [The order of task execution](#the_order_of_task_execution)
3.  [Deferred tasks](#Java_Deferred_tasks)
4.  [URL endpoints](#Java_URL_endpoints)
5.  [Calling Google Cloud Endpoints](#Java_calling_google_cloud_endpoints)
6.  [Push queues and the development server](#Java_Push_queues_and_the_development_server)
7.  [Quotas and limits for push queues](#Java_Quotas_and_limits_for_push_queues)

## Using push queues

A Java app sets up queues using a configuration file named `queue.xml`, in the `WEB-INF/` directory inside the WAR. See [Java Task Queue Configuration](https://web.archive.org/web/20160424230340/https://cloud.google.com/appengine/docs/java/config/queue). Every app has a push queue named `default` with some default settings.

To enqueue a task, you get a `Queue` using the `QueueFactory`, then call its `add()` method. You can get a named queue specified in the `queue.xml` file using the `getQueue()` method of the factory, or you can get the default queue using `getDefaultQueue()`. You can call the `Queue`'s `add()` method with a `TaskOptions` instance (produced by [`TaskOptions.Builder`](https://web.archive.org/web/20160424230340/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/taskqueue/TaskOptions.Builder)), or you can call it with no arguments to create a task with the default options for the queue.

The following code adds a task to a queue with options.

In `index.html`:

```
<!-- A basic index.html file served from the "/" URL. -->
<html>
  <body>
    <p>Enqueue a value, to be processed by a worker.</p>
    <form action="/enqueue" method="post">
      <input type="text" name="key">
      <input type="submit">
    </form>
  </body>
</html>
```

In `Enqueue.java`:

```
// The Enqueue servlet should be mapped to the "/enqueue" URL.
import com.google.appengine.api.taskqueue.Queue;
import com.google.appengine.api.taskqueue.QueueFactory;
import com.google.appengine.api.taskqueue.TaskOptions;

public class Enqueue extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        String key = request.getParameter("key");

        // Add the task to the default queue.
        Queue queue = QueueFactory.getDefaultQueue();
        queue.add(TaskOptions.Builder.withUrl("/worker").param("key", key));

        response.sendRedirect("/");
    }
}
```

In `Worker.java`:

```
// The Worker servlet should be mapped to the "/worker" URL.
public class Worker extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        String key = request.getParameter("key");
        // Do something with key.
    }
}
```

Tasks added to this queue will execute by calling the request handler at the URL `/worker` with the parameter `key`. They will execute at the rate set in the `queue.xml` file, or the default rate of 5 tasks per second.

## Push task execution

App Engine executes push tasks by sending HTTP POST requests to your app. Specifying a programmatic asynchronous callback as an HTTP request is sometimes called a [web hook](https://web.archive.org/web/20160424230340/https://en.wikipedia.org/wiki/Webhook). The web hook model enables efficient parallel processing.

The task's URL determines the *handler* for the task and the *module* that runs the handler.

The handler is determined by the path part of the URL (the forward-slash separated string following the hostname), which is specified by the `url` parameter in the [`TaskOptions`](https://web.archive.org/web/20160424230340/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/taskqueue/TaskOptions) that you include in your call to the [`Queue.add()`](https://web.archive.org/web/20160424230340/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/taskqueue/Queue#add(java.lang.Iterable)) method. The url must be relative and local to your application's root directory. <span id="set_module">The module and version in which the handler runs is determined by:</span>

-   The "Host" `header` parameter in the [`TaskOptions`](https://web.archive.org/web/20160424230340/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/taskqueue/TaskOptions) that you include in your call to the [`Queue.add()`](https://web.archive.org/web/20160424230340/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/taskqueue/Queue#add(java.lang.Iterable)) method.
-   The `target` directive in the [`queue.xml`](https://web.archive.org/web/20160424230340/https://cloud.google.com/appengine/docs/java/config/queue) or [`queue.yaml`](https://web.archive.org/web/20160424230340/https://cloud.google.com/appengine/docs/java/configyaml/queue) file.

The following code sample demonstrates how to create a push task addressed to instance `1` of a module named `backend1`, using the `target` directive:
```
import com.google.appengine.api.taskqueue.Queue;
import com.google.appengine.api.taskqueue.QueueFactory;
import com.google.appengine.api.taskqueue.TaskOptions;
import com.google.appengine.api.modules.ModulesServiceFactory;


// ...
    queue.add(TaskOptions.Builder.withUrl("/path/to/my/worker").param("key", key).header("Host",
        ModulesServiceFactory.getModulesService().getInstanceHostname("module1", null, 1)));
```

If you do not specify any of these parameters, the task will run in the same module/version in which it was enqueued, subject to these rules:

-   If the default version of the app enqueues a task, the task will run on the default version. Note that if the app enqueues a task and the default version is changed before the task actually runs, the task will be executed in the new default version.
-   If a non-default version enqueues a task, the task will always run on that same version.

**Note:** If you are using modules along with a [dispatch file](https://web.archive.org/web/20160424230340/https://cloud.google.com/appengine/docs/java/modules/routing#dispatch_file), a task's URL may be intercepted and re-routed to another module.

The namespace in which a push task runs is determined when the task is added to the queue. By default, a task will run in the current namespace of the process that created the task. You can override this behavior by explicitly setting the namespace before adding a task to a queue, as described on the [multitenancy](https://web.archive.org/web/20160424230340/https://cloud.google.com/appengine/docs/java/multitenancy/multitenancy#Java_Using_namespaces_with_the_Task_Queue) page.

### Task request headers

Requests from the Task Queue service contain the following HTTP headers:

-   `X-AppEngine-QueueName`, the name of the queue (possibly `default`)
-   `X-AppEngine-TaskName`, the name of the task, or a system-generated unique ID if no name was specified
-   `X-AppEngine-TaskRetryCount`, the number of times this task has been retried; for the first attempt, this value is 0. This number includes attempts where the task failed due to a lack of available instances and never reached the execution phase.
-   `X-AppEngine-TaskExecutionCount`, the number of times this task has previously failed during the execution phase. This number does not include failures due to a lack of available instances.
-   `X-AppEngine-TaskETA`, the target execution time of the task, specified in seconds since January 1st 1970.

These headers are set internally by Google App Engine. If your request handler finds any of these headers, it can trust that the request is a Task Queue request. If any of the above headers are present in an external user request to your app, they are stripped. The exception being requests from logged in administrators of the application, who are allowed to set the headers for testing purposes, and development servers, which do not strip headers.

Tasks may be created with the `X-AppEngine-FailFast` header, which specifies that a task running on a manual or basic scaled module fails immediately instead of waiting in a pending queue.

Google App Engine issues Task Queue requests from the IP address `0.1.0.2`.

### Task deadlines

A task must finish executing and send an HTTP response value between 200–299 within a time interval that depends on the [scaling type](https://web.archive.org/web/20160424230340/https://cloud.google.com/appengine/docs/java/modules#Java_Instance_scaling_and_class) of the App Engine module that receives the request. This deadline is separate from user requests, which have a 60-second deadline.

Tasks targeted at an automatic scaled module must finish execution within 10 minutes. If you have tasks that require more time or computing resources, they can be [sent to manual or basic scaling modules](#set_module), where they can run up to 24 hours. If the task fails to respond within the deadline, or returns an invalid response value, the task will be retried as described in the next section.

When a task's execution time nears the deadline, App Engine raises a [`DeadlineExceededException`](https://web.archive.org/web/20160424230340/https://cloud.google.com/appengine/docs/java/javadoc/com/google/apphosting/api/DeadlineExceededException) before the deadline is reached, so you can save your work or log whatever progress was made.

### Task retries

If a push task request handler returns an HTTP status code within the range 200–299, App Engine considers the task to have completed successfully. If the task returns a status code outside of this range, App Engine retries the task until it succeeds. The system backs off gradually to avoid flooding your application with too many requests, but schedules retry attempts for failed tasks to recur at a maximum of once per hour.

You can also configure your own scheme for task retries using the [retry-parameters element](https://web.archive.org/web/20160424230340/https://cloud.google.com/appengine/docs/java/config/queue#Configuring_Retry_Attempts_for_Failed_Tasks) in `queue.xml`.

When implementing the code for tasks (as worker URLs within your app), it is important to consider whether the task is [idempotent](https://web.archive.org/web/20160424230340/https://en.wikipedia.org/wiki/idempotent). App Engine's Task Queue API is designed to only invoke a given task once; however, it is possible in exceptional circumstances that a task may execute multiple times (such as in the unlikely case of major system failure). Thus, your code must ensure that there are no harmful side-effects of repeated execution.

### The rate of task execution

You set the maximum processing rate for the entire queue when you [configure the queue](https://web.archive.org/web/20160424230340/https://cloud.google.com/appengine/docs/java/config/queue). App Engine uses a [token bucket algorithm](https://web.archive.org/web/20160424230340/https://en.wikipedia.org/wiki/Token_bucket) to execute tasks once they've been delivered to the queue. Each queue has a token bucket, and each bucket holds a certain number of tokens. Your app consumes a token each time it executes a task. If the bucket runs out of tokens, the system pauses until the bucket has more tokens. The rate at which the bucket is refilled is the limiting factor that determines the rate of the queue. See [Defining Push Queues and Processing Rates](https://web.archive.org/web/20160424230340/https://cloud.google.com/appengine/docs/java/config/queue) for more details. See also [RetryOptions](https://web.archive.org/web/20160424230340/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/taskqueue/RetryOptions).

To ensure that the Task Queue system does not overwhelm your application, it may throttle the rate at which requests are sent. This throttled rate is known as the *enforced rate*. The enforced rate may be decreased when your application returns a 503 HTTP response code, or if there are no instances able to execute a request for an extended period of time. You can view the enforced rate on the Cloud Platform Console [Task queues page](https://web.archive.org/web/20160424230340/https://cloud.google.com/appengine/docs/developers-console/#queues) .

### The order of task execution

There are no guarantees or promises regarding the execution order of task queues. Even in the average case, execution order can be significantly different from FIFO order. The order in which tasks are executed depends on several factors:

-   **The position of the task in the queue.** App Engine attempts to process tasks based on FIFO (first in, first out) order. In general, tasks are inserted into the end of a queue and executed from the head of the queue. However, the logic that executes tasks may prioritize performance over preserving order. Users should expect tasks to arrive out of order regularly, often significantly out of order.
-   **The backlog of tasks in the queue.** The system attempts to deliver the lowest latency possible for any given task via specially optimized notifications to the scheduler. Thus, in the case that a queue has a large backlog of tasks, the system's scheduling may "jump" new tasks to the head of the queue.
-   **The value of the task's [etaMillis](https://web.archive.org/web/20160424230340/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/taskqueue/TaskOptions#etaMillis(long)) property**. This specifies the earliest time that a task can execute. App Engine always waits until after the specified ETA to process push tasks.
-   **The value of the task's [countdownMillis](https://web.archive.org/web/20160424230340/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/taskqueue/TaskOptions#countdownMillis(long)) property.** This specifies the minimum number of seconds to wait before executing a task. Countdown and eta are mutually exclusive; if you specify one, do not specify the other.

## Deferred tasks

Setting up a handler for each distinct task (as described in the previous sections) can be cumbersome, as can serializing and deserializing complex arguments for the task — particularly if you have many diverse but small tasks that you want to run on the queue. The Java SDK includes an interface called [DeferredTask](https://web.archive.org/web/20160424230340/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/taskqueue/DeferredTask). This interface lets you define a task as a single method. This interface uses Java serialization to package a unit of work into a Task Queue. A simple return from that method is considered success. Throwing any exception from that method is considered a failure.

<a href="https://web.archive.org/web/20160424230340/https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/taskqueue/deferred/src/main/java/com/google/cloud/taskqueue/samples/DeferSampleServlet.java" target="_blank" style="color: white;">taskqueue/deferred/src/main/java/com/google/cloud/taskqueue/samples/DeferSampleServlet.java</a>

<a href="https://web.archive.org/web/20160424230340/https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/taskqueue/deferred/src/main/java/com/google/cloud/taskqueue/samples/DeferSampleServlet.java" class="button" target="_blank" data-track-type="github" data-track-name="gitHubViewButton" data-track-metadata-link-destination="https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/taskqueue/deferred/src/main/java/com/google/cloud/taskqueue/samples/DeferSampleServlet.java">View on GitHub</a>

\[This section requires a browser that supports JavaScript and iframes.\]

**Warning:** While the DeferredTask API is a convenient way to handle serialization, you have to carefully control the serialization compatibility of objects passed into [payload](https://web.archive.org/web/20160424230340/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/taskqueue/TaskOptions#payload(byte%5B%5D,%20java.lang.String)) methods. Careful control is necessary because unprocessed objects remain in the task queue, even after the application code is updated. Tasks based on outdated application code will not deserialize properly when the task is decoded with new revisions of the application.

## URL endpoints

Push tasks reference their implementation via URL. For example, a task which fetches and parses an RSS feed might use a worker URL called `/app_worker/fetch_feed`. You can specify this worker URL or use the default. In general, you can use any URL as the worker for a task, so long as it is within your application; all task worker URLs must be specified as relative URLs:

```
import com.google.appengine.api.taskqueue.Queue;
import com.google.appengine.api.taskqueue.QueueFactory;
import com.google.appengine.api.taskqueue.TaskOptions.Method;
import com.google.appengine.api.taskqueue.TaskOptions;

// ...
    Queue queue = QueueFactory.getDefaultQueue();
    queue.add(TaskOptions.Builder.withUrl("/path/to/my/worker"));
    queue.add(TaskOptions.Builder.withUrl("/path?a=b&c=d").method(Method.GET));
```

If you do not specify a worker URL, the task uses a default worker URL named after the queue:

`/_ah/queue/queue_name`

A queue's default URL is used if, and only if, a task does not have a worker URL of its own. If a task does have its own worker URL, then it is only invoked at the worker URL, never another. Once inserted into a queue, its url endpoint cannot be changed.

**Warning:** If a task does not have a worker URL, then the task is invoked against the queue's default URL, even if there is currently no handler defined for the queue's default URL. This results in a 404 which will be available, along with the exact URL that was tried, in your application's logs. Due to the failure state of this 404, the system saves the task and retries it until it is eventually successful. You can clear (or 'purge') tasks that can't complete successfully using the Cloud Platform Console.

### Securing URLs for tasks

If a task performs sensitive operations (such as modifying important data), you might want to secure its worker URL to prevent a malicious external user from calling it directly. You can prevent users from accessing URLs of tasks by restricting access to administrator accounts. Task queues can access admin-only URLs. You can read about restricting URLs at [Security and Authentication](https://web.archive.org/web/20160424230340/https://cloud.google.com/appengine/docs/java/config/webxml#Security_and_Authentication). An example you would use in `web.xml` to restrict everything starting with `/tasks/` to admin-only is:

```
<security-constraint>
    <web-resource-collection>
        <web-resource-name>tasks</web-resource-name>
        <url-pattern>/tasks/*</url-pattern>
    </web-resource-collection>
    <auth-constraint>
        <role-name>admin</role-name>
    </auth-constraint>
</security-constraint>
```
**Note:** While task queues can use URL paths restricted with `<role-name>admin</role-name>`, they *cannot* use URL paths restricted with `<role-name>*</role-name>`.

For more on the format of `web.xml`, see the documentation on [the deployment descriptor](https://web.archive.org/web/20160424230340/https://cloud.google.com/appengine/docs/java/config/webxml).

To test a task web hook, sign in as an administrator and visit the URL of the handler in your browser.

## Calling Google Cloud Endpoints

You cannot call a [Google Cloud Endpoint](https://web.archive.org/web/20160424230340/https://cloud.google.com/appengine/features/#Endpoints) from a push queue. Instead, you should issue a request to a target that is served by a handler that's specified in your app's configuration file or in a dispatch file. That handler then calls the appropriate endpoint class and method.

## Push queues and the development server

When your app is running in the development server, tasks are automatically executed at the appropriate time just as in production.

To disable automatic execution of tasks, set the following jvm flag:

```
--jvm_flag=-Dtask_queue.disable_auto_task_execution=true
```

You can examine and manipulate tasks from the Cloud Platform Console [Task queues page](https://web.archive.org/web/20160424230340/https://cloud.google.com/appengine/docs/developers-console/#queues) .

To execute tasks, select the queue by clicking on its name, select the tasks to execute, and click **Run Now**. To clear a queue without executing any tasks, click **Purge Queue**.

The development server and the production server behave differently:

-   The development server doesn't respect the `<rate>` and `<bucket-size>` attributes of your queues. As a result, tasks are executed as close to their ETA as possible. Setting a rate of 0 doesn't prevent tasks from being executed automatically.
-   The development server doesn't retry tasks.
-   The development server doesn't preserve queue state across server restarts.

## Quotas and limits for push queues

Enqueuing a task in a push queue counts toward the following quotas:

-   Task Queue Stored Task Count
-   Task Queue Stored Task Bytes
-   Task Queue API Calls

The Task Queue Stored Task Bytes quota is configurable in queue.xml by setting `<total-storage-limit>`. This quota counts towards your Stored Data (billable) quota.

Execution of a task counts toward the following quotas:

-   Requests
-   Incoming Bandwidth
-   Outgoing Bandwidth

The act of executing a task consumes bandwidth-related quotas for the request and response data, just as if the request handler were called by a remote client. When the task queue processes a task, the response data is discarded.

Once a task has been executed or deleted, the storage used by that task is reclaimed. The reclaiming of storage quota for tasks happens at regular intervals, and this may not be reflected in the storage quota immediately after the task is deleted.

For more information on quotas, see [Quotas](https://web.archive.org/web/20160424230340/https://cloud.google.com/appengine/docs/quotas), and the Cloud Platform Console [Quota details page](https://web.archive.org/web/20160424230340/https://cloud.google.com/appengine/docs/developers-console/#quotas) ).

The following limits apply to the use of push queues:

<table>
<thead>
<tr class="header">
<th colspan="2">Push Queue Limits</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Maximum task size</td>
<td>100KB</td>
</tr>
<tr class="even">
<td>Queue execution rate</td>
<td>500 task invocations per second per queue</td>
</tr>
<tr class="odd">
<td>Maximum countdown/ETA for a task</td>
<td>30 days from the current date and time</td>
</tr>
<tr class="even">
<td>Maximum number of tasks that can be added in a batch</td>
<td>100 tasks</td>
</tr>
<tr class="odd">
<td>Maximum number of tasks that can be added in a transaction</td>
<td>5 tasks</td>
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
