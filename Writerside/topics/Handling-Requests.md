# Handling Requests

  

[Python](https://web.archive.org/web/20160424230611/https://cloud.google.com/appengine/docs/python/requests "View this page in the Python runtime") <span style="color: #ddd; padding: 0em .5em;">|</span><span style="color: black; font-weight:bold">Java</span> <span style="color: #ddd; padding: 0em .5em;">|</span>[PHP](https://web.archive.org/web/20160424230611/https://cloud.google.com/appengine/docs/php/requests "View this page in the PHP runtime") <span style="color: #ddd; padding: 0em .5em;">|</span>[Go](https://web.archive.org/web/20160424230611/https://cloud.google.com/appengine/docs/go/requests "View this page in the Go runtime")

1.  [Requests and domains](#Java_Requests_and_domains)
2.  [Requests and servlets](#Java_Requests_and_servlets)
3.  [Request headers](#Java_Request_headers)
4.  [Responses](#Java_Responses)
5.  [The request timer](#Java_The_request_timer)
6.  [SPDY](#Java_SPDY)
7.  [Logging](#Java_Logging)
8.  [The environment](#Java_The_environment)
9.  [Quotas and limits](#Java_Quotas_and_limits)

## Requests and domains

App Engine determines that an incoming request is intended for your application using the domain name of the request. A request whose domain name is `http://your_app_id.appspot.com` is routed to the application whose ID is `your_app_id`. Every application gets an `appspot.com` domain name for free.

`appspot.com` domains also support subdomains of the form `subdomain-dot-your_app_id.appspot.com`, where `subdomain` can be any string allowed in one part of a domain name (not `.`). Requests sent to any subdomain in this way are routed to your application.

You can set up a custom top-level domain using Google Apps. With Google Apps, you assign subdomains of your business's domain to various applications, such as Google Mail or Sites. You can also associate an App Engine application with a subdomain. For convenience, you can set up a Google Apps domain when you register your application ID, or later from the Cloud Platform Console. See [Deploying your Application on your Google Apps URL](https://web.archive.org/web/20160424230611/https://cloud.google.com/appengine/articles/domains) for more information.

Requests for these URLs all go to the version of your application that you have selected as the default version in the Cloud Platform Console. Each version of your application also has its own URL, so you can deploy and test a new version before making it the default version. The version-specific URL uses the version identifier from your app's configuration file in addition to the `appspot.com` domain name, in this pattern: `http://version_id-dot-latest-dot-your_app_id.appspot.com` You can also use subdomains with the version-specific URL: `http://subdomain-dot-version_id-dot-latest-dot-your_app_id.appspot.com`

The domain name used for the request is included in the request data passed to the application. If you want your app to respond differently depending on the domain name used to access it (such as to restrict access to certain domains, or redirect to an official domain), you can check the request data (such as the `Host` request header) for the domain from within the application code and respond accordingly.

If your app uses [modules](https://web.archive.org/web/20160424230611/https://cloud.google.com/appengine/docs/java/modules/), you can address requests to a specific module and optionally a specific version of that module. For more information about module addressability, please see [Routing Requests to Modules](https://web.archive.org/web/20160424230611/https://cloud.google.com/appengine/docs/java/modules/routing).

**Note:** After April 2013 Google does not issue SSL certificates for double-wildcard domains hosted at `appspot.com` (i.e. `*.*.appspot.com`). If you rely on such URLs for HTTPS access to your application, change any application logic to use "`-dot-`" instead of "`.`". For example, to access version v1 of application myapp use `https://v1-dot-myapp.appspot.com`. The certificate will not match if you use `https://v1.myapp.appspot.com`, and an error occurs for any User-Agent that expects the URL and certificate to match exactly.

## Requests and servlets

When App Engine receives a web request for your application, it invokes the servlet that corresponds to the URL, as described in the application's [deployment descriptor](https://web.archive.org/web/20160424230611/https://cloud.google.com/appengine/docs/java/config/webxml) (the `web.xml` file in the `WEB-INF/` directory). It uses the [Java Servlet API](https://web.archive.org/web/20160424230611/http://java.sun.com/products/servlet/), version 2.5, to provide the request data to the servlet and accept the response data .

App Engine runs multiple instances of your application, each instance has its own web server for handling requests. Any request can be routed to any instance, so consecutive requests from the same user are not necessarily sent to the same instance. The number of instances can be adjusted automatically as traffic changes.

By default, each web server processes only one request at a time. If you mark your application as thread-safe, App Engine may dispatch multiple requests to each web server in parallel. To do so, simply add a `<threadsafe>true</threadsafe>` element to `appengine-web.xml` as described in [Using Concurrent Requests](https://web.archive.org/web/20160424230611/https://cloud.google.com/appengine/docs/java/config/appconfig#Java_appengine_web_xml_Using_concurrent_requests).

The following example servlet class displays a simple message on the user's browser.

```
import java.io.IOException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class MyServlet extends HttpServlet {
    @Override
    public void doGet(HttpServletRequest req, HttpServletResponse resp)
            throws IOException {
        resp.setContentType("text/plain");
        resp.getWriter().println("Hello, world");
    }
}
```

## Request headers

An incoming HTTP request includes the HTTP headers sent by the client. For security purposes, some headers are sanitized or amended by intermediate proxies before they reach the application.

The following headers are removed from the request:

-   `Accept-Encoding`
-   `Connection`
-   `Keep-Alive`
-   `Proxy-Authorization`
-   `TE`
-   `Trailer`
-   `Transfer-Encoding`

In addition, the header `Strict-Transport-Security` is removed from requests served to any domains other than `appspot.com` or `*.appspot.com`.

These headers relate to the transfer of the HTTP data between the client and server, and are transparent to the application. For example, the server may automatically send a gzipped response, depending on the value of the `Accept-Encoding` request header. The application itself does not need to know which content encodings the client can accept.

**Note:** Entity headers (headers relating to the request body) are not sanitized or checked, so applications should not rely on them. In particular, the `Content-MD5` request header is sent unmodified to the application, so may not match the MD5 hash of the content. Also, the `Content-Encoding` request header is not checked by the server, so if the client sends a gzipped request body, it will be sent in compressed form to the application.

### App Engine specific headers

As a service to the app, App Engine adds the following headers to all requests:

###### `X-AppEngine-Country`

Country from which the request originated, as an [ISO 3166-1 alpha-2](https://web.archive.org/web/20160424230611/https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2) country code. App Engine determines this code from the client's IP address. Note that the country information is not derived from the WHOIS database; it's possible that an IP address with country information in the WHOIS database will not have country information in the `X-AppEngine-Country` header. Your application should handle the special country code `ZZ` (unknown country).

###### `X-AppEngine-Region`

Name of region from which the request originated. This value only makes sense in the context of the country in `X-AppEngine-Country`. For example, if the country is "US" and the region is "ca", that "ca" means "California", not Canada. The complete list of valid region values is found in the [ISO-3166-2](https://web.archive.org/web/20160424230611/https://en.wikipedia.org/wiki/ISO_3166-2) standard.

###### `X-AppEngine-City`

Name of the city from which the request originated. For example, a request from the city of Mountain View might have the header value `mountain view`. There is no canonical list of valid values for this header.

###### `X-AppEngine-CityLatLong`

Latitude and longitude of the city from which the request originated. This string might look like "37.386051,-122.083851" for a request from Mountain View.

App Engine services may add additional request headers:

[The Task Queue service](https://web.archive.org/web/20160424230611/https://cloud.google.com/appengine/docs/java/taskqueue/) adds [additional headers](https://web.archive.org/web/20160424230611/https://cloud.google.com/appengine/docs/java/taskqueue/overview-push#task_request_headers) to requests from that provide details about the task in the request, and the queue it is associated with.

Requests from the Cron Service will also contain a HTTP header:

`X-AppEngine-Cron: true`

See [Securing URLs for cron](https://web.archive.org/web/20160424230611/https://cloud.google.com/appengine/docs/java/config/cron#Java_appengine_web_xml_Securing_URLs_for_cron) for more details.

Requests coming from other App Engine applications will include a header identifying the app making the request:

`X-Appengine-Inbound-Appid`

See the [App Identity documentation](https://web.archive.org/web/20160424230611/https://cloud.google.com/appengine/docs/java/appidentity/) for more details.

## Responses

App Engine calls the servlet with a request object and a response object, then waits for the servlet to populate the response object and return. When the servlet returns, the data on the response object is sent to the user.

As explained below, there are limits that apply to the response you generate, and the response may be modified before it is returned to the client.

**Note:** This HTTP header documentation only applies to responses to inbound HTTP requests. For HTTP headers related to outbound requests originated by your App Engine code, see the [header documentation for URLFetch](https://web.archive.org/web/20160424230611/https://cloud.google.com/appengine/docs/java/urlfetch/#Java_Request_headers).

### Response size limits

Dynamic responses are limited to 32MB. If a script handler generates a response larger than this limit, the server sends back an empty response with a 500 Internal Server Error status code. This limitation does not apply to responses that serve data from the Blobstore or [Google Cloud Storage](https://web.archive.org/web/20160424230611/https://cloud.google.com/appengine/docs/java/googlestorage) .

### Streaming Responses

App Engine does not support streaming responses where data is sent in incremental chunks to the client while a request is being processed. All data from your code is collected as described above and sent as a single HTTP response.

### Response compression

If the client sends HTTP headers with the original request indicating that the client can accept compressed (gzipped) content, App Engine compresses the handler response data automatically and attaches the appropriate response headers. It uses both the `Accept-Encoding` and `User-Agent` request headers to determine if the client can reliably receive compressed responses.

Custom clients can indicate that they are able to receive compressed responses by specifying both `Accept-Encoding` and `User-Agent` headers with a value of `gzip`. The `Content-Type` of the response is also used to determine whether compression is appropriate; in general, text-based content types are compressed, whereas binary content types are not.

**Note:** In the absence of a `Content-Type` header provided by your application, a default `Content-Type: text/html` header will be set, but the automatic gzip compression will not be applied.

When responses are compressed automatically by App Engine, the `Content-Encoding` header is added to the response.

### Headers removed

The following headers are ignored and removed from the response:

-   `Connection`
-   `Content-Encoding`\*
-   `Content-Length`
-   `Date`
-   `Keep-Alive`
-   `Proxy-Authenticate`
-   `Server`
-   `Trailer`
-   `Transfer-Encoding`
-   `Upgrade`

\* May be re-added if the response is compressed by App Engine.

In addition, the header `Strict-Transport-Security` is removed from responses served from any domains other than `*.appspot.com`.

Headers with non-ASCII characters in either the name or value are also removed.

### Headers added or replaced

The following headers are added or replaced in the response:

###### `Cache-Control`, `Expires` and `Vary`

These headers specify caching policy to intermediate web proxies (such as Internet Service Providers) and browsers. If your script sets these headers, they will usually be unmodified, unless the response has a Set-Cookie header, or is generated for a user who is signed in using an administrator account. Static handlers will set these headers as directed by the [configuration file](https://web.archive.org/web/20160424230611/https://cloud.google.com/appengine/docs/java/config/appconfig#Java_appengine_web_xml_Setting_the_cache_expiration). If you do not specify a `Cache-Control`, the server may set it to `private`, and add a `Vary: Accept-Encoding` header.

If you have a Set-Cookie response header, the `Cache-Control` header will be set to `private` (if it is not already more restrictive) and the `Expires` header will be set to the current date (if it is not already in the past). Generally, this will allow browsers to cache the response, but not intermediate proxy servers. This is for security reasons, since if the response was cached publicly, another user could subsequently request the same resource, and retrieve the first user's cookie.

###### `Content-Encoding`

Depending upon the request headers and response `Content-Type`, the server may automatically compress the response body, as described above. In this case, it adds a `Content-Encoding: gzip` header to indicate that the body is compressed. See the section on [response compression](#response_compression) for more detail.

###### `Content-Length` or `Transfer-Encoding`

The server always ignores the `Content-Length` header returned by the application. It will either set `Content-Length` to the length of the body (after compression, if compression is applied), or delete `Content-Length`, and use chunked transfer encoding (adding a `Transfer-Encoding: chunked` header).

###### `Content-Type`

If not specified by the application, the server will set a default `Content-Type: text/html` header.

###### `Date`

Set to the current date and time.

###### `Server`

Set to `Google Frontend`.

If you access your site while signed in using an administrator account, App Engine includes per-request statistics in the response headers:

###### `X-AppEngine-Estimated-CPM-US-Dollars`

An estimate of what 1,000 requests similar to this request would cost in US dollars.

###### `X-AppEngine-Resource-Usage`

The resources used by the request, including server-side time as a number of milliseconds.

Responses with resource usage statistics will be made uncacheable.

If the `X-AppEngine-BlobKey` header is in the application's response, it and the optional `X-AppEngine-BlobRange` header will be used to replace the body with all or part of a blobstore blob's content. If `Content-Type` is not specified by the application, it will be set to the blob's MIME type. If a range is requested, the response status will be changed to `206 Partial Content`, and a `Content-Range` header will be added. The `X-AppEngine-BlobKey` and `X-AppEngine-BlobRange` headers will be removed from the response. You do not normally need to set these headers yourself, as the `blobstore_handlers.BlobstoreDownloadHandler` class sets them. See [Serving a Blob](https://web.archive.org/web/20160424230611/https://cloud.google.com/appengine/docs/java/blobstore/#Java_Serving_a_blob) for details.

### Response headers set in the application configuration

Custom HTTP Response headers can be set per URL for dynamic and static paths in your application's configuration file. See the `http_headers` sections in the [configuration documentation](https://web.archive.org/web/20160424230611/https://cloud.google.com/appengine/docs/python/config/appconfig) for more details.

## The request timer

A request handler has a limited amount of time to generate and return a response to a request, typically around 60 seconds. Once the deadline has been reached, the request handler is interrupted.

The Java runtime environment interrupts the servlet by throwing a `com.google.apphosting.api.DeadlineExceededException`. If there is no request handler to catch this exception, the runtime environment will return an HTTP 500 server error to the client.

If there is a request handler and the `DeadlineExceededException` is caught, then the runtime environment gives the request handler time (less than a second) to prepare a custom response. If the request handler takes more than a second after raising the exception to prepare a custom response, a `HardDeadlineExceededError` will be raised.

Both `DeadlineExceededExceptions` and `HardDeadlineExceededErrors` will force termination of the request and kill the instance.

To find out how much time remains before the deadline, the application can import [`com.google.apphosting.api.ApiProxy`](https://web.archive.org/web/20160424230611/https://cloud.google.com/appengine/docs/java/javadoc/com/google/apphosting/api/ApiProxy) and call [`ApiProxy.getCurrentEnvironment().getRemainingMillis()`](https://web.archive.org/web/20160424230611/https://cloud.google.com/appengine/docs/java/javadoc/com/google/apphosting/api/ApiProxy.Environment#getRemainingMillis()). This is useful if the application is planning to start on some work that might take too long; if you know it takes five seconds to process a unit of work but `getRemainingMillis()` returns less time, there's no point starting that unit of work.

While a request can take as long as 60 seconds to respond, App Engine is optimized for applications with short-lived requests, typically those that take a few hundred milliseconds. An efficient app responds quickly for the majority of requests. An app that doesn't will not scale well with App Engine's infrastructure.

Refer to [Dealing with DeadlineExceededErrors](https://web.archive.org/web/20160424230611/https://cloud.google.com/appengine/articles/deadlineexceedederrors.html) for common DeadlineExceededError causes and suggested workarounds.

[Backends](https://web.archive.org/web/20160424230611/https://cloud.google.com/appengine/docs/java/backends) allow you to avoid this request timer; with backends, there is no time limit for generating and returning a request.

**Warning:** The `DeadlineExceededException` can potentially be raised from anywhere in your program, including `finally` blocks, so it could leave your program in an invalid state. This can cause deadlocks or unexpected errors in threaded code, because locks may not be released. After the request completes, the runtime will terminate the server process, so future requests won't be affected; however, any concurrent requests being handled on the same process will be terminated. This is equivalent to calling [`Thread.stop`](https://web.archive.org/web/20160424230611/http://docs.oracle.com/javase/6/docs/api/java/lang/Thread.html#stop(java.lang.Throwable)) on the main thread. For more information, see [Why is Thread.stop deprecated?](https://web.archive.org/web/20160424230611/http://docs.oracle.com/javase/6/docs/technotes/guides/concurrency/threadPrimitiveDeprecation.html). To be safe, you should not rely on the `DeadlineExceededException`, and instead ensure that your requests complete well before the time limit (using [`getRemainingMillis()`](https://web.archive.org/web/20160424230611/https://cloud.google.com/appengine/docs/java/javadoc/com/google/apphosting/api/ApiProxy.Environment#getRemainingMillis()) if necessary).

## SPDY

App Engine applications will automatically use the [SPDY](https://web.archive.org/web/20160424230611/http://dev.chromium.org/spdy) protocol when accessed over SSL by a browser that supports SPDY. This is a replacement for HTTP designed by Google and intended to reduce the latency of web page downloads. The use of SPDY should be entirely transparent to both applications and users (applications can be written as if normal HTTP was being used). For more information, see the [SPDY project page](https://web.archive.org/web/20160424230611/http://dev.chromium.org/spdy).

## Logging

Your application can write information to the application logs using [java.util.logging.Logger](https://web.archive.org/web/20160424230611/http://java.sun.com/javase/6/docs/api/java/util/logging/Logger.html). Log data for your application can be viewed in the Cloud Platform Console [Logs page](https://web.archive.org/web/20160424230611/https://cloud.google.com/appengine/docs/developers-console/#logs), or downloaded using [appcfg.sh request\_logs](https://web.archive.org/web/20160424230611/https://cloud.google.com/appengine/docs/java/tools/uploadinganapp#Downloading_Logs). Each request logged is assigned a [request ID](https://web.archive.org/web/20160424230611/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/log/RequestLogs#getRequestId()), a globally unique identifier based on the request's start time. The Cloud Platform Console can recognize the `Logger` class's log levels, and interactively display messages at different levels.

Everything the servlet writes to the standard output stream (`System.out`) and standard error stream (`System.err`) is captured by App Engine and recorded in the application logs. Lines written to the standard output stream are logged at the "INFO" level, and lines written to the standard error stream are logged at the "WARNING" level. Any logging framework (such as log4j) that logs to the output or error streams will work. However, for more fine-grained control of the log level display in the Cloud Platform Console, the logging framework must use a `java.util.logging` adapter.

```
import java.util.logging.Logger;
// ...

public class MyServlet extends HttpServlet {
    private static final Logger log = Logger.getLogger(MyServlet.class.getName());

    @Override
    public void doGet(HttpServletRequest req, HttpServletResponse resp)
            throws IOException {

        log.info("An informational message.");

        log.warning("A warning message.");

        log.severe("An error message.");
    }
} 
```

The App Engine Java SDK includes a template `logging.properties` file, in the `appengine-java-sdk/config/user/` directory. To use it, copy the file to your `WEB-INF/classes` directory (or elsewhere in the WAR), then the system property `java.util.logging.config.file` to `"WEB-INF/logging.properties"` (or whichever path you choose, relative to the application root). You can set system properties in the [`appengine-web.xml` file](https://web.archive.org/web/20160424230611/https://cloud.google.com/appengine/docs/java/config/appconfig), as follows:

```
<appengine-web-app xmlns="http://appengine.google.com/ns/1.0">
    ...

    <system-properties>
        <property name="java.util.logging.config.file" value="WEB-INF/logging.properties" />
    </system-properties>

</appengine-web-app>
```

The servlet logs messages using the `INFO` log level (using `log.info()`). The default log level is `WARNING`, which suppresses `INFO` messages from the output. To change the log level, edit the `logging.properties` file. See the [Guestbook Form](https://web.archive.org/web/20160424230611/https://cloud.google.com/appengine/docs/java/gettingstarted/usingjsps#guestbook_form) application for a specific example on how to set log levels.

The Google Plugin for Eclipse new project wizard creates these logging configuration files for you, and copies them to `WEB-INF/classes/` automatically. For `java.util.logging`, you must set the system property to use this file.

## The environment

All system properties and environment variables are private to your application. Setting a system property only affects your application's view of that property, and not the JVM's view.

You can set system properties and environment variables for your app in the [deployment descriptor](https://web.archive.org/web/20160424230611/https://cloud.google.com/appengine/docs/java/config/webxml).

App Engine sets several system properties that identify the runtime environment:

-   `com.google.appengine.runtime.environment` is `"Production"` when running on App Engine, and `"Development"` when running in the development server.

    In addition to using `System.getProperty()`, you can access system properties using our [type-safe API](https://web.archive.org/web/20160424230611/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/utils/SystemProperty). For example:

    ```
    if (SystemProperty.environment.value() ==
        SystemProperty.Environment.Value.Production) {
        // The app is running on App Engine...
    }
    ```

-   `com.google.appengine.runtime.version` is the version ID of the runtime environment, such as `"1.3.0"`. You can get the version by invoking the following: `String version = SystemProperty.version.get();`

-   `com.google.appengine.application.id` is the application's ID. You can get the ID by invoking the following: `String ID = SystemProperty.applicationId.get();`

-   `com.google.appengine.application.version` is the major and minor version of the currently running application module, as "X.Y". The major version number ("X") is specified in the module's appengine-web.xml file. The minor version number ("Y") is set automatically when each version of the app is uploaded to App Engine. You can get the ID by invoking the following: `String ID = SystemProperty.applicationVersion.get();`

    On the development web server, the major version returned is always the default module's version, and the minor version is always "1".

App Engine also sets the following system properties when it initializes the JVM on an app server:

-   `file.separator`
-   `path.separator`
-   `line.separator`
-   `java.version`
-   `java.vendor`
-   `java.vendor.url`
-   `java.class.version`
-   `java.specification.version`
-   `java.specification.vendor`
-   `java.specification.name`
-   `java.vm.vendor`
-   `java.vm.name`
-   `java.vm.specification.version`
-   `java.vm.specification.vendor`
-   `java.vm.specification.name`
-   `user.dir`

### Instance IDs

You can retrieve the ID of the instance handling a request using this code:

```
com.google.apphosting.api.ApiProxy.getCurrentEnvironment().getAttributes().get("com.google.appengine.instance.id")
```

In the production environment, a logged-in admin can use the ID in a url: `http://[INSTANCE_ID].myApp.appspot.com/`. The request will be routed to that specific instance. If the instance cannot handle the request it returns an immediate 503.

### Request IDs

At the time of the request, you can save the request ID, which is unique to the request. The request ID can be used later to correlate a request with the logs for that request.

**Note:** Currently, App Engine doesn't support the use of the request ID to directly look up the related logs.

The following code shows how to get the request ID in the context of a request:

```
com.google.apphosting.api.ApiProxy.getCurrentEnvironment().getAttributes().get("com.google.appengine.runtime.request_log_id")
```

## Quotas and limits

Google App Engine automatically allocates resources to your application as traffic increases. However, this is bound by the following restrictions:

-   App Engine reserves automatic scaling capacity for applications with low latency, where the application responds to requests in less than one second. Applications with very high latency (over one second per request for many requests) and high throughput require [Silver, Gold, or Platinum support](https://web.archive.org/web/20160424230611/https://cloud.google.com/support/). Customers with this level of support can request higher throughput limits by contacting their support representative.
-   Applications that are heavily CPU-bound may also incur some additional latency in order to efficiently share resources with other applications on the same servers. Requests for static files are exempt from these latency limits.

Each incoming request to the application counts toward the **Requests** limit. Data sent in response to a request counts toward the **Outgoing Bandwidth (billable)** limit.

Both HTTP and HTTPS (secure) requests count toward the **Requests**, **Incoming Bandwidth (billable)**, and **Outgoing Bandwidth (billable)** limits. The Cloud Platform Console [Quota Details page](https://web.archive.org/web/20160424230611/https://cloud.google.com/appengine/docs/developers-console/#quotas) also reports **Secure Requests**, **Secure Incoming Bandwidth**, and **Secure Outgoing Bandwidth** as separate values for informational purposes. Only HTTPS requests count toward these values. See the [Quotas](https://web.archive.org/web/20160424230611/https://cloud.google.com/appengine/docs/quotas#Requests) page for more information.

In addition to system-wide safety limits, the following limits apply specifically to the use of request handlers:

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Limit</th>
<th>Amount</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>request size</td>
<td>32 megabytes</td>
</tr>
<tr class="even">
<td>response size</td>
<td>32 megabytes</td>
</tr>
<tr class="odd">
<td>request duration</td>
<td>60 seconds</td>
</tr>
<tr class="even">
<td>maximum total number of files (app files and static files)</td>
<td>10,000 total<br />
1,000 per directory</td>
</tr>
<tr class="odd">
<td>maximum size of an application file</td>
<td>32 megabytes</td>
</tr>
<tr class="even">
<td>maximum size of a static file</td>
<td>32 megabytes</td>
</tr>
<tr class="odd">
<td>maximum total size of all application and static files</td>
<td>first 1 gigabyte is free<br />
$ 0.026 per gigabyte per month after first 1 gigabyte</td>
</tr>
</tbody>
</table>
