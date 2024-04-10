# Scheduled Tasks With Cron for Java

  

The App Engine Cron Service allows you to configure regularly scheduled tasks that operate at defined times or regular intervals. These tasks are commonly known as *cron jobs*. These cron jobs are automatically triggered by the App Engine Cron Service. For instance, you might use this to send out a report email on a daily basis, to update some cached data every 10 minutes, or to update some summary information once an hour.

A cron job will invoke a URL, using an HTTP GET request, at a given time of day. An HTTP request invoked by cron is [subject to the same limits as other HTTP requests](https://web.archive.org/web/20160424230251/https://cloud.google.com/appengine/docs/java/modules/#Java_Instance_scaling_and_class), depending on the scaling type of the module.

Free applications can have up to 20 scheduled tasks. Paid applications can have up to 100 scheduled tasks.

**Important:** For a cron task to be considered successful it must return an HTTP status code between 200 and 299 (inclusive).

1.  [About cron.xml](#Java_appengine_web_xml_About_cron_xml)
2.  [The schedule format](#Java_appengine_web_xml_The_schedule_format)
3.  [Cron retries](#retry)
4.  [Securing URLs for cron](#Java_appengine_web_xml_Securing_URLs_for_cron)
5.  [Calling Google Cloud Endpoints](#Java_calling_google_cloud_endpoints)
6.  [Cron and app versions](#Java_appengine_web_xml_Cron_and_app_versions)
7.  [Uploading cron jobs](#Java_appengine_web_xml_Uploading_cron_jobs)
8.  [Cron support in the Cloud Platform Console](#Java_appengine_web_xml_Cron_support_in_the_Admin_Console)
9.  [Cron support in the development server](#Java_appengine_web_xml_Cron_support_in_the_development_server)

## About cron.xml

A `cron.xml` file in the `WEB-INF` directory of your application (alongside `appengine-web.xml`) controls cron for your Java application. The following is an example `cron.xml` file:

```
<?xml version="1.0" encoding="UTF-8"?>
<cronentries>
  <cron>
    <url>/recache</url>
    <description>Repopulate the cache every 2 minutes</description>
    <schedule>every 2 minutes</schedule>
  </cron>
  <cron>
    <url>/weeklyreport</url>
    <description>Mail out a weekly report</description>
    <schedule>every monday 08:30</schedule>
    <timezone>America/New_York</timezone>
  </cron>
  <cron>
    <url>/weeklyreport</url>
    <description>Mail out a weekly report</description>
    <schedule>every monday 08:30</schedule>
    <timezone>America/New_York</timezone>
    <target>version-2</target>
  </cron>
</cronentries>
```

For an XSD describing the format, check the file `docs/cron.xsd` in the SDK.

A `cron.xml` file consists of a number of job definitions. A job definition must have a `<url>` and a `<schedule>`. You can also optionally specify a `<description>`, `<timezone>`, and a `<target>`. The description is visible in the Cloud Platform Console.

The `url` url field is just a URL in your application. If the `url` element contains the special XML characters `&`, `<`, `>`, `'`, or `"`, you should escape them.

The format of the `schedule` field is covered more in [The Schedule Format](#Java_appengine_web_xml_The_schedule_format).

The `timezone` should be the name of a standard [zoneinfo](https://web.archive.org/web/20160424230251/https://en.wikipedia.org/wiki/List_of_zoneinfo_time_zones) time zone name. If you don't specify a timezone, jobs run in UTC (also known as GMT).

The `target` string is prepended to your app's hostname. It is usually the name of a module. The cron job will be routed to the default version of the named module. Note that if the default version of the module changes, the job will run in the new default version.

**Warning:** Be careful if you run a cron job with [traffic splitting](https://web.archive.org/web/20160424230251/https://cloud.google.com/appengine/docs/developers-console/#traffic-splitting) enabled. The request from the cron job is always sent from the same IP address, so if you've specified IP address splitting, the logic will route the request to the same version every time. If you've specified cookie splitting, the request will not be split at all, since there is no cookie accompanying the request.

If there is no module with the name assigned to `target`, the name is assumed to be a version of the default module, and App Engine will attempt to route the job to that version. See [About appengine-web.xml](https://web.archive.org/web/20160424230251/https://cloud.google.com/appengine/docs/java/config/appconfig#Java_appengine_web_xml_About_appengine_web_xml)

## The schedule format

Cron schedules are specified using a simple English-like format.

The following are examples of schedules:

```
every 12 hours
every 5 minutes from 10:00 to 14:00
every day 00:00
every monday 09:00
2nd,third mon,wed,thu of march 17:00
1st monday of sep,oct,nov 17:00
1 of jan,april,july,oct 00:00
```

If you don't need to run a recurring job at a specific time, but instead only need to run it at regular intervals, use the form:

```
every N (hours|mins|minutes) ["from" (time) "to" (time)]
```

The brackets are for illustration only, and quotes indicate a literal.

-   *N* specifies a number.
-   *hours* or *minutes* (you can also use *mins*) specifies the unit of time.
-   *time* specifies a time of day, as HH:MM in 24 hour time.

By default, an interval schedule starts the next interval after the last job has completed. If a *from...to* clause is specified, however, the jobs are scheduled at regular intervals independent of when the last job completed. For example:

```
every 2 hours from 10:00 to 14:00
```

This schedule runs the job three times per day at 10:00, 12:00, and 14:00, regardless of how long it takes to complete. You can use the literal "synchronized" as a synonym for *from 00:00 to 23:59*:

```
every 2 hours synchronized
```

If you want more specific timing, you can specify the schedule as:

```
("every"|ordinal) (days) ["of" (monthspec)] (time)
```

Where:

-   *ordinal* specifies a comma separated list of "1st", "first" and so forth (both forms are ok)
-   *days* specifies a comma separated list of days of the week (for example, "mon", "tuesday", with both short and long forms being accepted); "every day" is equivalent to "every mon,tue,wed,thu,fri,sat,sun"
-   *monthspec* specifies a comma separated list of month names (for example, "jan", "march", "sep"). If omitted, implies every month. You can also say "month" to mean every month, as in "1,8,15,22 of month 09:00".
-   *time* specifies the time of day, as HH:MM in 24 hour time.

**Note:** You cannot specify a schedule that includes both regular intervals and specific timing.

## Cron retries

If a cron job's request handler returns a status code that is not in the range 200–299 (inclusive) App Engine considers the job to have failed. By default, failed jobs are not retried. You can cause failed jobs to be retried by including a `retry-parameters` block in your configuration file.

Here is a sample cron.xml file that contains a single cron job configured to retry up to five times (the default) with a starting backoff of 2.5 seconds that doubles each time.
```
<cronentries>
  <cron>
    <url>/retry</url>
    <description>Retry on jsdk</description>
    <schedule>every 10 minutes</schedule>
    <retry-parameters>
      <min-backoff-seconds>2.5</min-backoff-seconds>
      <max-doublings>5</max-doublings>
    </retry-parameters>
  </cron>
</cronentries>
```

The retry parameters are described in the table below.

<table>
<thead>
<tr class="header">
<th>Setting</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>job‑retry‑limit</code></td>
<td>The maximum number of retry attempts for a failed cron job not to exceed '5'. If specified with <code>job‑age‑limit</code>, App Engine retries the cron job until both limits are reached. When omitted from the parameters, the limit is set to '5' by default.</td>
</tr>
<tr class="even">
<td><code>job‑age‑limit</code></td>
<td>The time limit for retrying a failed cron job, measured from when the cron job was first run. The value is a number followed by a unit of time, where the unit is s for seconds, m for minutes, h for hours, or d for days. For example, the value 5d specifies a limit of five days after the cron job's first execution attempt. If specified with <code>job‑retry‑limit</code>, App Engine retries the cron job until both limits are reached.</td>
</tr>
<tr class="odd">
<td><code>min‑backoff‑seconds</code></td>
<td>The minimum number of seconds to wait before retrying a cron job after it fails.</td>
</tr>
<tr class="even">
<td><code>max‑backoff‑seconds</code></td>
<td>The maximum number of seconds to wait before retrying a cron job after it fails.</td>
</tr>
<tr class="odd">
<td><code>max‑doublings</code></td>
<td>The maximum number of times that the interval between failed cron job retries will be doubled before the increase becomes constant. The constant is: 2**(<code>max‑doublings</code> - 1) * <code>min‑backoff‑seconds</code>.</td>
</tr>
</tbody>
</table>

## Securing URLs for cron

You can prevent users from accessing URLs used by scheduled tasks by restricting access to administrator accounts. Scheduled tasks can access admin-only URLs. You can read about restricting URLs at [Security and Authentication](https://web.archive.org/web/20160424230251/https://cloud.google.com/appengine/docs/java/config/webxml#Security_and_Authentication). An example you would use in `web.xml` to restrict everything starting with `/cron/` to admin-only is:

```
<security-constraint>
    <web-resource-collection>
        <web-resource-name>cron</web-resource-name>
        <url-pattern>/cron/*</url-pattern>
    </web-resource-collection>
    <auth-constraint>
        <role-name>admin</role-name>
    </auth-constraint>
</security-constraint>
```
**Note:** While cron jobs can use URL paths restricted with `<role-name>admin</role-name>`, they *cannot* use URL paths restricted with `<role-name>*</role-name>` because cron scheduled tasks are not run as any user. The `admin` restriction is satisfied by the inclusion of the `X-Appengine-Cron` header described below.

For more on the format of `web.xml`, see the documentation on [the deployment descriptor](https://web.archive.org/web/20160424230251/https://cloud.google.com/appengine/docs/java/config/webxml).

To test a cron job, sign in as an administrator and visit the URL of the handler in your browser.

Requests from the Cron Service will also contain a HTTP header:

```
X-Appengine-Cron: true
```

The `X-Appengine-Cron` header is set internally by Google App Engine. If your request handler finds this header it can trust that the request is a cron request. If the header is present in an external user request to your app, it is stripped. The exception being requests from logged in administrators of the application, who are allowed to set the header for testing purposes.

Google App Engine issues Cron requests from the IP address 0.1.0.1.

## Calling Google Cloud Endpoints

You cannot call a [Google Cloud Endpoint](https://web.archive.org/web/20160424230251/https://cloud.google.com/appengine/features/#Endpoints) from a cron job. Instead, you should issue a request to a target that is served by a handler that's specified in your app's configuration file or in a dispatch file. That handler then calls the appropriate endpoint class and method.

## Cron and app versions

If the `target` parameter has been set for a job, the request is sent to the specified version. Otherwise Cron requests are sent to the default version of the application.

## Uploading cron jobs

You can use `AppCfg` to upload cron jobs. When you upload your application to App Engine using `AppCfg update`, the Cron Service is updated with the contents of `cron.xml`. You can update just the cron configuration without uploading the rest of the application using `AppCfg update_cron`.

To delete all cron jobs, change the cron.xml file to just contain:

```
<?xml version="1.0" encoding="UTF-8"?>
<cronentries/>
```

## Cron support in the Cloud Platform Console

The Cloud Platform Console [Task queues page](https://web.archive.org/web/20160424230251/https://console.cloud.google.com/project/_/appengine/taskqueues) has a tab that shows the tasks that are running cron jobs.

You can also visit the [Logs page](https://web.archive.org/web/20160424230251/https://cloud.google.com/appengine/docs/developers-console/#logs) see when cron jobs were added or removed.

## Cron support in the development server

The development server doesn't automatically run your cron jobs. You can use your local desktop's cron or scheduled tasks interface to trigger the URLs of your jobs with [curl](https://web.archive.org/web/20160424230251/http://curl.haxx.se/) or a similar tool.
