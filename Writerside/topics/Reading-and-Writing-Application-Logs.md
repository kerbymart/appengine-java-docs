# Reading and Writing Application Logs

  

[Python](https://web.archive.org/web/20160425095405/https://cloud.google.com/appengine/docs/python/logs/ "View this page in the Python runtime") <span style="color: #ddd; padding: 0em .5em;">|</span><span style="color: black; font-weight:bold">Java</span> <span style="color: #ddd; padding: 0em .5em;">|</span>[PHP](https://web.archive.org/web/20160425095405/https://cloud.google.com/appengine/docs/php/logs/ "View this page in the PHP runtime") <span style="color: #ddd; padding: 0em .5em;">|</span>[Go](https://web.archive.org/web/20160425095405/https://cloud.google.com/appengine/docs/go/logs/ "View this page in the Go runtime")

[<img src="https://web.archive.org/web/20160425095405im_/https://cloud.google.com/cloud/images/stack_overflow_questions.png" title="Stack Overflow Questions" data-border="0" width="240" height="53" />](https://web.archive.org/web/20160425095405/http://stackoverflow.com/questions/tagged/google-app-engine+logging)

[](https://web.archive.org/web/20160425095405/http://stackoverflow.com/feeds/tag?sort=votes&tagnames=google-app-engine%2Blogging)

<span class="link gc-analytics-event" category="Sidebar" data-action="Stack Overflow question click"></span>

<a href="https://web.archive.org/web/20160425095405/http://stackoverflow.com/questions/tagged/google-app-engine+logging?sort=votes" class="gc-analytics-event" data-category="Sidebar" data-action="Stack Overflow See more link">See more...</a>

1.  [Overview](#Java_Overview)
2.  [Request logs vs application logs](#request_logs_vs_application_logs)
3.  [Writing application logs](#Java_writing_application_logs)
4.  [Reading logs via API](#Java_Getting_log_data)
5.  [Log URL formats in App Engine and developer consoles](#Java_log_formats_in_consoles)
6.  [Reading logs in the console](#reading_logs_in_the_console)
7.  [Understanding request log fields](#understanding_request_log_fields)
8.  [Quotas and limits](#Java_Quotas_and_limits)

## Overview

When a request is sent to your app, a request log is automatically written by App Engine. During the handling of the request, your app can also write application logs. In this page, we'll show you how to write application logs from your application, how to read both application and request logs programmatically using the Logs API, how to <a href="https://web.archive.org/web/20160425095405/https://console.cloud.google.com/logs" data-track-metadata-end-goal="viewLogs" data-track-metadata-position="body" data-track-name="consoleLink" data-track-type="tasks">view logging</a> in the Google Cloud Platform Console, and how to understand the request log data that App Engine writes during the request.

## Request logs vs application logs

There are two categories of log data: request logs and application logs. A request log is automatically written by App Engine for each request handled by your app, and contains information such as the app ID, HTTP version, and so forth. For a complete list of available properties for request logs, see [RequestLogs](https://web.archive.org/web/20160425095405/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/log/RequestLogs). See also the [request log table](#understanding_request_log_fields) for descriptions of the request log fields.

Each request log contains a list of application logs ([AppLogLine](https://web.archive.org/web/20160425095405/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/log/AppLogLine)) associated with that request, returned in the `RequestLogs.getAppLogLines()` method. Each app log contains the time the log was written, the log message, and the log level.

**Note:** A request log can only hold 1000 app logs. If you add more than 1000, the older logs will not be retained.

## Writing application logs

**Important:** Individual log entries from an application have an 8KB limit. Nothing beyond this limit will be written to the app log.

The App Engine Java SDK allows a developer to log the following levels of severity:

-   SEVERE
-   WARNING
-   INFO
-   CONFIG
-   FINE
-   FINER
-   FINEST

In order to add logging to your Java app, you'll first need to add the appropriate system properties to your project's `appengine-web.xml` file, and you may also need to modify the `logging.properties` file to set the desired log level. Both of these files are created for you when you create a new App Engine Java project [using Maven](https://web.archive.org/web/20160425095405/https://cloud.google.com/appengine/docs/java/tools/maven). You can find these files at the following locations:

![Maven Project Layout](https://web.archive.org/web/20160425095405im_/https://cloud.google.com/appengine/docs/java/tools/images/maven_layout.png)

Edit the `appengine-web.xml` file to add the following inside the `<appengine-web-app>` tags:

```
    <system-properties>
       <property name="java.util.logging.config.file" value="WEB-INF/logging.properties"/>
    </system-properties>
```

Notice that the default log level in `logging.properties` is `WARNING`, which suppresses `INFO` messages from the output. To change the log level for all classes in your app, edit the `logging.properties` file to change the level. For example you could change `.level = WARNING` to `.level = INFO`.

In your application code, write log messages as desired using the [java.util.logging.Logger](https://web.archive.org/web/20160425095405/http://docs.oracle.com/javase/6/docs/api/java/util/logging/Logger.html) API. The example below writes an Info log message:

```
import java.util.logging.Logger;
//...

public class MyClass {

  private static final Logger log = Logger.getLogger(MyClass.class.getName());
  log.info("Your information log message.");
  //....
```

When your app runs, App Engine records the messages and makes them available. You can read them programmatically using the Logs API as shown next, or you can browse them in the <a href="https://web.archive.org/web/20160425095405/https://console.cloud.google.com/logs" data-track-metadata-end-goal="viewLogs" data-track-metadata-position="body" data-track-name="consoleLink" data-track-type="article">Logs Viewer</a>. You can also download the log files using the [AppCfg](https://web.archive.org/web/20160425095405/https://cloud.google.com/appengine/docs/java/tools/uploadinganapp#Downloading_Logs) tool.

## Reading logs via API

**Note:** The Logs API only generates output for deployed apps.

The general process of getting logs using the Logs API is as follows:

1.  Use [`LogQuery`](https://web.archive.org/web/20160425095405/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/log/LogQuery) to specify which logs to return.
2.  Use [`LogServiceFactory.getLogService()`](https://web.archive.org/web/20160425095405/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/log/LogServiceFactory) to create the LogService
3.  Invoke `LogServiceFactory.getLogService().fetch()` to return an iterator for the request logs.
4.  In each iteration, for each `RequestLogs`, process the request properties as desired.
5.  Optionally, use `RequestLogs.getAppLogLines()` to get the list of app logs (`AppLogLine`) associated with this request.
6.  If you retrieved the app logs list, for each `AppLogLine`, process the property data as desired.

### Sample code

The following sample displays 5 request logs at at time, along with their application logs. It lets you cycle through each set of 5 logs using a **Next** link.

```
package testapp;

import java.io.IOException;
import javax.servlet.http.*;
import com.google.appengine.api.log.*;
import java.io.PrintWriter;
import java.util.Calendar;

// Get request logs along with their app log lines and display them 5 at
// a time, using a Next link to cycle through to the next 5.
public class TestAppServlet extends HttpServlet {
  @Override
  public void doGet(HttpServletRequest req, HttpServletResponse resp)
         throws IOException {

    resp.setContentType("text/html");
    PrintWriter writer = resp.getWriter();
    // We use this to break out of our iteration loop, limiting record
    // display to 5 request logs at a time.
    int limit = 5;

    // This retrieves the offset from the Next link upon user click.
    String offset = req.getParameter("offset");

    // We want the App logs for each request log
    LogQuery query = LogQuery.Builder.withDefaults();
    query.includeAppLogs(true);

    // Set the offset value retrieved from the Next link click.
    if (offset != null) {
      query.offset(offset);
    }

    // This gets filled from the last request log in the iteration
    String lastOffset = null;
    int i = 0;

    // Display a few properties of each request log.
    for (RequestLogs record : LogServiceFactory.getLogService().fetch(query)) {
      writer.println("<br />REQUEST LOG <br />");
      Calendar cal = Calendar.getInstance();
      cal.setTimeInMillis(record.getStartTimeUsec() / 1000);

      writer.println("IP: " + record.getIp()+"<br />");
      writer.println("Method: " + record.getMethod()+"<br />");
      writer.println("Resource " + record.getResource()+"<br />");
      writer.println(String.format("<br />Date: %s", cal.getTime().toString()));

      lastOffset = record.getOffset();

      // Display all the app logs for each request log.
      for (AppLogLine appLog : record.getAppLogLines()) {
        writer.println("<br />"+ "APPLICATION LOG" +"<br />");
        Calendar appCal = Calendar.getInstance();
        appCal.setTimeInMillis(appLog.getTimeUsec() / 1000);
        writer.println(String.format("<br />Date: %s",
                            appCal.getTime().toString()));
        writer.println("<br />Level: "+appLog.getLogLevel()+"<br />");
        writer.println("Message: "+ appLog.getLogMessage()+"<br /> <br />");
      } //for each log line

      if (++i >= limit) {
        break;
      }
    } // for each record

    // When the user clicks this link, the offset is processed in the
    // GET handler and used to cycle through to the next 5 request logs.
    writer.println(String.format("<br><a href=\"/?offset=%s\">Next</a>",
                             lastOffset));
  }  // end doGet
} //end class
```

In the sample, notice that the GET handler expects to be re-invoked by the user clicking on the Next link, and so it extracts the offset param, if present. That offset is used in the subsequent re-invocation of `LogServiceFactory.getLogService().fetch()` to "page through" each group of 5 request logs. There is nothing special about the number 5; it can be anything you want.

## Log URL format in the Google Cloud Platform Console

See the following sample URL for an example of the log URL format in the Cloud Platform Console:

`https://console.cloud.google.com/logs?filters=request_id:000000db00ff00ff827e493472570001737e73686966746361727331000168656164000100`

**Note:** The URL no longer includes the `filter_type`.

## Reading logs in the console

**Note:** the logs described here are shown as visible in the Log Viewer.

To view logs using the Log Viewer:

1.  In the Cloud Platform Console, go to <a href="https://web.archive.org/web/20160425095405/https://console.cloud.google.com/logs" data-track-metadata-end-goal="viewLogs" data-track-metadata-position="body" data-track-name="consoleLink" data-track-type="tasks">the logs for your project</a>.

2.  Use the desired filter to retrieve the logs you want to see. You can filter by various combinations of time, log level, module, and log filter label or regular expression.

    Notice that labels are regular expressions for filtering the logs by logging fields. Valid labels include the following:

    -   day
    -   month
    -   year
    -   hour
    -   minute
    -   second
    -   tzone
    -   remotehost
    -   identd\_user
    -   user
    -   status
    -   bytes
    -   referrer
    -   useragent
    -   method
    -   path
    -   querystring
    -   protocol
    -   request\_id

    For example, `path:/foo.* useragent:.*Chrome.*` gets logs for all requests to a path starting with `/foo` that were issued from a Chrome browser.

A typical App Engine log contains data in the [Apache combined log format](https://web.archive.org/web/20160425095405/http://httpd.apache.org/docs/1.3/logs.html#combined), along with some special App Engine fields, as shown in the following sample log:

```
192.0.2.0 - test [27/Jun/2014:09:11:47 -0700] "GET / HTTP/1.1" 200 414
"http://www.example.com/index.html"
Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"
"1-dot-calm-sylph-602.appspot.com" ms=195 cpu_ms=42 cpm_usd=0.000046
loading_request=1 instance=00c61b117cfeb66f973d7df1b7f4ae1f064d app_engine_release=1.9.34
```

## Understanding request log fields

The following table lists the fields in order of occurrence along with a description:

<table>
<thead>
<tr class="header">
<th>Field Order</th>
<th>Field Name</th>
<th>Always Present?</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>1</td>
<td>Client address</td>
<td>Yes</td>
<td>Client IP address. Example: <code>192.0.2.0</code></td>
</tr>
<tr class="even">
<td>2</td>
<td>RFC1413 identity</td>
<td>No</td>
<td>RFC1413 identity of the client. This is nearly always the character <code>-</code></td>
</tr>
<tr class="odd">
<td>3</td>
<td>User</td>
<td>No</td>
<td>Present only if the app uses the Users API and the user is logged in. This value is the "nickname" portion of the Google Account, for example, if the Google Account is <code>test@example.com</code>, the nickname that is logged in this field is <code>test</code>.</td>
</tr>
<tr class="even">
<td>4</td>
<td>Timestamp</td>
<td>Yes</td>
<td>Request timestamp. Example: <code>[27/Jun/2014:09:11:47 -0700]</code></td>
</tr>
<tr class="odd">
<td>5</td>
<td>Request querystring</td>
<td>Yes</td>
<td>First line of the request, containing method, path, and HTTP version. Example: <code>GET / HTTP/1.1</code></td>
</tr>
<tr class="even">
<td>6</td>
<td>HTTP Status Code</td>
<td>Yes</td>
<td>Returned HTTP status code. Example: <code>200</code></td>
</tr>
<tr class="odd">
<td>7</td>
<td>Response size</td>
<td>Yes</td>
<td>Response size in bytes. Example: <code>414</code></td>
</tr>
<tr class="even">
<td>8</td>
<td>Referrer path</td>
<td>No</td>
<td>If there is no referrer, the log contains no path, but only <code>-</code>. Example referrer path: <code>"http://www.example.com/index.html"</code>.</td>
</tr>
<tr class="odd">
<td>9</td>
<td>User-agent</td>
<td>Yes</td>
<td>Identifies the browser and operating system to the web server. Example: <code>Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36</code></td>
</tr>
<tr class="even">
<td>10</td>
<td>Hostname</td>
<td>Yes</td>
<td>The hostname used by the client to connect to the App Engine application. Example : (<code>1-dot-calm-sylph-602.appspot.com</code>)</td>
</tr>
<tr class="odd">
<td>11</td>
<td>Wallclock time</td>
<td>Yes</td>
<td>Total clock time in milliseconds spent by App Engine on the request. This time duration does not include time spent between the client and the server running the instance of your application. Example: <code>ms=195</code>.</td>
</tr>
<tr class="even">
<td>12</td>
<td>CPU milliseconds</td>
<td>Yes</td>
<td>CPU milliseconds required to fulfill the request. This is the number of milliseconds spent by the CPU actually executing your application code, expressed in terms of a baseline 1.2 GHz Intel x86 CPU. If the CPU actually used is faster than the baseline, the CPU milliseconds can be larger than the actual clock time defined above. Example: <code>cpu_ms=42</code></td>
</tr>
<tr class="odd">
<td>13</td>
<td>Exit code</td>
<td>No</td>
<td>Only present if the instance shut down after getting the request. In the format <code>exit_code=XXX</code> where <code>XXX</code> is a 3 digit number corresponding to the reason the instance shut down. The exit codes are not documented since they are primarily intended to help Google spot and fix issues.</td>
</tr>
<tr class="even">
<td>14</td>
<td>Estimated cost</td>
<td>Yes</td>
<td>DEPRECATED. Estimated cost of 1000 requests just like this one, in USD. Example: <code>cpm_usd=0.000046</code></td>
</tr>
<tr class="odd">
<td>15</td>
<td>Queue name</td>
<td>No</td>
<td>The name of the task queue used. Only present if request used a task queue. Example: <code>queue_name=default</code></td>
</tr>
<tr class="even">
<td>16</td>
<td>Task name</td>
<td>No</td>
<td>The name of the task executed in the task queue for this request. Only present if the request resulted in the queuing of a task. Example: <code>task_name=7287390692361099748</code></td>
</tr>
<tr class="odd">
<td>17</td>
<td>Pending queue</td>
<td>No</td>
<td>Only present if a request spent some time in a pending queue. If there are many of these in your logs and/or the values are high, it might be an indication that you need more instances to serve your traffic. Example: <code>pending_ms=195</code></td>
</tr>
<tr class="even">
<td>18</td>
<td>Loading request</td>
<td>No</td>
<td>Only present if the request is a loading request. This means an instance had to be started up. Ideally, your instances should be up and healthy for as long as possible, serving large numbers of requests before being recycled and needing to be started again. Which means you shouldn't see too many of these in your logs. Example: <code>loading_request=1</code>.</td>
</tr>
<tr class="odd">
<td>19</td>
<td>Instance</td>
<td>Yes</td>
<td>Unique identifier for the instance that handles the request. Example: <code>instance=00c61b117cfeb66f973d7df1b7f4ae1f064d</code></td>
</tr>
<tr class="even">
<td>20</td>
<td>Version</td>
<td>Yes</td>
<td>The current App Engine release version used in production App Engine: 1.9.34</td>
</tr>
</tbody>
</table>

## Quotas and limits

Your application is affected by the following [logs-related quotas](https://web.archive.org/web/20160425095405/https://cloud.google.com/appengine/docs/quotas#Logs):

-   Logs data retrieved via the Logs API.
-   Log storage, also called *logs retention*.

### Quota for data retrieved

The first 100 megabytes of logs data retrieved per day via the Logs API calls are free. After this amount is exceeded, no further Logs API calls will succeed unless billing is enabled for your app. If billing is enabled for your app, data in excess of 100 megabytes results in charges of $0.12/GB.

### Logs storage

You can control how much log data your application stores by means of its log retention settings in the Google Cloud Platform Console. By default, logs are stored for an application free of charge with the following per-application limits: a maximum of 1 gigabyte for a maximum of up to 90 days. If either limit is exceeded, more recent logs will be shown and older logs will be deleted to stay within the size limit. Logs older than the maximum retention time are also deleted.

If your app has billing enabled, you can pay for higher log size limits by <a href="https://web.archive.org/web/20160425095405/https://console.cloud.google.com/appengine/settings" data-track-metadata-end-goal="manageAppEngineSettings" data-track-metadata-position="body" data-track-name="consoleLink" data-track-type="tasks">specifying the desired maximum log size</a> in gigabytes in the Cloud Platform Console. You can also set the retention time by specifying the desired number of days to keep logs, up to a maximum of 365 days. The cost of this extra log storage is $0.026 per gigabyte utilized per month.

<table>
<thead>
<tr class="header">
<th>Limit</th>
<th>Amount</th>
<th>Cost past free threshold</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Maximum days storage per log</td>
<td>90 days free, 365 days if paid</td>
<td>$0.026 per gigabyte utilized per month</td>
</tr>
<tr class="even">
<td>Maximum total logs storage</td>
<td>1 gigabyte free, unlimited if paid</td>
<td>$0.026 per gigabyte utilized per month</td>
</tr>
</tbody>
</table>
