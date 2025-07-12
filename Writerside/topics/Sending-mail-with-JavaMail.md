# Sending mail with JavaMail

  

The Mail service Java API supports the <a href="https://web.archive.org/web/20160424230725/http://www.oracle.com/technetwork/java/javamail/index.html" class="external" target="java">JavaMail</a> (javax.mail) interface for sending email messages.

1.  [Sending email messages](#sending_email_messages)
2.  [Senders and recipients](#senders_and_recipients)
3.  [Messages and headers](#messages_and_headers)
4.  [Multi-part messages](#multi-part_messages)
5.  [JavaMail features not supported](#javamail_features_not_supported)
6.  [Features of the low-level API](#features_of_the_low-level_api)

<span id="Sending_Email_Messages"></span>

## Sending email messages

To send an email message, an app prepares a MimeMessage object, then sends it with the static method `send()` on the `Transport` class. The message is created using a JavaMail `Session` object. The `Session` and the `Transport` work with the App Engine Mail service without any additional configuration.

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

Calls to the Mail service are asynchronous, and return immediately. The Mail service manages the process of contacting the recipients' mail servers and delivering the message. If there is a problem sending the message to any recipient, or if a recipient's mail server returns a "bounce" message, the error message goes to the sender.

<span id="Senders_and_Recipients"></span>

## Senders and recipients

Sender and recipient email addresses are represented in JavaMail using instances of the `InternetAddress` class. The constructor takes the email address as a string, and raises an `AddressException` if the address does not resemble a valid email address. Optionally, you can provide a personal name as a string for the second parameter.

To set the sender address, the app calls the `setFrom()` method on the `MimeMessage` object. See the [Overview](https://web.archive.org/web/20160424230725/https://cloud.google.com/appengine/docs/java/mail/) section for information on which email addresses you can use for the Sender address.

Several methods on the `MimeMessage` object establish the recipients. The `addRecipient()` method takes a recipient type and an address, and adds it to the list of recipients for the type. The recipient type can be any of `Message.RecipientType.TO`, `Message.RecipientType.CC` or `Message.RecipientType.BCC`.

You can set a "reply to" address using the `setReplyTo()` method.

<span id="Messages_and_Headers"></span>

## Messages and headers

You establish the contents of the message by calling methods on the `MimeMessage` object. The `setSubject()` method sets the subject, and the `setText()` method sets the (plaintext) body content.

The Mail service allows you to specify a limited set of headers on outgoing email messages. For more information about the allowed headers, see the [Mail Service Overview](https://web.archive.org/web/20160424230725/https://cloud.google.com/appengine/docs/java/mail/#Java_Sending_mail_with_headers).

<span id="Multi_Part_Messages"></span>

## Multi-part messages

You can send a message with file attachments, or with an HTML message body in addition to the plaintext message body, using multi-part messages. You create a `MimeMultipart` object to contain the parts, then create a `MimeBodyPart` object for each attachment or alternate message body and add it to the container. Finally, you assign the container to the MimeMessage's content.

```
import java.io.InputStream;
import javax.activation.DataHandler;
import javax.mail.Multipart;
import javax.mail.internet.MimeBodyPart;
import javax.mail.internet.MimeMultipart;

// ...
    String htmlBody;        // ...
    byte[] attachmentData;  // ...

    Multipart mp = new MimeMultipart();

    MimeBodyPart htmlPart = new MimeBodyPart();
    htmlPart.setContent(htmlBody, "text/html");
    mp.addBodyPart(htmlPart);

    MimeBodyPart attachment = new MimeBodyPart();
    InputStream attachmentDataStream = new ByteArrayInputStream(attachmentData);
    attachment.setFileName("manual.pdf");
    attachment.setContent(attachmentDataStream, "application/pdf");
    mp.addBodyPart(attachment);

    message.setContent(mp);
```

For security purposes, message parts and attachments must be of one of several allowed types, and attachment filenames must end in a recognized filename extension for the type. For a list of allowed types and filename extensions, see [Overview: Attachments](https://web.archive.org/web/20160424230725/https://cloud.google.com/appengine/docs/java/mail/#Java_Sending_mail_with_attachments).

<span id="JavaMail_Features_Not_Supported"></span>

## JavaMail features not supported

An app cannot use the JavaMail interface to connect to other mail services for sending or receiving email messages. SMTP configuration added to the `Transport` or `Session` is ignored.

<span id="Features_of_the_Low_level_API"></span>

## Features of the low-level API

All features of the low-level Mail service API are available in the JavaMail interface.

**Note:** The low-level API includes a convenience method for sending mail to all of the application's administrators. One way to use this feature is shown in the <a href="https://web.archive.org/web/20160424230725/http://jeremyblythe.blogspot.co.uk/2010/05/app-engine-email-to-admins-gotcha.html" class="external" target="_blank">following blog.</a>
