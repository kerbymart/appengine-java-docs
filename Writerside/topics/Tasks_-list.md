# Tasks: list

  

**Requires [authorization](#auth)**

Lists all non-deleted Tasks in a TaskQueue, whether or not they are currently leased, up to a maximum of 100. [Try it now](#try-it).

The retrieved tasks will not include the task payload. Deleted tasks (whose names remain unavailable for up to seven days) do not appear in this list. Tasks are returned in the same order they are stored on the queue. The developer's email address must be specified as a [`user_email`](https://web.archive.org/web/20160424230500/https://cloud.google.com/appengine/docs/java/config/queue#Defining_Pull_Queues) in the `acl` element of `queue.yaml`.

## Request

### HTTP request

```
GET https://www.googleapis.com/taskqueue/v1beta2/projects/project/taskqueues/taskqueue/tasks
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
<tr id="taskqueue" class="odd">
<td><code>taskqueue</code></td>
<td><code class="apitype">string</code></td>
<td>The id of the taskqueue to list tasks from.</td>
</tr>
</tbody>
</table>

### Authorization

This request requires authorization with at least one of the following scopes ([read more about authentication and authorization](https://web.archive.org/web/20160424230500/https://cloud.google.com/appengine/docs/java/taskqueue/rest/about_auth)).

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

If successful, this method returns a response body with the following structure:

```
{
  "kind": "taskqueues#tasks",
  "items": [
    tasks Resource
  ]
}
```

<table id="response_properties_JSON" class="matchpre">
<thead>
<tr class="header">
<th>Property name</th>
<th>Value</th>
<th>Description</th>
<th>Notes</th>
</tr>
</thead>
<tbody>
<tr id="kind" class="odd">
<td><code>kind</code></td>
<td><code class="apitype">string</code></td>
<td>The kind of object returned, a list of tasks.</td>
<td></td>
</tr>
<tr id="items" class="even">
<td><code>items[]</code></td>
<td><code class="apitype">list</code></td>
<td>The actual list of tasks currently active in the TaskQueue.</td>
<td></td>
</tr>
</tbody>
</table>

## Try it!

Use the APIs Explorer below to call this method on live data and see the response. Alternatively, try the [standalone Explorer](https://web.archive.org/web/20160424230500/https://developers.google.com/apis-explorer/#p/taskqueue/v1beta2/taskqueue.tasks.list).
