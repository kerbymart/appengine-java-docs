# URL Fetch Java API Overview

  

[Python](https://web.archive.org/web/20160424225920/https://cloud.google.com/appengine/docs/python/urlfetch/ "View this page in the Python runtime") <span style="color: #ddd; padding: 0em .5em;">|</span><span style="color: black; font-weight:bold">Java</span> <span style="color: #ddd; padding: 0em .5em;">|</span>[PHP](https://web.archive.org/web/20160424225920/https://cloud.google.com/appengine/docs/php/urlfetch/ "View this page in the PHP runtime") <span style="color: #ddd; padding: 0em .5em;">|</span>[Go](https://web.archive.org/web/20160424225920/https://cloud.google.com/appengine/docs/go/urlfetch/ "View this page in the Go runtime")

[<img src="https://web.archive.org/web/20160424225920im_/https://cloud.google.com/cloud/images/stack_overflow_questions.png" title="Stack Overflow Questions" data-border="0" width="240" height="53" />](https://web.archive.org/web/20160424225920/http://stackoverflow.com/questions/tagged/google-app-engine+urlfetch)

[](https://web.archive.org/web/20160424225920/http://stackoverflow.com/feeds/tag?sort=votes&tagnames=google-app-engine%2Burlfetch)

<span class="link gc-analytics-event" category="Sidebar" data-action="Stack Overflow question click"></span>

<a href="https://web.archive.org/web/20160424225920/http://stackoverflow.com/questions/tagged/google-app-engine+urlfetch?sort=votes" class="gc-analytics-event" data-category="Sidebar" data-action="Stack Overflow See more link">See more...</a>

App Engine applications can communicate with other applications or access other resources on the web by fetching URLs. An app can use the URL Fetch service to issue HTTP and HTTPS requests and receive responses. The URL Fetch service uses Google's network infrastructure for efficiency and scaling purposes.

1.  [Fetching URLs with java.net](#Java_Fetching_URLs_with_java_net)
2.  [Making requests](#Java_Making_requests)
3.  [Secure connections and HTTPS](#Java_Secure_connections_and_HTTPS)
4.  [Request headers](#Java_Request_headers)
5.  [Responses](#Java_Responses)
6.  [URL Fetch and the development server](#Java_URL_Fetch_and_the_development_server)
7.  [Quotas and limits](#Java_Quotas_and_limits)

## Fetching URLs with java.net

You can use [`java.net.URLConnection`](https://web.archive.org/web/20160424225920/http://java.sun.com/javase/6/docs/api/java/net/URLConnection.html) and related classes from the Java standard library to make HTTP and HTTPS connections from your Java application. App Engine implements this interface using the URL Fetch service; your application does not make socket connections directly.

A simple way to get the contents of a page at a URL is to create a [`java.net.URL`](https://web.archive.org/web/20160424225920/http://java.sun.com/javase/6/docs/api/java/net/URL.html) object, then call its `openStream()` method. The method handles the details of creating the connection, issuing an HTTP GET request, and retrieving the response data.

```
import java.net.MalformedURLException;
import java.net.URL;
import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.io.IOException;

// ...
try {
    URL url = new URL("http://www.example.com/atom.xml");
    BufferedReader reader = new BufferedReader(new InputStreamReader(url.openStream()));
    String line;

    while ((line = reader.readLine()) != null) {
        // ...
    }
    reader.close();

} catch (MalformedURLException e) {
    // ...
} catch (IOException e) {
    // ...
}
```

For more sophisticated requests, you can call the URL object's `openConnection()` method to obtain a `URLConnection` object (either an `HttpURLConnection` or an `HttpsURLConnection`, depending on the URL). You can prepare this object with additional information before issuing the request. See [Using java.net](https://web.archive.org/web/20160424225920/https://cloud.google.com/appengine/docs/java/urlfetch/usingjavanet).

## Making requests

An app can fetch a URL using HTTP (normal) or HTTPS (secure). The URL specifies the scheme to use: `http://...` or `https://...`

The URL to be fetched can use any port number in the following ranges: `80`-`90`, `440`-`450`, `1024`-`65535`. If the port is not mentioned in the URL, the port is implied by the scheme: `http://...` is port `80`, `https://...` is port `443`.

The fetch can use any of the following HTTP methods: `GET` (common for requesting web pages and data), `POST` (common for submitting web forms), `PUT`, `HEAD`, and `DELETE`. The fetch can include HTTP request headers and a payload (an HTTP request body).

The URL Fetch service uses an HTTP/1.1 compliant proxy to fetch the result.

To prevent an app from causing an endless recursion of requests, a request handler is not allowed to fetch its own URL. It is still possible to cause an endless recursion with other means, so exercise caution if your app can be made to fetch requests for URLs supplied by the user.

You can set a deadline for a request, the most amount of time the service will wait for a response. By default, the deadline for a fetch is 5 seconds. The maximum deadline is 60 seconds for HTTP requests and 60 seconds for task queue and cron job requests. When using the URLConnection interface, the service uses the connection timeout (`setConnectTimeout()`) plus the read timeout (`setReadTimeout()`) as the deadline. You can adjust the default deadline for requests using the [appengine.api.urlfetch.defaultDeadline](https://web.archive.org/web/20160424225920/https://cloud.google.com/appengine/docs/java/config/appconfig#url_fetch_timeout) setting in the appengine-web.xml file.

The URL Fetch service supports both synchronous requests and asynchronous requests. With a synchronous request, the API call to fetch a URL waits until the remote host returns a result, then returns control to the application. The app can specify the maximum amount of time to wait when it makes the call. If the maximum wait time is exceeded, the call raises an exception.

An asynchronous request to the URL Fetch service starts the request, then returns immediately with an object. The application can perform other tasks while the URL is being fetched. When the application needs the results, it calls a method on the object, which waits for the request to finish if necessary, then returns the result. If any URL Fetch requests are pending when the request handler exits, the application server waits for all remaining requests to either return or reach their deadline before returning a response to the user.

In Java, the asynchronous interface is only available when using the low-level API directly. The `fetchAsync()` method returns a `java.util.concurrent.Future<HTTPResponse>`.

### Making requests to another App Engine app or Google service

If you are making requests to another App Engine app or Google service, you should consider telling the URL Fetch service to not follow redirects when invoking it. That is, your app must specify [doNotFollowRedirect](https://web.archive.org/web/20160424225920/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/urlfetch/FetchOptions#doNotFollowRedirects()) if it uses the [URLFetchService](https://web.archive.org/web/20160424225920/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/urlfetch/URLFetchService) class. If your app uses [`java.net`](https://web.archive.org/web/20160424225920/https://cloud.google.com/appengine/docs/java/urlfetch/usingjavanet), it must set the connection as follows: `connection.setInstanceFollowRedirects(false);`

**Note:** If you are making requests to another App Engine application, we recommend that you use its `appspot.com` domain name, rather than a custom domain for your app.

These settings do the following:

1.  Optimizes the network path to the backend service, potentially leading to faster calls.
2.  Adds an `X-Appengine-Inbound-Appid` header containing your app ID to the request, so the responding app can determine the source of incoming requests. For more information, see [Asserting identity to other App Engine apps](https://web.archive.org/web/20160424225920/https://cloud.google.com/appengine/docs/java/appidentity/).

## Secure connections and HTTPS

An app can fetch a URL with the HTTPS method to connect to secure servers. Request and response data are transmitted over the network in encrypted form.

In the high-level Java API, the proxy does not validate the host it is contacting. The proxy server cannot detect "man in the middle" attacks between App Engine and the remote host when using HTTPS.

However, you can use the [`FetchOptions`](https://web.archive.org/web/20160424225920/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/urlfetch/FetchOptions) class in the low-level API to enable host validation. In the near future, host validation will become the default in both low-level and the high-level APIs.

## Request headers

An app can set HTTP headers for the outgoing request.

When sending an HTTP POST request, if a `Content-Type` header is not set explicitly, the header is set to `x-www-form-urlencoded`. This is the content type used by web forms.

For security reasons, the following headers cannot be modified by the application:

-   `Content-Length`
-   `Host`
-   `Vary`
-   `Via`
-   `X-Appengine-Inbound-Appid`
-   `X-Forwarded-For`
-   `X-ProxyUser-IP`

These headers are set to accurate values by App Engine, as appropriate. For example, App Engine calculates the `Content-Length` header from the request data and adds it to the request prior to sending.

### Headers identifying request source

The following headers indicate the app ID of the requesting app:

-   `User-Agent`. This header can be modified but App Engine will append an identifier string to allow servers to identify App Engine requests. The appended string has the format `"AppEngine-Google; (+http://code.google.com/appengine; appid: APPID)"`, where `APPID` is your app's identifier.
-   `X-Appengine-Inbound-Appid`. This header cannot be modified, and is added automatically if the request is sent via the URL Fetch service when the follow redirects parameter is set to `False`.

## Responses

The URL Fetch service returns all response data, including the response code, header and body.

By default, if the URL Fetch service receives a response with a redirect code, the service will follow the redirect. The service will follow up to 5 redirect responses, then return the final resource. You can use the API to tell the URL Fetch service to not follow redirects and just return a redirect response to the application.

By default, if the incoming response exceeds the maximum response size limit, the URL fetch service raises an exception. (See [below](#Java_Quotas_and_limits) for the amount of this limit.) You can tell the API to truncate the response instead of raising an exception.

## URL Fetch and the development server

When your application is running in the development server on your computer, calls to the URL Fetch service are handled locally. The development server fetches URLs by contacting remote hosts directly from your computer, using whatever network configuration your computer is using to access the Internet.

When testing the features of your app that fetch URLs, be sure that your computer can access the remote hosts.

If your app is using the Google Secure Data Connector to access URLs on your intranet, be sure to test your app while connected to your intranet behind the firewall. Unlike App Engine, the development server does *not* use the SDC Agent to resolve intranet URLs. Only Google Apps and App Engine can authenticate with your SDC Agent.

## Quotas and limits

Each URL Fetch request counts toward the **URL Fetch API Calls** quota.

Data sent in an HTTP or HTTPS request using the URL Fetch service counts toward the following quotas:

-   **Outgoing Bandwidth (billable)**
-   **URL Fetch Data Sent**

In addition to these quotas, data sent in an HTTPS request also counts toward the following quota:

-   **Secure Outgoing Bandwidth (billable)**

Data received in response to an HTTP or HTTPS request using the URL Fetch service counts toward the following quotas:

-   **Incoming Bandwidth (billable)**
-   **URL Fetch Data Received**

In addition to these quotas, data received in response to an HTTPS request also counts toward the following quota:

-   **Secure Incoming Bandwidth (billable)**

For more information, see [Quotas](https://web.archive.org/web/20160424225920/https://cloud.google.com/appengine/docs/quotas) and [Quota details](https://web.archive.org/web/20160424225920/https://cloud.google.com/appengine/docs/developers-console/#quotas). You can see the current quota usage of your app by visiting the Google Cloud Platform Console [quota details](https://web.archive.org/web/20160424225920/https://console.cloud.google.com//project/_/appengine/quotadetails) tab for your project.

In addition to quotas, the following limits apply to the use of the URL Fetch service:

<table>
<thead>
<tr class="header">
<th>Limit</th>
<th>Amount</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Request size</td>
<td>10 megabytes</td>
</tr>
<tr class="even">
<td>Request header size</td>
<td>16 KB (Note that this limits the maximum length of the URL that can be specified in the header)</td>
</tr>
<tr class="odd">
<td>Response size</td>
<td>32 megabytes</td>
</tr>
<tr class="even">
<td>Maximum deadline (request handler)</td>
<td>60 seconds</td>
</tr>
<tr class="odd">
<td>Maximum deadline (task queue and cron job handler)</td>
<td>60 seconds</td>
</tr>
</tbody>
</table>
