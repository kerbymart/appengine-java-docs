# Java Task Queue Configuration

  

Applications define task queues in a configuration file called `queue.yaml`. For Java apps, this file is stored in the application's `WEB-INF` directory. You can use `queue.yaml` to configure both [push queues](https://web.archive.org/web/20160424225923/https://cloud.google.com/appengine/docs/java/taskqueue/overview-push) and [pull queues](https://web.archive.org/web/20160424225923/https://cloud.google.com/appengine/docs/java/taskqueue/overview-pull). This configuration file is optional for push queues, which have a [default queue](#Java_Configuring_the_default_queue). Pull queues must be specifically configured in `queue.yaml`.

1.  [About queues](#Java_About_queues)
2.  [Setting the storage limit for all queues](#Java_Setting_the_storage_limit_for_all_queues)
3.  [Defining push queues and processing rates](#Java_Defining_push_queues_and_processing_rates)
4.  [Defining pull queues](#Java_Defining_pull_queues)
5.  [Configuring retry attempts for failed tasks](#Java_Configuring_retry_attempts_for_failed_tasks)
6.  [Queue definitions](#Java_Queue_definitions)
7.  [Updating task queue configuration](#Java_Updating_task_queue_configuration)

## About queues

An app can define task queues using a file named `queue.yaml`, in the app's `WEB-INF/` directory in the WAR. The file specifies a directive named `queue`. Within this directive, you can name any number of individual queues and define their processing rates.

The app's queue configuration applies to all versions of the app. If a given version of an app enqueues a task, the queue uses the task handler for that version of the app. To control this behavior, see the [target parameter](#target).

An app can only add tasks to queues defined in `queue.yaml` and the default queue. If you upload a new `queue.yaml` file that removes a queue, but that queue still has tasks, the queue is "disabled" (its rate is set to 0) but not deleted. You can re-enable the deleted queue by uploading a new `queue.yaml` file with the queue defined.

## Setting the storage limit for all queues

You can use `queue.yaml` to define the total amount of storage that task data can consume over all queues. To define the total storage limit, include a directive named `total_storage_limit` at the top level just above each `queue` line, like this:

```
# Set the total storage limit for all queues to 120MB
total_storage_limit: 120M
queue:
- name: foo
  rate: 35/s
```

The value is a number followed by a unit: `B` for bytes, `K` for kilobytes, `M` for megabytes, `G` for gigabytes, `T` for terabytes. For example, `100K` specifies a limit of 100 kilobytes. If adding a task would cause the queue to exceed its storage limit, the call to add the task will fail. The default limit is `500M` (500 megabytes) for free apps. For billed apps there is no limit until you explicitly set one. You can use this limit to protect your app from a [fork bomb](https://web.archive.org/web/20160424225923/https://en.wikipedia.org/wiki/Fork_bomb) programming error in which each task adds multiple other tasks during its execution. If your app is receiving errors for insufficient quota when adding tasks, increasing the total storage limit may help. If you are using this feature, we strongly recommend setting a limit that corresponds to the storage required for several days' worth of tasks. In this way, your app is robust to its queues being temporarily backed up and can continue to accept new tasks while working through the backlog while still being protected from a fork bomb programming error.

## Defining push queues and processing rates

You can define any number of individual queues by providing a queue `name`. You can control the rate at which tasks are processed in each queue by defining other directives, such as `rate`, `bucket_size`, and `max_concurrent_requests`.

You can read more about these directives in the [Queue Definitions](#Java_Queue_definitions) section.

The task queue uses [token buckets](https://web.archive.org/web/20160424225923/https://wikipedia.org/wiki/Token_bucket) to control the rate of task execution. Each named queue has a token bucket that holds a certain number of tokens, defined by the `bucket_size` directive. Each time your application executes a task, it uses a token. Your app continues processing tasks in the queue until the queue's bucket runs out of tokens. App Engine refills the bucket with new tokens continuously based on the `rate` that you specified for the queue.

### Configuring the default queue

All apps have a push queue named `default`. This queue has a preset rate of 5 tasks per second, but you can change this rate by defining a default queue in `queue.yaml`. If you do not configure a default queue in `queue.yaml`, the `default` queue doesn't display in the Cloud Platform Console until the first time it is used. You can customize the settings for this queue by defining a queue named `default` in `queue.yaml`:

```
queue:
# Change the refresh rate of the default queue from 5/s to 1/s
- name: default
  rate: 1/s
```

### Configuring the processing rate

If your queue contains tasks to process, and the queue's bucket contains tokens, App Engine processes as many tasks as there are tokens remaining in the bucket. This can lead to bursts of processing, consuming system resources and competing with user-serving requests.

If you want to prevent too many tasks from running at once or to prevent datastore contention, you use `max_concurrent_requests`.

The following samples shows how to set `max_concurrent_requests` to limit tasks and also shows how to adjust the bucket size and rate based on your application's needs and available resources:

```
queue:
- name: optimize-queue
  rate: 20/s
  bucket_size: 40
  max_concurrent_requests: 10
```

### Configuring the maximum number of concurrent requests

You can further control the processing rate by setting `max_concurrent_requests`, which limits the number of tasks that can execute simultaneously.

If your application queue has a rate of 20/s and a bucket size of 40, tasks in that queue execute at a rate of 20/s and can burst up to 40/s briefly. These settings work fine if task latency is relatively low; however, if latency increases significantly, you'll end up processing significantly more concurrent tasks. This extra processing load can consume extra instances and slow down your application.

For example, let's assume that your normal task latency is 0.3 seconds. At this latency, you'll process at most around 40 tasks simultaneously. But if your task latency increases to 5 seconds, you could easily have over 100 tasks processing at once. This increase forces your application to consume more instances to process the extra tasks, potentially slowing down the entire application and interfering with user requests.

You can avoid this possibility by setting `max_concurrent_requests` to a lower value. For example, if you set `max_concurrent_requests` to 10, our example queue maintains about 20 tasks/second when latency is 0.3 seconds. However, when the latency increases over 0.5 seconds, this setting throttles the processing rate to ensure that no more than 10 tasks run simultaneously.

```
queue:
# Set the max number of concurrent requests to 50
- name: optimize-queue
  rate: 20/s
  bucket_size: 40
  max_concurrent_requests: 10
```

<span id="Defining_Pull_Queues"></span>

## Defining pull queues

You can specify any named queue as a [pull queue](https://web.archive.org/web/20160424225923/https://cloud.google.com/appengine/docs/java/taskqueue/overview-pull) by adding the `mode: pull` directive to `queue.yaml`.

If you are using the [Task Queue REST API](https://web.archive.org/web/20160424225923/https://cloud.google.com/appengine/docs/java/taskqueue/rest), you also need to create an access control list (ACL) using the [acl](#acl) directive. This directive allows you to restrict access to user email addresses corresponding to an account hosted by Google.

The `acl` element has two available parameters:

-   `user_email`: enables the user to list, get, lease, delete, and update tasks.
-   `writer_email`: enables the user to insert tasks.

In order to access all functions of the API, a developer's email address must be specified both as a `user_email` and a `writer_email`. The following code snippet creates a pull queue named *pull-queue* with two users in the ACL. The email account `bar@foo.com` can access all API calls:

```
queue:
- name: pull-queue
  mode: pull
  acl:
  - user_email: bar@foo.com      # can list, get, lease, delete, and update tasks
  - writer_email: user@gmail.com # can insert tasks
  - writer_email: bar@foo.com    # can insert tasks, in addition to rights granted by being a user_email above
```

## Configuring retry attempts for failed tasks

Tasks executing in the task queue can fail for many reasons. If a task fails to execute (by returning any HTTP status code outside of the range 200–299), App Engine retries the task until it succeeds. By default, the system gradually reduces the retry rate to avoid flooding your application with too many requests, but schedules retry attempts to recur at a maximum of once per hour until the task succeeds.

Push queues and pull queues differ in how they retry tasks, as described in the following sections.

**Important:** Retry options can be configured for the queue as well specified on the [`Queue.add()`](https://web.archive.org/web/20160424225923/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/taskqueue/RetryOptions) method. When both are specified, the **latter** takes precedence.

### Retrying tasks in push queues

In push queues, you can specify your own scheme for task retries by adding the [retry\_parameters](#retry_parameters) directive in `queue.yaml`. This addition allows you to specify the maximum number of times to retry failed tasks in a specific queue. You can also set a time limit for retry attempts and control the interval between attempts.

The following example demonstrates various retry scenarios:

-   In `fooqueue`, tasks are retried at least seven times and for up to two days from the first execution attempt. After both limits are passed, it fails permanently.
-   In `barqueue`, App Engine attempts to retry tasks, increasing the interval linearly between each retry until reaching the maximum backoff and retrying indefinitely at the maximum interval (so the intervals between requests are 10s, 20s, 30s, ..., 190s, 200s, 200s, ...).
-   In `bazqueue`, the interval increases to twice the minimum backoff and retries indefinitely at the maximum interval (so the intervals between requests are 10s, 20s, 40s, 80s, 120s, 160s, 200s, 200s, ...).

```
queue:
- name: fooqueue
  rate: 1/s
  retry_parameters:
    task_retry_limit: 7
    task_age_limit: 2d
- name: barqueue
  rate: 1/s
  retry_parameters:
    min_backoff_seconds: 10
    max_backoff_seconds: 200
    max_doublings: 0
- name: bazqueue
  rate: 1/s
  retry_parameters:
    min_backoff_seconds: 10
    max_backoff_seconds: 200
    max_doublings: 3
```

### Retrying tasks in pull queues

In pull queues, you can specify the number of times to retry a task using the retry\_parameters directive with the task\_retry\_limit field. The system counts each time you lease a task using [leaseTasks()](https://web.archive.org/web/20160424225923/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/taskqueue/Queue#leaseTasks(long,%20java.util.concurrent.TimeUnit,%20long)). When the count exceeds the `task_retry_limit`, the system deletes the task automatically. If you don't specify a `task_retry_limit`, the system never deletes a task automatically.

The following code sample shows how to specify a pull queue limited to seven retry attempts:

```
queue:
- name: pull-queue
  mode: pull
  retry_parameters:
    task_retry_limit: 7
```

## Queue definitions

The `queue.yaml` file is a YAML file whose root directive is `queue-entries`. This directive contains zero or more `queue` directives, one for each named queue.

A `queue` directive contains configuration for a queue. It can contain the following directives:

<span id="acl"></span>

###### `acl` (pull queues only)

Experimental. Creates an access control list (ACL) for Pull Queues using the Task Queue REST API. The ACL is composed of the specified email addresses. Accepts email addresses only from a [Google Account](https://web.archive.org/web/20160424225923/http://www.google.com/support/accounts/bin/answer.py?hl=en&answer=27439). Enter each email address on its own line as follows `- user_email: user@gmail.com`. For more information, please see [Defining Pull Queues](#Java_Defining_pull_queues).

<span id="bucket_size"></span>

###### `bucket_size` (push queues only)

Task queues use a "token bucket" algorithm for dequeueing tasks. The bucket size limits how fast the queue is processed when many tasks are in the queue and the rate is high. *The maximum value for bucket size is 100.* This allows you to have a high rate so processing starts shortly after a task is enqueued, but still limit resource usage when many tasks are enqueued in a short period of time.

If no `bucket_size` is specified for a queue, the default value is 5.

For more information on the algorithm, see [the Wikipedia article on token buckets](https://web.archive.org/web/20160424225923/https://en.wikipedia.org/wiki/Token_Bucket).

<span id="max_concurrent_requests"></span>

###### `max_concurrent_requests` (push queues only)

Sets the maximum number of tasks that can be executed simultaneously from the specified queue. The value is an integer. By default, the limit is 1000 tasks per queue.

Restricting the number of concurrent tasks gives you more control over the queue's rate of execution and can prevent too many tasks from running at once. It can also prevent datastore contention and make resources available for other queues or online processing.

<span id="mode"></span>

###### `mode`

Identifies the queue mode. This setting defaults to `push`, which identifies a queue as a [push queue](https://web.archive.org/web/20160424225923/https://cloud.google.com/appengine/docs/java/taskqueue/overview-push). If you wish to use [pull queues](https://web.archive.org/web/20160424225923/https://cloud.google.com/appengine/docs/java/taskqueue/overview-pull), set the mode to `pull`.

<span id="name"></span>

###### `name`

The name of the queue. This is the name you specify when you call `QueueFactory.getQueue()`.

A queue name can contain uppercase and lowercase letters, numbers, and hyphens. The maximum length for a queue name is 100 characters.

###### `rate` (push queues only)

How often tasks are processed on this queue. The value is a number followed by a slash and a unit of time, where the unit is `s` for seconds, `m` for minutes, `h` for hours, or `d` for days. For example, the value `5/m` says tasks will be processed at a rate of 5 times per minute.

If the number is `0` (such as `0/s`), the queue is considered "paused," and no tasks are processed.

<span id="retry_parameters"></span>

###### `retry_parameters`

Configures retry attempts for failed tasks. This addition allows you to specify the maximum number of times to retry failed tasks in a specific queue. You can also set a time limit for retry attempts and control the interval between attempts.

`task_retry_limit`  
The maximum number of retry attempts for a failed task. If specified with `task_age_limit`, App Engine retries the task until both limits are reached.

`task_age_limit` (push queues only)  
The time limit for retrying a failed task, measured from when the task was first run. The value is a number followed by a unit of time, where the unit is `s` for seconds, `m` for minutes, `h` for hours, or `d` for days. For example, the value `5d` specifies a limit of five days after the task's first execution attempt. If specified with `task_retry_limit`, App Engine retries the task until both limits are reached.

`min_backoff_seconds` (push queues only)  
The minimum number of seconds to wait before retrying a task after it fails.

`max_backoff_seconds` (push queues only)  
The maximum number of seconds to wait before retrying a task after it fails.

`max_doublings` (push queues only)  
The maximum number of times that the interval between failed task retries will be doubled before the increase becomes constant. The constant is: 2\*\*(max\_doublings - 1) \* min\_backoff\_seconds.

<span id="target"></span>

###### `target` (push queues only)

A string naming a module/version, a frontend version, or a backend, on which to execute all of the tasks enqueued onto this queue.

The string is prepended to the domain name of your app when constructing the HTTP request for a task. For example, if your app ID is `my-app` and you set the target to `my-version.my-module`, the URL hostname will be set to `my-version.my-module.my-app.appspot.com`.

If target is unspecified, then tasks are invoked on the same version of the application where they were enqueued. So, if you enqueued a task from the default application version without specifying a target on the queue, the task is invoked in the default application version. Note that if the default application version changes between the time that the task is enqueued and the time that it executes, then the task will run in the new default version.

**Note:** If you are using modules along with a [dispatch file](https://web.archive.org/web/20160424225923/https://cloud.google.com/appengine/docs/java/modules/routing#dispatch_file), your task's HTTP request may be intercepted and re-routed to another module.

## Updating task queue configuration

The task queue configuration for the app is updated when you upload the application using `appcfg.sh update`. You can update just the configuration for an app's task queues without uploading the full application. To upload the `queue.yaml` file, use the `appcfg.sh update_queues` command:

```
./appengine-java-sdk/bin/appcfg.sh update_queues myapp/war
```

See [Uploading and Managing a Java App](https://web.archive.org/web/20160424225923/https://cloud.google.com/appengine/docs/java/tools/uploadinganapp) for more information.
