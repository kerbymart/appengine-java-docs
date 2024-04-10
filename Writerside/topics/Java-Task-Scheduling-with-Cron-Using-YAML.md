# Java Task Scheduling with Cron Using YAML

  

The App Engine Cron Service allows you to configure regularly scheduled tasks that operate at defined times or regular intervals. These tasks are commonly known as *cron jobs*. These cron jobs are automatically triggered by the App Engine Cron Service. For instance, you might use this to send out a report email on a daily basis, to update some cached data every 10 minutes, or to update some summary information once an hour.

A cron job will invoke a URL, using an HTTP GET request, at a given time of day. An HTTP request invoked by cron is [subject to the same limits as other HTTP requests](https://web.archive.org/web/20160424230929/https://cloud.google.com/appengine/docs/java/modules/#Java_Instance_scaling_and_class), depending on the scaling type of the module.

Free applications can have up to 20 scheduled tasks. Paid applications can have up to 100 scheduled tasks.

**Important:** For a cron task to be considered successful it must return an HTTP status code between 200 and 299 (inclusive).

1.  [About cron.yaml](#Java_app_yaml_About_cron_yaml)
2.  [The schedule format](#Java_app_yaml_The_schedule_format)
3.  [Cron retries](#retry)
4.  [Securing URLs for cron](#Java_app_yaml_Securing_URLs_for_cron)
5.  [Calling Google Cloud Endpoints](#Java_calling_google_cloud_endpoints)
6.  [Cron and app versions](#Java_app_yaml_Cron_and_app_versions)
7.  [Uploading cron jobs](#Java_app_yaml_Uploading_cron_jobs)
8.  [Cron support in the Cloud Platform Console](#Java_app_yaml_Cron_support_in_the_Admin_Console)
9.  [Cron support in the development server](#Java_app_yaml_Cron_support_in_the_development_server)

## About cron.yaml

A `cron.yaml` file in the `WEB-INF` directory of your application (alongside `app.yaml`) configures scheduled tasks for your Java application. The following is an example `cron.yaml` file:

```
cron:
- description: daily summary job
  url: /tasks/summary
  schedule: every 24 hours
- description: monday morning mailout
  url: /mail/weekly
  schedule: every monday 09:00
  timezone: Australia/NSW
- description: new daily summary job
  url: /tasks/summary
  schedule: every 24 hours
  target: beta
```

The syntax of `cron.yaml` is the YAML format. For more information about this syntax, see [the YAML website](https://web.archive.org/web/20160424230929/http://www.yaml.org/) for more information.

A `cron.yaml` file consists of a number of job definitions. A job definition must have a `url` and a `schedule`. You can also optionally specify a `description`, `timezone`, and a `target`. The description is visible in the Cloud Platform Console and the development server's admin interface.

The `url` field specifies a URL in your application that will be invoked by the Cron Service. See [Securing URLs for Cron](#Java_app_yaml_Securing_URLs_for_cron) for more. The format of the schedule field is covered in [The Schedule Format](#Java_app_yaml_The_schedule_format).

The `timezone` should be the name of a standard [zoneinfo](https://web.archive.org/web/20160424230929/https://en.wikipedia.org/wiki/List_of_zoneinfo_time_zones) time zone name. If you don't specify a timezone, the schedule will be in UTC (also known as GMT).

The `target` string is prepended to your app's hostname. It is usually the name of a module. The cron job will be routed to the default version of the named module. Note that if the default version of the module changes, the job will run in the new default version.

**Warning:** Be careful if you run a cron job with [traffic splitting](https://web.archive.org/web/20160424230929/https://cloud.google.com/appengine/docs/developers-console/#traffic-splitting) enabled. The request from the cron job is always sent from the same IP address, so if you've specified IP address splitting, the logic will route the request to the same version every time. If you've specified cookie splitting, the request will not be split at all, since there is no cookie accompanying the request.

If there is no module with the name assigned to `target`, the name is assumed to be a version of the default module, and App Engine will attempt to route the job to that version.

If you use a [dispatch file](https://web.archive.org/web/20160424230929/https://cloud.google.com/appengine/docs/java/modules/routing#dispatch_file), your job may be re-routed. For example, given the following cron.yaml and dispatch.yaml files, the job will run in module2, even though its target is module1:

```
# cron.yaml
cron:
- description: test dispatch vs target
  url: /tasks/hello_module2
  schedule: every 1 mins
  target: module1

# dispatch.yaml:
dispatch:
- url: '*/tasks/hello_module2'
  module: module2
```

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

If a cron job's request handler returns a status code that is not in the range 200–299 (inclusive) App Engine considers the job to have failed. By default, failed jobs are not retried. You can cause failed jobs to be retried by including a `job_retry_parameters` block in your configuration file.

Here is a sample cron.yaml file that contains a single cron job configured to retry up to five times (the default) with a starting backoff of 2.5 seconds that doubles each time.
```
cron:
- description: retry demo
  url: /retry
  schedule: every 10 mins
- retry_parameters
  min_backoff_seconds: 2.5
  max_doublings: 5
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
<td><code>job_retry_limit</code></td>
<td>The maximum number of retry attempts for a failed cron job not to exceed '5'. If specified with <code>job_age_limit</code>, App Engine retries the cron job until both limits are reached. When omitted from the parameters, the limit is set to '5' by default.</td>
</tr>
<tr class="even">
<td><code>job_age_limit</code></td>
<td>The time limit for retrying a failed cron job, measured from when the cron job was first run. The value is a number followed by a unit of time, where the unit is s for seconds, m for minutes, h for hours, or d for days. For example, the value 5d specifies a limit of five days after the cron job's first execution attempt. If specified with <code>job_retry_limit</code>, App Engine retries the cron job until both limits are reached.</td>
</tr>
<tr class="odd">
<td><code>min_backoff_seconds</code></td>
<td>The minimum number of seconds to wait before retrying a cron job after it fails.</td>
</tr>
<tr class="even">
<td><code>max_backoff_seconds</code></td>
<td>The maximum number of seconds to wait before retrying a cron job after it fails.</td>
</tr>
<tr class="odd">
<td><code>max_doublings</code></td>
<td>The maximum number of times that the interval between failed cron job retries will be doubled before the increase becomes constant. The constant is: 2**(<code>max_doublings</code> - 1) * <code>min_backoff_seconds</code>.</td>
</tr>
</tbody>
</table>

## Securing URLs for cron

A cron handler is just a normal handler defined in `app.yaml`. You can prevent users from accessing URLs used by scheduled tasks by restricting access to administrator accounts. Scheduled tasks can access admin-only URLs. You can restrict a URL by adding `login: admin` to the handler configuration in `app.yaml`.

An example might look like this in `app.yaml`:

```
application: hello-cron
version: 1
runtime: java
api_version: 1

handlers:
- url: /report/weekly
  servlet: mysite.server.CronServlet
  login: admin
```
**Note:** While cron jobs can use URL paths restricted with `login: admin`, they *cannot* use URL paths restricted with `login: required` because cron scheduled tasks are not run as any user. The `admin` restriction is satisfied by the inclusion of the `X-Appengine-Cron` header described below.

To test a cron job, sign in as an administrator and visit the URL of the handler in your browser.

Requests from the Cron Service will also contain a HTTP header:

```
X-Appengine-Cron: true
```

The `X-Appengine-Cron` header is set internally by Google App Engine. If your request handler finds this header it can trust that the request is a cron request. If the header is present in an external user request to your app, it is stripped, except for requests from logged in administrators of the application, who are allowed to set the header for testing purposes.

Google App Engine issues Cron requests from the IP address `0.1.0.1`.

## Calling Google Cloud Endpoints

You cannot call a [Google Cloud Endpoint](https://web.archive.org/web/20160424230929/https://cloud.google.com/appengine/features/#Endpoints) from a cron job. Instead, you should issue a request to a target that is served by a handler that's specified in your app's configuration file or in a dispatch file. That handler then calls the appropriate endpoint class and method.

## Cron and app versions

If the `target` parameter has been set for a job, the request is sent to the specified version. Otherwise Cron requests are sent to the default version of the application.

## Uploading cron jobs

You can use `AppCfg` to upload cron jobs. When you upload your application to App Engine using `AppCfg update`, the Cron Service is updated with the contents of `cron.yaml`. You can update just the cron configuration without uploading the rest of the application using `AppCfg update_cron`.

To delete all cron jobs, change the `cron.yaml` file to just contain:

```
cron:
```

## Cron support in the Cloud Platform Console

The Cloud Platform Console [Task queues page](https://web.archive.org/web/20160424230929/https://console.cloud.google.com/project/_/appengine/taskqueues) has a tab that shows the tasks that are running cron jobs.

You can also visit the [Logs page](https://web.archive.org/web/20160424230929/https://cloud.google.com/appengine/docs/developers-console/#logs) see when cron jobs were added or removed.

## Cron support in the development server

The development server doesn't automatically run your cron jobs. You can use your local desktop's cron or scheduled tasks interface to trigger the URLs of your jobs with [curl](https://web.archive.org/web/20160424230929/http://curl.haxx.se/) or a similar tool.
