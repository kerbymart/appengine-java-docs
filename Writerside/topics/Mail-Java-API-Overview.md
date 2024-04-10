# Mail Java API Overview

  

[Python](https://web.archive.org/web/20160425095930/https://cloud.google.com/appengine/docs/python/mail/ "View this page in the Python runtime") <span style="color: #ddd; padding: 0em .5em;">|</span><span style="color: black; font-weight:bold">Java</span> <span style="color: #ddd; padding: 0em .5em;">|</span>[PHP](https://web.archive.org/web/20160425095930/https://cloud.google.com/appengine/docs/php/mail/ "View this page in the PHP runtime") <span style="color: #ddd; padding: 0em .5em;">|</span>[Go](https://web.archive.org/web/20160425095930/https://cloud.google.com/appengine/docs/go/mail/ "View this page in the Go runtime")

Related topics

-   <a href="https://web.archive.org/web/20160425095930/https://cloud.google.com/appengine/docs/java/mail/sendgrid" class="gc-analytics-event" data-category="Sidebar" data-action="Related topic" data-label="Sending email with SendGrid in App Engine">Sending email with SendGrid in App Engine</a>
-   <a href="https://web.archive.org/web/20160425095930/http://blog.vogella.com/2011/02/03/google-app-engine-sending-emails/" class="gc-analytics-event" data-category="Sidebar" data-action="Related topic" data-label="Google App Engine and Sending Emails (Vogella Blog)">Google App Engine and Sending Emails (Vogella Blog)</a>

[<img src="https://web.archive.org/web/20160425095930im_/https://cloud.google.com/cloud/images/stack_overflow_questions.png" title="Stack Overflow Questions" data-border="0" width="240" height="53" />](https://web.archive.org/web/20160425095930/http://stackoverflow.com/questions/tagged/google-app-engine+email)

[](https://web.archive.org/web/20160425095930/http://stackoverflow.com/feeds/tag?sort=votes&tagnames=google-app-engine%2Bemail)

<span class="link gc-analytics-event" category="Sidebar" data-action="Stack Overflow question click"></span>

<a href="https://web.archive.org/web/20160425095930/http://stackoverflow.com/questions/tagged/google-app-engine+email?sort=votes" class="gc-analytics-event" data-category="Sidebar" data-action="Stack Overflow See more link">See more...</a>

App Engine applications can send email messages on behalf of the app's email receiving addresses and on behalf of some users with Google Accounts. Apps can receive email at various addresses. Apps send messages using the Mail service and receive messages in the form of HTTP requests initiated by App Engine and posted to the app.

1.  [Sending mail](#Java_Sending_mail)
2.  [Sending mail with the JavaMail API](#Java_Sending_mail_with_the_JavaMail_API)
3.  [Receiving mail in Java](#Java_Receiving_mail_in_Java)
4.  [Receiving bounce notification](#Java_Receiving_bounce_notification)
5.  [Sending mail with attachments](#Java_Sending_mail_with_attachments)
6.  [Sending mail with headers](#Java_Sending_mail_with_headers)
7.  [Mail and the development server](#Java_Mail_and_the_development_server)
8.  [Authenticating mail: DKIM](#Java_Authenticating_mail_DKIM)
9.  [Bulk senders guidelines](#Java_Bulk_senders_guidelines)
10. [Quotas and limits](#Java_Quotas_and_limits)

## Sending mail

The Mail service can send email messages to one or more recipients. The message contains a subject, a plaintext body, and an optional HTML body. It can also contain file attachments, as well as a limited set of headers.

For security purposes, the sender address of a message must be one of the following:

-   The Gmail or Google Apps Account of the user who is currently signed in
-   Any email address of the form anything@appname.appspotmail.com or anything@appalias.appspotmail.com
-   Any email address listed in the Cloud Platform Console under [Email API Authorized Senders](https://web.archive.org/web/20160425095930/https://console.cloud.google.com/project/_/appengine/settings)

All email addresses on the [Email API Authorized Senders](https://web.archive.org/web/20160425095930/https://console.cloud.google.com/project/_/appengine/settings) list need to be valid Gmail or Google-hosted domain accounts. App Administrators can add the following accounts to the list of Authorized Senders:

-   Their own email address.
-   Any group for which they are an Owner or Manager.
-   For applications hosted in a Google Apps domain: noreply@domain.com, as long as noreply@domain.com is a valid account (user or group).

In addition, domain administrators of domains managed by Google Apps can add any user in their domain to the list.

If you will be sending email from a domain managed by Google Apps, you should set the SPF records for your domain to indicate that Google is a trusted source for your email. For instructions on how to do this, see [SPF records](https://web.archive.org/web/20160425095930/http://www.google.com/support/a/bin/answer.py?answer=33786) in the Google Apps help articles.

If you have one or more aliases set up for your Google Apps domain, you can send email from email addresses that use the domain alias. For example, adding "xyz@domain.com" to the Authorized Senders list will have the effect of also allowing sending email from "xyz@alias.com".

You can use any email address for a recipient. A recipient can be in the message's "to" field or the "cc" field, or the recipient can be hidden from the message header (a "blind carbon copy" or "bcc").

When an application calls the Mail service to send a message, the message is queued and the call returns immediately. The Mail service uses standard procedures for contacting each recipient's mail server, delivering the message, and retrying if the mail server cannot be contacted.

Mail that matches a known signature for spam, viruses or other malicious content may not be accepted for delivery.

If the Mail service cannot deliver a message, or if an recipient's mail server returns a bounce message (such as if there is no account for that address on that system), the error message is sent by email to the address of the sender for the message. The application itself does not receive any notification about whether delivery succeeded or failed.

## Sending mail with the JavaMail API

The Mail service Java API supports the [JavaMail](https://web.archive.org/web/20160425095930/http://java.sun.com/products/javamail/) ([javax.mail](https://web.archive.org/web/20160425095930/http://java.sun.com/products/javamail/javadocs/index.html)) interface for sending email messages.

All of the JavaMail classes you need are included with the App Engine SDK. Do not add Oracle®'s JavaMail JARs to your app; if you do, the app will throw exceptions.

When creating a JavaMail Session, you do not need to provide any SMTP server configuration. App Engine will always use the Mail service for sending messages.

```
import java.util.Properties;
import javax.mail.Message;
import javax.mail.MessagingException;
import javax.mail.Session;
import javax.mail.Transport;
import javax.mail.internet.AddressException;
import javax.mail.internet.InternetAddress;
import javax.mail.internet.MimeMessage;

// ...
Properties props = new Properties();
Session session = Session.getDefaultInstance(props, null);

String msgBody = "...";

try {
    Message msg = new MimeMessage(session);
    msg.setFrom(new InternetAddress("admin@example.com", "Example.com Admin"));
    msg.addRecipient(Message.RecipientType.TO,
     new InternetAddress("user@example.com", "Mr. User"));
    msg.setSubject("Your Example.com account has been activated");
    msg.setText(msgBody);
    Transport.send(msg);

} catch (AddressException e) {
    // ...
} catch (MessagingException e) {
    // ...
}
```

See [Using JavaMail to Send Mail](https://web.archive.org/web/20160425095930/https://cloud.google.com/appengine/docs/java/mail/usingjavamail) for more information and examples.

## Receiving mail in Java

You can set up your app to receive incoming email at addresses in the following format:

```
anything@appid.appspotmail.com
```
**Note:** Even if your app is deployed on a custom domain, your app can't receive email sent to addresses in that domain.

To receive email, you must put a section that enables incoming mail in your app's `appengine-web.xml` file:

```
<inbound-services>
  <service>mail</service>
</inbound-services>
```

Incoming email in App Engine works by posting HTTP requests containing MIME data to your app. The email's MIME data is supplied to your app as the contents of an HTTP POST request, and you process this data in your handler.

In your `web.xml` file, you must create mappings from URL paths that represent email addresses to handlers in your app's code:

```
<servlet>
  <servlet-name>mailhandler</servlet-name>
  <servlet-class>MailHandlerServlet</servlet-class>
</servlet>
<servlet-mapping>
  <servlet-name>mailhandler</servlet-name>
  <url-pattern>/_ah/mail/*</url-pattern>
</servlet-mapping>
<security-constraint>
  <web-resource-collection>
    <web-resource-name>mail</web-resource-name>
    <url-pattern>/_ah/mail/*</url-pattern>
  </web-resource-collection>
  <auth-constraint>
    <role-name>admin</role-name>
  </auth-constraint>
</security-constraint>
```

The URL path is `/_ah/mail/` followed by the incoming email address used. The pattern `/_ah/mail/*` matches all incoming email addresses.

In your app, create the servlet class to handle incoming mail requests. The JavaMail API includes the `MimeMessage` class which you can use to parse email messages in your handlers:

```
import java.io.IOException; 
import java.util.Properties; 
import javax.mail.Session; 
import javax.mail.internet.MimeMessage; 
import javax.servlet.http.*; 

public class MailHandlerServlet extends HttpServlet { 
    @Override
    public void doPost(HttpServletRequest req, HttpServletResponse resp) 
            throws IOException { 
                Properties props = new Properties(); 
                Session session = Session.getDefaultInstance(props, null); 
                MimeMessage message = new MimeMessage(session, req.getInputStream());
```

You can then use various methods to parse the `message` object.

See [Receiving Mail](https://web.archive.org/web/20160425095930/https://cloud.google.com/appengine/docs/java/mail/receiving) for more information and examples.

## Receiving bounce notification

By default, apps do not receive email bounce notifications. To turn this on for your app and to handle bounce notifications, see [Receiving Bounce Notification](https://web.archive.org/web/20160425095930/https://cloud.google.com/appengine/docs/java/mail/bounce).

## Sending mail with attachments

An outgoing email message can have zero or more file attachments.

An attachment has a filename and file data. The file data can come from any source, such as an application data file or the datastore. The MIME type of the attachment is determined from the filename.

The following is a list of MIME types and their corresponding filename extensions allowed for file attachments to an email message. You are not limited to these extensions. If you use an unknown extension, App Engine will assign it the mime type `application/octet-stream`.

<table>
<thead>
<tr class="header">
<th>MIME Type</th>
<th>Filename Extension(s)</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>application/msword</td>
<td>doc</td>
</tr>
<tr class="even">
<td>application/msword</td>
<td>docx</td>
</tr>
<tr class="odd">
<td>application/pdf</td>
<td>pdf</td>
</tr>
<tr class="even">
<td>application/rss+xml</td>
<td>rss</td>
</tr>
<tr class="odd">
<td>application/vnd.google-earth.kml+xml</td>
<td>kml</td>
</tr>
<tr class="even">
<td>application/vnd.google-earth.kmz</td>
<td>kmz</td>
</tr>
<tr class="odd">
<td>application/vnd.ms-excel</td>
<td>xls</td>
</tr>
<tr class="even">
<td>application/vnd.ms-excel</td>
<td>xlsx</td>
</tr>
<tr class="odd">
<td>application/vnd.ms-powerpoint</td>
<td>pptx</td>
</tr>
<tr class="even">
<td>application/vnd.ms-powerpoint</td>
<td>pps ppt</td>
</tr>
<tr class="odd">
<td>application/vnd.oasis.opendocument.presentation</td>
<td>odp</td>
</tr>
<tr class="even">
<td>application/vnd.oasis.opendocument.spreadsheet</td>
<td>ods</td>
</tr>
<tr class="odd">
<td>application/vnd.oasis.opendocument.text</td>
<td>odt</td>
</tr>
<tr class="even">
<td>application/vnd.sun.xml.calc</td>
<td>sxc</td>
</tr>
<tr class="odd">
<td>application/vnd.sun.xml.writer</td>
<td>sxw</td>
</tr>
<tr class="even">
<td>application/x-gzip</td>
<td>gzip</td>
</tr>
<tr class="odd">
<td>application/zip</td>
<td>zip</td>
</tr>
<tr class="even">
<td>audio/basic</td>
<td>au snd</td>
</tr>
<tr class="odd">
<td>audio/flac</td>
<td>flac</td>
</tr>
<tr class="even">
<td>audio/mid</td>
<td>mid rmi</td>
</tr>
<tr class="odd">
<td>audio/mp4</td>
<td>m4a</td>
</tr>
<tr class="even">
<td>audio/mpeg</td>
<td>mp3</td>
</tr>
<tr class="odd">
<td>audio/ogg</td>
<td>oga ogg</td>
</tr>
<tr class="even">
<td>audio/x-aiff</td>
<td>aif aifc aiff</td>
</tr>
<tr class="odd">
<td>audio/x-wav</td>
<td>wav</td>
</tr>
<tr class="even">
<td>image/gif</td>
<td>gif</td>
</tr>
<tr class="odd">
<td>image/jpeg</td>
<td>jpeg jpg jpe</td>
</tr>
<tr class="even">
<td>image/png</td>
<td>png</td>
</tr>
<tr class="odd">
<td>image/tiff</td>
<td>tiff tif</td>
</tr>
<tr class="even">
<td>image/vnd.wap.wbmp</td>
<td>wbmp</td>
</tr>
<tr class="odd">
<td>image/x-ms-bmp</td>
<td>bmp</td>
</tr>
<tr class="even">
<td>text/calendar</td>
<td>ics</td>
</tr>
<tr class="odd">
<td>text/comma-separated-values</td>
<td>csv</td>
</tr>
<tr class="even">
<td>text/css</td>
<td>css</td>
</tr>
<tr class="odd">
<td>text/html</td>
<td>htm html</td>
</tr>
<tr class="even">
<td>text/plain</td>
<td>text txt asc diff pot</td>
</tr>
<tr class="odd">
<td>text/x-vcard</td>
<td>vcf</td>
</tr>
<tr class="even">
<td>video/mp4</td>
<td>mp4</td>
</tr>
<tr class="odd">
<td>video/mpeg</td>
<td>mpeg mpg mpe</td>
</tr>
<tr class="even">
<td>video/ogg</td>
<td>ogv</td>
</tr>
<tr class="odd">
<td>video/quicktime</td>
<td>qt mov</td>
</tr>
<tr class="even">
<td>video/x-msvideo</td>
<td>avi</td>
</tr>
</tbody>
</table>

As a security measure to protect against viruses, you cannot send email attachments or zip files containing any of the following extensions:

-   ade
-   adp
-   bat
-   chm
-   cmd
-   com
-   cpl
-   exe
-   hta
-   ins
-   isp
-   jse
-   lib
-   mde
-   msc
-   msp
-   mst
-   pif
-   scr
-   sct
-   shb
-   sys
-   vb
-   vbe
-   vbs
-   vxd
-   wsc
-   wsf
-   wsh

## Sending mail with headers

An outgoing email can have zero or more extra headers. A header has a name and a value.

For security purposes, the name of a header must be of one of the allowed header names:

-   In-Reply-To
-   List-Id
-   List-Unsubscribe
-   On-Behalf-Of
-   References
-   Resent-Date
-   Resent-From
-   Resent-To

## Mail and the development server

When an application running in the development server calls the Mail service to send an email message, the message is printed to the log. The Java development server does not send the email message.

## Authenticating mail: DKIM

If your application sends messages from an email address that is part of a Google Apps domain, App Engine can utilize a Google Apps feature to cryptographically sign the emails it sends. This signature says that this mail that purports to be from `emma@example.com` really came from `example.com`. The recipient can check this signature; if the signature is there and correct, the recipient knows that the sender's domain wasn't spoofed. App Engine uses the DomainKeys Identified Mail (DKIM) standard to authenticate the sender's domain.

To enable DKIM authentication for messages sent from Google Apps email addresses, follow [these instructions](https://web.archive.org/web/20160425095930/http://support.google.com/a/bin/answer.py?answer=174124) in the Google Apps Help Center. Note that it may take up to 48 hours before DKIM authentication is active for your Google Apps domain.

App Engine will sign the application's outgoing mails if the sender address is part of a Google Apps domain with DKIM enabled. Additionally, the sender address must be formatted such that the domain part of the email address only consists of lowercase letters.

## Bulk senders guidelines

You must follow the guidelines in this section if your application is sending out bulk email, i.e. similar messages to numerous recipients. These guidelines will help to improve your inbox delivery rate to Gmail users, by ensuring that all recipients in your distribution list actually want to receive the email. If recipients manually mark your email as spam then that acts as a strong signal to Gmail to mark future emails from you as spam.

### Authentication and identification

-   Use the same sender for every bulk email. When calling the Mail API function to send email, the `From` header will be set to match the sender you specify.
-   Your sender address should be an account in a [Google Apps for Business](https://web.archive.org/web/20160425095930/http://www.google.com/intx/en/enterprise/apps/business/) domain. Google accounts that send too many emails that are marked as spam by Google, can be temporarily disabled if their domain is still in the free trial period or has less than six users. In these cases, the Mail API will throw an exception with an `Unauthorized sender` error message.
-   Sign your email with [DKIM](#Java_Authenticating_mail_DKIM), which requires a Google Apps domain if you are sending using App Engine.
-   Publish an [SPF record](https://web.archive.org/web/20160425095930/http://support.google.com/a/bin/answer.py?hl=en&answer=33786) to prevent spammers from spoofing your envelope sender address. SPF verifies that email is sent from an IP address that is published in the DNS records of the envelope sender. App Engine's envelope sender is in the `apphosting.bounces.google.com` domain, so your SPF record may not be used to determine if email from App Engine should be delivered.

### Sending limits

-   Your mail quota is shown in the Google Cloud Platform Console [Quota Details](https://web.archive.org/web/20160425095930/https://console.cloud.google.com/project/_/appengine/quotadetails) tab for your project. The quota is reset daily. You will get an [over quota exception](https://web.archive.org/web/20160425095930/https://cloud.google.com/appengine/docs/quotas#When_a_Resource_is_Depleted) if you exceed the daily quota. If you need a higher quota, you can [request a quota increase](https://web.archive.org/web/20160425095930/https://cloud.google.com/appengine/docs/quotas#up_mail_quota). See also [Quotas and Limits](#Java_Quotas_and_limits).
-   You should throttle sending of emails to avoid sending too many emails in a short burst, which could cause some emails to be silently dropped due to a safety limit on Google's side. You can calculate the maximum daily rate of sending emails per second by dividing your daily quota by 86,400, the number of seconds in a day. We recommend that you do not send bulk email with short bursts at higher than 50 times this long term rate.

### Subscription

-   Each user in your distribution list should opt-in to receive messages from you in one of the following ways:
    -   By sending you an email asking to subscribe
    -   By manually checking a box on a web form, or within a piece of software
-   Using an email address list purchased from a third-party is not considered opt-in. You also should not set a checkbox on a web form or within a piece of software to subscribe all users by default. Users should not be required to explicitly opt-out of mailings.
-   You should verify that the person that signed up by checking the box on the web form or in software is actually receiving emails at the address that was specified in the form, by sending an email that requires them to confirm receipt.

### Unsubscribing

-   A user must be able to unsubscribe in one of the following ways:
    -   Through a prominent link in the email with no further user interaction other than confirmation
    -   Via an email unsubscribe response
-   App Engine can only receive email sent to the `appid.appspotmail` domain. Therefore, you will need to set your sender to an address in this domain if you want to automatically handle email unsubscribe responses within App Engine.
-   Use the `List-Unsubscribe` header, which is [supported by the App Engine Mail API](#Java_Sending_mail_with_headers).
-   Automatically unsubscribe users whose addresses bounce multiple pieces of email. You can [configure your app to receive bounce notifications](#Java_Receiving_bounce_notification).
-   Periodically send email confirmations to users, offering the opportunity to unsubscribe from each list they are signed up for.
-   You should explicitly indicate the email address subscribed within your email because users may forward email from other accounts.

### Format

-   Format to [RFC 2822 SMTP standards](https://web.archive.org/web/20160425095930/http://www.ietf.org/rfc/rfc2822.txt) and, if using HTML, [w3.org standards](https://web.archive.org/web/20160425095930/http://www.w3.org/standards/techs/html).
-   Attempts to hide the true sender of the message or the true landing page for any web links in the message may result in non-delivery. For example, we recommend that you do not use URL shortener services in bulk email, since these can mask the real URLs contained in the body of your email.
-   The subject of each message should be relevant to the body's content and not be misleading.

### Delivery

-   The following factors will help messages arrive in Gmail users' inboxes:
    -   The `From` address is listed in the user's Contacts list.
    -   A user clicks "Not Spam" to alert Gmail that messages sent from that address are solicited.
-   If you send both promotional email and transactional email relating to your organization, we recommend separating email by purpose as much as possible. You can do this by:
    -   Using separate email addresses for each function.
    -   Sending email from different domains for each function.

### Third-party senders

-   If others use your service to send email, you are responsible for monitoring your users and/or clients' behavior. You must terminate, in a timely fashion, all users and/or clients who use your service to send spam email. The [Google Cloud Platform Acceptable Use Policy](https://web.archive.org/web/20160425095930/https://developers.google.com/cloud/terms/aup) specifically prohibits spam. Your application can be suspended if you violate this policy, as described in the [Google Cloud Platform Terms of Service](https://web.archive.org/web/20160425095930/https://developers.google.com/cloud/terms).
-   You must have an email address available for users and/or clients to report abuse, which should normally be `abuse@yourdomain.com`. You should also monitor `postmaster@yourdomain.com`.
-   Monitor email sent to app admins. Google may need to urgently contact app admins, for example to notify you of a violation of the Acceptable Use Policy. We can help you to resolve the problems more quickly if you respond promptly to our emails.
-   You must maintain up-to-date contact information in your WHOIS record maintained by your domain registrar, and on [abuse.net](https://web.archive.org/web/20160425095930/http://abuse.net/).

### Affiliate marketing programs

-   Affiliate marketing programs reward third-parties for bringing visitors to your site. These programs are attractive to spammers and can potentially do more harm than good. Please note the following:
    -   If your brand becomes associated with affiliate marketing spam, it can affect the email sent by you and your other affiliates.
    -   It is your responsibility to monitor your affiliates and remove them if they send spam.

### Alternatives to the App Engine Mail API

-   You can use a third-party email delivery service provider to send email from App Engine. These services may provide additional features that are not available in the Mail API and may be a better solution for some bulk email senders.
-   You can use the [Sockets API](https://web.archive.org/web/20160425095930/https://cloud.google.com/appengine/docs/java/sockets) to connect directly to an SMTP server to send email.

## Quotas and limits

Each Mail service request counts toward the **Mail API Calls** quota.

Each recipient email address for an email message counts toward the **Recipients Emailed (billable)** quota. Each recipient that is an administrator for the application also counts toward the **Admins Emailed** quota.

Data sent in the body of an email message counts toward the following quotas:

-   **Outgoing Bandwidth (billable)**
-   **Message Body Data Sent**

Each attachment included with an email message counts toward the **Attachments Sent** quota.

Data sent as an attachment to an email message counts toward the following quotas:

-   **Outgoing Bandwidth (billable)**
-   **Attachment Data Sent**

For more information, see [Quotas](https://web.archive.org/web/20160425095930/https://cloud.google.com/appengine/docs/quotas) and [Quota Details](https://web.archive.org/web/20160425095930/https://cloud.google.com/appengine/docs/developers-console/#quotas). You can see the current quota usage of your app by visiting the Google Cloud Platform Console [Quota Details](https://web.archive.org/web/20160425095930/https://console.cloud.google.com/project/_/appengine/quotadetails) tab for your project.

**Note:** Paid applications must pay their first weekly bill before they can send mail messages beyond the free quota. Once the first charge is cleared, each message sent above the free quota is charged at the normal paid rate.

In addition to quotas, the following limits apply to the use of the Mail service:

<table>
<thead>
<tr class="header">
<th>Limit</th>
<th>Amount</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>maximum size of outgoing mail messages, including attachments</td>
<td>31.5 megabytes</td>
</tr>
<tr class="even">
<td>maximum size of incoming mail messages, including attachments</td>
<td>31.5 megabytes</td>
</tr>
<tr class="odd">
<td>maximum size of message when an administrator is a recipient</td>
<td>16 kilobytes</td>
</tr>
</tbody>
</table>
