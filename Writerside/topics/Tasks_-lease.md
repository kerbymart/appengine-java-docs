# Tasks: lease

  

**Requires [authorization](#auth)**

Acquires a lease on the topmost N unowned tasks in the specified queue. [Try it now](#try-it).

The developer's email address must be specified as a [user\_email](https://web.archive.org/web/20160424225659/https://cloud.google.com/appengine/docs/java/config/queue#Defining_Pull_Queues) in the `acl` element of `queue.yaml`.

## Request

### HTTP request

```
POST https://www.googleapis.com/taskqueue/v1beta2/projects/project/taskqueues/taskqueue/tasks/lease
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
<td>The taskqueue to lease a task from.</td>
</tr>
<tr id="required-parameters" class="even alt">
<td colspan="3"><strong>Required query parameters</strong></td>
</tr>
<tr id="leaseSecs" class="odd">
<td><code>leaseSecs</code></td>
<td><code class="apitype">integer</code></td>
<td>How long to lease this task, in seconds.</td>
</tr>
<tr id="numTasks" class="even">
<td><code>numTasks</code></td>
<td><code class="apitype">integer</code></td>
<td>The number of tasks to lease.</td>
</tr>
<tr id="optional-parameters" class="odd alt">
<td colspan="3"><strong>Optional query parameters</strong></td>
</tr>
<tr id="groupByTag" class="even">
<td><code>groupByTag</code></td>
<td><code class="apitype">boolean</code></td>
<td>When True, returns tasks of the same tag. Specify which tag by using the <code>tag</code> parameter. If tag is not specified, returns tasks of the same tag as the oldest task in the queue.</td>
</tr>
<tr id="tag" class="odd">
<td><code>tag</code></td>
<td><code class="apitype">string</code></td>
<td>Only specify tag if <code>groupByTag</code> is true. If <code>groupByTag</code> is true and <code>tag</code> is not specified, the tag is assumed to be that of the oldest task by ETA. I.e., the first available tag.</td>
</tr>
</tbody>
</table>

### Authorization

This request requires authorization with at least one of the following scopes ([read more about authentication and authorization](https://web.archive.org/web/20160424225659/https://cloud.google.com/appengine/docs/java/taskqueue/rest/about_auth)).

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
  "kind": "taskqueue#tasks",
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
<td>The actual list of tasks returned as a result of the lease operation.</td>
<td></td>
</tr>
</tbody>
</table>

## Try it!

Use the APIs Explorer below to call this method on live data and see the response. Alternatively, try the [standalone Explorer](https://web.archive.org/web/20160424225659/https://developers.google.com/apis-explorer/#p/taskqueue/v1beta2/taskqueue.tasks.lease).
