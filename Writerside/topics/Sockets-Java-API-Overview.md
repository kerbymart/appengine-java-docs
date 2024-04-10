# Sockets Java API Overview

  

[Python](https://web.archive.org/web/20160424230604/https://cloud.google.com/appengine/docs/python/sockets/ "View this page in the Python runtime") <span style="color: #ddd; padding: 0em .5em;">|</span><span style="color: black; font-weight:bold">Java</span> <span style="color: #ddd; padding: 0em .5em;">|</span>[PHP](https://web.archive.org/web/20160424230604/https://cloud.google.com/appengine/docs/php/sockets/ "View this page in the PHP runtime") <span style="color: #ddd; padding: 0em .5em;">|</span>[Go](https://web.archive.org/web/20160424230604/https://cloud.google.com/appengine/docs/go/sockets/ "View this page in the Go runtime")

[<img src="https://web.archive.org/web/20160424230604im_/https://cloud.google.com/cloud/images/stack_overflow_questions.png" title="Stack Overflow Questions" data-border="0" width="240" height="53" />](https://web.archive.org/web/20160424230604/http://stackoverflow.com/questions/tagged/google-app-engine+sockets)

[](https://web.archive.org/web/20160424230604/http://stackoverflow.com/feeds/tag?sort=votes&tagnames=google-app-engine%2Bsockets)

<span class="link gc-analytics-event" category="Sidebar" data-action="Stack Overflow question click"></span>

<a href="https://web.archive.org/web/20160424230604/http://stackoverflow.com/questions/tagged/google-app-engine+sockets?sort=votes" class="gc-analytics-event" data-category="Sidebar" data-action="Stack Overflow See more link">See more...</a>

**Beta**

This is a Beta release of the Sockets API. This API is not covered by any SLA or deprecation policy and may be subject to backward-incompatible changes.

  
Sockets are only available for paid apps, and traffic from sockets is billed as [outgoing bandwidth](https://web.archive.org/web/20160424230604/https://cloud.google.com/appengine/docs/pricing#Billable_Resource_Unit_Costs). Sockets are also limited by daily and [per minute (burst) quotas](https://web.archive.org/web/20160424230604/https://cloud.google.com/appengine/docs/quotas?hl=en#Sockets). App Engine supports regular outbound Java sockets such as [java.net.Socket](https://web.archive.org/web/20160424230604/http://docs.oracle.com/javase/7/docs/api/java/net/Socket.html) and [java.net.DatagramSocket](https://web.archive.org/web/20160424230604/http://docs.oracle.com/javase/7/docs/api/java/net/DatagramSocket.html).

There is currently no support for sockets via [java.nio.SocketChannel](https://web.archive.org/web/20160424230604/http://docs.oracle.com/javase/7/docs/api/java/nio/channels/SocketChannel.html) or other `java.nio` classes. The Sockets API allows client code to call get/set options against sockets. (Previously, calls raised Not Implemented exceptions.)

The currently supported options are:

-   `SO_KEEPALIVE`
-   `SO_DEBUG`
-   `TCP_NODELAY`
-   `SO_LINGER`
-   `SO_OOBINLINE`
-   `SO_SNDBUF`
-   `SO_RCVBUF`
-   `SO_REUSEADDR`

## Limitations and restrictions

Although App Engine supports sockets, there are certain limitations and behaviors you need to be aware of when using sockets:

-   Sockets are available only for paid apps.

-   You cannot create a listen socket; you can only create outbound sockets.

-   FTP is not supported.

-   [`java.net.URL`](https://web.archive.org/web/20160424230604/http://docs.oracle.com/javase/7/docs/api/java/net/URL.html) is still configured to use the [URL Fetch API](https://web.archive.org/web/20160424230604/https://cloud.google.com/appengine/docs/java/urlfetch/); there is currently no way around this.

-   [InetAddress.isReachable](https://web.archive.org/web/20160424230604/https://docs.oracle.com/javase/1.5.0/docs/api/java/net/InetAddress.html) is a no-op.

-   Most classes in [`javax.net.ssl`](https://web.archive.org/web/20160424230604/http://docs.oracle.com/javase/7/docs/api/javax/net/ssl/package-summary.html) are supported.

-   You can only use TCP or UDP; arbitrary protocols are not allowed.

-   You cannot bind to specific IP addresses or ports.

-   Port 25 (SMTP) is blocked; you can still use authenticated SMTP on the submission port 587.

-   Private, broadcast, multicast, and Google IP ranges (except those whitelisted below), are blocked:

    -   Google Public DNS: `8.8.8.8`, `8.8.4.4`, `2001:4860:4860::8888`, `2001:4860:4860::8844` port 53
    -   Gmail SMTPS: `smtp.gmail.com` port 465 and 587
    -   Gmail POP3S: `pop.gmail.com` port 995
    -   Gmail IMAPS: `imap.gmail.com` port 993

    **Note:** Google Compute Engine IP addresses are not considered to be in Google IP ranges. You can use sockets to connect Google App Engine apps to Google Compute Engine instances.

-   Socket descriptors are associated with the App Engine app that created them and are non-transferable (cannot be used by other apps).

-   Sockets may be reclaimed after 2 minutes of inactivity; any socket operation keeps the socket alive for a further 2 minutes.

-   You cannot `Select` between multiple available sockets because that requires `java.nio.SocketChannel` which is not currently supported.)

## Using sockets with the development server

You can run and test code using sockets on the development server, without using any special command line parameters.

## App Engine sample using sockets

For a sample using sockets, see the [socket demo app](https://web.archive.org/web/20160424230604/https://github.com/GoogleCloudPlatform/appengine-sockets-python-java-go) in the Google Cloud Platform GitHub.
