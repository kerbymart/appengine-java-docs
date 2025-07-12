# Sending Email with Mailgun

  

[Python](https://web.archive.org/web/20160424230553/https://cloud.google.com/appengine/docs/python/mail/mailgun "View this page in the Python runtime") <span style="color: #ddd; padding: 0em .5em;">|</span><span style="color: black; font-weight:bold">Java</span> <span style="color: #ddd; padding: 0em .5em;">|</span><span style="color:lightgray" title="This page is not available in the PHP runtime">PHP</span> <span style="color: #ddd; padding: 0em .5em;">|</span>[Go](https://web.archive.org/web/20160424230553/https://cloud.google.com/appengine/docs/go/mail/mailgun "View this page in the Go runtime")

**Note:** Mailgun is a third-party company whose services are not covered by the [Google App Engine Service Level Agreement](https://web.archive.org/web/20160424230553/https://cloud.google.com/appengine/sla).

The Mailgun API is built on HTTP and is RESTful. It uses predictable, resource- oriented URLs and built-in HTTP capabilities for passing parameters and authentication. The Mailgun API responds with standard HTTP response codes to indicate errors, and returns JSON.

Mailgun has published <a href="https://web.archive.org/web/20160424230553/https://documentation.mailgun.com/libraries.html#libraries" target="mailgun">libraries for various languages</a>. You can use these libraries or your favorite HTTP/REST library to make HTTP calls to Mailgun.

For more code samples in other programming languages, see <a href="https://web.archive.org/web/20160424230553/https://documentation.mailgun.com/" target="mailgun">Mailgun's documentation</a>.

## Contents

1.  [Pricing](#pricing)
2.  [Setup](#setup)
3.  [Sending a plain text email](#sending_a_plain_text_email)
4.  [Sending an HTML and text email](#sending_an_html_and_text_email)
5.  [Learning more](#learning_more)

## Pricing

As a Google Cloud Platform user, your first 30,000 messages are free every month. See the monthly pricing calculator on <a href="https://web.archive.org/web/20160424230553/http://www.mailgun.com/google" target="mailgun">the sign up page</a> for pricing on additional messages and volume discounts.

## Setup

-   To ensure you get special pricing as a Google Cloud Platform customer, use the sign-up portal to <a href="https://web.archive.org/web/20160424230553/http://www.mailgun.com/google" target="mailgun">create a new Mailgun account</a>.

<!-- -->

-   Add the required <a href="https://web.archive.org/web/20160424230553/http://jersey.java.net/" target="_blank">jersey REST client</a> modules to your application. For example, if you use Maven, add to the project's pom.xml:

    <a href="https://web.archive.org/web/20160424230553/https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/appengine/mailgun/pom.xml" target="_blank" style="color: white;">appengine/mailgun/pom.xml</a>

    <a href="https://web.archive.org/web/20160424230553/https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/appengine/mailgun/pom.xml" class="button" target="_blank" data-track-type="github" data-track-name="gitHubViewButton" data-track-metadata-link-destination="https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/appengine/mailgun/pom.xml">View on GitHub</a>

    \[This section requires a browser that supports JavaScript and iframes.\]

## Sending a plain text email

<a href="https://web.archive.org/web/20160424230553/https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/appengine/mailgun/src/main/java/com/example/appengine/mailgun/MailgunServlet.java" target="_blank" style="color: white;">appengine/mailgun/src/main/java/com/example/appengine/mailgun/MailgunServlet.java</a>

<a href="https://web.archive.org/web/20160424230553/https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/appengine/mailgun/src/main/java/com/example/appengine/mailgun/MailgunServlet.java" class="button" target="_blank" data-track-type="github" data-track-name="gitHubViewButton" data-track-metadata-link-destination="https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/appengine/mailgun/src/main/java/com/example/appengine/mailgun/MailgunServlet.java">View on GitHub</a>

\[This section requires a browser that supports JavaScript and iframes.\]

## Sending an HTML and text email

Sending a message with HTML and text parts. This example also attaches files to the message:

<a href="https://web.archive.org/web/20160424230553/https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/appengine/mailgun/src/main/java/com/example/appengine/mailgun/MailgunServlet.java" target="_blank" style="color: white;">appengine/mailgun/src/main/java/com/example/appengine/mailgun/MailgunServlet.java</a>

<a href="https://web.archive.org/web/20160424230553/https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/appengine/mailgun/src/main/java/com/example/appengine/mailgun/MailgunServlet.java" class="button" target="_blank" data-track-type="github" data-track-name="gitHubViewButton" data-track-metadata-link-destination="https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/appengine/mailgun/src/main/java/com/example/appengine/mailgun/MailgunServlet.java">View on GitHub</a>

\[This section requires a browser that supports JavaScript and iframes.\]

Sample response:

```
{
    "message": "Queued. Thank you.",
    "id": "<20111114174239.25659.5817@samples.mailgun.org>"
}
```

## Learning more

For more detailed examples and information, including how to track and route messages, see <a href="https://web.archive.org/web/20160424230553/https://documentation.mailgun.com/" target="mailgun">Mailgun's documentation</a>.
