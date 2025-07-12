# Tasks

  

A Task resource represents a single task to be handled by a client. A task is defined by an arbitrary blob of data that should be understood by the client.

The standard usage is that a client calls [`task.lease`](https://web.archive.org/web/20160424230319/https://cloud.google.com/appengine/docs/java/taskqueue/rest/tasks/lease) to get the next available task, performs that task, and then calls [`task.delete`](https://web.archive.org/web/20160424230319/https://cloud.google.com/appengine/docs/java/taskqueue/rest/tasks/delete) on that task before the lease expires. If the client cannot finish the task before the lease expires, and has a reasonable chance of completing the task, it should call [`task.update`](https://web.archive.org/web/20160424230319/https://cloud.google.com/appengine/docs/java/taskqueue/rest/tasks/update) before the lease expires. If the client completes the task after the lease has expired, it still needs to delete the task. Tasks should be designed to be idempotent to avoid errors if multiple clients complete the same task.

For a list of [methods](#methods) for this resource, see the end of this page.

## Resource representations

```
{
  "kind": "taskqueues#task",
  "id": string,
  "queueName": string,
  "payloadBase64": string,
  "enqueueTimestamp": long,
  "leaseTimestamp": long,
  "retry_count": integer,
  "tag": string
}
```

<table id="properties" class="matchpre">
<colgroup>
<col style="width: 25%" />
<col style="width: 25%" />
<col style="width: 25%" />
<col style="width: 25%" />
</colgroup>
<thead>
<tr class="header">
<th>Property name</th>
<th>Value</th>
<th>Description</th>
<th>Notes</th>
</tr>
</thead>
<tbody>
<tr id="enqueueTimestamp" class="odd">
<td><code>enqueueTimestamp</code></td>
<td><code class="apitype">long</code></td>
<td>[Not mutable.] Time (in microseconds since the epoch) at which the task was enqueued.</td>
<td></td>
</tr>
<tr id="id" class="even">
<td><code>id</code></td>
<td><code class="apitype">string</code></td>
<td>Randomly generated name of the task. You may override the default generated name by specifying your own name when inserting the task.  We strongly recommend against specifying a task name yourself.</td>
<td></td>
</tr>
<tr id="kind" class="odd">
<td><code>kind</code></td>
<td><code class="apitype">string</code></td>
<td>[Not mutable.] The kind of object returned, in this case set to <code>task</code>.</td>
<td></td>
</tr>
<tr id="leaseTimestamp" class="even">
<td><code>leaseTimestamp</code></td>
<td><code class="apitype">long</code></td>
<td>The time at which the task lease will expire, in microseconds since the epoch. If this task has never been leased, it will be zero. If this this task has been previously leased and the lease has expired, this value will be &lt; Now().<br />
<br />
On <a href="https://web.archive.org/web/20160424230319/https://cloud.google.com/appengine/docs/java/taskqueue/rest/tasks/insert">task.insert</a>, this must be set to the time at which the task will first be available for lease. <br />
<span><br />
On </span><a href="https://web.archive.org/web/20160424230319/https://cloud.google.com/appengine/docs/java/taskqueue/rest/tasks/lease">task.lease</a><span>, this property is not mutable and reflects the time at which the task lease will expire.</span></td>
<td></td>
</tr>
<tr id="payloadBase64" class="odd">
<td><code>payloadBase64</code></td>
<td><code class="apitype">string</code></td>
<td>[Not mutable.] The bytes describing the task to perform. This is a base64-encoded string. The client is expected to understand the payload format. The maximum size is 1MB. This value will be empty in calls to <a href="https://web.archive.org/web/20160424230319/https://cloud.google.com/appengine/docs/java/taskqueue/rest/tasks/list"><code>tasks.list</code></a>.<br />
<br />
<strong>Required</strong> for calls to <a href="https://web.archive.org/web/20160424230319/https://cloud.google.com/appengine/docs/java/taskqueue/rest/tasks/insert"><code>tasks.insert</code></a>.</td>
<td></td>
</tr>
<tr id="queueName" class="even">
<td><code>queueName</code></td>
<td><code class="apitype">string</code></td>
<td>[Not mutable.] Name of the queue that the task is in.<br />
<br />
<span><strong>Required</strong> for calls to </span><a href="https://web.archive.org/web/20160424230319/https://cloud.google.com/appengine/docs/java/taskqueue/rest/tasks/insert"><code>tasks.insert</code></a><span>.</span></td>
<td></td>
</tr>
<tr id="retry_count" class="odd">
<td><code>retry_count</code></td>
<td><code class="apitype">integer</code></td>
<td>The number of leases applied to this task.</td>
<td></td>
</tr>
<tr id="tag" class="even">
<td><code>tag</code></td>
<td><code class="apitype">string</code></td>
<td>The tag for this task. Tagging tasks allows you to group them for processing using <code>lease.group_by_tag</code>.</td>
<td></td>
</tr>
</tbody>
</table>

## Methods

[delete](https://web.archive.org/web/20160424230319/https://cloud.google.com/appengine/docs/java/taskqueue/rest/tasks/delete)  
Deletes a task from a TaskQueue.

[get](https://web.archive.org/web/20160424230319/https://cloud.google.com/appengine/docs/java/taskqueue/rest/tasks/get)  
Gets the named task in a TaskQueue.

[insert](https://web.archive.org/web/20160424230319/https://cloud.google.com/appengine/docs/java/taskqueue/rest/tasks/insert)  
Insert a task into an existing queue.

[lease](https://web.archive.org/web/20160424230319/https://cloud.google.com/appengine/docs/java/taskqueue/rest/tasks/lease)  
Acquires a lease on the topmost N unowned tasks in the specified queue.

[list](https://web.archive.org/web/20160424230319/https://cloud.google.com/appengine/docs/java/taskqueue/rest/tasks/list)  
Lists all non-deleted Tasks in a TaskQueue, whether or not they are currently leased, up to a maximum of 100.

[patch](https://web.archive.org/web/20160424230319/https://cloud.google.com/appengine/docs/java/taskqueue/rest/tasks/patch)  
Update tasks that are leased out of a TaskQueue.

[update](https://web.archive.org/web/20160424230319/https://cloud.google.com/appengine/docs/java/taskqueue/rest/tasks/update)  
Update the duration of a task lease.
