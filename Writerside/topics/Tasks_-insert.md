# Tasks: insert

  

**Requires [authorization](#auth)**

Insert a task into an existing queue. [Try it now](#try-it).

The developer's email address must be specified as a [`writer_email`](https://web.archive.org/web/20160424231111/https://cloud.google.com/appengine/docs/java/config/queue#Defining_Pull_Queues) in the `acl` element of `queue.yaml`.

## Request

### HTTP request

```
POST https://www.googleapis.com/taskqueue/v1beta2/projects/project/taskqueues/taskqueue/tasks
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
<td>The project under which the queue lies</td>
</tr>
<tr id="taskqueue" class="odd">
<td><code>taskqueue</code></td>
<td><code class="apitype">string</code></td>
<td>The taskqueue to insert the task into</td>
</tr>
</tbody>
</table>

### Authorization

This request requires authorization with at least one of the following scopes ([read more about authentication and authorization](https://web.archive.org/web/20160424231111/https://cloud.google.com/appengine/docs/java/taskqueue/rest/about_auth)).

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

In the request body, supply a [Tasks resource](https://web.archive.org/web/20160424231111/https://cloud.google.com/appengine/docs/java/taskqueue/rest/tasks#resource).

## Response

If successful, this method returns a [Tasks resource](https://web.archive.org/web/20160424231111/https://cloud.google.com/appengine/docs/java/taskqueue/rest/tasks#resource) in the response body.

## Try it!

Use the APIs Explorer below to call this method on live data and see the response. Alternatively, try the [standalone Explorer](https://web.archive.org/web/20160424231111/https://developers.google.com/apis-explorer/#p/taskqueue/v1beta2/taskqueue.tasks.insert).
