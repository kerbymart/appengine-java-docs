# How Applications Scale

  

[Python](https://web.archive.org/web/20160424230901/https://cloud.google.com/appengine/docs/python/scaling "View this page in the Python runtime") <span style="color: #ddd; padding: 0em .5em;">|</span><span style="color: black; font-weight:bold">Java</span> <span style="color: #ddd; padding: 0em .5em;">|</span>[PHP](https://web.archive.org/web/20160424230901/https://cloud.google.com/appengine/docs/php/scaling "View this page in the PHP runtime") <span style="color: #ddd; padding: 0em .5em;">|</span>[Go](https://web.archive.org/web/20160424230901/https://cloud.google.com/appengine/docs/go/scaling "View this page in the Go runtime")

Instances are the basic building blocks of App Engine, providing all the resources needed to successfully host your application. This includes the language runtime, the App Engine APIs, and your application's code and memory. Each instance includes a security layer to ensure that instances cannot inadvertently affect each other.

Instances are the computing units that App Engine uses to automatically scale your application. At any given time, your application may be running on one instance or many instances, with requests being spread across all of them.

1.  [Introduction to Instances](#introduction_to_instances)
2.  [Scaling dynamic instances](#scaling_dynamic_instances)
3.  [Request throughput and latency](#request_throughput_and_latency)
4.  [Special kinds of requests](#special_kinds_of_requests)
    1.  [Loading requests](#loading_requests)
    2.  [Warmup requests](#warmup_requests)
5.  [Instance billing](#instance_billing)

## Introduction to Instances

Instances are *resident* or *dynamic*. A dynamic instance starts up and shuts down automatically based on the current needs. A resident instance runs all the time, which can improve your application's performance.

An instance instantiates the code which is included in an App Engine [module](https://web.archive.org/web/20160424230901/https://cloud.google.com/appengine/features/#modules).

When you configure your app, you specify how its modules scale (the initial number of instances for a module and how new instances are created and stopped in response to traffic), and how much time an instance is allowed to handle a request (its deadline).

The scaling class that you assign to a module determines the kind(s) of instances created:

-   Manual scaling modules use resident instances
-   Basic scaling modules use dynamic instances
-   Auto scaling modules use dynamic instances - but if you specify a number, N, of minimum idle instances, the first N instances will be resident, and additional dynamic instances will be created as necessary.

App Engine charges for instance usage on an hourly basis. You can track your instance usage on the Google Cloud Platform Console [Instances page](https://web.archive.org/web/20160424230901/https://cloud.google.com/appengine/docs/developers-console/#instances). If you want to set a limit on incurred instance costs, you can do so by [setting a spending limit](https://web.archive.org/web/20160424230901/https://cloud.google.com/appengine/docs/java/console#billing).

When App Engine was first released, the modules architecture did not exist. Applications were constructed with one frontend and multiple, optional [backends](https://web.archive.org/web/20160424230901/https://cloud.google.com/appengine/docs/python/backends). A frontend behaves like an auto scaling version of the default module. A backend can behave as either a manual or basic scaling module, depending on how it's configured. **Note that backends have been deprecated**.

## Scaling dynamic instances

App Engine applications are powered by any number of dynamic instances at a given time, depending on the volume of incoming requests. As requests for your application increase, so do the number of dynamic instances.

The App Engine scheduler decides whether to serve each new request with an existing instance (either one that is idle or accepts concurrent requests), put the request in a pending request queue, or start a new instance for that request. The decision takes into account the number of available instances, how quickly your application has been serving requests (its latency), and how long it takes to spin up a new instance.

Each instance has its own queue for incoming requests. App Engine monitors the number of requests waiting in each instance's queue. If App Engine detects that queues for an application are getting too long due to increased load, it automatically creates a new instance of the application to handle that load.

App Engine also scales instances in reverse when request volumes decrease. This scaling helps ensure that all of your application's current instances are being used to optimal efficiency and cost effectiveness.

When an application is not being used at all, App Engine turns off its associated dynamic instances, but readily reloads them as soon as they are needed. Reloading instances may result in loading requests and additional latency for users.

You can specify a minimum number of idle instances. Setting an appropriate number of idle instances for your application based on request volume allows your application to serve every request with little latency, unless you are experiencing abnormally high request volume.

## Request throughput and latency

Your application's latency has the biggest impact on the number of instances needed to serve your traffic. If you process requests quickly, a single instance can handle a lot of requests.

Single-threaded instances (Python or Java) can currently handle one concurrent request. Therefore, there is a direct relationship between the latency and number of requests that can be handled on the instance per second. For example, 10ms latency equals 100 request/second/instance, 100ms latency equals 10 request/second/instance, etc. Multi-threaded instances can handle many concurrent requests. Therefore, there is a direct relationship between the CPU consumed and the number of requests/second.

[Java](https://web.archive.org/web/20160424230901/https://cloud.google.com/appengine/docs/java/config/appconfig#Java_appengine_web_xml_Using_concurrent_requests) and [Python 2.7](https://web.archive.org/web/20160424230901/https://cloud.google.com/appengine/docs/python/config/appconfig#Python_app_yaml_Using_concurrent_requests) apps support concurrent requests, so a single instance can handle new requests while waiting for other requests to complete. Concurrency significantly reduces the number of instances your app requires, but you need to design your app specifically with multithreading in mind.

For example, if a [B4 instance](https://web.archive.org/web/20160424230901/https://cloud.google.com/appengine/docs/java/modules/#Java_Instance_scaling_and_class) (approx 2.4GHz) consumes 10 Mcycles/request, you can process 240 requests/second/instance. If it consumes 100 Mcycles/request, you can process 24 requests/second/instance, etc. These numbers are the ideal case but are fairly realistic in terms of what you can accomplish on an instance. Multi-Threaded instances are not available for the Python 2.5 runtime.

## Special kinds of requests

### Loading requests

When App Engine creates a new instance for your application, the instance must first load any libraries and resources required to handle the request. This happens during the first request to the instance, called a *Loading Request*. During a loading request, your application undergoes initialization which causes the request to take longer.

The following best practices allow you to reduce the duration of loading requests:

-   Load only the code needed for startup.
-   Access the disk as little as possible.
-   In some cases, loading code from a zip or jar file is faster than loading from many separate files.

### Warmup requests

Warmup requests are a specific type of loading request that load application code into an instance ahead of time, before any live requests are made. To learn more about how to use warmup requests, see [Warmup Requests](https://web.archive.org/web/20160424230901/https://cloud.google.com/appengine/docs/java/config/appconfig#Java_Warmup_requests).

**Note:** Warmup requests are not always called for every new instance. For example, if the new instance is the very first instance of your application, the user's request is sent to the application directly. As a result, you may still encounter loading requests, even with warmup requests enabled.

## Instance billing

In general, instances are charged per-minute for their uptime in addition to a 15-minute startup fee. Billing begins when the instance starts and ends fifteen minutes after the instance shuts down (see [Pricing](https://web.archive.org/web/20160424230901/https://cloud.google.com/appengine/pricing) for details). You will be billed only for idle instances up to the number of maximum idle instances that you set for each module. Runtime overhead is counted against the instance memory . This will be higher for Java applications than Python.

Billing is slightly different in resident and dynamic instances:

-   For resident instances, billing ends fifteen minutes after the instance is shut down.
-   For dynamic instances, billing ends fifteen minutes after the last request has finished processing.
