# App Engine Modules in Java

  

[Python](https://web.archive.org/web/20160425095823/https://cloud.google.com/appengine/docs/python/modules/ "View this page in the Python runtime") <span style="color: #ddd; padding: 0em .5em;">|</span><span style="color: black; font-weight:bold">Java</span> <span style="color: #ddd; padding: 0em .5em;">|</span>[PHP](https://web.archive.org/web/20160425095823/https://cloud.google.com/appengine/docs/php/modules/ "View this page in the PHP runtime") <span style="color: #ddd; padding: 0em .5em;">|</span>[Go](https://web.archive.org/web/20160425095823/https://cloud.google.com/appengine/docs/go/modules/ "View this page in the Go runtime")

[<img src="https://web.archive.org/web/20160425095823im_/https://cloud.google.com/cloud/images/stack_overflow_questions.png" title="Stack Overflow Questions" data-border="0" width="240" height="53" />](https://web.archive.org/web/20160425095823/http://stackoverflow.com/questions/tagged/google-app-engine+module)

[](https://web.archive.org/web/20160425095823/http://stackoverflow.com/feeds/tag?sort=votes&tagnames=google-app-engine%2Bmodule)

<span class="link gc-analytics-event" category="Sidebar" data-action="Stack Overflow question click"></span>

<a href="https://web.archive.org/web/20160425095823/http://stackoverflow.com/questions/tagged/google-app-engine+module?sort=votes" class="gc-analytics-event" data-category="Sidebar" data-action="Stack Overflow See more link">See more...</a>

App Engine Modules (or just "Modules" hereafter) let developers factor large applications into logical components that can share stateful services and communicate in a secure fashion. A deployed module behaves like a [microservice](https://web.archive.org/web/20160425095823/https://wikipedia.org/wiki/Microservices). By using multiple modules you can [deploy your app as a set of microservices](https://web.archive.org/web/20160425095823/https://cloud.google.com/solutions/microservices-on-app-engine), which is a popular design pattern.

For example, an app that handles customer requests might include separate modules to handle other tasks, such as:

-   API requests from mobile devices
-   Internal, admin-like requests
-   Backend processing such as billing pipelines and data analysis

Modules can be configured to use different runtimes and to operate with different performance settings.

1.  [Application hierarchy](#Java_Application_hierarchy)
2.  [Scaling types and instance classes](#Java_Instance_scaling_and_class)
3.  [Configuration](#Java_Configuration)
4.  [Uploading and deleting modules](#Java_Uploading_modules)
5.  [Instance states](#Java_Instance_states)
6.  [Instance uptime](#Java_Instance_uptime)
7.  [Background threads](#Java_Background_threads)
8.  [Monitoring resource usage](#Java_Monitoring_resource_usage)
9.  [Logging](#Java_Logging)
10. [Communication between modules](#Java_Communication_between_modules)
11. [Limits](#Java_Limits)

## Application hierarchy

At the highest level, an App Engine application is made up of one or more *modules*. Each module consists of source code and configuration files. The files used by a module represent a *version* of the module. When you deploy a module, you always deploy a specific version of the module. For this reason, whenever we speak of a module, it usually means a version of a module.

You can deploy multiple versions of the same module, to account for alternative implementations or progressive upgrades as time goes on.

Each module and each version must have a name. A name can contain numbers, letters, and hyphens. It cannot be longer than 63 characters and cannot start or end with a hyphen. Choose a unique name for each module and each version. Don't reuse names between modules and versions.

While running, a particular module/version will have one or more *instances*. Each instance runs its own separate executable. The number of instances running at any time depends on the module's scaling type and the amount of incoming requests:

![Hierarchy graph of modules/versions/instances](https://web.archive.org/web/20160425095823im_/https://cloud.google.com/appengine/docs/images/modules_hierarchy.png)

Stateful services (such as Memcache, Datastore, and Task Queues) are shared by all the modules in an application. Every module, version, and instance has its own unique URI (for example, `v1.my-module.my-app.appspot.com`). Incoming user requests are routed to an instance of a particular module/version according to [URL addressing conventions and an optional customized dispatch file](https://web.archive.org/web/20160425095823/https://cloud.google.com/appengine/docs/java/modules/routing).

**Note:** After April 2013 Google does not issue SSL certificates for double-wildcard domains hosted at `appspot.com` (i.e. `*.*.appspot.com`). If you rely on such URLs for HTTPS access to your application, change any application logic to use "`-dot-`" instead of "`.`". For example, to access version v1 of application myapp use `https://v1-dot-myapp.appspot.com`. The certificate will not match if you use `https://v1.myapp.appspot.com`, and an error occurs for any User-Agent that expects the URL and certificate to match exactly.

## Scaling types and instance classes

When you upload a version of a module, the configuration file specifies a *scaling type* and *instance class* that apply to every instance of that version. The *scaling type* controls how instances are created. The instance class determines compute resources and pricing. There are three scaling types: *manual*, *basic*, and *automatic*. Available instance classes depend on the scaling type.

<table>
<thead>
<tr class="header">
<th>Scaling Type</th>
<th>Description</th>
<th>Available Instance Classes</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Manual Scaling</td>
<td>A module with manual scaling runs continuously, allowing you to perform complex initialization and rely on the state of its memory over time.</td>
<td>B1, B2, B4, B4_1G, or B8</td>
</tr>
<tr class="even">
<td>Basic Scaling</td>
<td>A module with basic scaling will create an instance when the application receives a request. The instance will be turned down when the app becomes idle. Basic scaling is ideal for work that is intermittent or driven by user activity.</td>
<td>B1, B2, B4, B4_1G, or B8</td>
</tr>
<tr class="odd">
<td>Automatic Scaling</td>
<td>Automatic scaling is based on request rate, response latencies, and other application metrics.</td>
<td>F1, F2, F4, or F4_1G</td>
</tr>
</tbody>
</table>

This table summarizes the CPU, memory, and hourly billing rate of the various instance classes.

<table>
<thead>
<tr class="header">
<th>Instance Class</th>
<th>Memory Limit</th>
<th>CPU Limit</th>
<th>Cost per Hour per Instance</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>B1</td>
<td>128 MB</td>
<td>600 Mhz</td>
<td>$0.05</td>
</tr>
<tr class="even">
<td>B2</td>
<td>256 MB</td>
<td>1.2 Ghz</td>
<td>$0.10</td>
</tr>
<tr class="odd">
<td>B4</td>
<td>512 MB</td>
<td>2.4 Ghz</td>
<td>$0.20</td>
</tr>
<tr class="even">
<td>B4_1G</td>
<td>1024 MB</td>
<td>2.4 Ghz</td>
<td>$0.30</td>
</tr>
<tr class="odd">
<td>B8</td>
<td>1024 MB</td>
<td>4.8 Ghz</td>
<td>$0.40</td>
</tr>
<tr class="even">
<td>F1</td>
<td>128 MB</td>
<td>600 Mhz</td>
<td>$0.05</td>
</tr>
<tr class="odd">
<td>F2</td>
<td>256 MB</td>
<td>1.2 Ghz</td>
<td>$0.10</td>
</tr>
<tr class="even">
<td>F4</td>
<td>512 MB</td>
<td>2.4 Ghz</td>
<td>$0.20</td>
</tr>
<tr class="odd">
<td>F4_1G</td>
<td>1024 MB</td>
<td>2.4 Ghz</td>
<td>$0.30</td>
</tr>
</tbody>
</table>

Instances running in manual and basic scaling modules are billed at hourly rates based on uptime. Billing begins when an instance starts and ends fifteen minutes after a manual instance shuts down or fifteen minutes after a basic instance has finished processing its last request. Runtime overhead is counted against the instance memory limit. This will be higher for Java than for other languages.

**Important:** When you are billed for instance hours, you will not see any instance classes in your billing line items. Instead, you will see the appropriate multiple of instance hours. For example, if you use an F4 instance for one hour, you do not see "F4" listed, but you will see billing for four instance hours at the F1 rate.

This table compares the performance features of the three scaling types:

<table style="width:100%">
<thead>
<tr class="header">
<th style="width: 20%">Feature</th>
<th style="width: 25%">Automatic Scaling</th>
<th style="width: 30%">Manual Scaling</th>
<th style="width: 25%">Basic Scaling</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Deadlines</td>
<td>60-second deadline for HTTP requests, 10-minute deadline for tasks</td>
<td>Requests can run indefinitely. A manually-scaled instance can choose to handle <code>/_ah/start</code> and execute a program or script for many hours without returning an HTTP response code. Tasks can run up to 24 hours.</td>
<td>Same as manual scaling.</td>
</tr>
<tr class="even">
<td>Background Threads</td>
<td>Not allowed</td>
<td>Allowed</td>
<td>Allowed</td>
</tr>
<tr class="odd">
<td>Residence</td>
<td>Instances are evicted from memory based on usage patterns.</td>
<td>Instances remain in memory, and state is preserved across requests. When instances are restarted, an <code>/_ah/stop</code> request appears in the logs. If there is a registered stop callback method, it has 30 seconds to complete before shutdown occurs.</td>
<td>Instances are evicted based on the <code>idle-timeout</code> parameter. If an instance has been idle, i.e. has not received a request, for more than <code>idle-timeout</code>, then the instance is evicted.</td>
</tr>
<tr class="even">
<td>Startup and Shutdown</td>
<td>Instances are created on demand to handle requests and automatically turned down when idle.</td>
<td>Instances are sent a start request automatically by App Engine in the form of an empty GET request to <code>/_ah/start</code>. An instance that is stopped with <code>appcfg stop</code> (or from the Cloud Platform Console) has 30 seconds to finish handling requests before it is forcibly terminated.</td>
<td>Instances are created on demand to handle requests and automatically turned down when idle, based on the <code>idle-timeout</code> configuration parameter. As with manual scaling, an instance that is stopped with <code>appcfg stop</code> (or from the Cloud Platform Console) has 30 seconds to finish handling requests before it is forcibly terminated.</td>
</tr>
<tr class="odd">
<td>Instance Addressability</td>
<td>Instances are anonymous.</td>
<td>Instance "i" of version "v" of module "m" is addressable at the URL: <code>http://i.v.m.app_id.appspot.com</code>. If you have set up a <a href="https://web.archive.org/web/20160425095823/https://cloud.google.com/appengine/docs/java/console/using-custom-domains-and-ssl#wildcard_mappings">wildcard subdomain mapping</a> for a custom domain, you can also address a module or any of its instances via a URL of the form <code>http://m.domain.com</code> or <code>http://i.m.domain.com</code>. You can reliably cache state in each instance and retrieve it in subsequent requests.</td>
<td>Same as manual scaling.</td>
</tr>
<tr class="even">
<td>Scaling</td>
<td>App Engine scales the number of instances automatically in response to processing volume. This scaling factors in the <code>automatic_scaling</code> settings that are provided on a per-version basis in the configuration file uploaded with the module version.</td>
<td>You configure the number of instances of each module version in that module’s configuration file. The number of instances usually corresponds to the size of a dataset being held in memory or the desired throughput for offline work.</td>
<td>A basic scaling module version is configured with a maximum number of instances using the <code>basic_scaling</code> setting's <code>max_instances</code> parameter. The number of live instances scales with the processing volume.</td>
</tr>
<tr class="odd">
<td>Free Daily Usage Quota</td>
<td>28 instance-hours</td>
<td>8 instance-hours</td>
<td>8 instance-hours</td>
</tr>
</tbody>
</table>

## Configuration

An App Engine application that uses modules is organized as an unpacked Java Enterprise Archive (EAR) directory structure. The top-level EAR directory contains a single META-INF subdirectory, and a separate directory for each module in the app. These module directories are organized as unpacked Java Web Application Archives (WAR). Each WAR directory usually has the same name as the module it defines, but this is not required. Although Java EE supports WAR files, module configuration uses unpacked WAR directories only. App Engine's Java SDK includes an [Apache Maven plugin](https://web.archive.org/web/20160425095823/https://cloud.google.com/appengine/docs/java/tools/maven#creating_app_engine_applications_or_backend_apis_using_the_archetypes) with archetypes that you can use to build a skeletal EAR structure for you.

![Hierarchy graph of EAR directories](https://web.archive.org/web/20160425095823im_/https://cloud.google.com/appengine/docs/images/modules_file_hierarchy.png)

The META-INF directory has two configuration files: `appengine-application.xml` and `application.xml`. The `appengine-application.xml` file contains general information used by App Engine tools when your app is deployed. The `application.xml` file declares the list of modules and their WAR directories that comprise the application. The WAR directory for each module contains two configuration files: `appengine-web.xml`, and `web.xml`.

The WAR directory file [appengine-web.xml](https://web.archive.org/web/20160425095823/https://cloud.google.com/appengine/docs/java/config/appconfig) defines the configuration of modules. Each module has its own file, which defines the [scaling type and instance class](https://web.archive.org/web/20160425095823/https://cloud.google.com/appengine/docs/java/config/appconfig#scaling_and_instance_types) for a specific module/version. Different scaling parameters are used depending on which type of scaling you specify. If you do not specify scaling, automatic scaling is the default.

Note that while every `appengine-web.xml` file must contain the `<application>` tag, the name you supply there is ignored. The name of the application is taken from the `<application>` tag in the `appengine-application.xml` file.

For each module you can also specify settings that map URL requests to specific Java servlets and identify static files for better server efficiency. These settings are included in the `web.xml` and `appengine-web.xml` files and are described in the [Deployment Descriptor](https://web.archive.org/web/20160425095823/https://cloud.google.com/appengine/docs/java/config/webxml) and [App Config](https://web.archive.org/web/20160425095823/https://cloud.google.com/appengine/docs/java/config/appconfig) sections.

### The default module

Every application has a single default module. The default module is defined by the standard `appengine-web.xml` file or by an `appengine-web.xml` with the setting `<module>default</module>`. All configuration parameters relevant to modules can apply to the default module.

Also be sure to list the default module as the first module in the EAR directory's `META-INF/application.xml` file, as shown in the example below.

### Optional configuration files

These configuration files control optional features that apply to all the modules in an app:

-   `dispatch.xml`
-   `queue.xml`
-   `datastore-indexes.xml`
-   `cron.xml`
-   `dos.xml`

If you use any of these files, put them in the default module's WEB-INF directory.

### An example

Here is an example of how you would configure the various files in an EAR directory structure for an application that has two modules: a default module that handles web requests, plus another module (named `my-module`) for backend processing.

Assuming that the top-level EAR directory is "my-application," define the file `my-application/META-INF/appengine-application.xml`:

```
<?xml version="1.0" encoding="utf-8" standalone="no"?>
<appengine-application xmlns="http://appengine.google.com/ns/1.0">
  <application>my-application</application>
</appengine-application>
```

Create WAR directories for the two modules: `my-application/default` and `my-application/my-module`.

Now create an `appengine-web.xml` file in each WAR that specifies the parameters for the module. The file must include a version name for the module. To define the default module, you can explicitly include the `<module>default</module>` parameter or leave it out of the file. Here is the file `my-application/default/WEB-INF/appengine-web.xml` that defines the default module:

```
<?xml version="1.0" encoding="utf-8" standalone="no"?>
<appengine-web-app xmlns="http://appengine.google.com/ns/1.0">
  <application>my-application</application>
  <module>default</module>
  <version>uno</version>
  <threadsafe>true</threadsafe>
</appengine-web-app>
```

The file `my-application/my-module/WEB-INF/appengine-web.xml` defines the module that will handle background requests:

```
<?xml version="1.0" encoding="utf-8" standalone="no"?>
<appengine-web-app xmlns="http://appengine.google.com/ns/1.0">
  <application>my-application</application>
  <module>my-module</module>
  <version>uno</version>
  <threadsafe>true</threadsafe>
  <manual-scaling>
    <instances>5</instances>
  </manual-scaling>
</appengine-web-app>
```

Finally, define the file `my-application/META-INF/application.xml` that enumerates the modules. Note that the default module should be the first module listed.

```
<?xml version="1.0"
encoding="UTF-8"?>

<application
  xmlns="http://java.sun.com/xml/ns/javaee"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
                      http://java.sun.com/xml/ns/javaee/application_5.xsd"
  version="5">

  <description>GAE Java SuperFun app</description>
  <display-name>SuperFun</display-name>

  <!-- Modules -->
  <!-- The default module should be listed first -->
  <module>
    <web>
      <web-uri>default</web-uri>
      <context-root>default</context-root>
    </web>
  </module>
  <module>
    <web>
      <web-uri>my-module</web-uri>
      <context-root>my-module</context-root>
    </web>
  </module>

</application>
```

App Engine will ignore the `<context-root>` elements, so HTTP clients need not prepend it to the URL path when addressing a module.

## Uploading and deleting modules

To deploy the example above, use the [`appcfg update`](https://web.archive.org/web/20160425095823/https://cloud.google.com/appengine/docs/java/gettingstarted/uploading) command, passing the directory path of the EAR:
```
appcfg.sh update <directory-path>
```

To only upload a specific module, give the directory path for its WAR:

```
appcfg.sh update <directory-path>/my-application/my-module
```

If you are uploading the app for the first time, you must either pass the EAR path, or upload the WAR directory for the default module first.

You will receive verification via the command line as each module is successfully deployed. Once the application has been deployed you can access it at `http://my-application.appspot.com`.

Similarly, you can access each of the modules individually:

-   `http://default.my-application.appspot.com`
-   `http://my-module.my-application.appspot.com`

If you have uploaded multiple versions of the same module, you can access a specific version by prepending the version name to the URI. For example, `http://uno.default.my-application.appspot.com` will target version uno of the default module. [Routing Requests to Modules](https://web.archive.org/web/20160425095823/https://cloud.google.com/appengine/docs/java/modules/routing) explains addressing module instances in detail.

## Instance states

A manual or basic scaled instance can be in one of two states: `Running` or `Stopped`. All instances of a particular module/version share the same state. You can change the state of all the instances belonging to a module/version using the `appcfg` command or the Modules API.

### Startup

Each module instance is created in response to a start request, which is an empty GET request to `/_ah/start`. App Engine sends this request to bring an instance into existence; users cannot send a request to `/_ah/start`. Manual and basic scaling instances must respond to the start request before they can handle another request. The start request can be used for two purposes:

-   To start a program that runs indefinitely, without accepting further requests
-   To initialize an instance before it receives additional traffic

Manual, basic, and automatically scaling instances startup differently. When you start a manual scaling instance, App Engine immediately sends a `/_ah/start` request to each instance. When you start an instance of a basic scaling module, App Engine allows it to accept traffic, but the `/_ah/start` request is not sent to an instance until it receives its first user request. Multiple basic scaling instances are only started as necessary, in order to handle increased traffic. Automatically scaling instances do not receive any `/_ah/start` request.

When an instance responds to the `/_ah/start` request with an HTTP status code of `200–299` or `404`, it is considered to have successfully started and can handle additional requests. Otherwise, App Engine terminates the instance. Manual scaling instances are restarted immediately, while basic scaling instances are restarted only when needed for serving traffic.

### Shutdown

The shutdown process may be triggered by a variety of planned and unplanned events, such as:

-   You manually stop an instance using the `appcfg stop` command or the Modules API `stopVersion` function call.
-   Manually stop an instance from the Cloud Platform Console Instances page.
-   You update the module version using `appcfg update`.
-   The instance exceeds the maximum memory for its configured `instance_class`.
-   Your application runs out of Instance Hours quota.
-   The machine running the instance is restarted, forcing your instance to move to a different machine.
-   App Engine needs to move your instance to a different machine to improve load distribution.

There are two ways for an app to determine if a manual scaling instance is about to be shut down. First, the [`isShuttingDown()`](https://web.archive.org/web/20160425095823/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/LifecycleManager) method begins returning `true`. Second (and preferred), if you have [registered a shutdown hook](https://web.archive.org/web/20160425095823/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/LifecycleManager.ShutdownHook), it is called. When App Engine begins to shut down an instance, existing requests are given 30 seconds to complete, and new requests immediately return `404`.

As soon as App Engine starts to shut down an instance, all new requests immediately return `404`. If the instance is idle and there is a shutdown hook, the hook runs immediately. If the instance is handling a request, it is given 30 seconds to finish before the shutdown hook is called. If the handler does not complete, it will be interrupted and the hook will run.

When App Engine starts to shut down an instance, it sends an `/_ah/stop` request that appears in the log but is ignored by the request handling logic and cannot be handled by user code.

The following code sample demonstrates a basic shutdown hook:

```
LifecycleManager.getInstance().setShutdownHook(new ShutdownHook() {
  public void shutdown() {
    LifecycleManager.getInstance().interruptAllRequests();
  }
});
```

Alternatively, the following sample demonstrates how to use the isShuttingDown() method:

```
while (haveMoreWork()
&& !LifecycleManager.getInstance().isShuttingDown()) {
  doSomeWork();
  saveState();
}
```

## Instance uptime

App Engine attempts to keep manual and basic scaling instances running indefinitely. However, at this time there is no guaranteed uptime for manual and basic scaling instances. Hardware and software failures that cause early termination or frequent restarts can occur without prior warning and may take considerable time to resolve; thus, you should construct your application in a way that tolerates these failures. The App Engine team will provide more guidance on expected instance uptime as statistics become available.

Here are some good strategies for avoiding downtime due to instance restarts:

-   Use load balancing across multiple instances.
-   Configure more instances than are normally required to handle your traffic patterns.
-   Write fall-back logic that uses cached results when a manual scaling instance is unavailable.
-   Reduce the amount of time it takes for your instances to start up and shutdown.
-   Duplicate the state information across more than one instance.
-   For long-running computations, checkpoint the state from time to time so you can resume it if it doesn't complete.

It's also important to recognize that the shutdown hook is not always able to run before an instance terminates. In rare cases, an outage can occur that prevents App Engine from providing 30 seconds of shutdown time. Thus, we recommend periodically checkpointing the state of your instance and using it primarily as an in-memory cache rather than a reliable data store.

## Background threads

Code running on a manual scaling instance can start a background thread that may outlive the request that spawns it. This allows instances to perform arbitrary periodic or scheduled tasks or to continue working in the background after a request has returned to the user.

A background thread's logging entries are independent of those of the spawning thread. You can read more about background threads in App Engine's [`ThreadManager`](https://web.archive.org/web/20160425095823/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/ThreadManager) documentation.

```
import com.google.appengine.api.ThreadManager;
import java.util.concurrent.atomic.AtomicLong;

AtomicLong counter = new AtomicLong();

Thread thread = ThreadManager.createBackgroundThread(new Runnable() {
  public void run() {
    try {
      while (true) {
        counter.incrementAndGet();
        Thread.sleep(10);
      }
    } catch (InterruptedException ex) {
      throw new RuntimeException("Interrupted in loop:", ex);
    }
  }
});
thread.start();
```

If code running in an automatic scaling module attempts to start a background thread, it raises an exception.

The maximum number of concurrent background threads is 10 per instance.

## Monitoring resource usage

The Instances page of the [Cloud Platform Console](https://web.archive.org/web/20160425095823/https://cloud.google.com/appengine/docs/developers-console/#instances) provides visibility into how instances are performing. By selecting your module and version in the dropdowns, you can see the memory and CPU usage of each instance, uptime, number of requests, and other statistics. You can also manually initiate the shutdown process for any instance.

You also can use the Runtime API to access statistics showing the CPU and memory usage of your instances. These statistics help you understand how resource usage responds to requests or work performed, and also how to regulate the amount of data stored in memory in order to stay below the memory limit of your instance class.

## Logging

You can use the [Logs API](https://web.archive.org/web/20160425095823/https://cloud.google.com/appengine/docs/java/logs) to access your app's request and application logs. In particular, the `fetch()` function allows you to retrieve logs using various filters, such as request ID, timestamp, module ID, and version ID.

Application (user/app-generated) logs are periodically flushed while manual and basic scaling instances handle requests; since modules can run on a request a long time, logs may not flush for a while. You can tune the flush settings, or force an immediate flush, using the Logs API. When a flush occurs, a new log entry is created at the time of the flush, containing any log messages that have not yet been flushed. These entries show up in the Cloud Platform Console [Logs page](https://web.archive.org/web/20160425095823/https://cloud.google.com/appengine/docs/developers-console/#logs) marked with **flush**, and include the start time of the request that generated the flush.

## Communication between modules

Modules can share state by using the Datastore and Memcache. They can collaborate by assigning work between them using Task Queues. To access these shared services, use the corresponding App Engine APIs. Calls to these APIs are automatically mapped to the application’s namespace.

The Modules API provides functions to retrieve the address of a module, a version, or an instance. This allows an application to send requests from one module, version, or instance to another module, version, or instance. This works in both the development and production environments. The Modules API also provides functions that return information about the current operating environment (module, version, and instance).

The following code sample shows how to get the module name and instance id for a request:

```
import com.google.appengine.api.modules.ModulesService;
import com.google.appengine.api.modules.ModulesServiceFactory;

ModulesService modulesApi = ModulesServiceFactory.getModulesService();

// Get the module name handling the current request.
String currentModuleName = modulesApi.getCurrentModule();
// Get the instance handling the current request.
int currentInstance = modulesApi.getCurrentInstance();
```

The instance ID of an automatic scaled module will be returned as a unique base64 encoded value, e.g. `e4b565394caa`.

You can communicate between modules in the same app by fetching the hostname of the target module:

```
import com.google.appengine.api.modules.ModulesService;
import com.google.appengine.api.modules.ModulesServiceFactory;

import java.net.MalformedURLException;
import java.net.URL;
import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.io.IOException;

// ...

ModulesService modulesApi = ModulesServiceFactory.getModulesService();

// ...
    try {
        URL url = new URL("http://" +
            modulesApi.getVersionHostname("my-backend-module","v1") +
            "/fetch-stats");
        BufferedReader reader = new BufferedReader(
            new InputStreamReader(url.openStream()));
        String line;

        while ((line = reader.readLine()) != null) {
            // Do something...
        }
        reader.close();

    } catch (MalformedURLException e) {
        // ...
    } catch (IOException e) {
        // ...
    }
```

You can also use the [URL Fetch](https://web.archive.org/web/20160425095823/https://cloud.google.com/appengine/docs/java/urlfetch) service.

To be safe, the receiving module should validate that the request is coming from a valid client. You can check that the Inbound-AppId header or user-agent-string matches the app-id fetched with the [AppIdentity](https://web.archive.org/web/20160425095823/https://cloud.google.com/appengine/docs/java/appidentity) service. Or you can use [OAuth](https://web.archive.org/web/20160425095823/https://cloud.google.com/appengine/docs/java/oauth) to authenticate the request.

You can configure any manual or basic scaling module to accept requests from other modules in your app by restricting its handler to only allow administrator accounts, using a [`<security-constraint>`](https://web.archive.org/web/20160425095823/https://cloud.google.com/appengine/docs/java/config/webxml#Security_and_Authentication) with a `<role-name>admin</role-name>` in the module's `web.xml` deployment descriptor. With this restriction in place, any URLFetch from any other module in the app will be automatically authenticated by App Engine, and any request that is not from the application will be rejected.

If you want a module to receive requests from anywhere, you must code your own secure solution as you would for any App Engine application. This is usually done by implementing a custom API and authentication mechanism.

## Limits

The maximum number of modules and versions that you can deploy depends on your app's pricing:

<table>
<thead>
<tr class="header">
<th>Limit</th>
<th>Free App</th>
<th>Paid App</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Maximum Modules Per App</td>
<td>5</td>
<td>20</td>
</tr>
<tr class="even">
<td>Maximum Versions Per App</td>
<td>15</td>
<td>60</td>
</tr>
</tbody>
</table>

There is also a limit to the number of instances for each module with basic or manual scaling:

<table>
<thead>
<tr class="header">
<th colspan="3" style="text-align: center;">Maximum Instances per Manual/Basic Scaling Version</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<th>Free App</th>
<th>Paid App US</th>
<th>Paid App EU</th>
</tr>
&#10;<tr class="odd">
<td>20</td>
<td>200</td>
<td>25</td>
</tr>
</tbody>
</table>
