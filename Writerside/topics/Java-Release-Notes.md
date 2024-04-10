# Java Release Notes

## April 18, 2016 - Version 1.9.36

### App Engine notes

-   In response to your requests, the App Engine Users API joins the rest of App Engine in supporting IAM roles and group expansion. This means that any user who is a project Owner, Editor or Viewer or an App Engine Admin is considered an "admin" by the Users API, regardless of whether the user was granted the role directly or by membership in a group.
-   This release populates error details, when available, in error messages associated with the "OverQuota" exception type.

### Java runtime notes

-   Google no longer accepts quota increase requests for the mail service. Customers should use [Sendgrid](https://web.archive.org/web/20160424225258/https://cloud.google.com/appengine/docs/java/mail/sendgrid) instead.

## March 24, 2016 - Version 1.9.35

### App Engine notes

-   App Engine Managed VMs is renamed to [App Engine flexible environment](https://web.archive.org/web/20160424225258/https://cloud.google.com/appengine/docs/flexible/).
-   Fixes trace timestamps to match log timestamps.

### Java runtime notes

-   This release does not include a new Java SDK. Java users should continue to use the 1.9.34 SDK.

## March 16, 2016

### Java runtime notes

-   Version 1.9.34 of the Java SDK is available.

## March 4, 2016 - Version 1.9.34

### App Engine notes

-   Increases default quota for URL fetch for billed apps. Refer to the [Quotas page](https://web.archive.org/web/20160424225258/https://cloud.google.com/appengine/docs/quotas#UrlFetch) for details.

### Java runtime notes

-   This release does not include a new Java SDK. Java users should continue to use the 1.9.32 SDK.

## February 17, 2016 - Version 1.9.33

### App Engine notes

-   The URL path "/form" is now allowed and will be forwarded to applications. Previously, this path was blocked.

### Java runtime notes

-   This release does not include a new Java SDK. Java users should continue to use the 1.9.32 SDK.

## February 3, 2016 - Version 1.9.32

### App Engine notes

-   Container construction choices for Managed VMs

    The `gcloud preview app deploy` (and `mvn gcloud:deploy`) commands upload your artifacts to our servers and build a container to deploy your app to the Managed VM environment.

    There are two mechanisms for building the container image remotely. The default behavior is to build the container on a transient Compute Engine Virtual Machine which has Docker installed. Alternatively, you can use the [Container Builder](https://web.archive.org/web/20160424225258/https://cloud.google.com/container-builder/docs/) service, which is in Beta. To use the Container Builder service, follow these steps:

    1.  [Activate the Container Builder API](https://web.archive.org/web/20160424225258/https://support.google.com/cloud/answer/6158841) for your project.
    2.  Use the command `gcloud config set app/use_cloud_build True`. This will cause all invocations of `gcloud preview app deploy` to use the service. (To return to the default behavior, use the command `gcloud config set app/use_cloud_build False`.

### Java runtime notes

-   Improved exception handling for the low-level Java API for Datastore, Transaction.rollback(). Instead of an exception, it generates an INFO log message when an operation associated with the transaction has failed.

## January 14, 2016 - Version 1.9.31

### App Engine notes

-   App Engine now supports Google Groups: Adding a Google Group as a member of a project grants the members of the group access to App Engine. For example, if a Google Group is an Editor on a project, all members of the group now have Editor access to the App Engine application.

## November 30, 2015 - Version 1.9.30

### App Engine notes

-   Headers for push queue requests made for Task Queue tasks with no payload will now contain a Content-Length entry set to '0'. Previously headers for such requests contained no Content-Length entry.

## November 30, 2015 - Version 1.9.29

### App Engine notes

-   Stop calculating and storing queue depth for non-existent queues, queues marked for deletion, and in the case of queue table outages.
-   For developers using the [endpoints API](https://web.archive.org/web/20160424225258/https://cloud.google.com/appengine/docs/python/endpoints/create_api#defining_the_api_endpointsapi), added a discoverable boolean parameter to the @Api annotation to allow users to disable API discovery. Using this feature will prevent some client libraries (e.g. JavaScript) and the API Explorer from working, as they depend on discovery.

## October 29, 2015 - Version 1.9.28

### App Engine notes

-   The Prospective Search API, which was deprecated on July 14, 2015, is now restricted to existing users. It will fully shutdown on December 1, 2015.
-   Improved accuracy of Geo filtering in Search queries.

### Java runtime notes

-   Disabled Files API in the Java DevAppServer.

## September 25, 2015 - Version 1.9.27

### App Engine notes

-   Applications that are newly enabled for billing now default to an unlimited daily budget, and no longer default to a maximum daily budget of $0. This prevents unwanted outages due to running out of budget. To set a ceiling on your application's daily cost, after you enable billing, set a budget in the [app engine settings](https://web.archive.org/web/20160424225258/https://console.cloud.google.com/project/_/appengine/settings). For more information, see [Setting a daily budget](https://web.archive.org/web/20160424225258/https://cloud.google.com/appengine/docs/developers-console/#setting_a_daily_budget).

Datastore

-   Bugfix: Repeated numeric facets are now allowed.
-   Faceted Search is now GA.

## August 27, 2015 - Version 1.9.26

### App Engine notes

-   oauth2client library upgraded to version [1.4.2](https://web.archive.org/web/20160424225258/https://github.com/google/oauth2client/blob/master/CHANGELOG.md)
-   Adds "show in context" menu for MVM application logs that have thread\_id or request\_id as a field in their log entry. This allows sorting app logs based on either field.
-   Capability to provision applications for current load and configure elastic provisioning based on both VM and application level metrics.
-   Remote API can now be accessed using OAuth2 credentials using https://developers.google.com/identity/protocols/application-default-credentials
-   Use RequestPayloadTooLargeException for URLFetch requests with payloads that are too large.

### Java runtime notes

-   Java's URLFetch API gains a property to specify default fetch deadline. `appengine.api.urlfetch.defaultDeadline` is a floating point number in seconds that can be used to specify a default URLFetch timeout for Java in appengine-web.xml.

## August 14, 2015 - Version 1.9.25

### App Engine notes

-   Added PyAMF version 0.7.2 (Beta).
-   Admin Console menus start redirecting to Cloud Platform Console. Select services such as the Admin Logs will continue to be available in the Admin Console.
-   Datastore now allows properties to represent the empty list.
-   Failed tasks in queues configured with a ‘retry\_limit’ of zero will no longer be retried.

## Older release notes

(See earlier release notes in the [wiki](https://web.archive.org/web/20160424225258/https://code.google.com/p/googleappengine/wiki/SdkForJavaReleaseNotes))
