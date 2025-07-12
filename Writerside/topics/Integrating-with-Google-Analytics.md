# Integrating with Google Analytics

  

[Python](https://web.archive.org/web/20160424230452/https://cloud.google.com/appengine/docs/python/google-analytics "View this page in the Python runtime") <span style="color: #ddd; padding: 0em .5em;">|</span><span style="color: black; font-weight:bold">Java</span> <span style="color: #ddd; padding: 0em .5em;">|</span>[PHP](https://web.archive.org/web/20160424230452/https://cloud.google.com/appengine/docs/php/google-analytics "View this page in the PHP runtime") <span style="color: #ddd; padding: 0em .5em;">|</span>[Go](https://web.archive.org/web/20160424230452/https://cloud.google.com/appengine/docs/go/google-analytics "View this page in the Go runtime")

The [Google Analytics Platform](https://web.archive.org/web/20160424230452/https://cloud.google.com/analytics/devguides/platform/) lets you measure user interactions with your business across various devices and environments. The platform provides all the computing resources to collect, store, process, and report on these user-interactions.

[Analytics collection](https://web.archive.org/web/20160424230452/https://cloud.google.com/analytics/devguides/collection/) can take place on both the client and server side. Google Analytics provides easy to use APIs and SDKs to send data to Google Analytics. In addition to those, we have developed code that you can use in your App Engine applications to easily send server-side analytics to Google Analytics.

1.  [Client-side analytics collection](#client-side_analytics_collection)
2.  [App Engine server-side analytics collection](#app_engine_server-side_analytics_collection)
    1.  [Sample source code](#sample_source_code)

## Client-side analytics collection

With the collection APIs and SDKs, you can measure how users interact with your content and marketing initiatives. Once implemented, you will be able to view user-interaction data within Google Analytics or through the Reporting APIs. For more details on client-side analytics collection select the link below based on the type of your client:

-   [Web Tracking (analytics.js)](https://web.archive.org/web/20160424230452/https://cloud.google.com/analytics/devguides/collection/analyticsjs) - Measure user interaction with websites or web applications.
-   [Android](https://web.archive.org/web/20160424230452/https://cloud.google.com/analytics/devguides/collection/android) - Measure user interaction with Android applications.
-   [iOS](https://web.archive.org/web/20160424230452/https://cloud.google.com/analytics/devguides/collection/ios) - Measure user interaction with iOS applications.
-   [Measurement Protocol](https://web.archive.org/web/20160424230452/https://cloud.google.com/analytics/devguides/collection/protocol/v1/) - Measure user interaction in any environment with this low-level protocol.

## App Engine server-side analytics collection

Although App Engine already provides a mechanism for [logging events](https://web.archive.org/web/20160424230452/https://cloud.google.com/appengine/articles/logging) in your application, it may be advantageous to track specific server-side events in Google Analytics. Some of the benefits are as follows:

-   **Historical data analysis** - App Engine allows you to configure the maximum number of days, or size of your log file. After that time has passed you no longer have access to those log files. Tracking events in Google Analytics provides you a much longer lifespan into the visibility of past events.
-   **Track key events** - Log files can be verbose with various components of your application writing data to them. By using event tracking you can pinpoint just the key events that you are interested in monitoring and track those along with some additional metadata.
-   **Powerful user interface** - Take advantage of the rich user interface that Google Analytics provides to visualize, report and export these server side events.

This can be accomplished easily by integrating the sample source code below into your App Engine application. For additional information on this approach consult the Google Analytics developers guide for [Event Tracking](https://web.archive.org/web/20160424230452/https://cloud.google.com/analytics/devguides/collection/protocol/v1/devguide#event).

### Sample source code

### Java

\[This section requires a browser that supports JavaScript and iframes.\]

### Python

\[This section requires a browser that supports JavaScript and iframes.\]
