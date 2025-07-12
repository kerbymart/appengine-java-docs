# Glossary of App Engine Terms

  

[Python](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/python/glossary "View this page in the Python runtime") <span style="color: #ddd; padding: 0em .5em;">|</span><span style="color: black; font-weight:bold">Java</span> <span style="color: #ddd; padding: 0em .5em;">|</span>[PHP](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/php/glossary "View this page in the PHP runtime") <span style="color: #ddd; padding: 0em .5em;">|</span>[Go](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/go/glossary "View this page in the Go runtime")

This page defines App Engine-specific terms and concepts.

1.  [A](#a)
    1.  [Ancestor](#ancestor)
    2.  [Ancestor Query](#ancestor_query)
    3.  [Apache Ant](#apache_ant)
    4.  [AppCfg Command Line Tool](#appcfg_command_line_tool)
    5.  [appengine-web.xml](#appengine-webxml)
    6.  [Application ID](#application_id)
    7.  [Appspot](#appspot)
    8.  [Appstats](#appstats)
    9.  [app.yaml](#appyaml)
    10. [App Engine](#app_engine)
    11. [App Engine SDK](#app_engine_sdk)
    12. [Asynchronous Datastore](#asynchronous_datastore)
    13. [Authentication](#authentication)
2.  [B](#b)
    1.  [Backend Instance](#backend_instance)
    2.  [backends.xml](#backendsxml)
    3.  [backends.yaml](#backendsyaml)
    4.  [Billable Quota](#billable_quota)
    5.  [Blob](#blob)
    6.  [Blobstore API](#blobstore_api)
3.  [C](#c)
    1.  [Capabilities API](#capabilities_api)
    2.  [CGI](#cgi)
    3.  [Channel API](#channel_api)
    4.  [Common Gateway Interface (CGI)](#common_gateway_interface_cgi)
    5.  [Concurrency](#concurrency)
    6.  [Concurrency Control](#concurrency_control)
    7.  [Concurrent Request](#concurrent_request)
    8.  [Configuration File](#configuration_file)
    9.  [CPU Time](#cpu_time)
    10. [Cron Job](#cron_job)
    11. [cron.xml](#cronxml)
    12. [cron.yaml](#cronyaml)
    13. [Custom Domain](#custom_domain)
4.  [D](#d)
    1.  [Datastore](#datastore)
    2.  [Datastore Blob Property](#datastore_blob_property)
    3.  [Datastore Call Deadline](#datastore_call_deadline)
    4.  [Datastore Index](#datastore_index)
    5.  [Datastore Index Configuration](#datastore_index_configuration)
    6.  [Denial of Service Protection Service](#denial_of_service_protection_service)
    7.  [Deployment Descriptor](#deployment_descriptor)
    8.  [dev\_appserver](#dev_appserver)
    9.  [Development Console](#development_console)
    10. [Development Web Server](#development_web_server)
    11. [dev\_appserver.py](#dev_appserverpy)
    12. [dev\_appserver.sh](#dev_appserversh)
    13. [Django](#django)
    14. [DoS](#dos)
    15. [dos.xml](#dosxml)
    16. [dos.yaml](#dosyaml)
5.  [E](#e)
    1.  [Eclipse](#eclipse)
    2.  [Entity](#entity)
    3.  [Entity Group](#entity_group)
    4.  [Experimental](#experimental)
    5.  [Exploding Index](#exploding_index)
6.  [F](#f)
    1.  [Filter Class](#filter_class)
    2.  [Frontend Instance](#frontend_instance)
7.  [G](#g)
    1.  [Google App Engine Launcher](#google_app_engine_launcher)
    2.  [Google App Engine SDK](#google_app_engine_sdk)
    3.  [Google Cloud Platform Console](#console_name)
    4.  [Google Plugin for Eclipse](#google_plugin_for_eclipse)
    5.  [Google Protocol RPC Library](#google_protocol_rpc_library)
    6.  [Google Web Toolkit (GWT)](#google_web_toolkit_gwt)
    7.  [Go Programming Language](#go_programming_language)
    8.  [GWT](#gwt)
8.  [H](#h)
    1.  [High Replication Datastore (HRD)](#high_replication_datastore_hrd)
9.  [I](#i)
    1.  [Idle Instances](#idle_instances)
    2.  [Index](#index)
    3.  [index.yaml](#indexyaml)
    4.  [Instance](#instance)
10. [J](#j)
    1.  [Java Archive (JAR)](#java_archive_jar)
    2.  [Java Data Object (JDO)](#java_data_object_jdo)
    3.  [Java Development Server](#java_development_server)
    4.  [Java Persistence API (JPA)](#java_persistence_api_jpa)
    5.  [Java Runtime Environment (JRE)](#java_runtime_environment_jre)
    6.  [Java Servlet](#java_servlet)
    7.  [JRE Class White List](#jre_class_white_list)
11. [K](#k)
    1.  [Key](#key)
    2.  [Kind](#kind)
    3.  [Kindless Query](#kindless_query)
    4.  [Kindless Ancestor Query](#kindless_ancestor_query)
12. [L](#l)
13. [M](#m)
    1.  [Mail](#mail)
    2.  [MapReduce](#mapreduce)
    3.  [Master/Slave Datastore](#masterslave_datastore)
    4.  [Maximum Pending Latency](#maximum_pending_latency)
    5.  [Memcache](#memcache)
    6.  [Metadata Queries](#metadata_queries)
    7.  [Minimum Pending Latency](#minimum_pending_latency)
    8.  [Multitenancy](#multitenancy)
    9.  [M/S Datastore](#ms_datastore)
14. [N](#n)
    1.  [Namespaces API](#namespaces_api)
15. [O](#o)
    1.  [OAuth](#oauth)
16. [P](#p)
    1.  [Parent Entity](#parent_entity)
    2.  [Partitioning](#partitioning)
    3.  [Pending Latency](#pending_latency)
    4.  [Pending Request Queue](#pending_request_queue)
    5.  [Pull Queue](#pull_queue)
    6.  [Push Queue](#push_queue)
    7.  [Python Development Server](#python_development_server)
    8.  [Python Runtime Environment](#python_runtime_environment)
17. [Q](#q)
    1.  [Query](#query)
    2.  [Query Cursor](#query_cursor)
    3.  [queue.xml](#queuexml)
    4.  [queue.yaml](#queueyaml)
    5.  [Quota](#quota)
18. [R](#r)
    1.  [Read Policy](#read_policy)
    2.  [Remote Procedure Call (RPC)](#remote_procedure_call_rpc)
    3.  [Role](#role)
    4.  [Root Entity](#root_entity)
    5.  [RPC](#rpc)
19. [S](#s)
    1.  [Safety Limit](#safety_limit)
    2.  [Scheduler](#scheduler)
    3.  [Service Stub](#service_stub)
    4.  [Servlet](#servlet)
    5.  [Sharding](#sharding)
    6.  [Snapshot Isolation](#snapshot_isolation)
    7.  [Spending Limit](#spending_limit)
    8.  [Static Files](#static_files)
20. [T](#t)
    1.  [Task Queues](#task_queues)
    2.  [Token Bucket](#token_bucket)
    3.  [Transaction](#transaction)
    4.  [Transaction Isolation](#transaction_isolation)
21. [U](#u)
    1.  [Unit Testing](#unit_testing)
    2.  [Upload App](#upload_app)
    3.  [URL Fetch Service](#url_fetch_service)
    4.  [Users Service](#users_service)
22. [V](#v)
23. [W](#w)
    1.  [Webapp Framework](#webapp_framework)
    2.  [Web Application Archive (WAR)](#web_application_archive_war)
    3.  [Web Server Gateway Interface (WSGI)](#web_server_gateway_interface_wsgi)
    4.  [web.xml](#webxml)
    5.  [White List](#white_list)
    6.  [WSGI](#wsgi)
24. [X](#x)
    1.  [XMPP](#xmpp)
25. [Y](#y)
    1.  [YAML (Yet Another Markup Language)](#yaml_yet_another_markup_language)
26. [Z](#z)

## A

### Ancestor

A Datastore entity that is a parent of another entity.

### Ancestor Query

A query over a single [entity group](#term_entity_group) using the key of a parent entity. By default, the results of such a query are [strongly consistent](https://web.archive.org/web/20160424230711/https://en.wikipedia.org/wiki/Strong_consistency).

### Apache Ant

A third-party Java library used to build and test your App Engine applications. While [Apache Ant](https://web.archive.org/web/20160424230711/http://ant.apache.org/) is not a Google tool, App Engine's [Java SDK](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java) includes a set of [Ant](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/tools/ant) macros to perform common App Engine development tasks, including starting your [development server](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/tools/devserver) and uploading your apps to App Engine.

### AppCfg Command Line Tool

The [appcfg.py](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/python/tools/uploadinganapp) command uploads new versions of your code, configuration, and static files for your application to App Engine. You can also use this command to manage Datastore indexes and download log data.

### appengine-web.xml

`appengine-web.xml` is a required [configuration file](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/config/appconfig) for Java applications. At a minimum, this file specifies the application ID and version.

### Application ID

Your application ID is a unique name that you choose when you create your application. More details can be found in the [Java](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/gettingstarted/uploading), [Python](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/python/gettingstartedpython27/uploading) and [Go](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/go/gettingstarted/uploading) Getting Started Guides.

### Appspot

Your application serves from `http://_your_app_id_.appspot.com` by default.

### Appstats

The [Java](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/tools/appstats) and [Python](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/python/tools/appstats) SDKs each include a suite of tools called appstats for measuring the performance of your application. Appstats integrates with your application to record events, and it provides a web-based administrative interface for browsing statistics.

### app.yaml

Python, Go, and PHP applications must use [app.yaml](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/python/config/appconfig) is the main configuration file for Python applications. Java applications use [web.xml](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/config/appconfig) instead.

### App Engine

App Engine is Google's highly scalable platform for hosting web services. It consists of runtimes and SDKs for [Java](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/gettingstarted), [Python](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/python/gettingstartedpython27), and [Go](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/go/gettingstarted) programming languages.

### App Engine SDK

The App Engine Software Development Kits (SDKs) for [Java](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/downloads#Google_App_Engine_SDK_for_Java), [Python](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/downloads#Google_App_Engine_SDK_for_Python), and [Go](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/downloads#Google_App_Engine_SDK_for_Go) each include a web server application that emulates all of the App Engine services on your local computer. Each SDK includes all of the APIs and libraries available on App Engine. The web server also simulates the secure sandbox environment, including checks for attempts to access system resources disallowed in the App Engine runtime environment. Each SDK also includes a [tool](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/python/tools/uploadinganapp#Python_Uploading_the_app) to upload your application to App Engine.

### Asynchronous Datastore

The asynchronous Datastore API allows you to make parallel, non-blocking calls to the Datastore and to retrieve the results of these calls at a later point in the handling of the request. This API is available in both [Java](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/datastore/async) and [Python](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/python/datastore/async).

### Authentication

App Engine applications can [authenticate](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/python/users/) users using any one of three methods: Google Accounts, accounts on your own Google Apps domains, or OpenID identifiers. (Note that the support for OpenID is deprecated and scheduled for removal.) An application can detect whether the current user has signed in, and can redirect the user to the appropriate sign-in page. With a signed-in user, the application can access the user's email address (or OpenID identifier if your application is using OpenID). The application can also detect whether the current user is an administrator, making it easy to implement admin-only areas of the app.

#### Navigate to another letter:

[Top](#) | [A](#a) | [B](#b) | [C](#c) | [D](#d) | [E](#e) | [F](#f) | [G](#g) | [H](#h) | [I](#i) | [J](#j) | [K](#k) | [L](#l) | [M](#m) | [N](#n) | [O](#o) | [P](#p) | [Q](#q) | [R](#r) | [S](#s) | [T](#t) | [U](#u) | [V](#v) | [W](#w) | [X](#x) | [Y](#y) | [Z](#z)

## B

### Backend Instance

An [instance](#term_instance) that is exempt from request deadlines and has access to more memory and CPU than normal instances. Its duration is determined by your configuration with limited scaling based on your settings. App Engine provides [backends](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/python/backends/overview) for applications that need faster performance, large amounts of addressable memory, and continuous or long-running background processes.Backends are supported by [Java](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/backends), [Python](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/python/backends) and [Go](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/go/backends).

### backends.xml

[backends.xml](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/config/backends) is a file that Java applications can use to configure backends. Java applications can also use [backends.yaml](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/configyaml/backends).

### backends.yaml

Python applications must use [backends.yaml](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/python/config/backends) to configure backends. Java applications may use either [backends.yaml](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/configyaml/backends) or [backends.xml](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/config/backends).

### Billable Quota

A total measurement of resource use that combines usage covered by the free quotas and additional usage that is billed.

### Blob

An acronym for "Binary Large Object". A blob may refer to a large data object in the [blobstore API](#term_blobstore_api), or it may refer to a [property type](#term_datastore_blob_property) in the Datastore API.

### Blobstore API

Supported in [Java](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/blobstore/), [Python](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/python/blobstore/), and [Go](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/go/blobstore/), the blobstore API allows your application to serve data objects called blobs, which are much larger than the size allowed for objects in the Datastore service.

#### Navigate to another letter:

[Top](#) | [A](#a) | [B](#b) | [C](#c) | [D](#d) | [E](#e) | [F](#f) | [G](#g) | [H](#h) | [I](#i) | [J](#j) | [K](#k) | [L](#l) | [M](#m) | [N](#n) | [O](#o) | [P](#p) | [Q](#q) | [R](#r) | [S](#s) | [T](#t) | [U](#u) | [V](#v) | [W](#w) | [X](#x) | [Y](#y) | [Z](#z)

## C

### Capabilities API

Supported in [Java](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/capabilities), [Python](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/python/capabilities), and [Go](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/go/capabilities), with the capabilities API, your application can detect outages and scheduled downtime for specific API capabilities. With this information, you can disable the unavailable capability in your application before it impacts your users.

### CGI

See [common gateway interface](#common_gateway_interface_cgi).

### Channel API

Supported in [Java](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/channel/), [Python](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/python/channel/), and [Go](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/go/channel/), the channel API creates a persistent connection between your application and Google servers, allowing your application to send messages to JavaScript clients in real time without the use of polling. This is useful for applications designed to update users about new information immediately. Some example scenarios are collaborative applications, multi-player games, or chat rooms.

### Common Gateway Interface (CGI)

The [common gateway interface (CGI)](https://web.archive.org/web/20160424230711/http://www.w3.org/CGI/) is a standard that defines how web server software can delegate the generation of web pages to a stand-alone application. App Engine uses the [CGI standard with the Python runtime](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/python/#Python_Requests_and_CGI) to communicate the request data to the handler, and receive the response.

### Concurrency

[Concurrency](https://web.archive.org/web/20160424230711/https://en.wikipedia.org/wiki/Concurrency_(computer_science)) occurs when systems execute computations simultaneously, and these computations interact with each other.

### Concurrency Control

[Concurrency control](https://web.archive.org/web/20160424230711/https://en.wikipedia.org/wiki/Concurrency_control) ensures that systems generate correct results for concurrent operations while getting those results as quickly as possible.

### Concurrent Request

App Engine can send multiple requests in parallel to a web server. Concurrent Requests are supported by [Java](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/config/appconfig#Java_appengine_web_xml_Using_concurrent_requests), [Python 2.7](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/python/config/appconfig#Python_app_yaml_Using_concurrent_requests) and Go.

### Configuration File

All App Engine applications require a configuration file to set important details like the application ID and version number. [Python](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/python/config/appconfig), [PHP](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/php/config/appconfig), and [Go](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/go/config/appconfig) applications must use app.yaml as their main configuration file. Java applications use [web.xml](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/config/appconfig).

### CPU Time

[CPU time](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/quotas#Requests) is the amount of time for which a central processing unit (CPU) in a Google data center processes requests from your application.

### Cron Job

The App Engine cron service allows users to create tasks to run at regular intervals. Cron jobs are supported by [Java](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/config/cron), [Python](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/python/config/cron) and [Go](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/go/config/cron).

### cron.xml

For your Java application, a configuration file called [`cron.xml`](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/config/cron#Java_appengine_web_xml_About_cron_xml) or [`cron.yaml`](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/configyaml/cron) file controls scheduled tasks. It consists of a number of job definition pairs, each containing a `<url>` tag and a `<schedule>` tag.

### cron.yaml

For [Java](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/config/cron), [Python](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/python/config/cron) and [Go](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/go/config/cron) applications, a configuration file called `cron.yaml` controls scheduled tasks. It consists of a number of job definition pairs, each containing a `url` field and a `schedule` field.

### Custom Domain

To serve your application on another domain besides appspot.com, you need to [register your domain](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/domain) with Google apps.

#### Navigate to another letter:

[Top](#) | [A](#a) | [B](#b) | [C](#c) | [D](#d) | [E](#e) | [F](#f) | [G](#g) | [H](#h) | [I](#i) | [J](#j) | [K](#k) | [L](#l) | [M](#m) | [N](#n) | [O](#o) | [P](#p) | [Q](#q) | [R](#r) | [S](#s) | [T](#t) | [U](#u) | [V](#v) | [W](#w) | [X](#x) | [Y](#y) | [Z](#z)

## D

### Datastore

Google App Engine uses a Datastore to distribute, replicate, and load balance your data through a simple API. It also includes a powerful query engine and processes transactions. [Java](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/datastore), [Python](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/python/datastore) and [Go](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/go/datastore) can all make use of the Datastore.

### Datastore Blob Property

For Datastore storage, you must use the blob property value type to store unencoded byte strings longer than 1500 bytes. Blobs stored in this way are not indexed. For more information see the [Java](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/datastore/entities#Java_Properties_and_value_types), [Python](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/python/datastore/typesandpropertyclasses#Blob) or [Go](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/go/datastore) documentation

### Datastore Call Deadline

The Datastore call deadline is the maximum amount of time your API call to the Datastore can take. If your API call does not complete by the deadline, the Datastore aborts with an error and returns control to your application. For more information see the [Java](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/datastore/queries#Java_Data_consistency), [Python](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/python/datastore/queries#Python_Data_consistency) or [Go](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/go/datastore/#Go_Queries_and_indexes) documentation.

### Datastore Index

Every Datastore query uses an index, a table containing the results for the query in the desired order. For more information see the [Java](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/datastore/indexes), [Python](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/python/datastore/indexes) or [Go](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/go/datastore/#Go_Queries_and_indexes) documentation.

### Datastore Index Configuration

The App Engine Datastore uses indexes for every query your application makes. The Datastore updates these indexes whenever an [entity](#term_entity) changes, so the Datastore quickly returns results to you when your application makes a query. To do this, the Datastore needs to know in advance which queries the application will make. You specify which indexes your application needs in a configuration file. Java applications can use either [datastore-indexes.xml](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/config/indexconfig) or [index.yaml](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/configyaml/indexconfig). [Python](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/python/config/indexconfig) and [Go](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/go/config/indexconfig) applications must use index.yaml

### Denial of Service Protection Service

The App Engine denial of service (DoS) protection service allows you to protect your application from running out of quota when subjected to denial of service attacks or similar forms of abuse. You can blacklist IP addresses or subnets, and the service drops requests routed from those addresses or subnets before App Engine calls your code. DoS Protection can be configured for Java applications using either [dos.xml](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/config/dos) or [dos.yaml](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/configyaml/indexconfig). [Python](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/python/config/dos) and [Go](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/go/config/dos) applications must use dos.yaml

### Deployment Descriptor

Java web applications use a [deployment descriptor](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/config/webxml) file called `web.xml` to determine how URLs map to servlets, which URLs require authentication, and other information.

### dev\_appserver

See [development web server](#term_development_web_server).

### Development Console

The development web server includes a console web application. The consoles in [Java](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/tools/devserver#Java_The_Development_Console), [Python](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/python/tools/devserver#Python_The_Development_Console) and [Go](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/go/tools/devserver#Go_The_Development_Console), all let you browse the local Datastore, but each console also has unique features of its own.

### Development Web Server

The App Engine SDK includes a development web server for testing your application on your computer. In Java, the [development web server](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/tools/devserver) simulates the App Engine Java runtime environment and all of its services, including the Datastore. In Python, the [development web server](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/python/tools/devserver) simulates your application running in the App Engine Python runtime environment; this simulated environment enforces some sandbox restrictions such as restricted system functions and Python module imports. For Go, the [development web server](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/go/tools/devserver) simulates the App Engine Go runtime environment and all Go's supported services.

### dev\_appserver.py

A command-line tool for interacting with the [Python development server](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/python/tools/devserver).

### dev\_appserver.sh

A command-line tool for interacting with the [Java development server](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/tools/devserver).

### Django

[Django](https://web.archive.org/web/20160424230711/https://www.djangoproject.com/) is a high-level Python web framework that encourages rapid development.

### DoS

See [denial of service protection service](#term_denial_of_service_protection_service).

### dos.xml

A configuration file allowing Java applications to [configure denial of service](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/config/dos#Java_appengine_web_xml_About_dos_xml) protection. Java applications may also use [dos.yaml](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/configyaml/dos).

### dos.yaml

A configuration file allowing [Python](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/python/config/dos), [Java](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/configyaml/dos) or [Go](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/go/config/dos) applications to configure denial of service protection.

#### Navigate to another letter:

[Top](#) | [A](#a) | [B](#b) | [C](#c) | [D](#d) | [E](#e) | [F](#f) | [G](#g) | [H](#h) | [I](#i) | [J](#j) | [K](#k) | [L](#l) | [M](#m) | [N](#n) | [O](#o) | [P](#p) | [Q](#q) | [R](#r) | [S](#s) | [T](#t) | [U](#u) | [V](#v) | [W](#w) | [X](#x) | [Y](#y) | [Z](#z)

## E

### Eclipse

[Eclipse](https://web.archive.org/web/20160424230711/http://www.eclipse.org/) is an open-source integrated development environment (IDE) for Java developers. Java developers primarily use Eclipse, but it also supports other languages (including Python) because of its robust set of plugins.

### Entity

The Datastore writes data in objects known as entities, and each entity has a [key](#term_key) that identifies the entity. For more information see the entity documentation for [Java](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/datastore/entities), [Python](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/python/datastore/entities) or [Go](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/go/datastore/#Go_Entities).

### Entity Group

An entity group is a set of entities whose keys all specify the same root entity.

### Experimental

Features marked "Experimental"; are innovative new features that we are developing rapidly. The [App Engine SLA](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/sla) does not support these features, and Google may make backwards-incompatible changes to them at any time.

### Exploding Index

Custom indexes that refer to multiple properties with multiple values can get very large with only a few values. To completely record such properties, the [index table](#term_datastore_index) must include a row for every combination of values of the indexed properties. Because exploding indexes contain so many values, they increase the amount of Datastore CPU time that your application uses. The SDK attempts to detect exploding indexes and suggests an alternative, but in some cases you may require custom configuration. See the Datastore Indexes page for [Java](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/datastore/indexes#Java_Index_limits) or [Python](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/python/datastore/indexes#Python_Index_limits) for more information.

#### Navigate to another letter:

[Top](#) | [A](#a) | [B](#b) | [C](#c) | [D](#d) | [E](#e) | [F](#f) | [G](#g) | [H](#h) | [I](#i) | [J](#j) | [K](#k) | [L](#l) | [M](#m) | [N](#n) | [O](#o) | [P](#p) | [Q](#q) | [R](#r) | [S](#s) | [T](#t) | [U](#u) | [V](#v) | [W](#w) | [X](#x) | [Y](#y) | [Z](#z)

## F

### Filter Class

A [filter](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/config/webxml#Filters) is a class that acts on a request like a servlet, but may allow the handling of the request to continue with other filters or servlets.

### Frontend Instance

An [instance](#term_instance) running your code and scaling dynamically based on the incoming requests but limited in [how long a request can run](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/quotas#Requests)

#### Navigate to another letter:

[Top](#) | [A](#a) | [B](#b) | [C](#c) | [D](#d) | [E](#e) | [F](#f) | [G](#g) | [H](#h) | [I](#i) | [J](#j) | [K](#k) | [L](#l) | [M](#m) | [N](#n) | [O](#o) | [P](#p) | [Q](#q) | [R](#r) | [S](#s) | [T](#t) | [U](#u) | [V](#v) | [W](#w) | [X](#x) | [Y](#y) | [Z](#z)

## G

### Google App Engine Launcher

The Python SDK for Windows and Mac includes Google App Engine launcher, an application that runs on your computer and provides a graphical interface that simplifies many common App Engine development tasks.

### Google App Engine SDK

With App Engine, you can build web applications using either [Java](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/gettingstarted), [Python](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/python/gettingstartedpython27), or [Go](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/go/gettingstarted) programming languages. Your application runs in a secured “sandbox” environment to isolate your application from security threats.

### Google Cloud Platform Console

A web-based user interface for managing your application. You can use the [Cloud Platform Console](https://web.archive.org/web/20160424230711/https://console.cloud.google.com/) to create new apps, change which version of your application is serving, and perform tasks such as viewing error logs and analyzing client requests. You can also use the console to administer your Datastore, manage task queues, and test new versions of your app.

### Google Plugin for Eclipse

With the [Google Plugin for Eclipse](https://web.archive.org/web/20160424230711/https://developers.google.com/eclipse/docs/appengine), you create, test, and upload your Java App Engine applications from within Eclipse. The Google Plugin for Eclipse also makes it easy to develop applications using [Google Web Toolkit](https://web.archive.org/web/20160424230711/https://cloud.google.com/web-toolkit/) (GWT) to run on App Engine or in any other environment.

### Google Protocol RPC Library

The [Google Protocol RPC](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/python/tools/protorpc) library is a simple way to create well-defined, easy-to-use, web-based remote procedure call (RPC) services. An RPC service is a collection of message types and remote methods that provide a structured way for external applications to interact with web applications. Because it is possible to define messages and services using only the Python programming language, it's easy to get started developing your own services.

### Google Web Toolkit (GWT)

[Google Web Toolkit](https://web.archive.org/web/20160424230711/https://cloud.google.com/web-toolkit/) (GWT) is a free, open source development toolkit for building and optimizing complex browser-based applications. Its goal is to enable productive development of high-performance web applications without the developer having to be an expert in browser quirks, XMLHttpRequest, and JavaScript.

### Go Programming Language

By being expressive, concise, clean, and efficient, the [Go programming language](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/go) is an open source project to make programmers more productive. Its concurrency mechanisms make it easy to write programs optimized for multicore and networked machines while its novel type system enables flexible and modular program construction. Go compiles quickly to machine code, but it has the convenience of garbage collection and the power of run-time reflection.

### GWT

See [Google Web Toolkit](#term_google_web_toolkit).

#### Navigate to another letter:

[Top](#) | [A](#a) | [B](#b) | [C](#c) | [D](#d) | [E](#e) | [F](#f) | [G](#g) | [H](#h) | [I](#i) | [J](#j) | [K](#k) | [L](#l) | [M](#m) | [N](#n) | [O](#o) | [P](#p) | [Q](#q) | [R](#r) | [S](#s) | [T](#t) | [U](#u) | [V](#v) | [W](#w) | [X](#x) | [Y](#y) | [Z](#z)

## H

### High Replication Datastore (HRD)

The default data repository of the App Engine Datastore, providing high availability for reads and writes by storing data synchronously in multiple data centers. The result is a highly reliable storage solution, remaining available for reads and writes during planned downtime and extremely resilient in the face of catastrophic failure. [Java](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/datastore/), [Python](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/python/datastore/) and [Go](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/go/datastore) can all make use of the High Replication Datastore.

#### Navigate to another letter:

[Top](#) | [A](#a) | [B](#b) | [C](#c) | [D](#d) | [E](#e) | [F](#f) | [G](#g) | [H](#h) | [I](#i) | [J](#j) | [K](#k) | [L](#l) | [M](#m) | [N](#n) | [O](#o) | [P](#p) | [Q](#q) | [R](#r) | [S](#s) | [T](#t) | [U](#u) | [V](#v) | [W](#w) | [X](#x) | [Y](#y) | [Z](#z)

## I

### Idle Instances

Idle instances, or resident instances are instances primed to recieve additional load for your application. App Engine keeps these instances in reserve at all times. You can see these instances marked as "Resident" in the Instances page of the [Cloud Platform Console](#google_developers_console).

### Index

Every Datastore [query](#term_query_cursor) uses an index, a table containing the results for the query in the desired order. The Datastore maintains an index for every query an application intends to make. As the entities change, the Datastore updates the indexes with the correct results. When the application executes a query, the Datastore fetches the results directly from the corresponding index.

### index.yaml

A configuration file for [Datastore indexes](#term_index) available for [Python](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/python/config/indexconfig), [Java](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/configyaml/indexconfig) and [Go](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/go/config/indexconfig) applications.

### Instance

A small virtual environment to run your code with a reserved amount of [CPU and Memory](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/quotas#Requests).

#### Navigate to another letter:

[Top](#) | [A](#a) | [B](#b) | [C](#c) | [D](#d) | [E](#e) | [F](#f) | [G](#g) | [H](#h) | [I](#i) | [J](#j) | [K](#k) | [L](#l) | [M](#m) | [N](#n) | [O](#o) | [P](#p) | [Q](#q) | [R](#r) | [S](#s) | [T](#t) | [U](#u) | [V](#v) | [W](#w) | [X](#x) | [Y](#y) | [Z](#z)

## J

### Java Archive (JAR)

A [Java ARchive](https://web.archive.org/web/20160424230711/https://en.wikipedia.org/wiki/JAR_file) combines multiple files into one usually to distribute Java applications or Java libraries over the Internet.

### Java Data Object (JDO)

[Java Data Objects](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/datastore/jdo/overview) (JDO) is a standard interface for storing objects containing data into a database. The standard defines interfaces for annotating Java objects, retrieving objects with queries, and interacting with a database using transactions. An application that uses the JDO interface can work with different kinds of databases without using any database-specific code, including relational databases, hierarchical databases, and object databases.

### Java Development Server

The App Engine Java SDK includes a [development web server](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/tools/devserver) for testing your application on your computer. The development web server simulates the App Engine Java runtime environment and all of its services, including the Datastore. The [Google Plugin for Eclipse](#term_eclipse) can run the server in the Eclipse debugger. You can also run the development server from the command line.

### Java Persistence API (JPA)

[Java persistence API](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/datastore/jpa/overview) (JPA) is a standard interface for storing objects containing data into a relational database. The standard defines interfaces for annotating Java objects, retrieving objects using queries, and interacting with a database using transactions.

### Java Runtime Environment (JRE)

App Engine applications can be implemented using the Java programming language, and other languages that use the java virtual machine. The App Engine [Java runtime environment](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java) includes the Java 7 JVM and interfaces to the App Engine services that conform to industry standards, such as servlets and JDO/JPA. You can also use raw APIs to the services for implementing other interfaces.

### Java Servlet

A [servlet](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/#Java_Requests_and_servlets) is a Java programming language class used to extend the capabilities of servers that host applications accessed via a request-response programming model. Although servlets can respond to any type of request, you commonly use them to extend the applications hosted by web servers. When App Engine receives a web request from your application, it invokes the servlet that corresponds to the URL as described in your application's deployment descriptor. It uses the Java servlet API to provide the request data to the servlet, and accept the response data.

### JRE Class White List

The [list](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/jrewhitelist) of available Java classes in the Java standard library to develop App Engine applications.

#### Navigate to another letter:

[Top](#) | [A](#a) | [B](#b) | [C](#c) | [D](#d) | [E](#e) | [F](#f) | [G](#g) | [H](#h) | [I](#i) | [J](#j) | [K](#k) | [L](#l) | [M](#m) | [N](#n) | [O](#o) | [P](#p) | [Q](#q) | [R](#r) | [S](#s) | [T](#t) | [U](#u) | [V](#v) | [W](#w) | [X](#x) | [Y](#y) | [Z](#z)

## K

### Key

A key is a unique identifier for each [entity](#term_entity) in the Datastore. Keys consist of a [kind](#term_kind), a unique name assigned either by the app or the Datastore, and an optional ancestor path that specifies a parent entity. If an ancestor path is present, the entity belongs to an [entity group](#term_entity_group) defined by that parent.

### Kind

Each Datastore [entity](#term_entity) is of a particular kind, which is simply a name specified by the application. A kind categorizes the entity for the purpose of queries. For example, a human resources application might represent each employee at a company with an entity of the kind “Employee”. Unlike rows in a table, two entities of the same kind need not have the same properties. If required, an application can establish such a restriction in its data model. For more details on Kinds see the [Java](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/datastore/entities#Java_Kinds_and_identifiers), [Python](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/python/datastore/entities#Python_Kinds_and_identifiers) and [Go](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/go/datastore/#Go_Kinds_keys_and_identifiers) documentation.

### Kindless Query

In a kindless query, App Engine Datastore returns all entites matching query constraints regardless of kind. See the query documentation for [Java](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/datastore/queries#Java_Kindless_queries), [Python](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/python/datastore/queries#Python_Kindless_queries) and [Go](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/go/datastore/#Go_Queries_and_indexes) for more details.

### Kindless Ancestor Query

In a kindless ancestor query, App Engine Datastore returns all ancestors to a given entity in a query regardless of kind. See the query documentation for [Java](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/datastore/queries#Java_Kindless_ancestor_queries), [Python](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/python/datastore/queries#Kindless_Ancestor_Queries) and [Go](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/go/datastore/#Go_Queries_and_indexes) for more details.

#### Navigate to another letter:

[Top](#) | [A](#a) | [B](#b) | [C](#c) | [D](#d) | [E](#e) | [F](#f) | [G](#g) | [H](#h) | [I](#i) | [J](#j) | [K](#k) | [L](#l) | [M](#m) | [N](#n) | [O](#o) | [P](#p) | [Q](#q) | [R](#r) | [S](#s) | [T](#t) | [U](#u) | [V](#v) | [W](#w) | [X](#x) | [Y](#y) | [Z](#z)

## L

#### Navigate to another letter:

[Top](#) | [A](#a) | [B](#b) | [C](#c) | [D](#d) | [E](#e) | [F](#f) | [G](#g) | [H](#h) | [I](#i) | [J](#j) | [K](#k) | [L](#l) | [M](#m) | [N](#n) | [O](#o) | [P](#p) | [Q](#q) | [R](#r) | [S](#s) | [T](#t) | [U](#u) | [V](#v) | [W](#w) | [X](#x) | [Y](#y) | [Z](#z)

## M

### Mail

An App Engine API to [send email](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/python/mail/sendingmail) from the application. Sending email is supported by [Java](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/mail/), [Python](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/python/mail/) and [Go](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/go/mail/).

### MapReduce

A computing model developed by Google to do efficient distributed computing over large data sets. Mapreduce is supported on the [Java](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/dataprocessing/) and [Python](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/python/dataprocessing/) runtimes. Input data values in the MapReduce model are *mapped* (assigned lookup keys) and stored in intermediate storage, then the resulting key-value pairs are *shuffled* (collated by key), and finally the collated values are *reduced* (manipulated to yield desired results).

### Master/Slave Datastore

A now decommissioned data repository formerly providing asynchronous data replication among data centers. It is no longer available and is replaced by the [High Replication Datastore (HRD)](#term_high_replication_datastore).

### Maximum Pending Latency

The maximum length of time a request must wait in the [pending request queue](#term_pending_request_queue) before App Engine starts a new [instance](#term_instance) to serve it. A higher value means users may wait longer for your application to serve their request; a lower value means users don't have to wait as long, but your application may cost more to run. App Engine can automatically determine the maximum pending latency based on recent request data, or you can specify this setting manually in each module's [configuration file](#configuration_file).

### Memcache

A distributed in-memory data cache to speed up common Datastore queries. Memcache is supported on the [Java](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/memcache/), [Python](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/python/memcache/) and [Go](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/go/memcache/) runtimes.

### Metadata Queries

Metadata queries construct expressions that return metadata from your Datastore about namespaces, kinds, and properties. The queries return your metadata in dynamically generated entities. The most common use for metadata is to implement, for example, back-end administrative functions and meta-programming environments. For more information on using Metadata Queries see the [Java](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/datastore/metadataqueries) and [Python](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/python/datastore/metadataqueries) documentation.

### Minimum Pending Latency

The minimum amount of time a request can wait in the pending queue before being served by an instance. You can specify this setting manually in each module's [configuration file](#configuration_file), or you can let App Engine choose it for you automatically based on request volume.

### Multitenancy

[Multitenancy](https://web.archive.org/web/20160424230711/https://cloud.google.com/) is the name given to a software architecture in which one instance of an application, running on a remote server, serves many client organizations (also known as tenants). For more information on using Multitenancy see the [Java](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/multitenancy/#Java_About_multitenancy) and [Python](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/python/multitenancy/#Python_About_multitenancy) documentation.

### M/S Datastore

See [Master/Slave Datastore](#term_master_slave_datastore).

#### Navigate to another letter:

[Top](#) | [A](#a) | [B](#b) | [C](#c) | [D](#d) | [E](#e) | [F](#f) | [G](#g) | [H](#h) | [I](#i) | [J](#j) | [K](#k) | [L](#l) | [M](#m) | [N](#n) | [O](#o) | [P](#p) | [Q](#q) | [R](#r) | [S](#s) | [T](#t) | [U](#u) | [V](#v) | [W](#w) | [X](#x) | [Y](#y) | [Z](#z)

## N

### Namespaces API

The Namespaces API in Google App Engine allows you to segregate entities into specific namespaces. The Namespaces API is available in [Java](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/multitenancy/#Java_Building_a_multitenant_application_with_the_Namespaces_API) and [Python.](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/python/multitenancy/#Python_Building_a_multitenant_application_with_the_Namespaces_API)

#### Navigate to another letter:

[Top](#) | [A](#a) | [B](#b) | [C](#c) | [D](#d) | [E](#e) | [F](#f) | [G](#g) | [H](#h) | [I](#i) | [J](#j) | [K](#k) | [L](#l) | [M](#m) | [N](#n) | [O](#o) | [P](#p) | [Q](#q) | [R](#r) | [S](#s) | [T](#t) | [U](#u) | [V](#v) | [W](#w) | [X](#x) | [Y](#y) | [Z](#z)

## O

### OAuth

[OAuth](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/python/oauth/) is a protocol that allows users to grant a third party limited permission to access a web application, without sharing their credentials (username and password) with the third party. The third party can be a web application, or any other application with the capability of invoking a web browser for the users, such as a desktop application or an application running on a smart phone. Both [Java](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/oauth/) and [Python](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/python/oauth/) support OAuth.

#### Navigate to another letter:

[Top](#) | [A](#a) | [B](#b) | [C](#c) | [D](#d) | [E](#e) | [F](#f) | [G](#g) | [H](#h) | [I](#i) | [J](#j) | [K](#k) | [L](#l) | [M](#m) | [N](#n) | [O](#o) | [P](#p) | [Q](#q) | [R](#r) | [S](#s) | [T](#t) | [U](#u) | [V](#v) | [W](#w) | [X](#x) | [Y](#y) | [Z](#z)

## P

### Parent Entity

A parent entity is the root of an [entity group](#term_entity_group).

### Partitioning

See [sharding](#term_sharding).

### Pending Latency

The amount of time a request spends in the [pending request queue](#term_pending_request_queue) while waiting to be served. See [maximum pending latency](#term_maximum_pending_latency) for more information.

### Pending Request Queue

The pending request queue where pending requests wait when no [instances](#term_instance) are available to serve them. By default, App Engine automatically determines how long requests wait in the pending queue, but you can also manually configure this and other aspects of task processing in each module's [configuration file](#configuration_file).

### Pull Queue

A pull queue is a type of task queue in which a task consumer “pulls” tasks from your application, processes them outside your application, and then deletes them. The task consumer can be part of your App Engine application (such as a backend) or a system outside of App Engine (using the Task Queue REST API).

### Push Queue

A push queue is a type of [task queue](#term_task_queues) in which your application processes tasks using HTTP request handlers. Each Task object contains an application-specific URL with a request handler for the task, and an optional data payload that parameterizes the task. For example, consider a calendaring application that needs to notify an invitee, via email, that an event has been updated. The data payload for this task consists of the email address and name of the invitee, along with a description of the event. You can use push queues only within the App Engine environment; if you need to access App Engine tasks from outside of App Engine, use pull queues instead.

### Python Development Server

The App Engine Python SDK includes a [web server](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/python/tools/devserver) application you can run on your computer that simulates your application running in the App Engine Python runtime environment. The simulated environment enforces some sandbox restrictions, such as restricted system functions and Python module imports, but not others, like request time-outs or quotas. The server also simulates the services by performing their tasks locally.

### Python Runtime Environment

App Engine applications can be implemented using the Python programming language. The App Engine [Python runtime environment](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/python) includes a specialized version of the Python interpreter, the standard Python library, libraries and APIs for App Engine, and a standard interface to the web server layer.

#### Navigate to another letter:

[Top](#) | [A](#a) | [B](#b) | [C](#c) | [D](#d) | [E](#e) | [F](#f) | [G](#g) | [H](#h) | [I](#i) | [J](#j) | [K](#k) | [L](#l) | [M](#m) | [N](#n) | [O](#o) | [P](#p) | [Q](#q) | [R](#r) | [S](#s) | [T](#t) | [U](#u) | [V](#v) | [W](#w) | [X](#x) | [Y](#y) | [Z](#z)

## Q

### Query

A Datastore query retrieves [entities](#term_entity) that meet a specified set of conditions. The query specifies an entity kind, zero or more conditions based on entity property values (sometimes called "filters"), and zero or more sort order descriptions. When the query is executed, it fetches all entities of the given kind that meet all of the given conditions, sorted in the order described. Queries are supported in both [Java](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/datastore/queries) and [Python](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/python/datastore/queries).

### Query Cursor

Query cursors allow an application to perform a query and retrieve a batch of results, then fetch additional results for the same query in a subsequent web request without the overhead of a query offset. After the application fetches some results for a query, it can ask for an encoded string that represents the location in the result set after the last result fetched (the “cursor”). The application can use the cursor to fetch additional results starting from that point at a later time. Query cursors are supported in both [Java](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/datastore/queries#Java_Query_cursors) and [Python](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/python/datastore/queries#Python_Query_cursors).

### queue.xml

This file [configures task queues](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/config/queue) for Java applications. This file controls many parameters of task queues such as the storage quota, processing rate, the maximum number of concurrent requests, and other values. This configuration file is optional for push queues.

### queue.yaml

This file [configures task queues for Python applications](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/python/config/queue) and for Java applications that use [YAML configuration](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/configyaml/queue).

### Quota

An App Engine application can consume resources up to certain maximums, or [quotas](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/quotas). With quotas, App Engine ensures that other applications running on App Engine won't impact the performance of your app.

#### Navigate to another letter:

[Top](#) | [A](#a) | [B](#b) | [C](#c) | [D](#d) | [E](#e) | [F](#f) | [G](#g) | [H](#h) | [I](#i) | [J](#j) | [K](#k) | [L](#l) | [M](#m) | [N](#n) | [O](#o) | [P](#p) | [Q](#q) | [R](#r) | [S](#s) | [T](#t) | [U](#u) | [V](#v) | [W](#w) | [X](#x) | [Y](#y) | [Z](#z)

## R

### Read Policy

In order to increase data availability, you can set the Datastore read policy so that all reads and queries are [eventually consistent](https://web.archive.org/web/20160424230711/https://en.wikipedia.org/wiki/Eventual_consistency). While the API also allows you to explicitly set a [strong consistency](https://web.archive.org/web/20160424230711/https://en.wikipedia.org/wiki/Strong_consistency) policy, such a setting has no practical effect because non-ancestor queries are always eventually consistent regardless of policy. For more details on the Datastore read policy see the [Java](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/datastore/queries#Java_Data_consistency) and [Python](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/python/datastore/queries#Python_Data_consistency) documentation.

### Remote Procedure Call (RPC)

In a [remote procedure call](https://web.archive.org/web/20160424230711/https://en.wikipedia.org/wiki/Remote_procedure_call) (RPC), a computer program executes a method in another address space (commonly on another computer on a shared network) without the programmer explicitly coding the details for this remote interaction. From the programmer's point of view, the call is local to the executing program.

### Role

App Engine provides three roles—`Viewer`, `Editor`, `Owner`—supplying different levels of access to [Cloud Platform Console](#google_developers_console) features. Each progressively stronger role includes all permissions from the previous role.

### Root Entity

A Datastore [entity](#term_entity) without a [parent](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/python/datastore/entities#Python_Ancestor_paths). Root entities may serve as a parent entity for an [entity group](#term_entity_group).

### RPC

See [remote procedure call](#term_remote_procedure_call).

#### Navigate to another letter:

[Top](#) | [A](#a) | [B](#b) | [C](#c) | [D](#d) | [E](#e) | [F](#f) | [G](#g) | [H](#h) | [I](#i) | [J](#j) | [K](#k) | [L](#l) | [M](#m) | [N](#n) | [O](#o) | [P](#p) | [Q](#q) | [R](#r) | [S](#s) | [T](#t) | [U](#u) | [V](#v) | [W](#w) | [X](#x) | [Y](#y) | [Z](#z)

## S

### Safety Limit

[Safety limits](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/quotas#Safety_Limits_and_Billable_Resource_Quotas) are resource maximums set by App Engine to ensure the integrity of the system. These resources describe the boundaries of the architecture, and App Engine expects all applications to run within the same limits.

### Scheduler

The infrastructure component that determines how many [instances](#term_instance) are needed to serve your application's current traffic and to which instance to send a request.

### Service Stub

A service stub is a method that simulates the behavior of a service in the SDK. Both [Java](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/tools/localunittesting) and [Python](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/python/tools/localunittesting) support service stubs.

### Servlet

See [Java servlet](#term_java_servlet).

### Sharding

Sharding is a means of [partitioning data](https://web.archive.org/web/20160424230711/https://en.wikipedia.org/wiki/Shard_(database_architecture)) in a database.

### Snapshot Isolation

See [transaction isolation](#term_transaction_isolation).

### Spending Limit

[Spending limits](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/quotas#Billable_Resource_Quotas) are resource maximums set by the project owner and billing administrator to prevent the cost of an application from exceeding a daily limit. Every application gets an amount of each resource for free, but you can increase the amount of resources your application can use by enabling billing and then [setting a spending limit](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/console#billing). You will be charged only for the resources used above the free quota thresholds. By default, the spending limit is unlimited.

### Static Files

Static files are files to be served directly to the user for a given URL, such as images, CSS stylesheets, or JavaScript source files. Static file handlers describe which files in the application directory are static files, and which URLs serve them. In Java static files are set in the [appengine-web.xml](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/config/appconfig#Java_appengine_web_xml_Static_files_and_resource_files) file. For [Python](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/python/config/appconfig#Python_app_yaml_Static_file_handlers), [PHP](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/php/config/appconfig), and [Go](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/go/config/appconfig#Go_app_yaml_Static_file_handlers) they are configured in the app.yaml file.

#### Navigate to another letter:

[Top](#) | [A](#a) | [B](#b) | [C](#c) | [D](#d) | [E](#e) | [F](#f) | [G](#g) | [H](#h) | [I](#i) | [J](#j) | [K](#k) | [L](#l) | [M](#m) | [N](#n) | [O](#o) | [P](#p) | [Q](#q) | [R](#r) | [S](#s) | [T](#t) | [U](#u) | [V](#v) | [W](#w) | [X](#x) | [Y](#y) | [Z](#z)

## T

### Task Queues

Task queues enable your applications to schedule tasks to be done later in the background. As such, your application defines tasks, adds them to a queue, and then uses the queue to process them in aggregate. You can configure queue settings in `queue.yaml` or `queue.xml`. See [Queue Definitions](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/python/config/queue#Python_Queue_definitions) for more information about configuring queues. Task Queues are supported by [Java](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/taskqueue/), [Python](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/python/taskqueue/) and [Go](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/go/taskqueue/).

### Token Bucket

In App Engine task queues, the token bucket algorithm used by App Engine determines the rate at which the App Engine task queue processes push tasks. For more information on the token bucket see the [Java](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/config/queue#Queue_Definitions), [Python](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/python/config/queue#Python_Queue_definitions) or [Go](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/go/config/queue#Go_Queue_definitions) documentation.

### Transaction

The App Engine Datastore supports transactions. A transaction is an operation or set of operations that is atomic—either all of the operations in the transaction occur, or none of them occur. An application can perform multiple operations and calculations in a single transaction. Datastore transcations are supported by the [Java](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/datastore/transactions), [Python](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/python/datastore/transactions) and [Go](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/go/datastore/transactions) runtimes.

### Transaction Isolation

In the App Engine Datastore, the isolation level means how much impact a query has on other simultaneous queries being executed. Preferably, each query must run in its own vacuum to avoid any concurrency issues. For a more in-depth discussion on these concerns, see [transaction isolation](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/articles/transaction_isolation).

#### Navigate to another letter:

[Top](#) | [A](#a) | [B](#b) | [C](#c) | [D](#d) | [E](#e) | [F](#f) | [G](#g) | [H](#h) | [I](#i) | [J](#j) | [K](#k) | [L](#l) | [M](#m) | [N](#n) | [O](#o) | [P](#p) | [Q](#q) | [R](#r) | [S](#s) | [T](#t) | [U](#u) | [V](#v) | [W](#w) | [X](#x) | [Y](#y) | [Z](#z)

## U

### Unit Testing

You write unit tests while you develop your App Engine applications. This methodology provides small, maintainable, and reusable units of code and your tests run in your development environment without involving remote components. App Engine provides testing utilities that use local implementations of Datastore and other App Engine services. This means you can exercise your code's use of these services locally, without deploying your code to App Engine, by using service stubs. A service stub is a method that simulates the behavior of a service. Unit testing for [Java](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/tools/localunittesting) and [Python](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/python/tools/localunittesting) App Engine applications is supported.

### Upload App

Upload app is an [AppCfg](#term_appcfg_command) command that you use to upload your application to App Engine. This is a required step after you have registered your application in the [Cloud Platform Console](#google_developers_console).

### URL Fetch Service

App Engine applications can communicate with other applications or access other resources on the web by fetching URLs. An application can use the URL fetch service to issue HTTP and HTTPS requests and receive responses. The URL Fetch service is supported in [Java](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/urlfetch/), [Python](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/python/urlfetch/) and [Go](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/go/urlfetch/).

### Users Service

The Users service provides APIs for your application to integrate with Google user accounts. With the users service, your users can use the Google accounts they already have to sign in to your application. The URL Fetch service is supported in [Java](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/users), [Python](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/python/users) and [Go](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/go/users).

#### Navigate to another letter:

[Top](#) | [A](#a) | [B](#b) | [C](#c) | [D](#d) | [E](#e) | [F](#f) | [G](#g) | [H](#h) | [I](#i) | [J](#j) | [K](#k) | [L](#l) | [M](#m) | [N](#n) | [O](#o) | [P](#p) | [Q](#q) | [R](#r) | [S](#s) | [T](#t) | [U](#u) | [V](#v) | [W](#w) | [X](#x) | [Y](#y) | [Z](#z)

## V

#### Navigate to another letter:

[Top](#) | [A](#a) | [B](#b) | [C](#c) | [D](#d) | [E](#e) | [F](#f) | [G](#g) | [H](#h) | [I](#i) | [J](#j) | [K](#k) | [L](#l) | [M](#m) | [N](#n) | [O](#o) | [P](#p) | [Q](#q) | [R](#r) | [S](#s) | [T](#t) | [U](#u) | [V](#v) | [W](#w) | [X](#x) | [Y](#y) | [Z](#z)

## W

### Webapp Framework

The [webapp framework](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/python/tools/webapp) is a simple web application framework you can use for developing Python 2.5 web applications for App Engine. It is compatible with the WSGI standard for Python web application containers. In the Python 2.7 runtime, it is replaced with the backwards-compatible [webapp2](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/python/tools/webapp2) framework.

### Web Application Archive (WAR)

[Web application archives](https://web.archive.org/web/20160424230711/http://java.sun.com/j2ee/tutorial/1_3-fcs/doc/WCC3.html) bundle web clients to deploy in browsers over the Internet. They contain server-side utility classes, HTML files, image & sound files, and client-side classes such as applets.

### Web Server Gateway Interface (WSGI)

The [web server gateway interface](https://web.archive.org/web/20160424230711/https://en.wikipedia.org/wiki/Web_Server_Gateway_Interface) is a simple and universal interface between web servers and web applications or frameworks for the Python programming language.

### web.xml

See [deployment descriptor](#term_deployment_descriptor).

### White List

See [JRE class white list](#term_jre_class_white_list).

### WSGI

See [web server gateway interface](#term_web_server_gateway_interface).

#### Navigate to another letter:

[Top](#) | [A](#a) | [B](#b) | [C](#c) | [D](#d) | [E](#e) | [F](#f) | [G](#g) | [H](#h) | [I](#i) | [J](#j) | [K](#k) | [L](#l) | [M](#m) | [N](#n) | [O](#o) | [P](#p) | [Q](#q) | [R](#r) | [S](#s) | [T](#t) | [U](#u) | [V](#v) | [W](#w) | [X](#x) | [Y](#y) | [Z](#z)

## X

### XMPP

The XMMP ([Extensible Messaging and Presence Protocol](https://web.archive.org/web/20160424230711/https://en.wikipedia.org/wiki/Extensible_Messaging_and_Presence_Protocol)) API makes it possible to write App Engine applications that communicate with users—or even other applications—over the XMPP protocol. XMPP is also known as "Jabber" and is supported by XMPP-compatible chat clients. XMPP is supported in the [Java](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/java/xmpp), [Python](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/python/xmpp) and [Go](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/go/xmpp/) runtimes.

#### Navigate to another letter:

[Top](#) | [A](#a) | [B](#b) | [C](#c) | [D](#d) | [E](#e) | [F](#f) | [G](#g) | [H](#h) | [I](#i) | [J](#j) | [K](#k) | [L](#l) | [M](#m) | [N](#n) | [O](#o) | [P](#p) | [Q](#q) | [R](#r) | [S](#s) | [T](#t) | [U](#u) | [V](#v) | [W](#w) | [X](#x) | [Y](#y) | [Z](#z)

## Y

### YAML (Yet Another Markup Language)

[YAML](https://web.archive.org/web/20160424230711/http://cpansearch.perl.org/src/INGY/YAML-0.73/README) is a generic data serialization language that you can easily read. It can be used to express the data structures of most modern programming languages. Usually, you use YAML in configuration files, or as a way to print logging/debugging information. App Engine has several different YAML configuration files available for services such as the Datastore, task queue, and backends. For more information, see the [Python](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/python/config), [PHP](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/php/config), or [Go](https://web.archive.org/web/20160424230711/https://cloud.google.com/appengine/docs/go/config/appconfig) configuration documentation.

#### Navigate to another letter:

[Top](#) | [A](#a) | [B](#b) | [C](#c) | [D](#d) | [E](#e) | [F](#f) | [G](#g) | [H](#h) | [I](#i) | [J](#j) | [K](#k) | [L](#l) | [M](#m) | [N](#n) | [O](#o) | [P](#p) | [Q](#q) | [R](#r) | [S](#s) | [T](#t) | [U](#u) | [V](#v) | [W](#w) | [X](#x) | [Y](#y) | [Z](#z)

## Z

#### Navigate to another letter:

[Top](#) | [A](#a) | [B](#b) | [C](#c) | [D](#d) | [E](#e) | [F](#f) | [G](#g) | [H](#h) | [I](#i) | [J](#j) | [K](#k) | [L](#l) | [M](#m) | [N](#n) | [O](#o) | [P](#p) | [Q](#q) | [R](#r) | [S](#s) | [T](#t) | [U](#u) | [V](#v) | [W](#w) | [X](#x) | [Y](#y) | [Z](#z)
