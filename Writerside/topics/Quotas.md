# Quotas

An App Engine application can consume resources up to certain quotas. With these quotas, App Engine ensures that your application won't exceed your spending limit, and that other applications running on App Engine won't impact your application's performance.

## Contents

1.  [Billable limits and safety limits](#Safety_Quotas_and_Billable_Quotas)
2.  [How resources are replenished](#How_Resources_are_Replenished)
3.  [When a resource is depleted](#When_a_Resource_is_Depleted)
4.  [Resources](#Resources)
    -   [Blobstore](#Blobstore)
    -   [Channel](#Channel)
    -   [Code & static data](#Code)
    -   [Datastore](#Datastore)
    -   [Default Google Cloud Storage Bucket](#Default_Gcs_Bucket)
    -   [Deployments](#Deployments)
    -   [Instance hours](#Instances)
    -   [Logs](#Logs)
    -   [Mail](#Mail)
    -   [Requests](#Requests)
    -   [Search](#search)
    -   [Sockets](#Sockets)
    -   [Task queue](#Task_Queue)
    -   [URL fetch](#UrlFetch)
    -   [XMPP](#XMPP)

## Billable limits and safety limits

App Engine has three kinds of quotas or limits:

-   **Free Quotas:** Every application gets an amount of each resource for free. Free quotas can only be exceeded by paid applications, up to the application's spending limit or the safety limit, whichever applies first.
-   **Spending Limits:** Spending limits are set for paid apps and cannot be exceeded. If you are the [project owner](https://web.archive.org/web/20160424230405/https://cloud.google.com/appengine/docs/python/console/#permissions) and the [billing administrator](https://web.archive.org/web/20160424230405/https://support.google.com/cloud/answer/6293536), you can set the spending limit to manage application costs in the Google Cloud Platform Console in the [App Engine Settings](https://web.archive.org/web/20160424230405/https://console.cloud.google.com//project/_/appengine/settings).
-   **Safety Limits:** Safety limits are set by Google to protect the integrity of the App Engine system. These quotas ensure that no single app can over-consume resources to the detriment of other apps. If you go above these limits you'll get an error whether you are paid or free.

### Spending limits

If you are a [project owner](https://web.archive.org/web/20160424230405/https://cloud.google.com/appengine/docs/python/console/#permissions) and a [billing administrator](https://web.archive.org/web/20160424230405/https://support.google.com/cloud/answer/6293536), you can [enable billing](https://web.archive.org/web/20160424230405/https://cloud.google.com/appengine/docs/developers-console/#billing) for a project so that it can use additional resources beyond the free quotas. You will be charged for the resources your application uses above the free quota thresholds.

After you enable billing for your application, you may want to [set a spending limit](https://web.archive.org/web/20160424230405/https://cloud.google.com/appengine/docs/python/console/#billing) so there is a limit to the amount you can be charged per day. By default, the spending limit is unlimited. It's a good idea to specify a spending limit to gain more control over application costs.

When you enable billing for your application, the application's safety limits increase. See the [Resources](#Resources) section for details.

### Safety limits

Safety limits include *daily quotas* and *per-minute quotas*:

-   **Daily quotas** are refreshed daily at midnight Pacific time. Paid applications can exceed this free quota until their spending limit is exhausted.
-   **Per-minute quotas** protect the application from consuming all of its resources in very short periods of time, and prevent other applications from monopolizing a given resource. If your application consumes a resource too quickly and depletes one of the per-minute quotas, the word "Limited" appears next to the appropriate quota on the **Quota Details** screen in the Cloud Platform Console. Requests for resources that have hit their per-minute maximum will be denied.

See [When a Resource is Depleted](#When_a_Resource_is_Depleted) for details about what happens when a quota is exceeded and how to handle quota overage conditions.

**Tip:** For paid apps, the maximum per-minute quotas accommodate high traffic levels, enough to handle a spike in traffic from your site getting mentioned in news stories. If you believe a particular quota does not meet this requirement, please [create a feature request issue in the issue tracker](https://web.archive.org/web/20160424230405/http://code.google.com/p/googleappengine/). Please note that filing a feature request will not assure an actual quota bump for a particular app, but it will help us understand which quota is potentially too low for general use cases.  
  
If you're expecting extremely high traffic levels, or for some reason your app requires particularly high quotas (e.g. because of significant product launch or large load tests), we recommend you sign up for [Silver, Gold or Platinum support](https://web.archive.org/web/20160424230405/https://cloud.google.com/support/signup).

## How resources are replenished

App Engine tracks your application's resource usage against system quotas. App Engine resets all resource measurements at the beginning of each calendar day (except for Stored Data, which always represents the amount of datastore storage in use). When free applications reach their quota for a resource, they cannot use that resource until the quota is replenished. Paid apps can exceed the free quota until their spending limit is exhausted.

Daily quotas are replenished daily at midnight Pacific time. Per-minute quotas are refreshed every 60 seconds.

## When a resource is depleted

When an application consumes all of an allocated resource, the resource becomes unavailable until the quota is replenished. This may mean that your application will not work until the quota is replenished.

For resources that are required to initiate a request, when the resource is depleted, App Engine by default returns an HTTP 403 or 503 error code for the request instead of calling a request handler. The following resources have this behavior:

-   Bandwidth, incoming and outgoing
-   Instance hours

**Tip:** You can configure your application to serve a custom error page when your application exceeds a quota. For details, see Custom Error Responses documentation for [Python](https://web.archive.org/web/20160424230405/https://cloud.google.com/appengine/docs/python/config/appconfig#Python_app_yaml_Custom_error_responses), [Java](https://web.archive.org/web/20160424230405/https://cloud.google.com/appengine/docs/java/config/appconfig#Java_appengine_web_xml_Custom_error_responses), and [Go](https://web.archive.org/web/20160424230405/https://cloud.google.com/appengine/docs/go/config/appconfig#Go_app_yaml_Custom_error_responses).

For all other resources, when the resource is depleted, an attempt in the application to consume the resource results in an exception. This exception can be caught by the application and handled, such as by displaying a friendly error message to the user. In the Python API, this exception is `apiproxy_errors.OverQuotaError`. In the Java API, this exception is `com.google.apphosting.api.ApiProxy.OverQuotaException`. In the Go API, the `appengine.IsOverQuota` function reports whether an error represents an API call failure due to insufficient available quota.

The following example illustrates how to catch the `OverQuotaError`, which may be raised by the `SendMessage()` method if an email-related quota has been exceeded:

```
try:
  mail.SendMessage(to='test@example.com',
                   from='admin@example.com',
                   subject='Test Email',
                   body='Testing')
except apiproxy_errors.OverQuotaError, message:
  # Log the error.
  logging.error(message)
  # Display an informative message to the user.
  self.response.out.write('The email could not be sent. '
                          'Please try again later.')
```

**Is your app exceeding the default limits?** If you need a higher mail quota, you can [use SendGrid to send email](https://web.archive.org/web/20160424230405/https://cloud.google.com/appengine/docs/python/mail/sendgrid). For any other quota increase, if you have [Silver, Gold, or Platinum support package](https://web.archive.org/web/20160424230405/https://cloud.google.com/support/), you can [contact your support representative](https://web.archive.org/web/20160424230405/https://support.google.com/googleforwork/answer/142244?rd=1#cloud) to request higher throughput limits. Otherwise, you can [file a feature request](https://web.archive.org/web/20160424230405/http://code.google.com/p/googleappengine/).

## Resources

An application may use the following resources, subject to quotas. Resources measured against billable limits are indicated with "(billable)." Resource amounts represent an allocation over a 24 hour period.

The cost of additional resources is listed on the [Pricing](https://web.archive.org/web/20160424230405/https://cloud.google.com/appengine/docs/pricing) page.

### Default Google Cloud Storage Bucket

Applications can use a default Cloud Storage bucket, which has free quota and doesn't require billing to be enabled for the app. You create this free default bucket in the Google Cloud Platform Console [App Engine settings](https://web.archive.org/web/20160424230405/https://console.cloud.google.com/appengine/settings) page for your project.

The following quotas apply specifically to use of the default bucket.

Google Cloud Storage default bucket Stored Data  
The total amount of data stored in the default bucket. Available for both paid and free apps.

<table width="75%">
<thead>
<tr class="header">
<th width="30%">Resource</th>
<th width="35%">Free Default Limit</th>
<th width="35%">Billing Enabled Default Limit</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Default Google Cloud Storage Bucket Stored Data</td>
<td>5 GB</td>
<td>First 5 GB free; no maximum</td>
</tr>
</tbody>
</table>

### Blobstore

The following quotas apply specifically to use of the blobstore.

Blobstore Stored Data  
The total amount of data stored in the blobstore. Available for both paid and free apps.

<table width="75%">
<thead>
<tr class="header">
<th width="30%">Resource</th>
<th width="35%">Free Default Limit</th>
<th width="35%">Billing Enabled Default Limit</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Blobstore Stored Data</td>
<td>5 GB</td>
<td>First 5 GB free; no maximum</td>
</tr>
</tbody>
</table>

### Channel

Channel API Calls  
The total number of times the application accessed the Channel service.

Channels Created  
The number of channels created by the application.

Channel Hours Requested  
The number of hours of channel connect time requested by the application.

Channel Data Sent  
The amount of data sent over the Channel service. This also counts toward the Outgoing Bandwidth quota.

<table width="75%">
<thead>
<tr class="header">
<th width="30%">Resource</th>
<th colspan="2" width="35%">Free Default Limit</th>
<th colspan="2" width="35%">Billing Enabled Default Limit</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<th width="17.5%">Daily Limit</th>
<th width="17.5%">Maximum Rate</th>
<th width="17.5%">Daily Limit</th>
<th width="17.5%">Maximum Rate</th>
<th></th>
</tr>
&#10;<tr class="odd">
<td width="30%">Channel API Calls</td>
<td width="17.5%">657,000 calls</td>
<td width="17.5%">3,000 calls/minute</td>
<td width="17.5%">91,995,495 calls</td>
<td width="17.5%">32,000 calls/minute</td>
</tr>
<tr class="even">
<td>Channels Created</td>
<td>100 channels</td>
<td>6 creations/minute</td>
<td>Based on your spending limit</td>
<td>60 creations/minute</td>
</tr>
<tr class="odd">
<td>Channel Hours Requested</td>
<td>200 hours</td>
<td>12 hours requested/minute</td>
<td>Based on your spending limit</td>
<td>180 hours requested/minute</td>
</tr>
<tr class="even">
<td>Channel Data Sent</td>
<td>Up to the Outgoing Bandwidth quota</td>
<td>22 MB/minute</td>
<td>1 TB</td>
<td>740 MB/minute</td>
</tr>
</tbody>
</table>

### Code and static data storage

Static Data  
No single static data file may be larger than 32MB.

Total Storage  
The storage quota applies to the total amount of code and static data stored by all versions of your app. The total stored size of code and static files is listed in the Main Dashboard table. Individual sizes are displayed on the Versions and Backends screens respectively. Free apps may only upload up to 1 GB of code and static data. Paid apps may upload more, but will be charged $ 0.026 per GB per month for any code and static data storage that exceeds 1 GB.

<table width="75%">
<thead>
<tr class="header">
<th width="55%">Resource</th>
<th width="45%">Cost</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Code &amp; Static Data Storage - First 1 GB</td>
<td>Free</td>
</tr>
<tr class="even">
<td>Code &amp; Static Data Storage - Exceeding 1 GB</td>
<td>$ 0.026 per GB per month</td>
</tr>
</tbody>
</table>

### Datastore

The **Stored Data (billable)** quota refers to all data stored for the application in the Datastore, the Task Queue, and the Blobstore. Other quotas in the "Datastore" section of the **Quota Details** screen in the Google Cloud Platform Console refer specifically to the Datastore service.

Stored Data (billable)  
The total amount of data stored in datastore entities and corresponding indexes, in the task queue, and in the Blobstore.

It's important to note that data stored in the datastore may incur significant overhead. This overhead depends on the number and types of associated properties, and includes space used by built-in and custom indexes. Each entity stored in the datastore requires the following metadata:

-   The entity key, including the kind, the ID or key name, and the keys of the entity's ancestors.
-   The name and value of each property. Since the datastore is schemaless, the name of each property must be stored with the property value for any given entity.
-   Any built-in and custom index rows that refer to this entity. Each row contains the entity kind, any number of property values depending on the index definition, and the entity key.

See [How Entities and Indexes are Stored](https://web.archive.org/web/20160424230405/https://cloud.google.com/appengine/articles/storage_breakdown) for a complete breakdown of the metadata required to store entities and indexes at the Bigtable level.

Number of Indexes  
The number of Datastore indexes that exist for the application. This includes indexes that were created in the past and no longer appear in the application's configuration but have not been deleted using AppCfg's `vacuum_indexes` command.

Write Operations  
The total number of Datastore write operations.

Read Operations  
The total number of Datastore read operations.

Small Operations  
The total number of Datastore small operations. Small operations include calls to allocate Datastore IDs or keys-only queries.

<table width="75%">
<thead>
<tr class="header">
<th width="30%">Resource</th>
<th width="35%">Free Default Daily Limit</th>
<th width="35%">Billing Enabled Default Limit</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Stored Data (<a href="#Billable_Resource_Quotas">billable</a>)</td>
<td>1 GB <sup>*</sup></td>
<td>1 GB free; no maximum</td>
</tr>
<tr class="even">
<td>Number of Indexes</td>
<td>200 <sup>*</sup></td>
<td>200</td>
</tr>
<tr class="odd">
<td>Write Operations</td>
<td>50,000</td>
<td>Unlimited</td>
</tr>
<tr class="even">
<td>Read Operations</td>
<td>50,000</td>
<td>Unlimited</td>
</tr>
<tr class="odd">
<td>Small Operations</td>
<td>50,000</td>
<td>Unlimited</td>
</tr>
</tbody>
</table>

<sup>\*</sup>Not a daily limit but a total limit. Auto-generated single property ascending indexes are not counted towards this limit.

**Note:** Datastore operations generated by the [Datastore Admin](https://web.archive.org/web/20160424230405/https://cloud.google.com/appengine/docs/python/console/datastoreadmin) and [Datastore Viewer](https://web.archive.org/web/20160424230405/https://cloud.google.com/appengine/docs/adminconsole/datastorestats) count against your application quota.

### Deployments

Deployments  
The number of times the application has been uploaded by a developer. The current quota is 10,000 per day.

An application is limited to 10,000 uploaded files per version. Each file is limited to a maximum size of 32 megabytes. Additionally, if the total size of all files for all versions exceeds the initial free 1 gigabyte, then there will be a $ 0.026 per GB per month charge.

### Instance hours

Instance usage is billed by instance uptime, at a given hourly rate. Billable time starts when an instance starts, and ends fifteen minutes after it shuts down. There is no billing for idle instances above the maximum number of idle instances set in the Performance Settings tab of the Cloud Platform Console.

There are separate free daily quotas for frontend and backend instances. Note that when you use the [Modules](https://web.archive.org/web/20160424230405/https://cloud.google.com/appengine/docs/features#modules) API, the module's instance class determines which quota applies.

<table>
<thead>
<tr class="header">
<th>Resource or API Call</th>
<th>Free Quota</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Frontend Instances (Automatic Scaling Modules)</td>
<td>28 free instance-hours per day</td>
</tr>
<tr class="even">
<td>Backend Instances (Basic and Manual Scaling Modules)</td>
<td>9 free instance-hours per day</td>
</tr>
</tbody>
</table>

### Logs

The Logs API is metered when log data is retrieved, and is available for both paid and free apps.

Logs storage contains request logs and application logs for an application, and is available for both paid and free apps. For paid apps, you can increase total logs storage size and/or log data retention time, using the [Log Retention setting](https://web.archive.org/web/20160424230405/https://cloud.google.com/appengine/docs/adminconsole/applicationsettings#Retain_Application_Logs) in the Cloud Platform Console.

<table width="75%">
<thead>
<tr class="header">
<th width="30%">Resource</th>
<th width="35%">Free Default Limit</th>
<th width="35%">Billing Enabled Default Limit</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Logs data retrieval</td>
<td>100 megabytes</td>
<td>No maximum for paid app.</td>
</tr>
<tr class="even">
<td>Logs data</td>
<td>1 gigabyte</td>
<td>Log data kept for a maximum of 365 days if paid, 90 days if free.</td>
</tr>
</tbody>
</table>

### Mail

App Engine bills for email use "by message," counting each email to each recipient. For example, sending one email to ten recipients counts as ten messages.

Mail API Calls  
The total number of times the application accessed the mail service to send a message.

Messages Sent  
The total number of messages (email/recipient pairs) that have been sent by the application. Note that the maximum quota for Messages Sent stays at free levels until the first charge for your application has cleared.

Admin Emails  
The total number of messages to application admins that have been sent by the application. (The total size limit for each admin email, including headers, attachments, and body, is 16KB).

Message Body Data Sent  
The amount of data sent in the body of email messages. This also counts toward the Outgoing Bandwidth quota.

Attachments Sent  
The total number of attachments sent with email messages.

Attachment Data Sent  
The amount of data sent as attachments to email messages. This also counts toward the Outgoing Bandwidth quota.

<table width="75%">
<thead>
<tr class="header">
<th width="40%">Resource</th>
<th width="30%">Daily Limit</th>
<th width="30%">Maximum Rate</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td width="40%">Mail API Calls</td>
<td width="30%">100 calls</td>
<td width="30%">32 calls/minute</td>
</tr>
<tr class="even">
<td>Messages Sent</td>
<td>100 messages</td>
<td>8 messages/minute</td>
</tr>
<tr class="odd">
<td>Admins Emailed</td>
<td>5,000 mails</td>
<td>24 mails/minute</td>
</tr>
<tr class="even">
<td>Message Body Data Sent</td>
<td>60 MB</td>
<td>340 KB/minute</td>
</tr>
<tr class="odd">
<td>Attachments Sent</td>
<td>2,000 attachments</td>
<td>8 attachments/minute</td>
</tr>
<tr class="even">
<td>Attachment Data Sent</td>
<td>100 MB</td>
<td>10 MB/minute</td>
</tr>
</tbody>
</table>

#### Increasing your daily mail quota

If your app needs to send more than 100 messages per day, you can [send mail with SendGrid](https://web.archive.org/web/20160424230405/https://cloud.google.com/appengine/docs/python/mail/sendgrid), which has a higher quota.

### Requests

Outgoing Bandwidth (billable)  
The amount of data sent by the application in response to requests.

This includes:

-   data served in response to both secure requests and non-secure requests by application servers, static file servers, or the Blobstore
-   data sent in email messages
-   data sent over XMPP or the Channel API
-   data in outgoing HTTP requests sent by the URL fetch service.

Incoming Bandwidth  
The amount of data received by the application from requests. Each incoming HTTP request can be no larger than 32MB.

This includes:

-   data received by the application in secure requests and non-secure requests
-   uploads to the Blobstore
-   data received in response to HTTP requests by the URL fetch service

Secure Outgoing Bandwidth  
The amount of data sent by the application over a secure connection in response to requests. Secure outgoing bandwidth also counts toward the Outgoing Bandwidth quota.

Secure Incoming Bandwidth  
The amount of data received by the application over a secure connection from requests. Secure incoming bandwidth also counts toward the Incoming Bandwidth quota.

<table width="75%">
<thead>
<tr class="header">
<th width="30%">Resource</th>
<th colspan="2" width="35%">Free Default Limit</th>
<th colspan="2" width="35%">Billing Enabled Default Limit</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<th width="17.5%">Daily Limit</th>
<th width="17.5%">Maximum Rate</th>
<th width="17.5%">Daily Limit</th>
<th width="17.5%">Maximum Rate</th>
<th></th>
</tr>
&#10;<tr class="odd">
<td>Outgoing Bandwidth (<a href="#Billable_Resource_Quotas">billable</a>, includes HTTPS)</td>
<td>1 GB</td>
<td>56 MB/minute</td>
<td>1 GB free; 14,400 GB maximum</td>
<td>10 GB/minute</td>
</tr>
<tr class="even">
<td>Incoming Bandwidth (includes HTTPS)</td>
<td>1 GB; 14,400 GB maximum</td>
<td>56 MB/minute</td>
<td>None</td>
<td>None</td>
</tr>
</tbody>
</table>

### Search

Free quotas for Search are listed in the table below. Refer to the [Java](https://web.archive.org/web/20160424230405/https://cloud.google.com/appengine/docs/java/search/#Java_Search_API_quotas), [Python](https://web.archive.org/web/20160424230405/https://cloud.google.com/appengine/docs/python/search/#Python_Search_API_quotas), and [Go](https://web.archive.org/web/20160424230405/https://cloud.google.com/appengine/docs/go/search/#Go_Search_API_quotas) documentation for a detailed description of each type of Search call.

Once billing is turned on, Search API resources are charged according to the rates on the [pricing schedule](https://web.archive.org/web/20160424230405/https://cloud.google.com/appengine/docs/pricing#search_pricing).

<table>
<thead>
<tr class="header">
<th>Resource or API Call</th>
<th>Free Quota</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Total Storage (Documents and Indexes)</td>
<td>0.25 GB</td>
</tr>
<tr class="even">
<td>Queries</td>
<td>1000 queries per day</td>
</tr>
<tr class="odd">
<td>Adding documents to Indexes</td>
<td>0.01 GB per day</td>
</tr>
</tbody>
</table>

The application console quota section displays a raw count of API requests. Note that when indexing multiple documents in a single call, the call count is increased by the number of documents.

The Search API imposes these limits to ensure the reliability of the service:

-   100 aggregated minutes of query execution time per minute
-   15,000 Documents added/deleted per minute

In addition, there is a limit of 10GB storage per index. When an app tries to exceed this amount, an insufficient quota error is returned. This limit may be increased to up to 200GB by [submitting a request](https://web.archive.org/web/20160424230405/https://support.google.com/cloud/contact/cloud_search_api_index_size_increase_request_form).

**Note:** Although these limits are enforced by the minute, the Cloud Platform Console displays the daily totals for each. Customers with [Silver, Gold, or Platinum support](https://web.archive.org/web/20160424230405/https://cloud.google.com/support/) can request higher throughput limits by contacting their support representative.

### Sockets

Daily Data and Per-Minute (Burst) Data Limits  
Applications using sockets are rate limited on a per minute and a per day basis. Per minute limits are set to handle burst behavior from applications.

<table width="75%">
<thead>
<tr class="header">
<th width="40%">Resource</th>
<th width="30%">Per Day Limits</th>
<th width="30%">Per Minute (Burst) Limits</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Socket Bind Count</td>
<td>864,000</td>
<td>4,800</td>
</tr>
<tr class="even">
<td>Socket Create Count</td>
<td>864,000</td>
<td>4,800</td>
</tr>
<tr class="odd">
<td>Socket Connect Count</td>
<td>864,000</td>
<td>4,800</td>
</tr>
<tr class="even">
<td>Socket Send Count</td>
<td>663,552,000</td>
<td>3,686,400</td>
</tr>
<tr class="odd">
<td>Socket Receive Count</td>
<td>663,552,000</td>
<td>3,686,400</td>
</tr>
<tr class="even">
<td>Socket Bytes Received</td>
<td>20 GB</td>
<td>113 MB</td>
</tr>
<tr class="odd">
<td>Socket Bytes Sent</td>
<td>20 GB</td>
<td>113 MB</td>
</tr>
</tbody>
</table>

### Task queue

Task Queue API Calls  
The total number of times the application accessed the task queue service to enqueue a task.

Task Queue Stored Task Count  
The total number of tasks the application has enqueued that are not yet executed.

Task Queue Stored Task Bytes  
The bytes consumed by tasks the application has enqueued that are not yet executed. This quota is counted as part of Stored Data (billable).

These limits apply to all task queues:

<table width="75%">
<thead>
<tr class="header">
<th width="30%">Resource</th>
<th colspan="2" width="35%">Free Default Limit</th>
<th colspan="2" width="35%">Billing Enabled Default Limit</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<th width="17.5%">Daily Limit</th>
<th width="17.5%">Maximum Rate</th>
<th width="17.5%">Daily Limit</th>
<th width="17.5%">Maximum Rate</th>
<th></th>
</tr>
&#10;<tr class="odd">
<td width="30%">Task Queue API Calls</td>
<td width="17.5%">100,000</td>
<td width="17.5%"><em>n/a</em></td>
<td width="17.5%">1,000,000,000</td>
<td width="17.5%"><em>n/a</em></td>
</tr>
<tr class="even">
<td width="30%">Task Queue Management Calls (using the Cloud Platform Console)</td>
<td width="17.5%">10,000</td>
<td width="17.5%"><em>n/a</em></td>
<td width="17.5%">10,000</td>
<td width="17.5%"><em>n/a</em></td>
</tr>
</tbody>
</table>

<table width="75%">
<thead>
<tr class="header">
<th width="30%">Resource</th>
<th width="35%">Free Default Limit</th>
<th width="35%">Billing Enabled Default Limit</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Task Queue Stored Task Count</td>
<td>1,000,000</td>
<td>10,000,000,000</td>
</tr>
<tr class="even">
<td>Task Queue Stored Task Bytes</td>
<td>500 MB. Configurable up to 1 GB.</td>
<td>None. Configurable up to the <strong>Stored Data (billable).</strong></td>
</tr>
</tbody>
</table>

The following limits apply to task queues according to their type:

<table>
<thead>
<tr class="header">
<th colspan="2">Push Queue Limits</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Maximum task size</td>
<td>100KB</td>
</tr>
<tr class="even">
<td>Queue execution rate</td>
<td>500 task invocations per second per queue</td>
</tr>
<tr class="odd">
<td>Maximum countdown/ETA for a task</td>
<td>30 days from the current date and time</td>
</tr>
<tr class="even">
<td>Maximum number of tasks that can be added in a batch</td>
<td>100 tasks</td>
</tr>
<tr class="odd">
<td>Maximum number of tasks that can be added in a transaction</td>
<td>5 tasks</td>
</tr>
</tbody>
</table>

<table>
<thead>
<tr class="header">
<th colspan="2">Combined Limits (Push and Pull Queues)</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Maximum number of active queues (not including the default queue)</td>
<td>Free apps: 10 queues, Billed apps: 100 queues</td>
</tr>
</tbody>
</table>

<table>
<thead>
<tr class="header">
<th colspan="2">Pull Queue Limits</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Maximum task size</td>
<td>1MB</td>
</tr>
<tr class="even">
<td>Maximum countdown/ETA for a task</td>
<td>30 days from the current date and time</td>
</tr>
<tr class="odd">
<td>Maximum number of tasks that can be added in a batch</td>
<td>100 tasks</td>
</tr>
<tr class="even">
<td>Maximum number of tasks that can be added in a transaction</td>
<td>5 tasks</td>
</tr>
<tr class="odd">
<td>Maximum number of tasks that you can lease in a single operation</td>
<td>1000 tasks</td>
</tr>
<tr class="even">
<td>Maximum payload size when leasing a batch of tasks</td>
<td>32MB (1MB when using the REST API)</td>
</tr>
</tbody>
</table>

<table>
<thead>
<tr class="header">
<th colspan="2">Combined Limits (Push and Pull Queues)</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Maximum number of active queues (not including the default queue)</td>
<td>Free apps: 10 queues, Billed apps: 100 queues</td>
</tr>
</tbody>
</table>

**Note:** Pull queues are not available in PHP.

### URL fetch

URL Fetch API Calls  
The total number of times the application accessed the URL fetch service to perform an HTTP or HTTPS request.

URL Fetch Data Sent  
The amount of data sent to the URL fetch service in requests. This also counts toward the Outgoing Bandwidth quota.

URL Fetch Data Received  
The amount of data received from the URL fetch service in responses. This also counts toward the Incoming Bandwidth quota.

<table width="75%">
<thead>
<tr class="header">
<th width="30%">Resource</th>
<th colspan="2" width="35%">Free Default Limit</th>
<th colspan="2" width="35%">Billing Enabled Default Limit</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<th width="17.5%">Daily Limit</th>
<th width="17.5%">Maximum Rate</th>
<th width="17.5%">Daily Limit</th>
<th width="17.5%">Maximum Rate</th>
<th></th>
</tr>
&#10;<tr class="odd">
<td width="30%">UrlFetch API Calls</td>
<td width="17.5%">657,000 calls</td>
<td width="17.5%">3,000 calls/minute</td>
<td width="17.5%">860,000,000 calls</td>
<td width="17.5%">660,000 calls/minute</td>
</tr>
<tr class="even">
<td>UrlFetch Data Sent</td>
<td>4 GB</td>
<td>22 MB/minute</td>
<td>4.5 TB</td>
<td>3,600 MB/minute</td>
</tr>
<tr class="odd">
<td>UrlFetch Data Received</td>
<td>4 GB</td>
<td>22 MB/minute</td>
<td>4.5 TB</td>
<td>3,600 MB/minute</td>
</tr>
</tbody>
</table>

### XMPP

XMPP API Calls  
The total number of times the application accessed the XMPP service.

XMPP Data Sent  
The amount of data sent via the XMPP service. This also counts toward the Outgoing Bandwidth quota.

Recipients Messaged  
The total number of recipients to whom the application has sent XMPP messages.

Invitations Sent  
The total number of chat invitations sent by the application.

Stanzas Sent  
XMPP stanzas sent when the application sends a message, invitation, or presence information.

<table width="75%">
<thead>
<tr class="header">
<th width="30%">Resource</th>
<th colspan="2" width="35%">Free Default Limit</th>
<th colspan="2" width="35%">Billing Enabled Default Limit</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<th width="17.5%">Daily Limit</th>
<th width="17.5%">Maximum Rate</th>
<th width="17.5%">Daily Limit</th>
<th width="17.5%">Maximum Rate</th>
<th></th>
</tr>
&#10;<tr class="odd">
<td width="30%">XMPP API Calls</td>
<td width="17.5%">46,000,000 calls</td>
<td width="17.5%">257,280 calls/minute</td>
<td width="17.5%">46,000,000 calls</td>
<td width="17.5%">257,280 calls/minute</td>
</tr>
<tr class="even">
<td>XMPP Data Sent</td>
<td>1 GB</td>
<td>5.81 GB/minute</td>
<td>1,046 GB</td>
<td>5.81 GB/minute</td>
</tr>
<tr class="odd">
<td>XMPP Recipients Messaged</td>
<td>46,000,000 recipients</td>
<td>257,280 recipients/minute</td>
<td>46,000,000 recipients</td>
<td>257,280 recipients/minute</td>
</tr>
<tr class="even">
<td>XMPP Invitations Sent</td>
<td>100,000 invitations</td>
<td>2,000 invitations/minute</td>
<td>100,000 invitations</td>
<td>2,000 invitations/minute</td>
</tr>
<tr class="odd">
<td>XMPP Stanzas Sent</td>
<td>10,000 stanzas</td>
<td>n/a</td>
<td>Based on your spending limit</td>
<td>n/a</td>
</tr>
</tbody>
</table>
