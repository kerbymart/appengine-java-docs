# Adding Application Code and UI

  

1.  [Creating a JSP template](#creating_a_jsp_template)
    1.  [Creating a CSS file for the JSP](#creating_a_css_file_for_the_jsp)
2.  [Testing the app in a development server](#testing_the_app_in_a_development_server)
3.  [Creating a servlet class](#creating_a_servlet_class)
4.  [Configuring URL paths](#configuring_url_paths)
5.  [Testing the finished sample](#testing_the_finished_sample)
6.  [Project directory structure and files](#project_directory_structure_and_files)
7.  [Next](#next)

In this section, you'll create an app that integrates with Google Accounts so users can sign in using their Google accounts. An app can integrate with Google accounts using the App Engine [Users](https://web.archive.org/web/20160424225709/https://cloud.google.com/appengine/docs/java/users/) service. You'll use this to personalize our application's greeting.

This is what the app will look like at the end of this section:

![Hello\_UI](https://web.archive.org/web/20160424225709im_/https://cloud.google.com/appengine/docs/java/gettingstarted/images/hello-1.png)

We will look at two ways to implement request handlers:

-   A [JSP template](https://web.archive.org/web/20160424225709/http://docs.oracle.com/javaee/5/tutorial/doc/bnagy.html)
-   A servlet class based on [javax.servlet.http.HttpServlet](https://web.archive.org/web/20160424225709/https://tomcat.apache.org/tomcat-5.5-doc/servletapi/javax/servlet/http/HttpServlet.html)

**Note:** If you're following along using [the Github repo for this tutorial](https://web.archive.org/web/20160424225709/https://github.com/GoogleCloudPlatform/appengine-java-guestbook-multiphase), the code in this section is in the **phase1** subdirectory. In later steps, we will extend this example to match the code in the **final** subdirectory.

## Creating a JSP template

By default, any file in `webapp/` or in a subdirectory other than `WEB-INF/` that has the file suffix `.jsp` is automatically mapped to a URL path consisting of the path to the `.jsp` file, including the filename. This JSP will be mapped automatically to the URL `/guestbook.jsp`:

Let's start with a simple JSP template. In `guestbook/src/main/webapp`, create a file named `guestbook.jsp` with the following contents:

<a href="https://web.archive.org/web/20160424225709/https://github.com/GoogleCloudPlatform/appengine-java-guestbook-multiphase/blob/master/phase1/src/main/webapp/guestbook.jsp" target="_blank" style="color: white;">phase1/src/main/webapp/guestbook.jsp</a>

<a href="https://web.archive.org/web/20160424225709/https://github.com/GoogleCloudPlatform/appengine-java-guestbook-multiphase/blob/master/phase1/src/main/webapp/guestbook.jsp" class="button" target="_blank" data-track-type="github" data-track-name="gitHubViewButton" data-track-metadata-link-destination="https://github.com/GoogleCloudPlatform/appengine-java-guestbook-multiphase/blob/master/phase1/src/main/webapp/guestbook.jsp">View on GitHub</a>

\[This section requires a browser that supports JavaScript and iframes.\]

This JSP imports the library for the App Engine `Users` service, and calls the service to detect whether the user is signed in, fetch the user's "nickname," and calculate URLs that the user can visit to sign in or sign out.

### Creating a CSS file for the JSP

Our JSP template uses a CSS file, so create a directory named `stylesheets` in `guestbook/src/main/webapp`. In this new directory, create a file named `main.css` with the following contents:

<a href="https://web.archive.org/web/20160424225709/https://github.com/GoogleCloudPlatform/appengine-java-guestbook-multiphase/blob/master/phase1/src/main/webapp/stylesheets/main.css" target="_blank" style="color: white;">phase1/src/main/webapp/stylesheets/main.css</a>

<a href="https://web.archive.org/web/20160424225709/https://github.com/GoogleCloudPlatform/appengine-java-guestbook-multiphase/blob/master/phase1/src/main/webapp/stylesheets/main.css" class="button" target="_blank" data-track-type="github" data-track-name="gitHubViewButton" data-track-metadata-link-destination="https://github.com/GoogleCloudPlatform/appengine-java-guestbook-multiphase/blob/master/phase1/src/main/webapp/stylesheets/main.css">View on GitHub</a>

\[This section requires a browser that supports JavaScript and iframes.\]

## Testing the app in a development server

**Note:** The feature shown here (Users service) is fully operational only when you deploy the application to App Engine. If you test in the local development server, as shown below, the current user isn't checked, and you can supply any name or email you want to the login form.

You can test this JSP file in the App Engine development server without any further changes:

1.  From the `guestbook` directory, run the following Maven command:

    ```
     mvn appengine:devserver
    ```

2.  Wait for the success message, which looks something like this:

    ```
     [INFO] INFO: Module instance default is running at http://localhost:8080/
     [INFO] Apr 29, 2015 3:28:38 PM com.google.appengine.tools.development.AbstractModule startup
     [INFO] INFO: The admin console is running at http://localhost:8080/_ah/admin
     [INFO] Apr 29, 2015 3:28:38 PM com.google.appengine.tools.development.DevAppServerImpl doStart
     [INFO] INFO: Dev App Server is now running
    ```

    **Note:** If there is a failure indicating that the App Engine Maven plugin is missing, this could be due to the fact that a new version was recently pushed to the Maven repo. To remedy this, invoke Maven with the -U option: `mvn appengine:devserver -U`

3.  In your browser, visit this URL:

    [http://localhost:8080/guestbook.jsp](https://web.archive.org/web/20160424225709/http://localhost:8080/guestbook.jsp)

4.  The app serves a page inviting you to sign in. Try clicking the "Sign in" link, then sign in with any email address. The development server emulates the Google Account sign-in process for testing purposes. While signed in, the app displays a greeting with the email address you entered.

    You may notice that visiting the URL [http://localhost:8080](https://web.archive.org/web/20160424225709/http://localhost:8080/) (with no path) returns an error. So far, we have only defined a handler for the `/guestbook.jsp` URL path by creating this file in `guestbook/src/main/webapp/`. We will configure more URL handlers in the next section.

    If you get a runtime error when you first visit `localhost:8080`, referring to a restricted class, for example, `java.lang.NoClassDefFoundError: java.nio.charset.StandardCharsets is a restricted class.`, check the settings for `guestbook/pom.xml`. The App Engine version must be set to the most recent App Engine SDK version: 1.9.34.

5.  Terminate the development server by pressing Control-C.

## Creating a servlet class

App Engine Java applications use the [Java Servlet API](https://web.archive.org/web/20160424225709/http://download.oracle.com/javaee/5/api/javax/servlet/Servlet.html) to interact with the web server. As above, JSP files act as servlets mapped to URL paths similar to their file paths. You can also implement servlets using Java servlet classes. A servlet class extends either the [javax.servlet.GenericServlet](https://web.archive.org/web/20160424225709/http://download.oracle.com/javaee/5/api/javax/servlet/GenericServlet.html) class or the [javax.servlet.http.HttpServlet](https://web.archive.org/web/20160424225709/http://download.oracle.com/javaee/5/api/javax/servlet/http/HttpServlet.html) class.

In the `guestbook/src/main/java` directory, create a package path for your servlet class called `com/example/guestbook`. In Linux or Mac OS X, you can run the following command:

```
mkdir -p com/example/guestbook
```

In this new directory, create a file named `GuestbookServlet.java`, and give it the following contents:

<a href="https://web.archive.org/web/20160424225709/https://github.com/GoogleCloudPlatform/appengine-java-guestbook-multiphase/blob/master/phase1/src/main/java/com/example/guestbook/GuestbookServlet.java" target="_blank" style="color: white;">phase1/src/main/java/com/example/guestbook/GuestbookServlet.java</a>

<a href="https://web.archive.org/web/20160424225709/https://github.com/GoogleCloudPlatform/appengine-java-guestbook-multiphase/blob/master/phase1/src/main/java/com/example/guestbook/GuestbookServlet.java" class="button" target="_blank" data-track-type="github" data-track-name="gitHubViewButton" data-track-metadata-link-destination="https://github.com/GoogleCloudPlatform/appengine-java-guestbook-multiphase/blob/master/phase1/src/main/java/com/example/guestbook/GuestbookServlet.java">View on GitHub</a>

\[This section requires a browser that supports JavaScript and iframes.\]

The code above checks for the user currently logged into a Google Account, and if the user isn't logged in, displays a login form for the user to log in. Note that this behavior is what happens when you deploy. If you test it locally, the current user isn't checked, and you can supply any name or email you want to the login form.

The servlet also demonstrates how to read query parameters with the `getParameter()` method.

## Configuring URL paths

The servlet engine needs to know which URL paths go to each servlet class. You specify this using the [deployment descriptor](https://web.archive.org/web/20160424225709/https://cloud.google.com/appengine/docs/java/config/webxml), a file named `web.xml`.

Locate `web.xml` in the `guestbook/src/main/webapp/WEB-INF` directory. Edit the file, then give it the following contents:

<a href="https://web.archive.org/web/20160424225709/https://github.com/GoogleCloudPlatform/appengine-java-guestbook-multiphase/blob/master/phase1/src/main/webapp/WEB-INF/web.xml" target="_blank" style="color: white;">phase1/src/main/webapp/WEB-INF/web.xml</a>

<a href="https://web.archive.org/web/20160424225709/https://github.com/GoogleCloudPlatform/appengine-java-guestbook-multiphase/blob/master/phase1/src/main/webapp/WEB-INF/web.xml" class="button" target="_blank" data-track-type="github" data-track-name="gitHubViewButton" data-track-metadata-link-destination="https://github.com/GoogleCloudPlatform/appengine-java-guestbook-multiphase/blob/master/phase1/src/main/webapp/WEB-INF/web.xml">View on GitHub</a>

\[This section requires a browser that supports JavaScript and iframes.\]

This deployment descriptor does three things:

-   The `<servlet>` directive declares that the `GuestbookServlet` class is a servlet named `guestbook`.
-   The `<servlet-mapping>` directive associates the `guestbook` servlet with the URL path `/guestbook`.
-   The `<welcome-file-list>` directive tells the server to use the JSP file you created earlier to serve requests for the home page.

## Testing the finished sample

You can test the complete sample (JSP and servlet) as follows:

1.  Stop the development server with Control-C if it is still running, then restart it (again, from the `guestbook` directory):

    mvn appengine:devserver

2.  Visit the `/guestbook` URL to see the results of the `GuestbookServlet` class:

    [http://localhost:8080/guestbook](https://web.archive.org/web/20160424225709/http://localhost:8080/guestbook)

3.  This servlet demonstrates how to read query parameters with the `getParameter()` method. Try adding a `testing` parameter to the URL, to display testing information:

    [http://localhost:8080/guestbook?testing](https://web.archive.org/web/20160424225709/http://localhost:8080/guestbook?testing)

4.  Try visiting the root (`/`) URL to see that JSP file that used for the home page:

    [http://localhost:8080/](https://web.archive.org/web/20160424225709/http://localhost:8080/)

## Project directory structure and files

Your project directory structure looks like this:

![Simple Hello World](https://web.archive.org/web/20160424225709im_/https://cloud.google.com/appengine/docs/java/gettingstarted/images/phase1_layout.png)

You can see the JSP file (`guestbook.jsp`) and the servlet (`GuestBookServlet.java`) that we just added above, along with the stylesheet `main.css`, and the `web.xml` configuration file.

Notice also the file `appengine-web.xml`. This is the location where the version is specified. If you use the Maven artifacts and sample projects described in this tutorial, the version value in this file is set using the `pom.xml` file, as we'll show later in [Uploading Your Application](https://web.archive.org/web/20160424225709/https://cloud.google.com/appengine/docs/java/gettingstarted/uploading#specifying_version).

Finally, notice the file `logging.properties`, which is used if you want to write application logs, as described in [Writing application logs](https://web.archive.org/web/20160424225709/https://cloud.google.com/appengine/docs/java/logs/#Java_writing_application_logs)

## Next

Continue to the next section to add persistent data storage functionality to the guestbook app.

<a href="https://web.archive.org/web/20160424225709/https://cloud.google.com/appengine/docs/java/gettingstarted/usingdatastore" class="button">Storing Data in Datastore &gt;&gt;</a>
