# SMS and Voice Services via Twilio

  

[Python](https://web.archive.org/web/20160424230625/https://cloud.google.com/appengine/docs/python/sms/twilio "View this page in the Python runtime") <span style="color: #ddd; padding: 0em .5em;">|</span><span style="color: black; font-weight:bold">Java</span> <span style="color: #ddd; padding: 0em .5em;">|</span>[PHP](https://web.archive.org/web/20160424230625/https://cloud.google.com/appengine/docs/php/sms/twilio "View this page in the PHP runtime") <span style="color: #ddd; padding: 0em .5em;">|</span><span style="color:lightgray" title="This page is not available in the Go runtime">Go</span>

**Note:** Twilio is a third-party company whose services are not covered by the [Google App Engine Service Level Agreement](https://web.archive.org/web/20160424230625/https://cloud.google.com/appengine/sla).

1.  [Pricing](#pricing)
2.  [Platform](#platform)
    1.  [TwiML](#twiml)
    2.  [REST API](#rest_api)
3.  [Setting Up](#setting_up)
4.  [Receiving an Incoming Call](#receiving_an_incoming_call)
5.  [Sending an SMS](#sending_an_sms)
6.  [Learning More about Twilio](#learning_more_about_twilio)

Twilio is powering the future of business communications, enabling developers to embed voice, VoIP, and messaging into applications. They virtualize all infrastructure needed in a cloud-based, global environment, exposing it through the Twilio communications API platform. Applications are simple to build and scalable. Enjoy flexibility with pay-as-you go pricing, and benefit from cloud reliability.

**Twilio Voice** enables your application to make and receive phone calls. **Twilio SMS** enables your application to send and receive text messages. **Twilio Client** allows you to make VoIP calls from any phone, tablet, or browser and supports WebRTC.

## Pricing

Google App Engine customers receive [complimentary credit for 2,000 SMS messages or inbound minutes](https://web.archive.org/web/20160424230625/http://ahoy.twilio.com/googlecloudplatform) when you upgrade. Redeem this Twilio credit and [get started here](https://web.archive.org/web/20160424230625/http://ahoy.twilio.com/googlecloudplatform).

Twilio is a pay-as-you-go service. There are no set-up fees and you can close your account at any time. You can find more details at [Twilio Pricing](https://web.archive.org/web/20160424230625/http://www.twilio.com/pricing).

## Platform

The Twilio platform consists of the Twilio Markup Language (TwiML), a RESTful API and VoIP SDKs for web browsers, Android and iOS. Helper libraries are available in multiple languages. You can find the full list at [Twilio Helper Libraries](https://web.archive.org/web/20160424230625/http://www.twilio.com/docs/libraries).

### TwiML

TwiML is a set of instructions you can use to tell Twilio what to do when you receive an incoming call or SMS. When someone makes a call or sends an SMS to one of your Twilio numbers, Twilio will look up the URL associated with that phone number and make a request to that URL. Twilio will read TwiML instructions at that URL to determine what to do:

-   `<Say>` - text to speech
-   `<Record>` - record the call
-   `<Play>` - play a message for the caller
-   `<Gather>` - prompt the caller to press digits on their keypad
-   `<Sms>` - send an SMS

Learn about the other verbs and capabilities from the [Twilio Markup Language documentation](https://web.archive.org/web/20160424230625/http://www.twilio.com/docs/api/twiml).

### REST API

The Twilio REST API allows you to query metadata about your account, phone numbers, calls, text messages, and recordings. You can also do some fancy things like initiate outbound calls and send text messages.

Since the API is based on REST principles, it's very easy to write and test applications. You can use your browser to access URLs, and you can use pretty much any HTTP client in any programming language to interact with the API. Learn more about the [Twilio REST API](https://web.archive.org/web/20160424230625/http://www.twilio.com/docs/api/rest).

## Setting Up

The first thing you will need is a Google App Engine development environment. For Java, the two primary options are using [Google's Eclipse plugin](https://web.archive.org/web/20160424230625/https://cloud.google.com/appengine/docs/java/gettingstarted/installing), or [building via Ant scripts](https://web.archive.org/web/20160424230625/https://cloud.google.com/appengine/docs/java/tools/ant). For the purposes of this tutorial, we will assume you are developing with Eclipse, but in broad strokes the step would be the same for Ant/other environments. Create a new App Engine Java project using either of these methods.

If you'd like, it's possible to consume the Twilio REST API directly using App Engine's [URL Fetch service](https://web.archive.org/web/20160424230625/https://cloud.google.com/appengine/docs/java/urlfetch/), without any setup or external SDKs. If you would rather not add any dependencies to your project and want to use HTTP APIs directly, you can simply refer to the [REST API documentation](https://web.archive.org/web/20160424230625/http://www.twilio.com/docs/api/rest) for available endpoints, secured with HTTP Basic Authentication.

However, it will probably be easier to use [Twilio's helper library](https://web.archive.org/web/20160424230625/http://www.twilio.com/docs/libraries) (bundled with all dependent Java libraries), which provides a simple object-oriented interface to Twilio's REST API. To use the Twilio helper library in your GAE Java project, you will first need to copy it into your Eclipse project to the `war/WEB-INF/lib` directory.

Once the JAR file has been copied into your project, you will need to put it on your project's build path to get code completion in Eclipse, and remove any compile errors in the IDE. To add a JAR file to your project's build path, right-click on your project folder, and select "Build Path -&gt; Configure Build Path...". On that screen, click the "Add JARs" button, and navigate your project folder to select the jar you already copied into your project. You are now ready to use the Twilio REST API!

For more information, see the [Javadocs for the Twilio helper library](https://web.archive.org/web/20160424230625/http://twilio.github.com/twilio-java/), and this [example of sending a text message from a Twilio number in a Java Servlet](https://web.archive.org/web/20160424230625/https://gist.github.com/kwhinnery/5119402).

## Receiving an Incoming Call

Twilio can send an HTTP request to our App Engine application whenever a call or text message is received. Use [this example of how to set up a Java Servlet to respond to incoming voice calls](https://web.archive.org/web/20160424230625/https://gist.github.com/kwhinnery/5119456) with the message "Hello Monkey!"

In order to expose this servlet in our web application, we will need to use [this code to add the appropriate mapping in our deployment descriptor](https://web.archive.org/web/20160424230625/https://gist.github.com/kwhinnery/5119475) (`web.xml`).

Once our servlet is configured, we will need to [deploy our application to App Engine](https://web.archive.org/web/20160424230625/https://cloud.google.com/appengine/docs/java/gettingstarted/uploading). You will then be able to send a POST request to `http://<your app>.appspot.com/twiml`, which should return the following text:

```
<?xml version="1.0" encoding="UTF-8"?>
<Response>
  <Say>Hello Monkey!</Say>
</Response>
```

Copy and paste the URL of your servlet into the "Voice" URL on the Numbers page of your Twilio Account.

Now call your Twilio number! You should hear a voice say "Hello Monkey!" in response. When you call, Twilio will fetch your URL, and execute the XML instructions listed above. Then, Twilio will hang up, because there are no more instructions.

## Sending an SMS

Sending an outbound text message via our [helper library](https://web.archive.org/web/20160424230625/http://twilio.github.com/twilio-java/) is done using Java classes included in the JAR file. In this example file that uses the helper library, you'll see [a simple servlet application, configured to send an SMS text message](https://web.archive.org/web/20160424230625/https://gist.github.com/kwhinnery/5254045).

Note that you’ll need to fill in your `ACCOUNT_SID` and `AUTH_TOKEN`, which are found at `[https://www.twilio.com/user/account](https://www.twilio.com/user/account)`. You will also need to configure the `HashMap` to contain valid parameters for the REST request. The `from` and `to` parameters must use real phone numbers. The `from` number must be a valid Twilio phone number in your account. The `to` number can be any outgoing number, your cell phone for example.

## Learning More about Twilio

Now that you've learned some of the basics, learn more about additional features and some best practices for building secure and scalable applications:

-   [Twilio Security Guidelines](https://web.archive.org/web/20160424230625/http://www.twilio.com/docs/security)
-   [Twilio HowTo’s and Example Code](https://web.archive.org/web/20160424230625/http://www.twilio.com/docs/howto)
-   [Twilio Quickstart Tutorials](https://web.archive.org/web/20160424230625/http://www.twilio.com/docs/quickstart)
-   [Twilio on GitHub](https://web.archive.org/web/20160424230625/https://github.com/twilio)
-   [Talk to Twilio Support](https://web.archive.org/web/20160424230625/http://www.twilio.com/help/contact)
