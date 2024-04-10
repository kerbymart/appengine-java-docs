# Tasks: get

  

**Requires [authorization](#auth)**

Gets the named task in a TaskQueue. [Try it now](#try-it).

This does not [`lease`](https://web.archive.org/web/20160424230449/https://cloud.google.com/appengine/docs/java/taskqueue/rest/tasks/lease) the task. You shouldn't need to call this method in your typical work flow. The developer's email address must be specified as a [`user_email`](https://web.archive.org/web/20160424230449/https://cloud.google.com/appengine/docs/java/config/queue#Defining_Pull_Queues) in the `acl` element of `queue.yaml`.

## Request

### HTTP request

```
GET https://www.googleapis.com/taskqueue/v1beta2/projects/project/taskqueues/taskqueue/tasks/task
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
<td>The task to get properties of.</td>
</tr>
<tr id="taskqueue" class="even">
<td><code>taskqueue</code></td>
<td><code class="apitype">string</code></td>
<td>The taskqueue in which the task belongs.</td>
</tr>
</tbody>
</table>

### Authorization

This request requires authorization with at least one of the following scopes ([read more about authentication and authorization](https://web.archive.org/web/20160424230449/https://cloud.google.com/appengine/docs/java/taskqueue/rest/about_auth)).

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

If successful, this method returns a [Tasks resource](https://web.archive.org/web/20160424230449/https://cloud.google.com/appengine/docs/java/taskqueue/rest/tasks#resource) in the response body.

## Try it!

Use the APIs Explorer below to call this method on live data and see the response. Alternatively, try the [standalone Explorer](https://web.archive.org/web/20160424230449/https://developers.google.com/apis-explorer/#p/taskqueue/v1beta2/taskqueue.tasks.get).
