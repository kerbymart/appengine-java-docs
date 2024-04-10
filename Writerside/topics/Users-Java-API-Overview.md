# Users Java API Overview

  

[Python](https://web.archive.org/web/20160425095409/https://cloud.google.com/appengine/docs/python/users/ "View this page in the Python runtime") <span style="color: #ddd; padding: 0em .5em;">|</span><span style="color: black; font-weight:bold">Java</span> <span style="color: #ddd; padding: 0em .5em;">|</span>[PHP](https://web.archive.org/web/20160425095409/https://cloud.google.com/appengine/docs/php/users/ "View this page in the PHP runtime") <span style="color: #ddd; padding: 0em .5em;">|</span>[Go](https://web.archive.org/web/20160425095409/https://cloud.google.com/appengine/docs/go/users/ "View this page in the Go runtime")

[<img src="https://web.archive.org/web/20160425095409im_/https://cloud.google.com/cloud/images/stack_overflow_questions.png" title="Stack Overflow Questions" data-border="0" width="240" height="53" />](https://web.archive.org/web/20160425095409/http://stackoverflow.com/questions/tagged/google-app-engine+user)

[](https://web.archive.org/web/20160425095409/http://stackoverflow.com/feeds/tag?sort=votes&tagnames=google-app-engine%2Buser)

<span class="link gc-analytics-event" category="Sidebar" data-action="Stack Overflow question click"></span>

<a href="https://web.archive.org/web/20160425095409/http://stackoverflow.com/questions/tagged/google-app-engine+user?sort=votes" class="gc-analytics-event" data-category="Sidebar" data-action="Stack Overflow See more link">See more...</a>

App Engine applications can authenticate users using one of these methods: Google Accounts or accounts on your own Google Apps domains. An application can detect whether the current user has signed in, and can redirect the user to the appropriate sign-in page to sign in or, if your app uses Google Accounts authentication, create a new account. While a user is signed in to the application, the app can access the user's email address . The app can also detect whether the current user is an administrator, making it easy to implement admin-only areas of the app.

1.  [User authentication in Java](#Java_User_authentication_in_Java)
2.  [Authentication options](#Java_Authentication_options)
3.  [Signing in and out](#Java_Signing_in_and_out)
4.  [Accessing account information](#Java_Accessing_account_information)
5.  [Users and the Datastore](#Java_Users_and_the_Datastore)
6.  [Google accounts and the development server](#Java_Google_accounts_and_the_development_server)

## User authentication in Java

The following example greets a user who has signed in to the app with a personalized message and a link to sign out. If the user is not signed in, the app offers a link to the sign-in page for Google Accounts.

You can test if the user is signed in and get the user's email address using the standard servlet API, with the request object's `getUserPrincipal()` method. You can use the User service API to generate sign-in and sign-out URLs.

```
import java.io.IOException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import com.google.appengine.api.users.UserService;
import com.google.appengine.api.users.UserServiceFactory;

public class MyServlet extends HttpServlet {
    @Override
    public void doGet(HttpServletRequest req, HttpServletResponse resp)
            throws IOException {
        UserService userService = UserServiceFactory.getUserService();

        String thisURL = req.getRequestURI();

        resp.setContentType("text/html");
        if (req.getUserPrincipal() != null) {
            resp.getWriter().println("<p>Hello, " +
                                     req.getUserPrincipal().getName() +
                                     "!  You can <a href=\"" +
                                     userService.createLogoutURL(thisURL) +
                                     "\">sign out</a>.</p>");
        } else {
            resp.getWriter().println("<p>Please <a href=\"" +
                                     userService.createLoginURL(thisURL) +
                                     "\">sign in</a>.</p>");
        }
    }
}
```

The User service API can return the current user's information as a User object. Although User objects can be stored as a property value in the datastore, we strongly recommend that you avoid doing so because this includes the email address along with the user's unique ID. If a user changes their email address and you compare their old, stored `User` to the new `User` value, they won't match. Instead, consider using the `User` *user ID value* as the user's stable unique identifier.

### Enforcing sign in and admin access with web.xml

If you have pages that the user should not be able to access unless signed in, you can establish a security constraint for those pages in the deployment descriptor (the [`web.xml`](https://web.archive.org/web/20160425095409/https://cloud.google.com/appengine/docs/java/config/webxml) file). If a user accesses a URL with a security constraint and the user is not signed in, App Engine redirects the user to the sign-in page automatically (for Google Accounts or Google Apps authentication), then directs the user back to the URL after signing in or registering successfully.

A security constraint can also require that the user be a registered administrator for the application. This makes it easy to build administrator-only sections of the site, without having to implement a separate authorization mechanism.

To learn how to set security constraints for URLs, see [The Deployment Descriptor: Security and Authentication](https://web.archive.org/web/20160425095409/https://cloud.google.com/appengine/docs/java/config/webxml#Security_and_Authentication) for `web.xml`.

## Authentication options

Your app can authenticate users using one of these options:

-   A Google Account
-   An account on your Google Apps domain

### Choosing an authentication option

After you create your app, you can choose the authentication option you want to use. By default, your app will use Google Accounts for authentication. To choose another option, such as Google Apps domain, go to the [settings](https://web.archive.org/web/20160425095409/https://console.cloud.google.com//project/_/appengine/settings) page for your project in the Google Cloud Platform Console and click **Edit**. In the *Google authentication* dropdown menu, select the desired authentication type, and then click **Save**.

## Signing in and out

An application can detect whether a user has signed in to the app with your app's chosen authentication option. If the user is not signed in, the app can direct the user to Google Accounts to sign in or create a new Google account. The app gets the URL for the sign-in page by calling a method of the Users API. The app can display this URL as a link, or it can issue an HTTP redirect to the URL when the user visits a page that requires authentication.

If your app uses Google Accounts or Google Apps for authentication, the name of your application appears on the sign-in page when the user signs in to your application, using the application name you chose when registering the application. You can change your application name in the Google Cloud Platform Console [projects page](https://web.archive.org/web/20160425095409/https://console.cloud.google.com//project). Click on the pencil icon for the project you want to rename.

Once the user has signed in or created a Google account, the user is redirected back to your application. The app provides the redirect URL to the method that generates the sign-in URL.

The Users API includes a method to generate a URL for signing out of the app. The sign-out URL de-authenticates the user from the app, then redirects back to the app's URL without displaying anything.

A user is not signed in to an application until she is prompted to do so by the app and enters her account's email address and password. This is true even if the user has signed in to other applications using her Google Account.

## Accessing account information

While a user is signed in to an app, the app can access the account's email address for every request the user makes to the app. The app can also access a user ID that identifies the user uniquely, even if the user changes the email address for her account.

The app can also determine whether the current user is an administrator (a "developer") for the app. You can use this feature to build administrative features for the app, even if you don't authenticate other users. The Go, Java, PHP and Python APIs make it easy to configure URLs as "administrator only".

**Note:** Every user has the same user ID for all App Engine applications. If your app uses the user ID in public data, such as by including it in a URL parameter, you should use a hash algorithm with a "salt" value added to obscure the ID. Exposing raw IDs could allow someone to associate a user's activity in one app with that in another, or get the user's email address by coercing the user to sign in to another app.

## Users and the Datastore

The User service API can return the current user's information as a User object. Although User objects can be stored as a property value in the datastore, we strongly recommend that you avoid doing so because this includes the email address along with the user's unique ID. If a user changes their email address and you compare their old, stored `User` to the new `User` value, they won't match. Instead, consider using the `User` *user ID value* as the user's stable unique identifier.

## Google accounts and the development server

The development server simulates the Google Accounts system using a dummy sign-in screen. When your application calls the Users API to get the URL for the sign-in screen, the API returns a special development server URL that prompts for an email address, but no password. You can type any email address into this prompt, and the app will behave as if you are signed in with an account with that address.

The dummy sign-in screen also includes a checkbox that indicates whether the dummy account is an administrator. If you check this box, the app will behave as if you are signed in using an administrator account.

Similarly, the Users API returns a sign-out URL that cancels the dummy sign-in.
