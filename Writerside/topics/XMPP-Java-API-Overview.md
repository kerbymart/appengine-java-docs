# XMPP Java API Overview

  

[Python](https://web.archive.org/web/20160424225731/https://cloud.google.com/appengine/docs/python/xmpp/ "View this page in the Python runtime") <span style="color: #ddd; padding: 0em .5em;">|</span><span style="color: black; font-weight:bold">Java</span> <span style="color: #ddd; padding: 0em .5em;">|</span><span style="color:lightgray" title="This page is not available in the PHP runtime">PHP</span> <span style="color: #ddd; padding: 0em .5em;">|</span>[Go](https://web.archive.org/web/20160424225731/https://cloud.google.com/appengine/docs/go/xmpp/ "View this page in the Go runtime")

[<img src="https://web.archive.org/web/20160424225731im_/https://cloud.google.com/cloud/images/stack_overflow_questions.png" title="Stack Overflow Questions" data-border="0" width="240" height="53" />](https://web.archive.org/web/20160424225731/http://stackoverflow.com/questions/tagged/google-app-engine+xmpp)

[](https://web.archive.org/web/20160424225731/http://stackoverflow.com/feeds/tag?sort=votes&tagnames=google-app-engine%2Bxmpp)

<span class="link gc-analytics-event" category="Sidebar" data-action="Stack Overflow question click"></span>

<a href="https://web.archive.org/web/20160424225731/http://stackoverflow.com/questions/tagged/google-app-engine+xmpp?sort=votes" class="gc-analytics-event" data-category="Sidebar" data-action="Stack Overflow See more link">See more...</a>

An App Engine application can send and receive chat messages to and from any XMPP-compatible chat messaging service. An app can send and receive chat messages, send chat invites, request a user's chat presence and status, and provide a chat status. Incoming XMPP messages are handled by request handlers, similar to web requests.

Some possible uses of chat messages include automated chat participants ("chat bots"), chat notifications, and chat interfaces to services. A rich client with a connection to an XMPP server can use XMPP to interact with an App Engine application in real time, including receiving messages initiated by the app.

Currently, an app cannot participate in group chats. An app can only receive messages of types "chat" and "normal". An app can send messages of any type defined in [RFC 3921](https://web.archive.org/web/20160424225731/http://www.ietf.org/rfc/rfc3921.txt).

1.  [Sending chat messages](#Java_Sending_chat_messages)
2.  [Handling incoming calls](#Java_Handling_incoming_calls)
3.  [XMPP addresses](#Java_XMPP_addresses)
4.  [JIDs and resources](#Java_JIDs_and_resources)
5.  [Sending invitations](#Java_Sending_invitations)
6.  [Sending the application's chat status](#Java_Sending_the_applications_chat_status)
7.  [Quotas and limits](#Java_Quotas_and_limits)

## Sending chat messages

An app can send chat messages to any XMPP address by calling the XMPP service API. A chat message can be of any of the five types defined in [RFC 3921](https://web.archive.org/web/20160424225731/http://www.ietf.org/rfc/rfc3921.txt). Note that an app can only receive two types of messages:

1.  chat messages via `/_ah/xmpp/message/chat/`
2.  normal messages via `/_ah/xmpp/message/chat/`

For the body of the message, an app can provide plain text that is displayed to the user, or it can provide an XML stanza that is included in the XMPP XML data. You can use XML message data to communicate structured data to a custom client.

The following example tests for a user's availability, then sends a chat message.

```
import com.google.appengine.api.xmpp.JID;
import com.google.appengine.api.xmpp.Message;
import com.google.appengine.api.xmpp.MessageBuilder;
import com.google.appengine.api.xmpp.SendResponse;
import com.google.appengine.api.xmpp.XMPPService;
import com.google.appengine.api.xmpp.XMPPServiceFactory;

// ...
        JID jid = new JID("example@gmail.com");
        String msgBody = "Someone has sent you a gift on Example.com. To view: http://example.com/gifts/";
        Message msg = new MessageBuilder()
            .withRecipientJids(jid)
            .withBody(msgBody)
            .build();

        boolean messageSent = false;
        XMPPService xmpp = XMPPServiceFactory.getXMPPService();
        SendResponse status = xmpp.sendMessage(msg);
        messageSent = (status.getStatusMap().get(jid) == SendResponse.Status.SUCCESS);

        if (!messageSent) {
            // Send an email message instead...
        }
```

When an app running in the development server sends an XMPP message, the fields of the message are printed to the console, and no message is sent. The development server cannot send XMPP messages on behalf of the app.

## Handling incoming calls

App Engine provides three inbound services to manage messages, subscriptions, and presence:

-   `xmpp_message` allows your application to receive chat messages from XMPP-compliant chat services.
-   `xmpp_subscribe` allows subscriptions between users and your application for the purpose of exchanging data, such as chat messages, presence information, and status messages.
-   `xmpp_presence` allows your application to determine a user's chat presence (available or unavailable).
-   `xmpp_error` allows your application to receive error stanzas.

You can [enable these inbound services](https://web.archive.org/web/20160424225731/https://cloud.google.com/appengine/docs/java/config/appconfig#Java_appengine_web_xml_Inbound_services) in `appengine-web.xml`:

```
<inbound-services>
  <service>xmpp_message</service>
  <service>xmpp_presence</service>
  <service>xmpp_subscribe</service>
  <service>xmpp_error</service>
</inbound-services>
```

The following sections show you how to handle these inbound services in your application:

-   [Chat messages](#Java_Receiving_chat_messages)
-   [User subscriptions](#Java_Handling_subscriptions)
-   [User presence](#Java_Handling_user_presence)
-   [XMPP errors](#Java_Handling_error_stanzas)

### Receiving chat messages

Users of XMPP-compliant chat services can send chat messages to App Engine applications. For an app to receive chat messages, the XMPP message service must be enabled in the app's configuration.

To enable the XMPP service for a Java app, edit the `appengine-web.xml` file and include the following lines:

```
<inbound-services>
  <service>xmpp_message</service>
</inbound-services>
```

With the XMPP service enabled, App Engine makes an HTTP POST request to the following URL path when your app receives a chat message:

`/_ah/xmpp/message/chat/`

To handle incoming messages, you simply create a request handler that accepts POST requests at this URL path.

This URL path is restricted to app administrators automatically. The XMPP service connects to the app with "administrator" status for the purposes of accessing this URL path. You can configure the path to have this restriction explicitly if you like, but this is not necessary. Only the XMPP service and clients authenticated as administrators using Google Accounts can access this URL path.

The POST request data represents the message as a MIME multipart message ([RFC 2388](https://web.archive.org/web/20160424225731/http://www.ietf.org/rfc/rfc2388.txt)). Each part is a named POST parameter:

-   `from`, the address of the sender of the message
-   `to`, the address of the recipient as described by the sender (see below)
-   `body`, the body of the message
-   `stanza`, the full XMPP message in its original XML form

An app can receive messages of two XMPP types: "chat" and "normal." If App Engine receives an XMPP message of an unsupported type, the message is ignored and the request handler is not called.

The Java Servlet API does not provide a way to parse MIME multipart POST data directly. The XMPP API includes a helper method for parsing this data from the HttpServletRequest object. The following example illustrates how to use this helper class to parse incoming XMPP messages.

```
import java.io.IOException;
import javax.servlet.http.*;
import com.google.appengine.api.xmpp.JID;
import com.google.appengine.api.xmpp.Message;
import com.google.appengine.api.xmpp.XMPPService;
import com.google.appengine.api.xmpp.XMPPServiceFactory;

@SuppressWarnings("serial")
public class XMPPReceiverServlet extends HttpServlet {
    @Override
    public void doPost(HttpServletRequest req, HttpServletResponse res)
          throws IOException {
        XMPPService xmpp = XMPPServiceFactory.getXMPPService();
        Message message = xmpp.parseMessage(req);

        JID fromJid = message.getFromJid();
        String body = message.getBody();
        // ...
    }
}
```

To map this servlet to the XMPP URL path, put the following section in your `web.xml` file, inside the `<web-app>` element:

```
<servlet>
  <servlet-name>xmppreceiver</servlet-name>
  <servlet-class>XMPPReceiverServlet</servlet-class>
</servlet>
<servlet-mapping>
  <servlet-name>xmppreceiver</servlet-name>
  <url-pattern>/_ah/xmpp/message/chat/</url-pattern>
</servlet-mapping>
```

### Handling subscriptions

Enabling the `xmpp_subscribe` inbound service allows your application to receive updates to a user's subscription information via POSTs to the following URL paths:

-   POSTs to `/_ah/xmpp/subscription/subscribe/` signal that the user wishes to subscribe to the application's presence.
-   POSTs to `/_ah/xmpp/subscription/subscribed/` signal that the user has allowed the application to receive its presence.
-   POSTs to `/_ah/xmpp/subscription/unsubscribe/` signal that the user is unsubscribing from the application's presence.
-   POSTs to `/_ah/xmpp/subscription/unsubscribed/` signal that the user has denied the application's subscription request, or cancelled a previously granted subscription.

These URLs correspond to the stanzas used for managing subscriptions described in the [XMPP Instant Messages and Presence Protocol](https://web.archive.org/web/20160424225731/http://xmpp.org/rfcs/rfc3921.html#sub).

The following code sample demonstrates how to track subscriptions using webhooks to the above handlers. The following code sample uses POSTs to the above handlers to maintain a roster of subscribed users in the datastore:

```
// In the handler for _ah/xmpp/presence/subscribe
XMPPService xmppService = XMPPServiceFactory.getXMPPService();
Subscription sub = xmppService.parseSubscription(req);

// Split the bare XMPP address (e.g., user@gmail.com)
// from the resource (e.g., gmail.CD6EBC4A)
String from = sub.getFromJid().getId().split("/")[0];

// Add an object tracking the roster to the contact

this.roster.addContact(sub.getFromJid());
```

### Handling user presence

Enabling the `xmpp_presence` inbound service allows your application to detect changes to a user's chat presence. When you enable `xmpp_presence`, your application receives POSTs to the following URL paths:

-   POSTs to `/_ah/xmpp/presence/available/` signal that the user is available and provide the user's chat status.
-   POSTs to `/_ah/xmpp/presence/unavailable/` signal that the user is unavailable.
-   POSTs to `/_ah/xmpp/presence/probe/` request a user's current presence.

Your application can register handlers to these paths in order to receive notifications. You can use these notifications to track which users are currently online. If you wish to check the most current status, you can use the [`sendPresence`](https://web.archive.org/web/20160424225731/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/xmpp/XMPPService#sendPresence) method with the `type` parameter set to `PresenceType.PROBE`. This will result in POSTs to the above paths reflecting the user's current status.

If a Google talk user has subscribed to an app (has accepted an invitation or has invited the app to chat), the app can discover the user's availability by looking for a POST to `/_ah/xmpp/presence/available`. If the user is subscribed and available, you can send them your application's presence and status:

```
// In the handler for _ah/xmpp/presence/available
XMPPService xmppService = XMPPServiceFactory.getXMPPService();
Presence presence = xmppService.parsePresence(req);

// Split the XMPP address (e.g., user@gmail.com)
// from the resource (e.g., gmail.CD6EBC4A)
String from = presence.getFromJid().getId().split("/")[0];

// Mirror the contact's presence back to them
xmppService.sendPresence(from, PresenceType.AVAILABLE, presence.getPresenceShow(), presence.getStatus());
```

### Handling error stanzas

Enabling the `xmpp_error` inbound service allows your application to receive XMPP error stanzas. Error stanzas are most commonly sent as a result of sending a message to a user who's offline.

With the error service enabled, App Engine makes an HTTP POST request to the following path when your app receives an error stanza:

`/_ah/xmpp/error/`

You can register a handler at this address to be notified of XMPP errors. This handler may be responsible for eg. diagnostics or logging.

```
// In the handler for _ah/xmpp/error

// Parse the POST data, which is sent as a MIME stream containing the stanza.
ByteArrayOutputStream baos = new ByteArrayOutputStream();
ServletInputStream inputStream = req.getInputStream();
ByteStreams.copy(inputStream, baos);

// Log the error
log.warning("Error stanza received: " + baos.toString());
```

## XMPP addresses

An application can send and receive messages using several kinds of addresses, or "JIDs." One kind of address uses the application ID and the `appspot.com` domain, as follows:

`your_app_id@appspot.com`

An app can also use a custom address in the following format, where `anything` is any string containing letters, numbers and hyphens:

`anything@your_app_id.appspotchat.com`

**Note:** `anything` must not include certain characters that would otherwise be allowed by the XMPP standard (even if they're escaped), such as: `, ; @ ( ) [ ]`.

**Note:** Also note the different domain name for custom addresses: `appspotchat.com`

The use of Google Apps domains in XMPP addresses is not yet supported for apps.

The app can choose which address is used as the sender address when sending a message. All messages sent to any address in the supported formats are routed to the application.

### Addresses and app versions

**Note:** XMPP is not supported in non-default modules.

The `@appspot.com` address and the `@your_app_id.appspotchat.com` addresses all refer to the default version of the app. Messages sent to these addresses are routed to the default version. When the default version of an app sends a message, the requested address is used as the sender of the message. Replies to the app's messages are routed to the default version, even if the default version has changed since the original message was sent.

Each version of an app has its own set of XMPP addresses in the following format:

`anything@version.latest.your_app_id.appspotchat.com`

If App Engine receives a message for an address in this format, it is routed to the corresponding version of the app (if that version still exists).

If a version of an app sends a message and that version is not the default, the sender address of the message is changed to an address in this format. If the sender is `your_app_id@appspot.com`, the sender address becomes `your_app_id@version.latest.your_app_id.appspotchat.com`. This ensures that replies to a message sent by a specific non-default version of the app are routed back to that version.

## JIDs and resources

An XMPP address, also known as a "JID," has several components: a node (such as the name of a user of a chat service), a domain (the domain name of the service), and an optional resource. These components are separated by punctuation: `node@domain/resource`.

When establishing a chat conversation with a human user of a chat service, you typically use just the node and domain parts of the JID. Each chat client the user has connected to the service may have its own resource, and sending a message to the JID without the resource sends it to all of the user's clients.

A JID without a resource is known as a "bare JID". A JID with a resource is a "full JID". App Engine's XMPP service supports both bare and full JIDs for recipient and sender addresses of messages.

When sending a message, if you use a bare JID as the sender address, the XMPP service automatically uses the resource `/bot`. If another XMPP user sends a message to the app using a full JID with a custom resource, the app can see which resource was used by inspecting the `to` field of the incoming message.

## Sending invitations

Chat servers only accept messages for users that are "subscribed" to the sender, either because the user invited the sender to chat or because the user accepted an invitation to chat sent by the sender. An App Engine app can send chat invitations using [`sendInvitation()`](https://web.archive.org/web/20160424225731/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/xmpp/XMPPService#sendInvitation(com.google.appengine.api.xmpp.JID)).

As with sending email, a best practice is to send a chat invitation only when the user asks, such as by clicking a button on a web page. Alternatively, the app can ask the user to send an invitation to the app's XMPP address to enable receiving of messages.

App Engine accepts all chat invitations and automatically registers subscriptions as described in the [Handling Subscriptions](#Java_Handling_subscriptions) section. App Engine routes all chat messages to the application, regardless of whether the sender previously sent an invitation to the app.

## Sending the application's chat status

The [`sendPresence()`](https://web.archive.org/web/20160424225731/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/xmpp/XMPPService#sendPresence(com.google.appengine.api.xmpp.JID,%20com.google.appengine.api.xmpp.PresenceType,%20com.google.appengine.api.xmpp.PresenceShow,%20java.lang.String)) API allows the application to send the application's presence, along with an optional status message, to a given JID. By default, the application appears available with no status message. You can set the status message as follows:

```
xmppService.sendPresence(toJid, PresenceType.AVAILABLE, PresenceShow.NONE, "My app's status")
```

## Quotas and limits

Each XMPP service request counts toward the **XMPP API Calls** quota.

Each outgoing XMPP chat message counts toward the following quotas:

-   **XMPP Data Sent**
-   **XMPP Recipients Messaged**
-   **Outgoing Bandwidth (billable)**

Each outgoing XMPP chat invitation counts toward the following quotas:

-   **XMPP Data Sent**
-   **XMPP Invitations Sent**
-   **Outgoing Bandwidth (billable)**

Each incoming XMPP message counts toward the following quotas:

-   **Requests**
-   **Incoming Bandwidth (billable)**

Each incoming presence and subscription message counts as a normal HTTP request.

Computation performed in a request handler for incoming XMPP messages applies toward the same quotas as with web requests and tasks.

For more information, see [Quotas](https://web.archive.org/web/20160424225731/https://cloud.google.com/appengine/docs/quotas) and [Quota details](https://web.archive.org/web/20160424225731/https://cloud.google.com/appengine/docs/developers-console/#quotas). You can see the current quota usage of your app by visiting the Google Cloud Platform Console [quota details](https://web.archive.org/web/20160424225731/https://console.cloud.google.com//project/_/appengine/quotadetails) page for your project.

In addition to quotas, the following limits apply to the use of the XMPP service:

<table>
<thead>
<tr class="header">
<th>Limit</th>
<th>Amount</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>incoming message size</td>
<td>100 kilobytes</td>
</tr>
<tr class="even">
<td>outgoing message size</td>
<td>100 kilobytes</td>
</tr>
<tr class="odd">
<td>invitations sent</td>
<td>100,000/day</td>
</tr>
</tbody>
</table>

There is no explicit limit to the number of recipients for a message. However, each message is sent and metered separately for each recipient. A message with 30 recipients consumes 30 times as much of the affected quotas as a message with 1 recipient. Also, the total size of the API call is limited to 1 megabyte, including the total size of the recipient list and the message body.
