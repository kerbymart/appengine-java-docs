# Using the Google Cloud Platform Console for App Engine

  

[Python](https://web.archive.org/web/20160424230219/https://cloud.google.com/appengine/docs/python/console "View this page in the Python runtime") <span style="color: #ddd; padding: 0em .5em;">|</span><span style="color: black; font-weight:bold">Java</span> <span style="color: #ddd; padding: 0em .5em;">|</span>[PHP](https://web.archive.org/web/20160424230219/https://cloud.google.com/appengine/docs/php/console "View this page in the PHP runtime") <span style="color: #ddd; padding: 0em .5em;">|</span>[Go](https://web.archive.org/web/20160424230219/https://cloud.google.com/appengine/docs/go/console "View this page in the Go runtime")

The Google Cloud Platform Console lets you manage your App Engine application along with other Google Cloud Platform resources. Use your [Google account](https://web.archive.org/web/20160424230219/https://support.google.com/accounts/answer/27441?source=gsearch) to access the Cloud Platform Console at [https://console.cloud.google.com/](https://web.archive.org/web/20160424230219/https://console.cloud.google.com/).

Use the Cloud Platform Console to monitor application performance and control various serving and configuration settings, depending on your [user role](#roles).

**Note:** To learn more about the many Google Cloud Platform features you can manage from the Cloud Platform Console, go to the Cloud Platform Console [help center](https://web.archive.org/web/20160424230219/https://support.google.com/cloud#topic=3340599).

## Contents

1.  [Managing a Cloud Platform project](#managing_a_cloud_platform_project)
    1.  [Creating and deleting a project](#create)
    2.  [Setting the server location](#server-location)
    3.  [Enabling billing and setting a spending limit](#billing)
    4.  [Disabling billing](#disabling_billing)
    5.  [Managing billing](#managing_billing)
    6.  [Managing permissions](#permissions)
    7.  [Managing cookies, authentication, and logs retention](#managing_cookies_authentication_and_logs_retention)
    8.  [Using custom domains and SSL](#using_custom_domains_and_ssl)
2.  [Managing an App Engine application](#managing_an_app_engine_application)
    1.  [Viewing snapshots of the current status](#dashboard)
    2.  [Viewing the status of running instances](#instances)
    3.  [Managing Versions](#versions)
    4.  [Viewing logs](#logs)
    5.  [Using security scans](#security)
3.  [Managing application resources](#managing_application_resources)
    1.  [Viewing quota details](#quotas)
    2.  [Managing task queues](#queues)
    3.  [Managing memcache](#memcache)
    4.  [Managing datastore](#datastore)
    5.  [Search](#search)

## Managing a Cloud Platform project

### Creating and deleting a project

In the Google Cloud Platform Console, [**manage all projects**](https://web.archive.org/web/20160424230219/https://console.cloud.google.com/project).

-   To create a project, click **Create Project**, enter a name, and click **Create**.
-   To delete a project, click the trash can icon for the appropriate project. This will delete the project and all the resources used within the project.

For more information, see the Cloud Platform Console help page on [Creating, deleting, and restoring projects](https://web.archive.org/web/20160424230219/https://support.google.com/cloud/answer/6251787).

### Setting the server location

When you create your project, you can specify the [location](https://web.archive.org/web/20160424230219/https://cloud.google.com/docs/geography-and-regions) from which it will be served. In the new project dialog, click on the link to **Show Advanced Options**, and select a location from the pulldown menu:

-   us-central
-   us-east1
-   europe-west

If you select **us-east1** your project will be served from a single region in South Carolina. The **us-central** and **europe-west** locations contain multiple regions in the United States and western Europe, respectively. Projects deployed to either **us-central** or **europe-west** may be served from any one of the regions they contain. If you want to colocate your App Engine instances with other single-region services, such as Google Compute Engine, you should select **us-east1**.

Note the following limitations:

-   You cannot change the location after the project has been created.
-   [Python 2.5 support is deprecated](https://web.archive.org/web/20160424230219/https://cloud.google.com/appengine/docs/python/python25/diff27) and new Python 2.5 apps cannot be created in any location. Apps written in Python 2.7, and all other App Engine languages, can be served from any location.
-   [The flexible environment](https://web.archive.org/web/20160424230219/https://cloud.google.com/appengine/docs/flexible/) is not currently available for projects located in europe-west.

### Enabling billing and setting a spending limit

If your application needs more resources than the free [quotas](https://web.archive.org/web/20160424230219/https://cloud.google.com/appengine/docs/quotas), you can enable billing to pay for the additional usage. If you have a billing account when you create a project, then billing will automatically be enabled. For more information, see [Billing settings](https://web.archive.org/web/20160424230219/https://cloud.google.com/appengine/pricing#billing_settings).

1.  If you don't have a billing account, add one in the Cloud Platform Console.

    1.  Go to the <a href="https://web.archive.org/web/20160424230219/https://console.cloud.google.com/billing" data-track-metadata-end-goal="addBillingAccount" data-track-metadata-position="body" data-track-name="consoleLink" data-track-type="tasks"><strong>Billing accounts</strong></a>.
    2.  Click **New billing account** and follow the on-screen instructions to set up a billing account.

2.  Select a project, and enable billing for your project.

    1.  Go to the [**Project billing settings**](https://web.archive.org/web/20160424230219/https://console.cloud.google.com/project/_/settings).
    2.  Click **Enable billing**. If billing is already enabled, then the billing account for the project is listed.

    After you enable billing, there is no limit to the amount you may be charged until you set a daily spending limit. It's a good idea to specify a spending limit to gain more control over application costs.

3.  Create or change the spending limit.

    1.  Go to the <a href="https://web.archive.org/web/20160424230219/https://console.cloud.google.com/project/_/appengine/settings" data-track-metadata-end-goal="enableBilling" data-track-metadata-position="body" data-track-name="consoleLink" data-track-type="tasks"><strong>Application Settings</strong></a>.
    2.  Click **Edit** and specify a spending limit. Click **Save**.

    The spending limit only applies to [App Engine resources](https://web.archive.org/web/20160424230219/https://cloud.google.com/appengine/pricing#cost_resource) for the selected project:

    -   You may still be charged for other Google Cloud Platform resources.
    -   If you have multiple projects, you may want to set the spending limit for each project.

    When you increase the daily spending limit, the new limit takes effect immediately. However, if you lower the spending limit, the change will take effect immediately only if the current daily usage is below the old limit. Otherwise, the new spending limit takes effect the next day.

    For more information, see [Spending Limits](https://web.archive.org/web/20160424230219/https://cloud.google.com/appengine/pricing#spending_limit).

### Disabling billing

Once you have enabled billing, if you want to stop automatic payments for a project, you must [disable billing for the project](https://web.archive.org/web/20160424230219/https://support.google.com/cloud/answer/6293499#disable-billing). Alternatively, if you also want to release all the resources used in a project, you can [shut down the project](https://web.archive.org/web/20160424230219/https://support.google.com/cloud/answer/6251787#delete-project).

### Managing billing

To manage a billing account, go to <a href="https://web.archive.org/web/20160424230219/https://console.cloud.google.com/billing" data-track-metadata-end-goal="manageBilling" data-track-metadata-position="body" data-track-name="consoleLink" data-track-type="tasks"><strong>Billing accounts</strong></a> and select the account. You can then:

-   View your transaction history for the account and make a payment on the **History** tab.
-   Change your payment method on the **Settings** tab.
-   Add billing administrators on the **Administrators** tab.
-   Set up an alert for when monthly charges exceed a threshold on the **Alerts** tab.

### Managing permissions

To manage permissions for both [members](https://web.archive.org/web/20160424230219/https://developers.google.com/console/help/new/#differentroles) and [service accounts](https://web.archive.org/web/20160424230219/https://developers.google.com/console/help/new/#serviceaccounts), go to the [Permissions page](https://web.archive.org/web/20160424230219/https://console.cloud.google.com/permissions/projectpermissions). You can assign edit or view permissions to project members and service accounts. You can also designate members as project owners.

#### Assigning project member roles

Add project members and assign each member a role in the <a href="https://web.archive.org/web/20160424230219/https://console.cloud.google.com/permissions/projectpermissions" data-track-metadata-end-goal="addProjectUser" data-track-metadata-position="body" data-track-name="consoleLink" data-track-type="tasks">Permissions page</a>. A role determines the level of access to a project and its application and resources. For App Engine applications, the role also controls the permissible actions in the [gcloud](https://web.archive.org/web/20160424230219/https://cloud.google.com/sdk/gcloud/reference/preview/) and [appcfg](https://web.archive.org/web/20160424230219/https://cloud.google.com/appengine/docs/python/tools/uploadinganapp) command line tools that are used to deploy and manage applications.

<table>
<thead>
<tr class="header">
<th>Role</th>
<th>Google Cloud Platform Console permissions</th>
<th>appcfg permissions</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>Viewer</code></td>
<td>View application information</td>
<td>Request logs</td>
</tr>
<tr class="even">
<td><code>Editor</code></td>
<td>View application information and edit application settings</td>
<td>Upload/rollback application code, update indexes/queues/crons</td>
</tr>
<tr class="odd">
<td><code>Owner</code></td>
<td>All viewer and editor privileges, plus invite users, change user roles, and delete an application.</td>
<td>Upload/rollback application code, update indexes/queues/crons</td>
</tr>
</tbody>
</table>

#### Managing service account permissions

If your project uses server-to-server interactions such as those between a web application and Google Cloud Storage, then you may need a service account, which is an account that belongs to your application instead of to an individual end user.

To add a [service account](https://web.archive.org/web/20160424230219/https://developers.google.com/console/help/new/#serviceaccounts) or to edit service account permissions, go to the [Permissions page](https://web.archive.org/web/20160424230219/https://console.cloud.google.com/project/_/permissions).

### Managing cookies, authentication, and logs retention

If you have owner permissions, in the <a href="https://web.archive.org/web/20160424230219/https://console.cloud.google.com/project/_/appengine/settings" data-track-metadata-end-goal="manageAppEngineSettings" data-track-metadata-position="body" data-track-name="consoleLink" data-track-type="tasks">Application Settings</a>, you may set:

-   The Google login cookie expiration interval. Cookies can expire in one day, one week, or two weeks.

-   The Google authentication method. If you do not wish to authenticate against a Google Apps domain, you can switch to authentication using [Google Accounts API](https://web.archive.org/web/20160424230219/https://developers.google.com/accounts/).

-   The logs retention criteria (storage limit and time interval). By default, application logs are stored free of charge for up to 90 days and with a 1 GB maximum size limit. If you enable billing, you can increase the size limit and extend the time limit up to one year. The cost for log storage above the free quota is $0.026 per gigabyte per month.

### Using custom domains and SSL

Specify a custom domain for your App Engine application using the <a href="https://web.archive.org/web/20160424230219/https://console.cloud.google.com/project/_/appengine/settings/domains" data-track-metadata-end-goal="addCustomDomain" data-track-metadata-position="body" data-track-name="consoleLink" data-track-type="tasks">domains</a> page. Optionally, you can also specify an SSL certificate to be used with your custom domain via the [certificates](https://web.archive.org/web/20160424230219/https://console.cloud.google.com/project/_/appengine/settings/certificates) page.

For complete instructions, see [Using Custom Domains and SSL](https://web.archive.org/web/20160424230219/https://cloud.google.com/appengine/docs/using-custom-domains-and-ssl).

## Managing an App Engine application

### Viewing snapshots of the current status

View usage graphs and current status tables for your app in the <a href="https://web.archive.org/web/20160424230219/https://console.cloud.google.com/project/_/appengine" data-track-metadata-end-goal="viewDashboard" data-track-metadata-position="body" data-track-name="consoleLink" data-track-type="tasks">Dashboard</a>.

Change the scope of the information displayed by selecting the [modules](https://web.archive.org/web/20160424230219/https://cloud.google.com/appengine/features/#modules) and versions of your app using drop-down menus at the top of the dashboard. If you have enabled [traffic splitting](#traffic-splitting), the percentage of traffic routed to each version is indicated in parenthesis. Click on the link at the top right to view your app for the scope you've selected.

#### Usage graphs

View specific performance metrics or resources using the drop-down menu at the top left, below the module/version selectors. Select a time period (from 1 hour to 30 days) at the top right, then select one of the graph types:

<table>
<thead>
<tr class="header">
<th>Graph</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Error Details</td>
<td>The error count per-second broken down into client (4xx) and server (5xx) errors. This graph is similar to the error data shown in the <strong>Summary</strong> graph.</td>
</tr>
<tr class="even">
<td>Instances</td>
<td>A count of instances-per second. It breaks instance activity into the number of new instances created and the number of active instances (those that have served requests).</td>
</tr>
<tr class="odd">
<td>Latency</td>
<td>The average number of milliseconds spent serving dynamic requests only. This includes processing time, but not the time it takes to deliver the request to the client.</td>
</tr>
<tr class="even">
<td>Loading Latency</td>
<td>The average number of milliseconds used to respond to the first request to a new instance. This includes the instance loading and initializing time as well as the time required to process the request. It does not include the time it takes to deliver the request to the client.</td>
</tr>
<tr class="odd">
<td>Memory Usage</td>
<td>The total memory usage across all instances, in MB.</td>
</tr>
<tr class="even">
<td>Memcache Operations</td>
<td>The count-per-second of all memcache key operations. Note that each item in a batch request counts as one key operation.</td>
</tr>
<tr class="odd">
<td>Memcache Compute Units</td>
<td>This graph approximates the resource cost of memcache. It is measured in compute units per second, which is computed as a function of the characteristics of a memcache operation (value size, read vs. mutation, etc.).</td>
</tr>
<tr class="even">
<td>Memcache Traffic</td>
<td>Measured in bytes-per-second. It is broken down into memcache bytes sent and received.</td>
</tr>
<tr class="odd">
<td>Requests by Type</td>
<td>Static and dynamic requests per-second.</td>
</tr>
<tr class="even">
<td>Summary</td>
<td>The per-second count of requests and errors. Total Requests includes static, dynamic, and cached requests. Errors are broken down into client (4xx) and server (5xx) errors.</td>
</tr>
<tr class="odd">
<td>Traffic</td>
<td>The number of bytes-per-second received and sent while handling all requests.</td>
</tr>
<tr class="even">
<td>Utilization</td>
<td>The total CPU activity in cycles-per-second.</td>
</tr>
</tbody>
</table>

#### Current status tables

Below the dashboard graph, you can view tables with the current status of the module(s) and version(s) that you've specified at the top of the page:

<table>
<thead>
<tr class="header">
<th>Table</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Instances</td>
<td>Instances are grouped by the App Engine release number. For each release, the table shows the App Engine release that's running the module, the total number of instances, and the average QPS, latency, and memory.</td>
</tr>
<tr class="even">
<td>Billing Status</td>
<td>Displays the current usage and cost of billable resources.</td>
</tr>
<tr class="odd">
<td>Current Load</td>
<td>Application activity is shown by URI. For each URI the table shows requests/minute, total request in the past 24 hours, and the number of runtime mcycles and average latency in the last hour.</td>
</tr>
<tr class="even">
<td>Server Errors</td>
<td>Reports the URIs that have generated the most errors in the past 24 hours. The total number of errors is shown, along with the percentage ratio of errors to total requests for the URI.</td>
</tr>
<tr class="odd">
<td>Client Errors</td>
<td>Reports the URIs that have generated the most errors in the past 24 hours. The total number of errors is shown, along with the percentage ratio of errors to total requests for the URI.</td>
</tr>
</tbody>
</table>

You can sort the tables on most of the columns. Place the mouse to the right of a column title and click on the caret that appears to sort by increasing or decreasing order.

### Viewing the status of running instances

You can <a href="https://web.archive.org/web/20160424230219/https://console.cloud.google.com/project/_/appengine/instances" data-track-metadata-end-goal="viewAppEngineInstances" data-track-metadata-position="body" data-track-name="consoleLink" data-track-type="tasks">view information for every instance</a>.

The top of the page is similar to the [dashboard](#dashboard). Use the drop-down menus to select a specific module and version. You can see the same dashboard graphs on this page.

Below the graph, a table lists every instance in the scope you selected. If the selected module is running in a the [flexible environment](https://web.archive.org/web/20160424230219/https://cloud.google.com/appengine/docs/flexible/), a column indicates if its VM is being managed by Google or the user. The other columns in the table show:

-   The average Queries Per Second and latency (ms) over the last minute
-   The number of requests received in the last minute
-   The number of errors reported in the last minute
-   Current memory usage (MB)
-   The start time for the instance, indicating its age - how it's been running.
-   A link to the logs viewer for the instance
-   The instance's availability type; resident or dynamic.

### Managing Versions

In the Cloud Platform Console, you can <a href="https://web.archive.org/web/20160424230219/https://console.cloud.google.com/project/_/appengine/versions" data-track-metadata-end-goal="manageAppEngineVersions" data-track-metadata-position="body" data-track-name="consoleLink" data-track-type="tasks">manage different versions of your app</a>.

Use the drop-down menu to select a specific module. All of the module's deployed versions will appear in a table below, showing:

-   The version name
-   The percent of traffic routed to each version
-   The version size
-   The version runtime
-   The number of instances of each version
-   The deployment time

If you have owner or edit permissions, you can delete a version, or use [traffic splitting](#traffic-splitting) and [traffic migration](#traffic-migration) to control the way requests are routed to versions of modules.

**Note:** The performance settings for modules are included in the module's [configuration](https://web.archive.org/web/20160424230219/https://cloud.google.com/appengine/docs/java/config/appconfig#scaling_and_instance_types) file. These settings are made at deployment time and cannot be changed from the Cloud Platform Console.

#### Traffic Splitting

Traffic splitting lets you specify a distribution (by percent) of traffic across two or more versions of a module. Traffic splitting is applied to URLs that do not explicitly target a version, like `<your-project>.appspot.com` (which distributes traffic to versions of the default module) or `<your-module>.<your-project>.appspot.com` (distributing traffic to versions of `<your-module>`). This allows you to roll out features for your app slowly over a period of time. It can also be used for <a href="https://web.archive.org/web/20160424230219/https://en.wikipedia.org/wiki/A/B_testing" target="_blank">A/B Testing</a>.

At the bottom of the Versions page is a **Traffic Splitting** section. Click **Edit** and select two or more versions, specifying the traffic percentage assigned to each. The sum of all percentages should be 100%.

To disable traffic splitting, select a single version from the version list and press **Route all traffic**.

When you have specified two or more versions for splitting, you must choose whether to split the traffic by IP address or an HTTP cookie. It's easier to set up an IP address split, but a cookie split is more precise.

##### IP Address Splitting

When the application receives a request, it hashes the IP address to a value between 0–999, and uses that number to route the request. IP address splitting has some significant limitations:

-   Because IP addresses are independently assigned to versions, the resulting traffic split will differ somewhat from what you specify. For example, if you ask for 5% of traffic to be delivered to an alternate version, the actual percent of traffic the the version might be between 3–7%. The more traffic your application receives, the more accurate the split will be.
-   IP addresses are reasonably sticky, but are not permanent. Users connecting from cell phones may have a shifting IP address throughout a single session. Similarly, a user on a laptop may be moving from home to a cafe to work, and hence also shifting through IP addresses. As a result, the user may have an inconsistent experience with your app as their IP address changes.
-   Requests sent to your app from outside of Google's cloud infrastructure will work normally. However, requests sent from your app to another app (or to itself) are not sent to a version because they all originate from the a small number of IP addresses. This applies to any traffic between apps within Google's cloud infrastructure. If you need to send requests between apps, use cookie splitting instead.

##### Cookie Splitting

The application looks in the HTTP request header for a cookie named `GOOGAPPUID` that contains a value between 0–999. If the cookie exists the number is used to route the request. If there is no such cookie the request is routed randomly - and when the response is sent the app adds a `GOOGAPPUID` cookie, with a random value between 0–999. (The cookie is added only when traffic split by cookie is enabled and the response does not already contain a `GOOGAPPUID` cookie.)

Splitting traffic using cookies makes it easier to accurately assign of users to versions, which allows more precision in traffic routing (as small as 0.1%). Cookie splitting also has some limitations:

-   If you are writing a mobile app or running a desktop client, it needs to manage the `GOOGAPPUID` cookies. When your app server sends back a response with a `Set-Cookie` header, you must store the cookie and include it with each subsequent request. (Browser-based apps already manage cookies in this way automatically.)

-   Splitting internal requests requires extra work. Requests sent server-side from your app to another app (or to itself) can be sent to a version, but doing so requires that you forward the user's cookie with the request. Note that internal requests that don't originate from the user are not recommended for versions.

##### Caching and traffic splitting

Caching issues can exist for any App Engine application, especially when deploying a new version. Traffic splitting often makes subtle caching problems more apparent.

For example, assume you are splitting traffic between two versions, A and B, and some external cacheable resource (like a css file) changed between versions. Now assume the client makes a request, and the response contains an external reference to the cached file. The local HTTP cache will retrieve the file if it's in the cache - no matter which version of the file is cached and which version of the application served up the response. The cached resource could be incompatible with the data in the response.

Avoid caching problems for dynamic resources by setting the [Cache-Control](https://web.archive.org/web/20160424230219/http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.9) and [Expires](https://web.archive.org/web/20160424230219/http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.21) headers. These headers tell proxies that the resource is dynamic. It is best to set both headers, since not all proxy servers support the HTTP/1.1 `Cache-Control` header properly. If you want more information on caching in general, try these web pages:

-   [Header Fields in the HTTP/1.1 RFC](https://web.archive.org/web/20160424230219/http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html)
-   [Document Caching in the Browser Security Handbook](https://web.archive.org/web/20160424230219/http://code.google.com/p/browsersec/wiki/Part2#Document_caching)

For cacheable static resources that vary between versions, you can change the resource's URL between versions. If each version of a static resource is serving from a different URL, the versions can happily co-exist in proxy servers and browser caches.

If the app sets the [Vary: Cookie](https://web.archive.org/web/20160424230219/https://en.wikipedia.org/wiki/HTTP_Cookie) header, the uniqueness of a resource is computed by combining the cookies and the URL for the request. This approach increases the burden on cache servers. There are 1000 possible values of `GOOGAPPUID`, and hence 1000 possible entries for each URL for your app. Depending on the load on the proxies between your users and your app, this may decrease the cache hit rate. Also note that for the 24 hours after adding a new batch of users to a version, they may still see cached resources. However, using `Vary: Cookie` can make it easier to rename static resources that are changing between versions.

The `Vary: Cookie` technique doesn't work in all circumstances. In general, if your app is using cookies for other purposes, you must consider how this affects the burden on proxy servers. If `codeninja` had its own cookie that had 100 possible values, then the space of all possible cache entries becomes a very big number (100 \* 1000 = 100,000). In the worst case, there is a unique cookie for every user. Two common examples of this are Google Analytics (`__utma`) and SiteCatalyst (`s_vi`). In these cases, every user gets a unique copy, which severely degrades cache performance and may also increase the billable instance hours consumed by your app.

#### Traffic migration

When 100% of traffic is routed to one version, you can use traffic migration to re-route requests to some other version. Select an available version in the <a href="https://web.archive.org/web/20160424230219/https://console.cloud.google.com/appengine/versions" id="consoleLink" data-track-="" data-track-metadata-end-="" data-goal="manageAppEngineVersions" data-track-metadata-position="body" data-track-type="tasks">versions list</a> and press **Migrate Traffic**. Migration takes a short amount of time (possibly a few minutes), the exact interval depends on how much traffic your app is receiving and how many instances are running. Once the migration is complete, the new version receives 100% of the traffic.

**Note:** Traffic migration is only available between versions running in the sandbox enviroment. You cannot migrate to or from a version running in the flexible environment.

##### Warmup requests

When using traffic migration you must enable *warmup requests* on the target version (the version you are migrating *to*).

When a request from a user requires the creation of a new instance, the instance receives a *loading request* first, in order to initialize and load the application code. This can increase latency when handling the first user request. Warmup requests are sent to new instances *before* they receive user requests, which improves response time.

Warmup requests are already enabled by default in Java modules. For Python, Go, and PHP modules, you need to enable warmup requests by including this line in your `app.yaml` file:

```
inbound_services:
- warmup
```

For more information, read about [warmup requests](https://web.archive.org/web/20160424230219/https://cloud.google.com/appengine/docs/java/config/appconfig#Java_Warmup%20_requests).

### Viewing logs

<a href="https://web.archive.org/web/20160424230219/https://console.cloud.google.com/project/_/logs" data-track-metadata-end-goal="viewLogs" data-track-metadata-position="body" data-track-name="consoleLink" data-track-type="tasks">View the logs for all of a project's services, including App Engine</a>. See [How to read a log](https://web.archive.org/web/20160424230219/https://cloud.google.com/appengine/docs/java/logs/#Java_how_to_read_a_log) for an explanation of the log fields.

### Using security scans

<a href="https://web.archive.org/web/20160424230219/https://console.cloud.google.com/project/_/appengine/securityscan" data-track-metadata-end-goal="securityScans" data-track-metadata-position="body" data-track-name="consoleLink" data-track-type="tasks">Identify security vulnerabilities</a> in your Google App Engine web applications.

The Google Cloud Security Scanner crawls your application, following all links within the scope of your starting URLs, and attempts to exercise as many user inputs and event handlers as possible to discover vulnerabilities.

In order to use the security scanner, you must be an owner of the project. For more information, read the security scanner [instructions](https://web.archive.org/web/20160424230219/https://cloud.google.com/tools/security-scanner).

## Managing application resources

### Viewing quota details

<a href="https://web.archive.org/web/20160424230219/https://console.cloud.google.com/project/_/appengine/quotadetails" data-track-metadata-end-goal="viewQuota" data-track-metadata-position="body" data-track-name="consoleLink" data-track-type="tasks">See the daily usage and quota consumption for your project</a> in the Cloud Platform Console.

Resources are grouped by API. If some project resource exceeds 50% of its quota halfway through the day, it may exceed the quota before the day is over. For more information about quotas, see the App Engine [quota page](https://web.archive.org/web/20160424230219/https://cloud.google.com/appengine/docs/quotas?hl=en_US) and [Why is My App Over Quota?](https://web.archive.org/web/20160424230219/https://cloud.google.com/appengine/kb/general?hl=en_US#quota)

You can also use the [billing export](https://web.archive.org/web/20160424230219/https://support.google.com/cloud/answer/4524336?hl=en&ref_topic=2991962) feature to write daily Google Cloud Platform usage and cost estimates to a CSV or JSON file that's stored in a Google Cloud Storage bucket you specify.

### Managing task queues

Use [task queues](https://web.archive.org/web/20160424230219/https://cloud.google.com/appengine/features/#taskqueue) so that a module that handles requests can pass off work to a background task that executes asynchronously.

In the Cloud Platform Console, you can <a href="https://web.archive.org/web/20160424230219/https://console.cloud.google.com/project/_/appengine/taskqueues" data-track-metadata-end-goal="manageTaskQueues" data-track-metadata-position="body" data-track-name="consoleLink" data-track-type="tasks">see scheduled tasks, delete tasks, and manage a queue</a>. This page displays information about both types of task queues, pull queues and push queues, and also tasks that belong to [cron jobs](https://web.archive.org/web/20160424230219/https://cloud.google.com/appengine/features/#cron). Select the type of task using the menu bar at the top of the page. When you are viewing push and pull queue reports, a **quotas** link appears at the upper right that hides and shows your app's use of [task queue quotas](https://web.archive.org/web/20160424230219/https://cloud.google.com/appengine/docs/quotas#Task_Queue).

A table lists all of your application's queues, filtered by the type you selected. Click on a queue name to show the tasks scheduled to run that queue. Click on a task name to see detailed information about the task's payload, headers, and previous run (if any).

You can manually delete individual tasks or purge every task from a queue. This is useful if a task cannot be completed successfully and is stuck waiting to be retried. You can also pause and resume a queue.

### Managing memcache

Use a distributed in-memory [data cache](https://web.archive.org/web/20160424230219/https://cloud.google.com/appengine/features/#memcache) to improve performance by storing data, such as the results of common datastore queries. Memcache contains key/value pairs, and the actual pairs in memory at any time change as items are written and retrieved from the cache.

In the Cloud Platform Console, [view the top keys, manage keys, and monitor memcache](https://web.archive.org/web/20160424230219/https://console.cloud.google.com/project/_/appengine/memcache). The top of the page displays key information about the state of your project's memcache:

-   Memcache service level (shared or dedicated memcache). The shared level is free, and is provided on a "best effort" basis with no space guarantees. The dedicated class assigns resources exclusively to your application and is billed by the gigabyte-hour of cache space (it requires billing to be enabled). If you are an owner of the project, you can switch between the two service levels.

-   Hit ratio (as a percentage) and the raw number of memcache hits and misses.

-   Number of items in the cache.

-   The age of the oldest cached item. Note that the age of an item is reset every time it is used, either read or written.

-   Total cache size.

-   Top keys by MCU (for dedicated memcache). See [Viewing top keys](#viewing_top_keys) for more information.

#### Viewing top keys

For dedicated memcache, the memcache page displays a list of the 20 top keys by MCU over the past hour, which can be useful for identifying problematic "hot keys". The list is created by sampling API calls; only the most frequently accessed keys are tracked. Although the viewer displays 20 keys, more may have been tracked. The list gives each key's relative operation count as a percentage of all memcache traffic to the shard that stores that key (note that each key is stored in at most one shard). If an application is a heavy user of memcache and some keys are used very often, the display may include warning indicators.

For tips on distributing load across the keyspace, refer to the article [Best Practices for App Engine Memcache](https://web.archive.org/web/20160424230219/https://cloud.google.com/appengine/articles/best-practices-for-app-engine-memcache#distribute_load_across_the_keyspace).

#### Managing keys

You can add a new key to the cache and retrieve an existing key.

#### Monitoring memcache

Dedicated memcache is rated in operations per second per GB, where an operation is defined as an individual cache item access, e.g. a `get`, `set`, `delete`, etc. The operation rate varies by item size approximately according to the following table. Exceeding these ratings may result in increased API latency or errors.

The following table provides the maximum number of sustained, exclusive `get-hit` or `set` operations per GB of cache. (Note that a `get-hit` operation is a `get` call that finds that there is a value stored with the specified key, and returns that value.)

<table data-border="0">
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<tbody>
<tr class="odd">
<td><table width="45%">
<thead>
<tr class="header">
<th>Item size (KB)</th>
<th>Maximum <code>get-hit</code> ops/s</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>≤1</td>
<td>10,000</td>
</tr>
<tr class="even">
<td>100</td>
<td>2,000</td>
</tr>
<tr class="odd">
<td>512</td>
<td>500</td>
</tr>
</tbody>
</table></td>
<td><table width="45%">
<thead>
<tr class="header">
<th>Item size (KB)</th>
<th>Maximum <code>set</code> ops/s</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>≤1</td>
<td>5,000</td>
</tr>
<tr class="even">
<td>100</td>
<td>1,000</td>
</tr>
<tr class="odd">
<td>512</td>
<td>250</td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

An app configured for multiple GB of cache can in theory achieve an aggregate operation rate computed as the number of GB multiplied by the per-GB rate. For example, an app configured for 5GB of cache could reach 50,000 memcache operations/sec on 1KB items. Achieving this level requires a good distribution of load across the memcache keyspace, as described in [Best Practices for App Engine Memcache](https://web.archive.org/web/20160424230219/https://cloud.google.com/appengine/articles/best-practices-for-app-engine-memcache#distribute-load).

For each IO pattern, the limits listed above are for reads *or* writes. For simultaneous reads *and* writes, the limits are on a sliding scale. The more reads being performed, the fewer writes can be performed, and vice versa. Each of the following are example IOPs limits for simultaneous reads and writes of 1KB values per 1GB of cache:

<table>
<thead>
<tr class="header">
<th>Read IOPs</th>
<th>Write IOPs</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>10000</td>
<td>0</td>
</tr>
<tr class="even">
<td>8000</td>
<td>1000</td>
</tr>
<tr class="odd">
<td>5000</td>
<td>2500</td>
</tr>
<tr class="even">
<td>1000</td>
<td>4500</td>
</tr>
<tr class="odd">
<td>0</td>
<td>5000</td>
</tr>
</tbody>
</table>

##### Memcache compute units (MCU)

**Note:** The way that Memcache Compute Units (MCU) are computed is subject to change.

Memcache throughput can vary depending on the item size and operation. To help developers roughly associate a cost with operations and estimate the traffic capacity that they can expect from dedicated memcache, we define a unit called Memcache Compute Unit (MCU). Developers can expect 10,000 MCU per second per GB of dedicated memcache. The Google Cloud Platform Console shows how much MCU your app is currently using.

Note that MCU is a rough statistical estimation and also it's not a linear unit. Each cache operation that reads or writes a value has a corresponding MCU cost that depends on the size of the value. The MCU for a `set` depends on the value size: it is 2 times the cost of a successful `get-hit` operation.

<table>
<thead>
<tr class="header">
<th>Value item size (KB)</th>
<th>MCU cost for <code>get-hit</code></th>
<th>MCU cost for <code>set</code></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>≤1</td>
<td>1.0</td>
<td>2.0</td>
</tr>
<tr class="even">
<td>2</td>
<td>1.3</td>
<td>2.6</td>
</tr>
<tr class="odd">
<td>10</td>
<td>1.7</td>
<td>3.4</td>
</tr>
<tr class="even">
<td>100</td>
<td>5.0</td>
<td>10.0</td>
</tr>
<tr class="odd">
<td>512</td>
<td>20.0</td>
<td>40.0</td>
</tr>
<tr class="even">
<td>1024</td>
<td>50.0</td>
<td>100.0</td>
</tr>
</tbody>
</table>

Operations that do not read or write a value have a fixed MCU cost:

<table>
<thead>
<tr class="header">
<th>Operation</th>
<th>MCU</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>get-miss</code></td>
<td>1.0</td>
</tr>
<tr class="even">
<td><code>delete</code></td>
<td>2.0</td>
</tr>
<tr class="odd">
<td><code>increment</code></td>
<td>2.0</td>
</tr>
<tr class="even">
<td><code>flush</code></td>
<td>100.0</td>
</tr>
<tr class="odd">
<td><code>stats</code></td>
<td>100.0</td>
</tr>
</tbody>
</table>

Note that a `get-miss` operation is a `get` that finds that there is no value stored with the specified key.

### Managing datastore

Use a schemaless, scalable [datastore](https://web.archive.org/web/20160424230219/https://cloud.google.com/appengine/features/#datastore) for structured data objects. <a href="https://web.archive.org/web/20160424230219/https://console.cloud.google.com/project/_/datastore/stats" data-track-metadata-end-goal="manageDatastore" data-track-metadata-position="body" data-track-name="consoleLink" data-track-type="tasks">Manage your application's datastore</a> in the Cloud Platform Console.

#### Datastore dashboard

View data for the entities in your application's Datastore, as well as statistics for the built-in and composite indexes in the <a href="https://web.archive.org/web/20160424230219/https://console.cloud.google.com/project/_/datastore/stats" data-track-metadata-end-goal="viewEntities" data-track-metadata-position="body" data-track-name="consoleLink" data-track-type="tasks">Cloud Datastore Dashboard</a>. The Statistics page displays data in various ways:

-   A pie chart that shows datastore space used by each property type (string, double, blob, etc.).

-   A pie chart showing datastore space by entity kind.

-   A table with the total space used by each property type. The "Metadata" property type represents space consumed by storing properties inside an entry that is not used by the properties directly. The "Datastore Stats" entity, if any, shows the space consumed by the statistics data itself in your datastore.

-   A table showing total size, average size, entry count, and the size of all entities, and the built-in and composite indexes.

By default, the pie charts display statistics for all entities. To restrict the pie charts to a particular entity kind, use the drop-down menu.

The statistics data is stored in your app's datastore. To make sure there's room for your app's data, Google App Engine will only store statistics if they consume less than 10 megabytes if namespaces are not used, or 20 megabytes if namespaces are used. If your app's stats go over the limit, kind-specific stats are not emitted. If subsequently these "non-kind" stats exceed the limits, then no stats are updated until stats storage drops below the size limitation. (Any previously reported statistics will remain.) You can determine whether this is happening by looking at the timestamps on stat records. If the timestamp gets older than 2 days, and stats in other applications are being updated regularly then this can indicate that you have run into stat storage size limits for your app.

The space consumed by the statistics data increases in proportion to the number of different entity kinds and property types used by your app. The more different entities and properties used by your app, the more likely you are to reach the stat storage limit. Also, if you use namespaces, remember that each namespace contains a complete copy of the stats for that namespace.

**Note:** Datastore operations generated by Datastore Statistics count against your application [quota](https://web.archive.org/web/20160424230219/https://cloud.google.com/appengine/docs/quotas#Datastore).

#### Indexes

In the Cloud Platform Console, you can <a href="https://web.archive.org/web/20160424230219/https://console.cloud.google.com/project/_/appengine/indexes" data-track-metadata-end-goal="viewIndexes" data-track-metadata-position="body" data-track-name="consoleLink" data-track-type="tasks">create a new entity and view a table of all indexes</a>.

#### Query

In the Cloud Platform Console, <a href="https://web.archive.org/web/20160424230219/https://console.cloud.google.com/project/_/appengine/query" data-track-metadata-end-goal="queryDatastore" data-track-metadata-position="body" data-track-name="consoleLink" data-track-type="tasks">select an entity kind and construct a query by applying filters</a>. You can also create a new entity in this page.

#### Backup/restore, copy, delete

To perform bulk operations on the entities in your datastore, go to the [Datastore Admin page](https://web.archive.org/web/20160424230219/https://console.cloud.google.com/datastore/settings).

For more information, see [Managing Datastore from the Google Cloud Platform Console](https://web.archive.org/web/20160424230219/https://cloud.google.com/appengine/docs/java/console/datastoreadmin).

### Search

App Engine provides a [Search](https://web.archive.org/web/20160424230219/https://cloud.google.com/appengine/features/#search) service that stores structured documents in one or more indexes. A document contains data-typed fields.

If your app uses the Search service, you can <a href="https://web.archive.org/web/20160424230219/https://console.cloud.google.com//project/_/appengine/search" data-track-metadata-end-goal="searchDatastore" data-track-metadata-position="body" data-track-name="consoleLink" data-track-type="tasks">search an index for documents with fields that match queries</a>. You can also view all the indexes in a project and the documents contained in each index.
