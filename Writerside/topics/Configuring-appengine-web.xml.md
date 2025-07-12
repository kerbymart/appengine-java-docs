# Configuring appengine-web.xml

  

[Python](https://web.archive.org/web/20160424230144/https://cloud.google.com/appengine/docs/python/config/appconfig "View this page in the Python runtime") <span style="color: #ddd; padding: 0em .5em;">|</span><span style="color: black; font-weight:bold">Java</span> <span style="color: #ddd; padding: 0em .5em;">|</span>[PHP](https://web.archive.org/web/20160424230144/https://cloud.google.com/appengine/docs/php/config/appconfig "View this page in the PHP runtime") <span style="color: #ddd; padding: 0em .5em;">|</span>[Go](https://web.archive.org/web/20160424230144/https://cloud.google.com/appengine/docs/go/config/appconfig "View this page in the Go runtime")

In addition to the [`web.xml` deployment descriptor](https://web.archive.org/web/20160424230144/https://cloud.google.com/appengine/docs/java/config/webxml), an App Engine Java application uses a configuration file, named `appengine-web.xml`, to specify the app's registered application ID and the version identifier of the latest code, and to identify which files in the app's WAR are static files (like images) and which are resource files used by the application. The [appcfg](https://web.archive.org/web/20160424230144/https://cloud.google.com/appengine/docs/java/tools/uploadinganapp) command uses this information when you upload the app.

**Note:** If you created your project using the [Google Cloud Platform Console](https://web.archive.org/web/20160424230144/https://console.cloud.google.com/), your project has a title and an ID. In the instructions that follow, the *project* title and ID can be used wherever an *application* title and ID are mentioned. They are the same thing.

1.  [About `appengine-web.xml`](#Java_appengine_web_xml_About_appengine_web_xml)
2.  [Scaling and instance types](#scaling_and_instance_types)
3.  [Static files and resource files](#Java_appengine_web_xml_Static_files_and_resource_files)
    -   [Including and excluding files](#Java_appengine_web_xml_Including_and_excluding_files)
    -   [Changing the MIME Type for static files](#Java_appengine_web_xml_Changing_the_MIME_type_for_static_files)
    -   [Changing the root directory](#Java_appengine_web_xml_Changing_the_root_directory)
    -   [Static cache expiration](#Java_appengine_web_xml_Static_cache_expiration)
4.  [System properties and environment variables](#Java_appengine_web_xml_System_properties_and_environment_variables)
5.  [Configuring Secure URLs (SSL)](#Java_appengine_web_xml_Configuring_secure_URLs__SSL_)
6.  [Enabling sessions](#Java_appengine_web_xml_Enabling_sessions)
7.  [Reserved URLs](#Java_appengine_web_xml_Reserved_URLs)
8.  [Inbound services](#Java_appengine_web_xml_Inbound_services)
9.  [Warmup requests](#Java_Warmup_requests)
10. [Disabling precompilation](#Java_appengine_web_xml_Disabling_precompilation)
11. [Custom error responses](#Java_appengine_web_xml_Custom_error_responses)
12. [Using concurrent requests](#Java_appengine_web_xml_Using_concurrent_requests)
13. [Custom PageSpeed configuration](#Java_appengine_web_xml_Custom_PageSpeed_configuration) (Deprecated)
14. [Auto ID policy](#Java_appengine_web_xml_Auto_ID_policy)
15. [URLFetch timeout](#url_fetch_timeout)

## About `appengine-web.xml`

An App Engine Java app must have a file named `appengine-web.xml` in its WAR, in the directory `WEB-INF/`. This is an XML file whose root element is `<appengine-web-app>`. A minimal file that specifies the application ID, a version identifier, and no static files or resource files looks like this:

```
<?xml version="1.0" encoding="utf-8"?>
<appengine-web-app xmlns="http://appengine.google.com/ns/1.0">
  <application>_your_app_id_</application>
  <version>alpha-001</version>
  <threadsafe>true</threadsafe>
</appengine-web-app>
```
**Note:** The XML Schema Definition of `appengine-web.xml` can be found [here](https://web.archive.org/web/20160424230144/https://code.google.com/p/googleappengine/source/browse/trunk/java/src/main/com/google/appengine/tools/development/appengine-web.xsd). The raw XSD file can be downloaded from this [link](https://web.archive.org/web/20160424230144/https://googleappengine.googlecode.com/svn/trunk/java/src/main/com/google/appengine/tools/development/appengine-web.xsd).

The `<application>` element contains the application's ID. This is the application ID you register when you create your application in the [Google Cloud Platform Console](https://web.archive.org/web/20160424230144/https://console.cloud.google.com/). When you upload your app, `appcfg` gets the application ID from this file.

The `<version>` element contains the version identifier for the latest version of the app's code. The version identifier can contain lowercase letters, digits, and hyphens. It cannot begin with the prefix "ah-" and the names "default" and "latest" are reserved and cannot be used.

**Note:** Version names should begin with a letter, to distinguish them from numeric instances which are always specified by a number. This avoids the ambiguity with URLs like `123.my-module.appspot.com`, which can be interpreted two ways: If version “123” exists, the target will be version “123” of the given module. If that version does not exist, the target will be instance number 123 of the default version of the module.

`appcfg` uses this version identifier when it uploads the application, telling App Engine to either create a new version of the app with the given identifier, or replace the version of the app with the given identifier if one already exists. You can test new versions of your app with a URL using "-dot-" as a subdomain separator in the URL, e.g., `https://_version_id_-dot- _your_app_id_.appspot.com`. Using the [Google Cloud Platform Console](https://web.archive.org/web/20160424230144/https://console.cloud.google.com/), you can select the default version of your app. The default version is loaded when no version, or an invalid version, is specified.

The `<threadsafe>` element defines whether App Engine can send multiple requests at the same time to a given web server. It is further described in the section [Using concurrent requests](#Java_appengine_web_xml_Using_concurrent_requests).

The `<static-files>` and `<resource-files>` elements are described in the next section.

You can find the DTD and schema specifications for this file in the SDK's `docs/` directory.

**Note:** The standard `appengine-web.xml` file defines the default module. You will need to specify additional configuration parameters for each non-default module. These are described in the [Modules](https://web.archive.org/web/20160424230144/https://cloud.google.com/appengine/docs/java/modules/#Java_Configuration) documentation. You may also apply these configuration parameters to the default module.

## Scaling and instance types

### Manual Scaling

```
<appengine-web-app xmlns="http://appengine.google.com/ns/1.0">
  <application>simple-app</application>
  <module>default</module>
  <version>uno</version>
  <threadsafe>true</threadsafe>
  <instance-class>B8</instance-class>
  <manual-scaling>
    <instances>5</instances>
  </manual-scaling>
</appengine-web-app>
```

###### `instance-class:`

The instance class size for this module. When using manual scaling, the B1, B2, B4, B4\_1G, and B8 instance classes are available. If you do not specify a class, B2 is assigned by default.

###### `manual-scaling:`

Required to enable manual scaling for a module.

###### `instances:`

The number of instances to assign to the module at the start. This number can later be altered by using the Modules API `setNumInstances()` function.

### Basic Scaling

```
<appengine-web-app xmlns="http://appengine.google.com/ns/1.0">
  <application>simple-app</application>
  <module>default</module>
  <version>uno</version>
  <threadsafe>true</threadsafe>
  <instance-class>B8</instance-class>
  <basic-scaling>
    <max-instances>11</max-instances>
    <idle-timeout>10m</idle-timeout>
  </basic-scaling>
</appengine-web-app>
```

###### `instance-class:`

The instance class size for this module. When using basic scaling, the B1, B2, B4, B4\_1G, and B8 instance classes are available. If you do not specify a class, B2 is assigned by default.

###### `basic-scaling:`

Required to enable basic scaling for a module.

###### `max-instances:`

Required. The maximum number of instances for App Engine to create for this module version. This is useful to limit the costs of a module.

###### `idle-timeout:`

Optional. The instance will be shut down this amount of time after receiving its last request. The default is 5 minutes.

Note: Basic scaling is not currently available in the Java Development Server.

### Automatic Scaling

```
<appengine-web-app xmlns="http://appengine.google.com/ns/1.0">
  <application>simple-app</application>
  <module>default</module>
  <version>uno</version>
  <threadsafe>true</threadsafe>
  <instance-class>F2</instance-class>
  <automatic-scaling>
    <min-idle-instances>5</min-idle-instances>
    <!-- automatic is the default value. -->
    <max-idle-instances>automatic</max-idle-instances>
    <!-- automatic is the default value. -->
    <min-pending-latency>30ms</min-pending-latency>
    <max-pending-latency>automatic</max-pending-latency>
    <max-concurrent-requests>50</max-concurrent-requests>
  </automatic-scaling>
</appengine-web-app>
```

###### `instance-class:`

The Instance Class size for this module. When using automatic scaling, only the F1, F2, F4, and F4\_1G instance classes are available. If you do not specify a class, F1 is assigned by default.

###### `automatic-scaling:`

Optional. Automatic scaling is assumed by default.

###### `min-idle-instances:`

The minimum number of idle instances that App Engine should maintain for this version. Only applies to the default version of a module, since other versions are not expected to receive significant traffic. Please keep in mind:

-   A low minimum helps keep your running costs down during idle periods, but means that fewer instances may be immediately available to respond to a sudden load spike.

-   <span id="minimum">A high minimum allows you to prime the application for rapid spikes in request load. App Engine keeps the minimum number of “resident instances” running at all times, so an instance is always available to serve an incoming request. You are charged for resident instances, whether or not they are handling requests. For resident instances to function properly, you must be sure that [warmup requests](https://web.archive.org/web/20160424230144/https://cloud.google.com/appengine/docs/java/config/appconfig#Java_Warmup_requests) are enabled and your application handles warmup requests. The **Availability** column of the [Cloud Platform Console](https://web.archive.org/web/20160424230144/https://console.cloud.google.com/) **Instance** page indicates whether an instance is resident or dynamic.</span>

    If you set a minimum number of idle instances, pending latency will have less effect on your application's performance. Because App Engine keeps idle instances in reserve, it is unlikely that requests will enter the pending queue except in exceptionally high load spikes. You will need to test your application and expected traffic volume to determine the ideal number of instances to keep in reserve.

###### `max-idle-instances:`

The maximum number of idle instances that App Engine should maintain for this version. Please keep in mind:

-   A high maximum reduces the number of idle instances more gradually when load levels return to normal after a spike. This helps your application maintain steady performance through fluctuations in request load, but also raises the number of idle instances (and consequent running costs) during such periods of heavy load.
-   A low maximum keeps running costs lower, but can degrade performance in the face of volatile load levels.

**Note:** When settling back to normal levels after a load spike, the number of idle instances may temporarily exceed your specified maximum. However, you will not be charged for more instances than the maximum number you've specified.

###### `min-pending-latency:`

The minimum amount of time that App Engine should allow a request to wait in the pending queue before starting a new instance to handle it.

-   A low minimum means requests must spend less time in the pending queue when all existing instances are active. This improves performance but increases the cost of running your application.
-   A high minimum means requests will remain pending longer if all existing instances are active. This lowers running costs but increases the time users must wait for their requests to be served.

###### `max-pending-latency:`

The maximum amount of time that App Engine should allow a request to wait in the pending queue before starting a new instance to handle it.

-   A low maximum means App Engine will start new instances sooner for pending requests, improving performance but raising running costs.
-   A high maximum means users may wait longer for their requests to be served (if there are pending requests and no idle instances to serve them), but your application will cost less to run.

###### `max-concurrent-requests:`

Optional. The number of concurrent requests an automatic scaling instance can accept before the scheduler spawns a new instance (Default: 8, Maximum: 80). You may experience increased API latency if this setting is too high. Note that the scheduler may spawn a new instance before the actual maximum number of requests is reached.

## Static files and resource files

Many web applications have files that are served directly to the user's browser, such as images, CSS style sheets, or browser JavaScript code. These are known as *static files* because they do not change, and can benefit from web servers dedicated just to static content. App Engine serves static files from dedicated servers and caches that are separate from the application servers.

Files that are accessible by the application code using the filesystem are called *resource files*. These files are stored on the application servers with the app.

By default, all files in the WAR are treated as both static files and resource files, except for JSP files, which are compiled into servlet classes and mapped to URL paths, and files in the `WEB-INF/` directory, which are never served as static files and always available to the app as resource files.

You can adjust which files are considered static files and which are considered resource files using elements in the `appengine-web.xml` file. The `<static-files>` element specifies patterns that match file paths to include and exclude from the list of static files, overriding or amending the default behavior. Similarly, the `<resource-files>` element specifies which files are considered resource files.

**Note:** App Engine resource files are read using `java.io.File` or `javax.servlet.ServletContext.getResource/getResourceAsStream`. They are not accessible via `Class.getResourceAsStream()`

### Including and excluding files

Path patterns are specified using zero or more `<include>` and `<exclude>` elements. In a pattern, `'*'` represents zero or more of any character in a file or directory name, and `**` represents zero or more directories in a path. Files and directories matching `<exclude>` patterns will not be uploaded when you deploy your app to App Engine. However, these files and directories will still be accessible to your application when running on the local Development Server.

An `<include>` element overrides the default behavior of including all files. An `<exclude>` element applies after all `<include>` patterns (as well as the default if no explicit `<include>` is provided).

The following example demonstrates how to designate all `.png` files as static files (except those in the `data/` directory and all of its subdirectories):

```
<static-files>
  <include path="/**.png" />
  <exclude path="/data/**.png" />
```

Similarly, the following sample demonstrates how to designate all `.xml` files as resource files (except those in the `feeds/` directory and all of its subdirectories):

```
<resource-files>
  <include path="/**.xml" />
  <exclude path="/feeds/**.xml" />
```

You can also set HTTP headers to use when responding to requests to these resources.

```
<static-files>
  <include path="/my_static-files" >
    <http-header name="Access-Control-Allow-Origin" value="http://example.org" />
  </include>
</static-files>
```
**Note:** If the `path` string doesn't start with a slash, then the HTTP headers (if any) work on App Engine but do **not** work on the Development Server.

### Changing the MIME type for static files

By default, static files are served using a MIME type selected based on the filename extension. You can associate custom MIME types with filename extensions for static files in [`web.xml`](https://web.archive.org/web/20160424230144/https://cloud.google.com/appengine/docs/java/config/webxml) using `<mime-mapping>` elements.

### Changing the root directory

The `<public-root>` is a directory in your application that contains the static files for your application. When a request for a static file is made, the `<public-root>` for your application is prepended to the request path. This gives the path of an application file containing the content that is being requested.

The default `<public-root>` is `/`.

For example, the following would map the URL path `/index.html` to the application file path `/static/index.html`:

```
<public-root>/static</public-root>
```

### Static cache expiration

Unless told otherwise, web proxies and browsers retain files they load from a website for a limited period of time.

You can configure a cache duration for specific static file handlers by providing an `expiration` attribute to `<static-files><include ... >`. The value is a string of numbers and units, separated by spaces, where units can be `d` for days, `h` for hours, `m` for minutes, and `s` for seconds. For example, `"4d 5h"` sets cache expiration to 4 days and 5 hours after the file is first requested. If the expiration time is omitted, the production server defaults to 10 minutes.

**Note:** Script handlers can set cache durations by returning the appropriate HTTP headers to the browser.

For example:

```
<static-files>
  <include path="/**.png" expiration="4d 5h" />
</static-files>
```
**Important:** The expiration time will be sent in the `Cache-Control` and `Expires` HTTP response headers, and therefore, the files are likely to be cached by the user's browser, as well as intermediate caching proxy servers such as Internet Service Providers. Once a file is transmitted with a given expiration time, there is generally **no way** to clear it out of intermediate caches, even if the user clears their own browser cache. Re-deploying a new version of the app will **not** reset any caches. Therefore, if you ever plan to modify a static file, it should have a short (less than one hour) expiration time. In most cases, the default 10-minute expiration time is appropriate.

## System properties and environment variables

The `appengine-web.xml` file can define system properties and environment variables that are set when the application is running.

```
<system-properties>
  <property name="myapp.maximum-message-length" value="140" />
  <property name="myapp.notify-every-n-signups" value="1000" />
  <property name="myapp.notify-url" value="http://www.example.com/signupnotify" />
</system-properties>

<env-variables>
  <env-var name="DEFAULT_ENCODING" value="UTF-8" />
</env-variables>
```

To avoid conflicts with your local environment, the development server does not set environment variables based on this file, and requires that the local environment have these variables already set to matching values. (This does not apply to system properties.)

```
export DEFAULT_ENCODING="UTF-8"
dev_appserver war
```

When running on App Engine, the environment is created with these variables already set.

## Configuring Secure URLs (SSL)

By default, any user can access any URL using either HTTP or HTTPS. You can configure an app to require HTTPS for certain URLs in the deployment descriptor. See [Deployment Descriptor: Secure URLs](https://web.archive.org/web/20160424230144/https://cloud.google.com/appengine/docs/java/config/webxml#Secure_URLs).

If you want to disallow the use of HTTPS for the application, put the following in the `appengine-web.xml` file:

```
<ssl-enabled>false</ssl-enabled>
```

There is no way to disallow HTTPS for some URL paths and not others in the Java runtime environment.

## Enabling sessions

App Engine includes an implementation of sessions, using [the servlet session interface](https://web.archive.org/web/20160424230144/https://docs.oracle.com/javaee/7/api/javax/servlet/http/HttpSession.html). The implementation stores session data in the App Engine datastore for persistence, and also uses memcache for speed. As with most other servlet containers, the session attributes that are set with `session.setAttribute()` during the request are persisted at the end of the request.

This feature is off by default. To turn it on, add the following to `appengine-web.xml`:

```
<sessions-enabled>true</sessions-enabled>
```

The implementation creates datastore entities of the kind `_ah_SESSION`, and memcache entries using keys with a prefix of `_ahs`.

**Note:** Because App Engine stores session data in the datastore and memcache, all values stored in the session must implement the `java.io.Serializable` interface.

It's possible to reduce request latency by configuring your application to asynchronously write HTTP session data to the datastore:
```
<async-session-persistence enabled="true" />
```

With async session persistence turned on, App Engine will submit a Task Queue task to write session data to the datastore before writing the data to memcache. By default the task will be submitted to the `default` queue. If you'd like to use a different queue, add the `queue-name` attribute:

```
<async-session-persistence enabled="true" queue-name="myqueue"/>
```
**Note:** Note, session data is always written synchronously to memcache. If a request tries to read the session data when memcache is not available (or the session data has been flushed), it will fail over to the datastore, which may not yet have the most recent session data. This means that asynchronous session persistence may cause your application to see stale session data. However, for most applications the latency benefit far outweighs the risk.

## Reserved URLs

All URLs that begin with `/_ah/` are reserved by App Engine for features or administrative purposes. Some URLs are routed to App Engine feature handlers, while others are called by App Engine for special purposes and are expected to be mapped to request handlers in your app (for example, `/_ah/warmup` for [Warm-up Requests](#Java_Warmup_requests)).

## Inbound services

Before an application can receive email or XMPP messages, the application must be configured to enable the service. You enable the service for a Java app by including an `<inbound-services>` section in the `appengine-web.xml` file. The following inbound services are available:

-   `channel_presence` - registers your application for notifications when a client [connects or disconnects from a channel](https://web.archive.org/web/20160424230144/https://cloud.google.com/appengine/docs/java/channel/#Java_Tracking_client_connections_and_disconnections).
-   `mail` - allows your application to [receive mail](https://web.archive.org/web/20160424230144/https://cloud.google.com/appengine/docs/java/mail/)
-   `xmpp_message` - allows your application to [receive instant messages](https://web.archive.org/web/20160424230144/https://cloud.google.com/appengine/docs/java/xmpp/#Java_Handling_incoming_calls)
-   `xmpp_presence` - allows your application to receive a user's chat presence
-   `xmpp_subscribe` - allows your application to receive user subscription POSTs.
-   `warmup` - enables [warmup requests](#Java_Warmup_requests)

For example, you can enable mail and warmup by specifying the following in `appengine-web.xml`:

```
<inbound-services>
  <service>mail</service>
  <service>warmup</service>
</inbound-services>
```

## Disabling precompilation

App Engine uses a "precompilation" process with the Java bytecode of an app to enhance the performance of the app in the Java runtime environment. Precompiled code functions identically to the original bytecode.

If for some reason you prefer that your app not use precompilation, you can turn it off by adding the following to your `appengine-web.xml` file:

```
<precompilation-enabled>false</precompilation-enabled>
```

## Warmup requests

App Engine frequently needs to load application code into a fresh instance. This happens when you redeploy the application, when the load pattern has increased beyond the capacity of the current instances, or simply due to maintenance or repairs of the underlying infrastructure or physical hardware.

Loading new application code on a fresh instance can result in [loading requests](https://web.archive.org/web/20160424230144/https://cloud.google.com/appengine/docs/scaling#loading_requests). Loading requests can result in increased request latency for your users, but you can avoid this latency using **warmup requests**. Warmup requests load application code into a new instance before any live requests reach that instance.

App Engine attempts to detect when your application needs a new instance, and (assuming that warmup requests are enabled for your application) initiates a warmup request to initialize the new instance. However, these detection attempts do not work in every case. As a result, you may encounter loading requests, even if warmup requests are enabled in your app. For example, if your app is serving no traffic, the first request to the app will always be a loading request, not a warmup request.

Warmup requests use instance hours like any other request to your App Engine application. In most cases, you won't notice an increase in instance hours, since your application is simply initializing in a warmup request instead of a loading request. Your instance hour usage will likely increase if you decide to do more work (such as precaching) during a warmup request. If you set a minimum number of idle instances, you may encounter warmup requests when those instances first start, but they will remain available after that time.

The default warmup request causes all JAR files to be indexed in memory and initializes your application and filters. The following sections describe how to configure warmup requests and create warmup logic for your application:

-   [Configuration](#configuration)
-   [Using a `<load-on-startup>` servlet](#using_a_load-on-startup_servlet)
-   [Using a ServletContextListener](#using_a_servletcontextlistener)
-   [Using a custom warmup servlet](#using_a_custom_warmup_servlet)

#### Configuration

Warmup requests are enabled by default for Java applications configured via `appengine-web.xml`.

With warmup requests enabled, the App Engine infrastructure issues `GET` requests to `/_ah/warmup`, initializing [](#using_a_load-on-startup_servlet) servlets, [ServletContextListeners](#using_a_servletcontextlistener), and [custom warmup servlets](#using_a_custom_warmup_servlet) - which allow you to initialize your application's code as it requires. You may or may not need to implement your own handler for `/_ah/warmup` depending on which of these methods you choose.

To disable warmup requests, specify the optional `<warmup-requests-enabled>` element in `appengine-web.xml` with a value of `false`:

```
<warmup-requests-enabled>false</warmup-requests-enabled>
```

#### Using a `<load-on-startup>` servlet

The easiest way to provide warmup logic is to mark your own servlets as `<load-on-startup>` in `web.xml`. This method requires no changes to your application code, and initializes all specified servlets when your application initializes. The following example demonstrates how to load `my-servlet` on startup:

```
<servlet>
  <servlet-name>my-servlet</servlet-name>
  <servlet-class>com.company.MyServlet</servlet-class>
  <load-on-startup>1</load-on-startup>
</servlet>
```

These lines load the specified servlet class and invoke the servlet's `init()` method. The warmup request initializes the specified servlets before servicing any live requests. However, if there is no warmup request, the servlets specified in `<load-on-startup>` are registered upon the first request to a new instance, which result in a loading request. As noted earlier, App Engine may not issue a warmup request every time your application needs a new instance.

#### Using a `ServletContextListener`

If you have custom logic that you want to run before any of your servlets is invoked, the standard Java mechanism to arrange for that code to be executed is to register a [`ServletContextListener`](https://web.archive.org/web/20160424230144/http://docs.oracle.com/javaee/7/api/javax/servlet/ServletContextListener.html) in `web.xml`.

```
<listener>
  <listener-class>com.company.MyListener</listener-class>
</listener>
```

And then supply a class alongside your servlet and filter code:

```
public class MyListener implements ServletContextListener {
  public void contextInitialized(ServletContextEvent event) {
    // This will be invoked as part of a warmup request, or the first user
    // request if no warmup request was invoked.
  }
  public void contextDestroyed(ServletContextEvent event) {
    // App Engine does not currently invoke this method.
  }
}
```

The `ServletContextListener` runs during a warmup request. If there is no warmup request, it runs upon the first request to a new instance. This may result in loading requests.

#### Using a custom warmup servlet

As noted earlier, the prediction algorithm to initiate warmup requests for new instances does not work in every case. Even with this feature enabled, you may encounter loading requests. Executing expensive logic (such as precaching) during a loading request may incur additional load time when a new instance starts to run.

The custom warmup servlet invokes the servlet's `service` method *only* during a warmup request. By placing expensive logic in a custom warmup servlet, you can avoid increased load times on loading requests.

To create a custom warmup servlet, simply override the built-in servlet definition for `_ah_warmup` in `web.xml`:

```
<servlet>
  <servlet-name>_ah_warmup</servlet-name>
  <servlet-class>com.company.MyWarmupServlet</servlet-class>
```

## Custom error responses

When certain errors occur, App Engine serves a generic error page. You can configure your app to serve a custom static file instead of these generic error pages, so long as the custom error data is less than 10 kilobytes. You can set up different static files to be served for each supported error code by specifying the files in your app's `appengine-web.xml` file. To serve custom error pages, add a `<static-error-handlers>` section to your `appengine-web.xml`, as in this example:

```
<static-error-handlers>
  <handler file="default_error.html" />
  <handler file="over_quota.html" error-code="over_quota" />
</static-error-handlers>
```
**Warning:** Make sure that the path to the error response file does not overlap with static file handler paths.

Each `file` entry indicates a static file that should be served in place of the generic error response. The `error-code` indicates which error code should cause the associated file to be served. Supported error codes are as follows:

-   `over_quota`, which indicates the app has [exceeded a resource quota](https://web.archive.org/web/20160424230144/https://cloud.google.com/appengine/docs/quotas);
-   `dos_api_denial`, which is served to any client blocked by your app's [DoS Protection configuration](https://web.archive.org/web/20160424230144/https://cloud.google.com/appengine/docs/java/config/dos);
-   `timeout`, served if a deadline is reached before there is a response from your app.

The `error-code` is optional; if it's not specified, the given file is the default error response for your app.

You can optionally specify a `mime-type` to use when serving the custom error. See [http://www.iana.org/assignments/media-types/](https://web.archive.org/web/20160424230144/http://www.iana.org/assignments/media-types/) for a complete list of MIME types.

## Using concurrent requests

When the `threadsafe` element in `appengine-web.xml` is `false`, App Engine sends requests serially to a given web server. When the value is `true`, App Engine can send multiple requests in parallel:

```
<threadsafe>true</threadsafe>
```
**Note:** If you wish to use concurrent requests, your application code needs to use proper thread synchronization before you enable `threadsafe`.

## Auto ID policy

If you are [setting entity identifiers automatically](https://web.archive.org/web/20160424230144/https://cloud.google.com/appengine/docs/java/datastore/entities#Java_Assigning_identifiers), you can change the method employed by setting the auto ID policy. You can choose default (default is also applied if you specify nothing) or legacy. Please note however that the legacy option will be deprecated in a future release and will eventually be removed. For more information, see [our blog post](https://web.archive.org/web/20160424230144/http://googlecloudplatform.blogspot.com/2013/05/update-on-datastore-auto-ids.html) where we announced the change.

```
<auto-id-policy>
  default
</auto-id-policy>
```

## URLFetch timeout

You can set a deadline for each [URLFetch](https://web.archive.org/web/20160424230144/https://cloud.google.com/appengine/docs/java/urlfetch/#Java_Making_requests) request. By default, the deadline for a fetch is 5 seconds. You can change this default by including the following setting in your `appengine-web.xml` configuration file. Specify the timeout in seconds:

```
<system-properties>
    <property name="appengine.api.urlfetch.defaultDeadline" value="10"/>
</system-properties>
```
