# Sending Email with SendGrid

  

**Note:** SendGrid is a third-party company whose services are not covered by the [Google App Engine Service Level Agreement](https://web.archive.org/web/20160424230431/https://cloud.google.com/appengine/sla).

You can use SendGrid to power your emails on Google App Engine. Using SendGrid can improve your deliverability and provide transparency into what actually happens to those emails your app sends. You can see statistics on opens, clicks, unsubscribes, spam reports and more through either the SendGrid interface or its API.

1.  [Pricing](#pricing)
2.  [Prerequisites](#prerequisites)
3.  [Sending an email](#sending_an_email)
4.  [Getting real-time information](#getting_real-time_information)
    1.  [Using the Event API](#using_the_event_api)
    2.  [Using the Inbound Parse API](#using_the_inbound_parse_api)

## Pricing

Google App Engine customers can send 12,000 emails every month for free. [Create a SendGrid account](https://web.archive.org/web/20160424230431/http://sendgrid.com/partner/google)\* to claim the free emails and to see higher volume plans.

## Prerequisites

To begin, you'll need to:

1.  [Create a SendGrid account](https://web.archive.org/web/20160424230431/http://sendgrid.com/partner/google)\* and as a Google App Engine developer, you can start with 12,000 free emails per month
2.  [Create a new project](https://web.archive.org/web/20160424230431/https://support.google.com/cloud/answer/6251787#create-a-project) or use an existing app

## Sending an email

It's easy to get started with the [Java library](https://web.archive.org/web/20160424230431/https://github.com/sendgrid/sendgrid-google-java) for SendGrid to send emails from your App Engine apps.

With the [prerequisites](#prerequisites) complete, make sure you are [set up for Java](https://web.archive.org/web/20160424230431/https://cloud.google.com/appengine/docs/java/gettingstarted/) on your local machine. The last thing you'll need before writing code is to copy [Sendgrid.java](https://web.archive.org/web/20160424230431/https://github.com/sendgrid/sendgrid-google-java) to the `src` directory of your app. You'll import this class so that you can create a SendGrid instance and send mail with simple commands.

At the top of your App Engine Servlet, import the `Sendgrid` library:

```
import packageName.Sendgrid
```

Now, from within your app, you can send email with the following few lines:

```
// set credentials
Sendgrid mail = new Sendgrid("<sendgrid_username>","<sendgrid_password>");

// set email data
mail.setTo("foo@bar.com").setFrom("me@bar.com").setSubject("Subject goes here").setText("Hello World!").setHtml("<strong>Hello World!</strong>");

// send your message
mail.send();
```

You'll want to add your own account details. Then edit the email address and other message content. Of course, you'll also need to upload your app either directly in Eclipse or through the `appcfg` on your local machine's command line.

If you'd rather start with a full, working app, check out this [sample Google App Engine app](https://web.archive.org/web/20160424230431/https://github.com/sendgrid/google-java-sample-app). For more information about email in your apps, browse the [SendGrid documentation](https://web.archive.org/web/20160424230431/http://sendgrid.com/docs/).

## Getting real-time information

In addition to sending email, SendGrid can help you receive email or make sense of the email you’ve already sent. The two real-time webhook solutions can greatly enhance the role email plays in your application.

### Using the Event API

Once you start sending email from your app, you'll want to know more about how it's performing. The statistics within SendGrid are one of its best features. The [Event API](https://web.archive.org/web/20160424230431/http://sendgrid.com/docs/API_Reference/Webhooks/event.html) lets you see all this data as one giant firehose. Whenever a recipient opens or clicks an email, for example, SendGrid can send a small bit of descriptive JSON to your Google App Engine app. You can react to the event or store the data for future use.

There are many different applications of event data. Some common uses are to integrate email stats into internal dashboards or use it to respond immediately to unsubscribes and spam reports. Advanced users of the Event API raise the engagement of their emails by sending only to those who have clicked or opened within the last few months.

The [Event API documentation](https://web.archive.org/web/20160424230431/http://sendgrid.com/docs/API_Reference/Webhooks/event.html) shows how to set up the webhook, outlines the nine event types and shows the fields included in event callbacks.

### Using the Inbound Parse API

You can also use SendGrid to receive email. The [Inbound Parse API](https://web.archive.org/web/20160424230431/http://sendgrid.com/docs/API_Reference/Webhooks/parse.html) can be used for powerful, interactive applications. For example, automate support tickets from customer emails, or collect data via short emails employees dispatch from the road. [NudgeMail's reminder application](https://web.archive.org/web/20160424230431/http://sendgrid.com/resources/success_stories/nudgemail-case-study) is even built entirely on the Parse API.

Like the Event API, the Parse API is a webhook that sends data to your application when something new is available. In this case, the webhook is called whenever a new email arrives at the domain you've associated with incoming email. Due to intricacies in the way DNS works for email, you need to assign all incoming mail to the domain or subdomain you use for the Parse API.

Emails are sent to your application structured as JSON, with sender, recipients, subject and body as different fields. You can even accept attachments, within SendGrid’s limit of messages up to 20 MB in size.

The [Parse API documentation](https://web.archive.org/web/20160424230431/http://sendgrid.com/docs/API_Reference/Webhooks/parse.html) has more details, including additional fields sent with every email, as well as instructions for DNS setup and usage.

\* *Google will be compensated for customers who sign up for a non-free account.*
