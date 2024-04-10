# What is Stackdriver Monitoring?

> **Beta**
> This is a Beta release of Stackdriver Monitoring. This feature is not covered by any SLA or deprecation policy and may be subject to backward-incompatible changes.

Stackdriver Monitoring provides dashboards and alerts for your cloud-powered applications. You configure Stackdriver Monitoring using the [Stackdriver Monitoring Console](https://web.archive.org/web/20160424225556/https://app.google.stackdriver.com/). Review performance metrics for cloud services, virtual machines, and common open source servers such as MongoDB, Apache, Nginx, Elasticsearch, and more. Use the Stackdriver Monitoring API to retrieve monitoring data and create custom metrics.

**Stackdriver Monitoring is simple to set up.** Get charting and alerting tools out of the box. Use the smart defaults or define your own custom dashboards.

**Receive alerts when issues occur.** Receive alerts via email, SMS, PagerDuty, HipChat, and more. Alert on individual metrics and thresholds or on aggregate group performance.

**Integrate with common open source software.** Stackdriver Monitoring collects metrics from many common open source servers with minimal configuration. It helps you discover trends unique to your Cassandra clusters or Nginx servers.

To provide feedback on the beta release of Stackdriver Monitoring, click **Send Feedback** on the **Help** menu in the [Stackdriver Monitoring Console](https://web.archive.org/web/20160424225556/https://app.google.stackdriver.com/).

Stackdriver Monitoring concepts
-------------------------------

### Resources and groups

A **resource** is an abstract object provided by certain products, platforms, or services within Google Cloud Platform. For example, Google Compute Engine provides a resource called **VM instance**, Cloud SQL provides a **database instance**, Cloud Pub/Sub provides **topics**, and so forth.

A **group** is a collection of resources, such as "all VM instances in my project that are running Cassandra." Stackdriver Monitoring can automatically detect resources that are related and group them together. You can create your own groups based on resource names, tags and other attributes. Groups are not presently available in App Engine-only projects.

### Metrics

A **metric** is a measured value that can be used to assess a system. Examples of metrics in the Google Cloud Platform include CPU utilization, request processing latency, amount of storage consumed, and so forth. Metrics are typically qualified, or _labeled_, by particular resources. For example, you can measure the CPU utilization _of VM instance A_, the request processing latency _in web server B_, the amount of storage used _by project C_, and so forth. When a metric is measured over time, it produces a **time series** of data.

A **service metric** or **platform metric** is a metric that is provided for you when you use a particular service or platform. For example, Google App Engine provides a set of platform metrics to its users.

You can define **custom metrics** to measure any aspect of your system. Custom metrics might include cart checkouts, user logins, or business KPIs. You are responsible for sending your custom metrics data to Stackdriver Monitoring using the [custom-metrics API](https://web.archive.org/web/20160424225556/https://cloud.google.com/monitoring/custom-metrics).

### Charts and dashboards

A **chart** is a named, visual representation of one or more metrics. You can create a chart that displays the time series data for one or more metrics, or you can aggregate several metrics. For example, you can create a chart named "Web tier CPU utilization" that shows the average CPU utilization for a group of VMs.

A **dashboard** is a collection of charts and other information such as event logs and incident lists. Dashboards are organized in a single page view. You can create a dashboard to show a snapshot of your system's overall health or to show specific information about factors influencing that health. You can add or remove charts from dashboards as needed.

### Uptime checks and events

**Uptime checks** is a service of Stackdriver Monitoring. You configure the service to check your system's health by sending requests to your applications, services, or URLs from various locations around the world. You can use the results of the checks as conditions in your alert policies, so you will be notified if system health is degraded.

An **event log** is a time-ordered list of system events related to your application and to the platforms and services it uses. Examples of events include downtime of the cloud infrastructure, notifications related to your alert policy, and code deployments. You can also add your own events to the log. Event logs are supported by Stackdriver Monitoring and are different from the logs supported by Stackdriver Logging.

### Alerts, notifications, and incidents

An **alert policy** is a set of rules that determine whether your resources or groups are operating normally. The rules are logical conditions involving metric thresholds and uptime checks. For example, you can create a rule that your web site's average response latency must not exceed five seconds over a period of two minutes.

An **alert** occurs when an alert policy's conditions are met, causing an **incident** to appear in the **Incidents** section of the Stackdriver Monitoring Console. Incidents remain open until the alert policy rules are no longer in violation or until the incident is manually closed.

You can associate **notifications** with alert policies. For example, alerts can send email or SMS notifications to people or services.