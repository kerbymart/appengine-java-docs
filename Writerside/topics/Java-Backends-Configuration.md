# Java Backends Configuration

  

The Backend API is deprecated as of March 13, 2014. Although Google will continue to support the Backend API in accordance with our [terms of service](//web.archive.org/web/20160424230750/https://developers.google.com/appengine/terms#Deprecation), it is strongly recommended that all new applications use the [Modules API](https://web.archive.org/web/20160424230750/https://cloud.google.com/appengine/docs/java/modules) instead.  
  
For information on converting existing apps using the Backend API to the Modules API, see [Converting Apps to Modules](https://web.archive.org/web/20160424230750/https://cloud.google.com/appengine/docs/java/modules/converting)

You can add backends to your application in a configuration file called `backends.yaml`, which declares the name and desired properties of each backend server. The `backends.yaml` file is stored in the application's `WEB-INF` directory.

1.  [About backends.yaml](#Java_About_backendsyaml)
2.  [Types of backends](#Java_Types_of_backends)
3.  [Request handling](#Java_Request_handling)
4.  [Defining handlers](#Java_Defining_handlers)
5.  [Instance classes](#Java_Instance_classes)
6.  [Backends definitions](#Java_Backends_definitions)

## About backends.yaml

Backends have several unique and important differences from default App Engine instances; for more information about how backends work, please see the [Backends documentation](https://web.archive.org/web/20160424230750/https://cloud.google.com/appengine/docs/java/backends). The following code sample shows a sample `backends.yaml` file. This configuration file defines four separate backends:

1.  A backend named `memdb`. This backend has five instances of the [instance class](#Java_Instance_classes) B8.
2.  A backend named `crawler` uses the default class and options, but with ten instances.
3.  A backend named `worker` using the default class with one instance, and configured with failfast to disable pending queue behavior.
4.  A backend named `cmdline`, used to execute long-running commands issued by admins of the application.

```
backends:
- name: memdb
  class: B8
  instances: 5
- name: crawler
  instances: 10
- name: worker
  options: failfast
- name: cmdline
  options: dynamic
```

## Types of backends

App Engine supports two backend types: resident and dynamic. *Resident* backends are the default type of backend; they use resident instances, which remain in memory, even when idle, and are restarted automatically. This allows you to perform larger tasks or tasks requiring continuous processing. By default, a backend is configured with one instance, but you can add more in the configuration file.

The following table compares the major features of resident and dynamic backends:

<table>
<colgroup>
<col style="width: 25%" />
<col style="width: 25%" />
<col style="width: 25%" />
<col style="width: 25%" />
</colgroup>
<thead>
<tr class="header">
<th width="10%"> </th>
<th width="30%">Resident Backends</th>
<th width="30%">Dynamic Backends</th>
<th width="30%">Other Notes</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<th>Instance type</th>
<td>Resident</td>
<td>Dynamic</td>
<td>Both resident and dynamic backends have one instance by default. You can configure the number of instances using the optional <a href="#instances">instances</a> element.</td>
</tr>
<tr class="even">
<th>Startup</th>
<td>Manual, via the <a href="https://web.archive.org/web/20160424230750/https://cloud.google.com/appengine/docs/java/backends/#Java_Administering_backends">Admin Console</a> or <a href="https://web.archive.org/web/20160424230750/https://cloud.google.com/appengine/docs/java/backends/#Java_Administering_backends">command-line tool</a></td>
<td>Upon receipt of an HTTP request</td>
<td></td>
</tr>
<tr class="odd">
<th>Shutdown</th>
<td>Usually manual, via the <a href="https://web.archive.org/web/20160424230750/https://cloud.google.com/appengine/docs/java/backends/overview#Administering_Backends">Admin Console</a> or <a href="https://web.archive.org/web/20160424230750/https://cloud.google.com/appengine/docs/java/backends/#Java_Administering_backends">command-line tool</a></td>
<td>After sitting idle for a few minutes.</td>
<td>Other factors can affect shutdown. See <a href="https://web.archive.org/web/20160424230750/https://cloud.google.com/appengine/docs/java/backends/#Java_Shutdown">Shutdown</a> for more information.</td>
</tr>
<tr class="even">
<th>State</th>
<td>Preserved until manually shut down, or until an unexpected shutdown event takes place. See <a href="https://web.archive.org/web/20160424230750/https://cloud.google.com/appengine/docs/java/backends/#Java_Shutdown">Shutdown</a> for more details on shutdown events.</td>
<td>Erased when the backend is shutdown after sitting idle for a few minutes.</td>
<td> </td>
</tr>
<tr class="odd">
<th>Type of Processing</th>
<td>Continuous or driven by requests</td>
<td>Driven by requests</td>
<td> </td>
</tr>
<tr class="even">
<th>Cost</th>
<td>Relatively more, because resident backends run continuously.</td>
<td>Relatively less, because dynamic backends terminate when idle.</td>
<td><a href="https://web.archive.org/web/20160424230750/https://cloud.google.com/appengine/docs/pricing">Billed by the hour</a>, even if idle</td>
</tr>
<tr class="odd">
<th>Uses</th>
<td>Any work that requires continuous processing or preserved state, such as:
<ul>
<li>Storing a game state in memory</li>
<li>Caching a social graph or web index</li>
<li>Running a web crawler</li>
</ul></td>
<td>Any request-based work, such as:
<ul>
<li>Task queue requests requiring more power, time, or memory to complete.</li>
<li>Report generation</li>
<li>Large or complex user requests</li>
<li>Ad-hoc queries or admin scripts</li>
</ul></td>
<td>Both resident and dynamic backends allow you to address tasks to a specific instance of a backend. You can obtain the instance and backend names programmatically; the endpoint takes the form <code>http://[instance].[backend_name].[your_app_id].appspot.com</code>.</td>
</tr>
<tr class="even">
<th>Load Balancing</th>
<td>Requests are evenly balanced across all instances of the backend.</td>
<td>Requests are concentrated on the lowest-numbered instances first, and are served by additional instances as traffic increases.</td>
<td> </td>
</tr>
</tbody>
</table>

## Request handling

Resident and dynamic backends both use pending queues to handle incoming traffic. Requests to a backend that is busy handling other requests wait in a pending queue for a short period of time (10 seconds) before being aborted.

You can disable the backend's pending queue using the `failfast` feature. You can configure failfast either on a per-request basis or on an entire backend (though the backend option is currently experimental). To use failfast for a single request, add the following header to the task request:

```
X-AppEngine-FailFast
```

To make a backend failfast, add the `failfast` option to the backend definition in `backends.yaml`.

## Defining handlers

Backends share the request handlers defined in `app.yaml` with your main application version. It is not possible to configure a separate set of handlers for each backend, so mark any handlers that you wish to restrict to admins with `login: admin`. This login declaration blocks external requests, while allowing internal requests (for example, from another instance of your application, or a Task Queue) to access a handler without additional configuration.

The `start` directive allows each backend to define a separate handler for `/_ah/start`.

## Instance classes

App Engine provides four different classes of backend instances, each with different memory and CPU limits. These classes allow you to configure your backend with the processing capacity you need to perform your work. Each class has a specific hourly billing rate. See [Pricing](https://web.archive.org/web/20160424230750/https://cloud.google.com/appengine/docs/pricing#Billable_Resource_Unit_Costs) for more information.

The default class for backends is `B2`, which gives you 256MB of memory and 1200MHz of CPU capacity.

You can change the class of a backend using the `class` directive in `backends.yaml`:

```
backends:
- name: cmdline
  class: B4
```

The `class` directive has the following values:

<table>
<thead>
<tr class="header">
<th>Class configuration</th>
<th>Memory limit</th>
<th>CPU limit</th>
<th>Cost per hour*</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>B1</code></td>
<td>128MB</td>
<td>600MHz</td>
<td>$0.05</td>
</tr>
<tr class="even">
<td><code>B2</code> (default)</td>
<td>256MB</td>
<td>1.2GHz</td>
<td>$0.10</td>
</tr>
<tr class="odd">
<td><code>B4</code></td>
<td>512MB</td>
<td>2.4GHz</td>
<td>$0.20</td>
</tr>
<tr class="even">
<td><code>B4_1G</code></td>
<td>1024MB</td>
<td>2.4GHz</td>
<td>$0.30</td>
</tr>
<tr class="odd">
<td><code>B8</code></td>
<td>1024MB</td>
<td>4.8GHz</td>
<td>$0.40</td>
</tr>
</tbody>
</table>

\*See [Billing, Quotas, and Limits](https://web.archive.org/web/20160424230750/https://cloud.google.com/appengine/docs/java/backends/#Java_Billing_quotas_and_limits) for more information about Backends Billing.

## Backends definitions

The `backends.yaml` file has a single list element called `backends`. Each element in the list represents a backend for the application. Backends are subject to [Billing, Quotas, and Limits](https://web.archive.org/web/20160424230750/https://cloud.google.com/appengine/docs/java/backends/#Java_Billing_quotas_and_limits).

A `backends` directive can have the following directives:

###### `name` (required)

The server name, composed of letters, digits, and hyphens. The name "default" is reserved and cannot be used. You also cannot use a name that is already used as an application version. This name is also referred to as the `BACKEND_ID`.

###### `class`

Determines the backend's [instance class](#Java_Instance_classes), which determines the memory and CPU limits for each instance. The default class is `B2`.

###### `instances`

An integer between `1` and `20` indicating the number of instances to assign to the given backend. Defaults to `1` if unspecified. The instance number is also referred to as the `INSTANCE_ID`.

Each instance can handle multiple requests if your application is [marked as threadsafe](https://web.archive.org/web/20160424230750/https://cloud.google.com/appengine/docs/java/#Java_Requests_and_servlets) in `appengine-web.xml`. You can also configure the [maximum number of concurrent requests](https://web.archive.org/web/20160424230750/https://cloud.google.com/appengine/docs/java/config/appconfig#Using_Concurrent_Requests). This feature is experimental. Backends have pending queues; requests to a busy instance retry for 10 seconds before aborting.

###### `max_concurrent_requests`

Specifies the maximum number of requests that each instance can handle simultaneously. If unset, App Engine determines the concurrent request limit dynamically.

###### `options`

A list of settings, which can include one or more of the following options.

-   **dynamic**: Defaults to resident instances. This option sets the backend to use dynamic instances rather than resident instances. Backends default to resident instances. Backends can be configured to always remain in memory, so state is preserved across requests. This means that the backends are configured to always remain in memory, as described in the [Types of Backends](#Java_Types_of_backends) section above. A resident server is only restarted when the underlying appserver restarts.
-   **failfast** (Experimental!): The `failfast` option disables pending queues, so that an instance that is busy returns an HTTP 503 error immediately when it receives a new request. You can achieve this same behavior by adding an `X-AppEngine-FailFast` header to requests, and the header is not experimental. <span id="options_public"></span>
-   **public** (Experimental!): Sets backend access to `public`. Public backends can serve external HTTP requests from the web. See [Public and Private Backends](https://web.archive.org/web/20160424230750/https://cloud.google.com/appengine/docs/java/backends/#Java_Public_and_private_backends) for more information.
