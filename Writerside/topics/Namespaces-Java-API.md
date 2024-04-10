# Namespaces Java API

  

[Python](https://web.archive.org/web/20160425094147/https://cloud.google.com/appengine/docs/python/multitenancy/ "View this page in the Python runtime") <span style="color: #ddd; padding: 0em .5em;">|</span><span style="color: black; font-weight:bold">Java</span> <span style="color: #ddd; padding: 0em .5em;">|</span><span style="color:lightgray" title="This page is not available in the PHP runtime">PHP</span> <span style="color: #ddd; padding: 0em .5em;">|</span>[Go](https://web.archive.org/web/20160425094147/https://cloud.google.com/appengine/docs/go/multitenancy/ "View this page in the Go runtime")

[<img src="https://web.archive.org/web/20160425094147im_/https://cloud.google.com/cloud/images/stack_overflow_questions.png" title="Stack Overflow Questions" data-border="0" width="240" height="53" />](https://web.archive.org/web/20160425094147/http://stackoverflow.com/questions/tagged/google-app-engine+multi-tenant)

[](https://web.archive.org/web/20160425094147/http://stackoverflow.com/feeds/tag?sort=votes&tagnames=google-app-engine%2Bmulti-tenant)

<span class="link gc-analytics-event" category="Sidebar" data-action="Stack Overflow question click"></span>

<a href="https://web.archive.org/web/20160425094147/http://stackoverflow.com/questions/tagged/google-app-engine+multi-tenant?sort=votes" class="gc-analytics-event" data-category="Sidebar" data-action="Stack Overflow See more link">See more...</a>

The Namespaces API in Google App Engine makes it easy to compartmentalize your Google App Engine data. This API is implemented via a new package called the [namespace manager](https://web.archive.org/web/20160425094147/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/NamespaceManager), and is incorporated in certain [namespace-enabled APIs](#Java_App_Engine_APIs_that_use_namespaces).

When you set a namespace in the namespace manager, these APIs get the current namespace and use it globally. You can explicitly declare a namespace locally, but you need to exercise caution when explicitly declaring namespaces, because you could inadvertently create data leaks and other bugs. Any App Engine request can access any namespace, leaving the application to enforce an access control policy across namespaces.

You can use the Namespaces API to build a wide range of applications. One of the most compelling uses of this API is for multitenant applications, as described below.

1.  [About multitenancy](#Java_About_multitenancy)
2.  [Building a multitenant application with the Namespaces API](#Java_Building_a_multitenant_application_with_the_Namespaces_API)
3.  [App Engine APIs that use namespaces](#Java_App_Engine_APIs_that_use_namespaces)
4.  [Sample projects using namespaces](#Java_Sample_projects_using_namespaces)
5.  [Other Uses for the Namespaces API](#Java_Other_uses_for_the_Namespaces_API)

## About multitenancy

*Multitenancy* is the name given to a software architecture in which one instance of an application, running on a remote server, serves many client organizations (also known as *tenants*).

Using a multitenant architecture simplifies administration and provisioning of tenants. You can provide a more streamlined, customized user experience, and also aggregate different silos of data under a single database schema. As a result, your applications become more scalable as well as more cost-effective as you scale. Data becomes easier to segregate and analyze across tenants because all tenants share the same database schema. Different user groups see custom content wrapped within a more efficient application.

## Building a multitenant application with the Namespaces API

Using the Namespaces API, you can easily partition data across tenants simply by specifying a unique namespace string for each tenant. You simply [set the namespace](https://web.archive.org/web/20160425094147/https://cloud.google.com/appengine/docs/java/multitenancy/multitenancy#Java_Setting_the_current_namespace) for each tenant globally using the namespace manager (as opposed to setting it explicitly for a specific request). The [namespace-enabled APIs](#Java_App_Engine_APIs_that_use_namespaces) always use this current namespace by default.

The Namespaces API is integrated with Google Apps, allowing you to use your Google Apps domain as the current namespace. Because Google Apps lets you deploy your app to any domain that you own, you can easily [set unique namespaces](https://web.archive.org/web/20160425094147/https://cloud.google.com/appengine/docs/java/multitenancy/multitenancy#Java_Setting_the_current_namespace) for all domains linked to your Google Apps account.

When designing multitenant applications, you need to prevent data from leaking across namespaces. For more information, please see [Avoiding Data Leaks](https://web.archive.org/web/20160425094147/https://cloud.google.com/appengine/docs/java/multitenancy/multitenancy#Java_Avoiding_data_leaks).

## App Engine APIs that use namespaces

App Engine currently supports namespaces in the following APIs:

-   [Datastore](https://web.archive.org/web/20160425094147/https://cloud.google.com/appengine/docs/java/multitenancy/multitenancy#Java_Using_namespaces_with_the_Datastore)
-   [Memcache](https://web.archive.org/web/20160425094147/https://cloud.google.com/appengine/docs/java/multitenancy/multitenancy#Java_Using_namespaces_with_the_Memcache)
-   [Task queue](https://web.archive.org/web/20160425094147/https://cloud.google.com/appengine/docs/java/multitenancy/multitenancy#Java_Using_namespaces_with_the_Task_Queue)
-   [Search](https://web.archive.org/web/20160425094147/https://cloud.google.com/appengine/docs/java/multitenancy/multitenancy#Java_Using_namespaces_with_Search)

## Sample projects using namespaces

Two sample guestbook applications using namespaces are provided:

-   Python App Engine: [appengine-multitenancy](https://web.archive.org/web/20160425094147/https://github.com/GoogleCloudPlatform/python-docs-samples/tree/master/appengine/multitenancy) – A namespace-aware sample guestbook application.
-   Java App Engine: [appengine-gwtguestbook-namespaces-java](https://web.archive.org/web/20160425094147/https://github.com/GoogleCloudPlatform/appengine-gwtguestbook-namespaces-java) – A namespace aware sample guestbook application using GWT.

## Other uses for the Namespace API

While the Namespaces API enables multitenancy on App Engine, it has a number of other uses, including:

-   Compartmentalizing user information
-   Separating admin data from application data
-   Creating separate datastore instances for testing and production
-   Running multiple apps on a single app engine instance
