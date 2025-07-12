# Receiving Bounce Notification

  

To receive email bounce notifications, you need to configure your app to enable bounce notification and you need to handle the incoming notification in your app.

## Configuring Apps for Email Bounce Notification

By default, applications do not receive bounce notifications for email that could not be delivered. To enable the incoming bounce notification service, you must modify your app's `appengine-web.xml` and `web.xml` configuration files:

1.  In `appengine-web.xml`, add an inbound-services section to enable the incoming bounce service as follows:

    ```
    <inbound-services>
      <service>mail_bounce</service>
    </inbound-services>
    ```

2.  In `web.xml`, add a mapping that associate the bounce URL `/_ah/bounce` with your bounce handling servlet, as follows:

    ```
    <servlet>
      <servlet-name>bouncehandler</servlet-name>
      <servlet-class>BounceHandlerServlet</servlet-class>
    </servlet>
    <servlet-mapping>
      <servlet-name>bouncehandler</servlet-name>
      <url-pattern>/_ah/bounce</url-pattern>
    </servlet-mapping>
    <security-constraint>
      <web-resource-collection>
        <web-resource-name>bounce</web-resource-name>
        <url-pattern>/_ah/bounce</url-pattern>
      </web-resource-collection>
      <auth-constraint>
        <role-name>admin</role-name>
      </auth-constraint>
    </security-constraint>
    ```

## Handling Bounce Notifications

The JavaMail API includes the [BounceNotificationParser class](https://web.archive.org/web/20160424230351/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/mail/BounceNotificationParser), which you use to parse incoming bounce notifications, as shown in this snippet:

```
import java.io.IOException;
import javax.servlet.http.*;
import com.google.appengine.api.mail.BounceNotification;
import com.google.appengine.api.mail.BounceNotificationParser;

public class BounceHandlerServlet extends HttpServlet {
    @Override
    public void doPost(HttpServletRequest req,
                       HttpServletResponse resp)
            throws IOException {
       BounceNotification bounce = BounceNotificationParser.parse(req);
       // The following data is available in a BounceNotification object
       // bounce.getOriginal().getFrom() 
       // bounce.getOriginal().getTo() 
       // bounce.getOriginal().getSubject() 
       // bounce.getOriginal().getText() 
       // bounce.getNotification().getFrom() 
       // bounce.getNotification().getTo() 
       // bounce.getNotification().getSubject() 
       // bounce.getNotification().getText() 
       ...
   }
```
