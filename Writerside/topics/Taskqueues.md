# Taskqueues

  

A TaskQueue resource represents a queue created by an App Engine application. You create TaskQueue resources within a project namespace defined by your App Engine App ID. You can use the TaskQueue resource to get information about a queue. By default, a user has no rights on a TaskQueue unless they are [explicitly granted rights](https://web.archive.org/web/20160424230643/https://cloud.google.com/appengine/docs/java/config/queue#Defining_Pull_Queues) in queue.yaml.

  

Each project can hold an unlimited number of TaskQueue resources. It is currently not possible to create, modify, or delete queues using the REST API.

For a list of [methods](#methods) for this resource, see the end of this page.

## Resource representations

```
{
  "kind": "taskqueues#taskqueue",
  "id": string,
  "maxLeases": integer,
  "stats": {
    "totalTasks": integer,
    "oldestTask": long,
    "leasedLastMinute": long,
    "leasedLastHour": long
  },
  "acl": {
    "consumerEmails": [
      string
    ],
    "producerEmails": [
      string
    ],
    "adminEmails": [
      string
    ]
  }
}
```

<table id="properties" class="matchpre">
<thead>
<tr class="header">
<th>Property name</th>
<th>Value</th>
<th>Description</th>
<th>Notes</th>
</tr>
</thead>
<tbody>
<tr id="acl" class="odd">
<td><code>acl</code></td>
<td><code class="apitype">object</code></td>
<td>ACLs that are applicable to this TaskQueue object.</td>
<td></td>
</tr>
<tr id="acl.adminEmails" class="even">
<td><span class="quiet"><code>acl.</code></span><code>adminEmails[]</code></td>
<td><code class="apitype">list</code></td>
<td>Email addresses of users who are "admins" of the TaskQueue. This means they can control the queue, eg set ACLs for the queue.</td>
<td></td>
</tr>
<tr id="acl.consumerEmails" class="odd">
<td><span class="quiet"><code>acl.</code></span><code>consumerEmails[]</code></td>
<td><code class="apitype">list</code></td>
<td>Email addresses of users who can "consume" tasks from the TaskQueue. This means they can Dequeue and Delete tasks from the queue.</td>
<td></td>
</tr>
<tr id="acl.producerEmails" class="even">
<td><span class="quiet"><code>acl.</code></span><code>producerEmails[]</code></td>
<td><code class="apitype">list</code></td>
<td>Email addresses of users who can "produce" tasks into the TaskQueue. This means they can Insert tasks into the queue.</td>
<td></td>
</tr>
<tr id="id" class="odd">
<td><code>id</code></td>
<td><code class="apitype">string</code></td>
<td>[Mutable only on insert.] Name of the taskqueue. This name is required and must be unique among all queue names within this project. See the <a href="https://web.archive.org/web/20160424230643/https://cloud.google.com/appengine/docs/java/taskqueue/overview#Task_Names">naming restrictions</a>.</td>
<td></td>
</tr>
<tr id="kind" class="even">
<td><code>kind</code></td>
<td><code class="apitype">string</code></td>
<td>[Not mutable.] The kind of REST object returned, in this case taskqueue.</td>
<td></td>
</tr>
<tr id="maxLeases" class="odd">
<td><code>maxLeases</code></td>
<td><code class="apitype">integer</code></td>
<td>[Mutable.] The number of times a client can lease a specific task before it will be automatically dropped from the queue. If not specified, all tasks will be available for indefinite retries. In practice, you should set this value to a reasonably small value so that a difficult task does not block the whole queue.</td>
<td></td>
</tr>
<tr id="stats" class="even">
<td><code>stats</code></td>
<td><code class="apitype">object</code></td>
<td>[Not mutable.] Object that holds statistics for the queue. </td>
<td></td>
</tr>
<tr id="stats.leasedLastHour" class="odd">
<td><span class="quiet"><code>stats.</code></span><code>leasedLastHour</code></td>
<td><code class="apitype">long</code></td>
<td>[Not mutable.] Number of tasks leased in the last hour.</td>
<td></td>
</tr>
<tr id="stats.leasedLastMinute" class="even">
<td><span class="quiet"><code>stats.</code></span><code>leasedLastMinute</code></td>
<td><code class="apitype">long</code></td>
<td>[Not mutable.] Number of tasks leased in the last minute.</td>
<td></td>
</tr>
<tr id="stats.oldestTask" class="odd">
<td><span class="quiet"><code>stats.</code></span><code>oldestTask</code></td>
<td><code class="apitype">long</code></td>
<td>[Not mutable.] The timestamp (in seconds since the epoch) of the oldest unfinished task.</td>
<td></td>
</tr>
<tr id="stats.totalTasks" class="even">
<td><span class="quiet"><code>stats.</code></span><code>totalTasks</code></td>
<td><code class="apitype">integer</code></td>
<td>[Not mutable.] Number of tasks in the queue.</td>
<td></td>
</tr>
</tbody>
</table>

## Methods

[get](https://web.archive.org/web/20160424230643/https://cloud.google.com/appengine/docs/java/taskqueue/rest/taskqueues/get)  
Get detailed information about a TaskQueue.
