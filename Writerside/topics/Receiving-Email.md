# Receiving Email

  

Email messages sent to your app are implemented as HTTP requests. To process incoming email messages, you associate email addresses with servlets in your app configuration, then include the servlet code with your app. Incoming email generates HTTP requests, which are passed to the appropriate servlets for handling.

## Configuring Your App to Receive Email

When you create a new app, incoming email is disabled by default. To enable the incoming email service, you must modify your app's configuration files in two ways:

1.  In `appengine-web.xml`, add an `inbound-services` section that enables the incoming email service.
2.  In `web.xml`, add mappings that associate URL-mapped email addresses with servlets.

To enable the incoming email service, add a short section to the root element of your `appengine-web.xml` file, like the following:

```
<inbound-services>
  <service>mail</service>
</inbound-services>
```

You can verify that you have enabled incoming email in your app by going to the Administration Console and checking the Application Settings section. Note that you can only check there to see if the incoming email service is enabled; you must change the `appengine-web.xml` file to actually enable or disable the service. If you don't enable incoming email by including this section in your configuration file, incoming email is disabled, and email messages sent to the app are ignored.

Your app can receive email at addresses of the following form:

```
  string@appid.appspotmail.com
```

Email messages are sent to your app as HTTP POST requests using the following URL:

```
  /_ah/mail/<address>
```

where *address* is a full email address, including domain name. To handle incoming email in your app, you map email URLs to servlets in the `web.xml` file:

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

In this example, `/_ah/mail/*` matches all email addressed to the app. Mail servlets run in the default module (or application version).

### Pattern-Based Dispatching of Incoming Messages

If your app uses pattern matching, consider using a filter-based approach based on the following code snippets.

#### Concrete Handler

```
public class HandleDiscussionEmail extends MailHandlerBase {

  public HandleDiscussionEmail() { super("discuss-(.*)@(.*)"); }

  @Override
  protected boolean processMessage(HttpServletRequest req, HttpServletResponse res)
    throws ServletException
  {
    MimeMessage msg = getMessageFromRequest(req);
    Matcher match = getMatcherFromRequest(req);
    ...
  }

}
```

The concrete handler would be registered using the following snippet in web.xml

```
  <filter>
      <filter-name>HandleDiscussionEmail</filter-name>
      <filter-class>pkg.HandleDiscussionEmail</filter-class>
  </filter>
  <filter-mapping>
      <filter-name>HandleDiscussionEmail</filter-name>
      <url-pattern>/_ah/mail/*</url-pattern>
  </filter-mapping>
```

**Note:** `security-constraint` directives are not possible on filters; security policies on the handler will have to be introduced some other way.

#### <span id="AbstractHandler"></span>Abstract Handler

```
public abstract class MailHandlerBase implements Filter {

  private Pattern pattern = null;

  protected MailHandlerBase(String pattern) {
    if (pattern == null || pattern.trim().length() == 0)
    {
      throw new IllegalArgumentException("Expected non-empty regular expression");
    }
    this.pattern = Pattern.compile("/_ah/mail/"+pattern);
  }

  @Override public void init(FilterConfig config) throws ServletException { }

  @Override public void destroy() { }

  /**
   * Process the message. A message will only be passed to this method
   * if the servletPath of the message (typically the recipient for
   * appengine) satisfies the pattern passed to the constructor. If
   * the implementation returns false, control is passed
   * to the next filter in the chain. If the implementation returns
   * true, the filter chain is terminated.
   *
   * The Matcher for the pattern can be retrieved via
   * getMatcherFromRequest (e.g. if groups are used in the pattern).
   */
  protected abstract boolean processMessage(HttpServletRequest req, HttpServletResponse res) throws ServletException;

  @Override
  public void doFilter(ServletRequest sreq, ServletResponse sres, FilterChain chain)
      throws IOException, ServletException {

    HttpServletRequest req = (HttpServletRequest) sreq;
    HttpServletResponse res = (HttpServletResponse) sres;

    MimeMessage message = getMessageFromRequest(req);
    Matcher m = applyPattern(req);

    if (m != null && processMessage(req, res)) {
      return;
    }

    chain.doFilter(req, res); // Try the next one

  }

  private Matcher applyPattern(HttpServletRequest req) {
    Matcher m = pattern.matcher(req.getServletPath());
    if (!m.matches()) m = null;

    req.setAttribute("matcher", m);
    return m;
  }

  protected Matcher getMatcherFromRequest(ServletRequest req) {
    return (Matcher) req.getAttribute("matcher");
  }

  protected MimeMessage getMessageFromRequest(ServletRequest req) throws ServletException {
    MimeMessage message = (MimeMessage) req.getAttribute("mimeMessage");
    if (message == null) {
      try {
        Properties props = new Properties();
        Session session = Session.getDefaultInstance(props, null);
        message = new MimeMessage(session, req.getInputStream());
        req.setAttribute("mimeMessage", message);

      } catch (MessagingException e) {
        throw new ServletException("Error processing inbound message", e);
      } catch (IOException e) {
        throw new ServletException("Error processing inbound message", e);
      }
    }
    return message;
  }



}
```

## Handling Incoming Email

The JavaMail API includes the `MimeMessage` class which you can use to parse incoming email messages. `MimeMessage` has a constructor that accepts a `java.io.InputStream` and a JavaMail session, which can have an empty configuration.

You can create a `MimeMessage` instance like this:

```
import java.io.IOException; 
import java.util.Properties; 
import javax.mail.Session; 
import javax.mail.internet.MimeMessage; 
import javax.servlet.http.*; 

public class MailHandlerServlet extends HttpServlet { 
    @Override
    public void doPost(HttpServletRequest req, 
                       HttpServletResponse resp) 
            throws IOException { 
        Properties props = new Properties(); 
        Session session = Session.getDefaultInstance(props, null); 
        MimeMessage message = new MimeMessage(session, req.getInputStream());
```

You can then use various methods to parse the `message` object:

-   Call `getFrom()` to return the sender of the message.
-   The `getContent()` method returns an object that implements the `Multipart` interface. You can then call `getCount()` to determine the number of parts and getBodyPart(int index) to return a particular body part.

Once you set up your app to handle incoming email, you can use the development server console to simulate incoming email messages. You can access the development server by going to [http://localhost:8888/\_ah/admin/](https://web.archive.org/web/20160424231029/http://localhost:8888/_ah/admin/) (or if your app is running on a port other than 8888, use that value instead). In the development server, click Inbound Mail on the left side, fill out the form that appears, and click Send Email. To learn more, including how to get the development server running, see [The Java Development Server](https://web.archive.org/web/20160424231029/https://cloud.google.com/appengine/docs/java/tools/devserver.html).
