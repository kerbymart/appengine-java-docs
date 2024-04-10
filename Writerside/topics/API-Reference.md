# API Reference

Source: https://web.archive.org/web/20160424225301/https://cloud.google.com/appengine/docs/java/taskqueue/rest/  

This API reference is organized by resource type. Each resource type has one or more data representations and one or more methods.

## Resource types

1.  [Taskqueues](#Taskqueues)
2.  [Tasks](#Tasks)

## Taskqueues

For Taskqueues Resource details, see the [resource representation](https://web.archive.org/web/20160424225301/https://cloud.google.com/appengine/docs/java/taskqueue/rest/taskqueues#resource) page.

<table>
<thead>
<tr class="header">
<th>Method</th>
<th>HTTP request</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd alt">
<td colspan="3">URIs relative to https://www.googleapis.com/taskqueue/v1beta2/projects, unless otherwise noted</td>
</tr>
<tr class="even">
<td><a href="https://web.archive.org/web/20160424225301/https://cloud.google.com/appengine/docs/java/taskqueue/rest/taskqueues/get">get</a></td>
<td><code>GET  /</code><var class="apiparam">project</var><code>/taskqueues/</code><var class="apiparam">taskqueue</var></td>
<td>Get detailed information about a TaskQueue.</td>
</tr>
</tbody>
</table>

## Tasks

For Tasks Resource details, see the [resource representation](https://web.archive.org/web/20160424225301/https://cloud.google.com/appengine/docs/java/taskqueue/rest/tasks#resource) page.

<table>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<thead>
<tr class="header">
<th>Method</th>
<th>HTTP request</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd alt">
<td colspan="3">URIs relative to https://www.googleapis.com/taskqueue/v1beta2/projects, unless otherwise noted</td>
</tr>
<tr class="even">
<td><a href="https://web.archive.org/web/20160424225301/https://cloud.google.com/appengine/docs/java/taskqueue/rest/tasks/delete">delete</a></td>
<td><code>DELETE  /</code><var class="apiparam">project</var><code>/taskqueues/</code><var class="apiparam">taskqueue</var><code>/tasks/</code><var class="apiparam">task</var></td>
<td>Deletes a task from a TaskQueue.</td>
</tr>
<tr class="odd">
<td><a href="https://web.archive.org/web/20160424225301/https://cloud.google.com/appengine/docs/java/taskqueue/rest/tasks/get">get</a></td>
<td><code>GET  /</code><var class="apiparam">project</var><code>/taskqueues/</code><var class="apiparam">taskqueue</var><code>/tasks/</code><var class="apiparam">task</var></td>
<td>Gets the named task in a TaskQueue.</td>
</tr>
<tr class="even">
<td><a href="https://web.archive.org/web/20160424225301/https://cloud.google.com/appengine/docs/java/taskqueue/rest/tasks/insert">insert</a></td>
<td><code>POST  /</code><var class="apiparam">project</var><code>/taskqueues/</code><var class="apiparam">taskqueue</var><code>/tasks</code></td>
<td>Insert a task into an existing queue.</td>
</tr>
<tr class="odd">
<td><a href="https://web.archive.org/web/20160424225301/https://cloud.google.com/appengine/docs/java/taskqueue/rest/tasks/lease">lease</a></td>
<td><code>POST  /</code><var class="apiparam">project</var><code>/taskqueues/</code><var class="apiparam">taskqueue</var><code>/tasks/lease</code></td>
<td>Acquires a lease on the topmost N unowned tasks in the specified queue.
<strong>Required query parameters:</strong> <code>leaseSecs</code>, <code>numTasks</code></td>
</tr>
<tr class="even">
<td><a href="https://web.archive.org/web/20160424225301/https://cloud.google.com/appengine/docs/java/taskqueue/rest/tasks/list">list</a></td>
<td><code>GET  /</code><var class="apiparam">project</var><code>/taskqueues/</code><var class="apiparam">taskqueue</var><code>/tasks</code></td>
<td>Lists all non-deleted Tasks in a TaskQueue, whether or not they are currently leased, up to a maximum of 100.</td>
</tr>
<tr class="odd">
<td><a href="https://web.archive.org/web/20160424225301/https://cloud.google.com/appengine/docs/java/taskqueue/rest/tasks/patch">patch</a></td>
<td><code>PATCH  /</code><var class="apiparam">project</var><code>/taskqueues/</code><var class="apiparam">taskqueue</var><code>/tasks/</code><var class="apiparam">task</var></td>
<td>Update tasks that are leased out of a TaskQueue.
<strong>Required query parameters:</strong> <code>newLeaseSeconds</code></td>
</tr>
<tr class="even">
<td><a href="https://web.archive.org/web/20160424225301/https://cloud.google.com/appengine/docs/java/taskqueue/rest/tasks/update">update</a></td>
<td><code>POST  /</code><var class="apiparam">project</var><code>/taskqueues/</code><var class="apiparam">taskqueue</var><code>/tasks/</code><var class="apiparam">task</var></td>
<td>Update the duration of a task lease.
<strong>Required query parameters:</strong> <code>newLeaseSeconds</code></td>
</tr>
</tbody>
</table>
