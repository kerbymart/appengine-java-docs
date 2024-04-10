# Monitoring Latency with Stackdriver Trace

# Stackdriver Trace - Find Perf Bottlenecks - Google Cloud Platform — Google Cloud Platform
Detailed Performance Insights
-----------------------------

Stackdriver Trace is a **distributed tracing system** for Google Cloud Platform that collects latency data from your applications and displays it in the Google Cloud Platform Console. You can see detailed **near real-time insights** into **application performance**. Stackdriver Trace automatically analyzes all your application traces to generate in-depth **performance reports** to surface application performance degradations and call flow performance bottlenecks.

![](https://web.archive.org/web/20160424223151im_/https://cloud.google.com/images/products/stackdriver-trace/what-is-it.png)

Find Performance Bottlenecks
----------------------------

Using Stackdriver Trace, you can view detailed latency information for a single request all the way to aggregated latency for the entire application. Using the various tools and filters provided, you can quickly find where the bottlenecks are occurring and their root cause.

![](https://web.archive.org/web/20160424223151im_/https://cloud.google.com/images/products/stackdriver-trace/performance-bottlenecks.png)

Fast, Automatic Issue Detection
-------------------------------

Stackdriver Trace continuously gathers trace data for your project. Trace continuously analyzes trace data associated with individual requests to automatically identify performance bottlenecks in your call graphs. At an aggregate level, it continuously looks for performance degradation of your application. When generating reports comparing latency distribution over time or versions, Trace also analyzes for latency shifts so that you can catch problems even before they become customer impacting issues.

![](https://web.archive.org/web/20160424223151im_/https://cloud.google.com/images/products/stackdriver-trace/fast-automatic-issue.png)

Trace Your Custom Workloads
---------------------------

If you have your own custom workloads that you wish to trace and analyze, the Trace API and Trace SDK can now be used to optimize the performance of those workloads. The Trace SDK can be used to add custom spans. Custom spans can be used to capture latency information for your own custom workloads. You can upload custom Trace data using the Trace API. The trace data will be combined with the remaining application traces. Currently the Trace SDK is available for Java and Node.js. A REST API is available for all.

![](https://web.archive.org/web/20160424223151im_/https://cloud.google.com/images/products/stackdriver-trace/trace-workloads.png)

# Cloud Stackdriver Trace Features

## Find performance bottlenecks in production.

Easy Set Up

All app engine applications are automatically traced. All performance reports and analysis described above is available out of the box.

Automatic Analysis

Automatic daily performance reports are created for each app engine application. You can also generate reports on demand.

Latency Shift Detection

Performance reports of your application are evaluated over time to identify latency degradation of your application over time.

Performance Insights

Each end point level trace is evaluated automatically for performance bottlenecks.

Extensibility For Custom Workloads

Trace API and language specific SDK available to custom trace your applications and write data back to Stackdriver Trace.


> “ Analytics in trace allowed us to quickly establish that latency had changed. Stackdriver Trace timelines showed us where the latency bottlenecks were occurring and helped us get back on our feet. ”
>
> **\- Keith Marsh** _Principal Online Technologist, Dovetail Games_