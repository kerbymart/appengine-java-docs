# Tasks: delete

  

**Requires [authorization](#auth)**

Deletes a task from a TaskQueue. [Try it now](#try-it).

You should only call this method if you own the lease for a task and have completed it, unless you have some other compelling reason to remove a task from the queue. The developer's email address must be specified as a [`user_email`](https://web.archive.org/web/20160424230330/https://cloud.google.com/appengine/docs/java/config/queue#Defining_Pull_Queues) in the `acl` element of `queue.yaml`.

## Request

### HTTP request

```
DELETE https://www.googleapis.com/taskqueue/v1beta2/projects/project/taskqueues/taskqueue/tasks/task
```

### Parameters

<table id="request_parameters" class="matchpre">
<thead>
<tr class="header">
<th>Parameter name</th>
<th>Value</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr id="required-parameters" class="odd alt">
<td colspan="3"><strong>Path parameters</strong></td>
</tr>
<tr id="project" class="even">
<td><code>project</code></td>
<td><code class="apitype">string</code></td>
<td>The project under which the queue lies.</td>
</tr>
<tr id="task" class="odd">
<td><code>task</code></td>
<td><code class="apitype">string</code></td>
<td>The id of the task to delete.</td>
</tr>
<tr id="taskqueue" class="even">
<td><code>taskqueue</code></td>
<td><code class="apitype">string</code></td>
<td>The taskqueue to delete a task from.</td>
</tr>
</tbody>
</table>

### Authorization

This request requires authorization with at least one of the following scopes ([read more about authentication and authorization](https://web.archive.org/web/20160424230330/https://cloud.google.com/appengine/docs/java/taskqueue/rest/about_auth)).

<table class="matchpre">
<thead>
<tr class="header">
<th>Scope</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>https://www.googleapis.com/auth/taskqueue</code></td>
</tr>
<tr class="even">
<td><code>https://www.googleapis.com/auth/taskqueue.consumer</code></td>
</tr>
</tbody>
</table>

### Request body

Do not supply a request body with this method.

## Response

If successful, this method returns an empty response body.

## Try it!

Use the APIs Explorer below to call this method on live data and see the response. Alternatively, try the [standalone Explorer](https://web.archive.org/web/20160424230330/https://developers.google.com/apis-explorer/#p/taskqueue/v1beta2/taskqueue.tasks.delete).
