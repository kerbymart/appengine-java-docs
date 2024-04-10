# Taskqueues: get

  

**Requires [authorization](#auth)**

Get detailed information about a TaskQueue. [Try it now](#try-it).

Returns information about a specified TaskQueue resource. The developer's email address must be specified as a [`user_email`](https://web.archive.org/web/20160424230835/https://cloud.google.com/appengine/docs/java/config/queue#Defining_Pull_Queues) in the `acl` element of `queue.yaml`.

## Request

### HTTP request

```
GET https://www.googleapis.com/taskqueue/v1beta2/projects/project/taskqueues/taskqueue
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
<td>The <code>id</code> of the taskqueue of which to get properties.</td>
</tr>
<tr id="optional-parameters" class="even alt">
<td colspan="3"><strong>Optional query parameters</strong></td>
</tr>
<tr id="getStats" class="odd">
<td><code>getStats</code></td>
<td><code class="apitype">boolean</code></td>
<td> Whether to populate the stats field of the returned resource. Default value is false. Optional.</td>
</tr>
</tbody>
</table>

### Authorization

This request requires authorization with at least one of the following scopes ([read more about authentication and authorization](https://web.archive.org/web/20160424230835/https://cloud.google.com/appengine/docs/java/taskqueue/rest/about_auth)).

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

If successful, this method returns a [Taskqueues resource](https://web.archive.org/web/20160424230835/https://cloud.google.com/appengine/docs/java/taskqueue/rest/taskqueues#resource) in the response body.

## Try it!

Use the APIs Explorer below to call this method on live data and see the response. Alternatively, try the [standalone Explorer](https://web.archive.org/web/20160424230835/https://developers.google.com/apis-explorer/#p/taskqueue/v1beta2/taskqueue.taskqueues.get).
