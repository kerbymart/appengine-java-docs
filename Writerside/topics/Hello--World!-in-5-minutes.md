# Hello, World! in 5 minutes

This tutorial creates a tiny application that displays a short message.

## Contents

1.  [Download Maven](#download_maven)
2.  [Creating a simple HTTPServlet](#creating_a_simple_httpservlet)
3.  [Understanding the configuration files](#understanding_the_configuration_files)
    1.  [pom.xml](#pomxml)
    2.  [appengine-web.xml](#appengine-webxml)
    3.  [web.xml](#webxml)
4.  [Testing the application](#testing_the_application)
5.  [Iterative development](#iterative_development)
6.  [Uploading the application](#uploading_the_application)
7.  [Congratulations!](#congratulations)
8.  [Expanding on Hello, World!](#expanding_on_hello_world)

## Download Maven

Before getting started, be sure to [Download Apache Maven](https://web.archive.org/web/20160424204148/https://maven.apache.org/download.cgi) version 3.1 or greater and then [install](https://web.archive.org/web/20160424204148/https://maven.apache.org/install) it. When you use Maven as described in these instructions, the required App Engine SDK libraries for Java will be automatically downloaded into your project.

## Creating a simple `HTTPServlet`

Clone the contents of [this GitHub project](https://web.archive.org/web/20160424204148/https://github.com/GoogleCloudPlatform/java-docs-samples/tree/master/appengine/helloworld) by running the following commands:

```
$ git clone https://github.com/GoogleCloudPlatform/java-docs-samples.git
$ cd  java-doc-samples/appengine/helloworld
```

In the resulting `helloworld` files you'll find the `src` directory for a package called `com.example.appengine.helloworld` that implements a simple `HTTPServlet`.

<a href="https://web.archive.org/web/20160424204148/https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/appengine/helloworld/src/main/java/com/example/appengine/helloworld/HelloServlet.java" target="_blank" style="color: white;">appengine/helloworld/src/main/java/com/example/appengine/helloworld/HelloServlet.java</a>

<a href="https://web.archive.org/web/20160424204148/https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/appengine/helloworld/src/main/java/com/example/appengine/helloworld/HelloServlet.java" class="button" target="_blank" data-track-type="github" data-track-name="gitHubViewButton" data-track-metadata-link-destination="https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/appengine/helloworld/src/main/java/com/example/appengine/helloworld/HelloServlet.java">View on GitHub</a>

\[This section requires a browser that supports JavaScript and iframes.\]

This servlet responds to any request by sending a response containing the message `Hello, world!`.

## Understanding the configuration files

There are three config files to look at in addition to `HelloServlet.java`.

### `pom.xml`

The `helloworld` app uses Maven, which means you must specify a [Project Object Model](https://web.archive.org/web/20160424204148/https://maven.apache.org/guides/introduction/introduction-to-the-pom.html), or POM, which contains information about the project and configuration details used by Maven to build the project.

<a href="https://web.archive.org/web/20160424204148/https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/appengine/helloworld/pom.xml" target="_blank" style="color: white;">appengine/helloworld/pom.xml</a>

<a href="https://web.archive.org/web/20160424204148/https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/appengine/helloworld/pom.xml" class="button" target="_blank" data-track-type="github" data-track-name="gitHubViewButton" data-track-metadata-link-destination="https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/appengine/helloworld/pom.xml">View on GitHub</a>

\[This section requires a browser that supports JavaScript and iframes.\]

### `appengine-web.xml`

You will also need to configure App Engine's behavior using `appengine-web.xml`. You can use this file to specify a version for your application. More importantly, you'll eventually need to enter the Project ID for your Google Cloud Platform project before you [upload this application](#uploading_the_application). But don't worry about that now.

<a href="https://web.archive.org/web/20160424204148/https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/appengine/helloworld/src/main/webapp/WEB-INF/appengine-web.xml" target="_blank" style="color: white;">appengine/helloworld/src/main/webapp/WEB-INF/appengine-web.xml</a>

<a href="https://web.archive.org/web/20160424204148/https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/appengine/helloworld/src/main/webapp/WEB-INF/appengine-web.xml" class="button" target="_blank" data-track-type="github" data-track-name="gitHubViewButton" data-track-metadata-link-destination="https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/appengine/helloworld/src/main/webapp/WEB-INF/appengine-web.xml">View on GitHub</a>

\[This section requires a browser that supports JavaScript and iframes.\]

### `web.xml`

Java web applications use a deployment descriptor file to determine how URLs map to servlets, which URLs require authentication, and other information. This file is named `web.xml`, and resides in the app's WAR under the `WEB-INF/` directory. `web.xml` is part of the servlet standard for web applications.

<a href="https://web.archive.org/web/20160424204148/https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/appengine/helloworld/src/main/webapp/WEB-INF/web.xml" target="_blank" style="color: white;">appengine/helloworld/src/main/webapp/WEB-INF/web.xml</a>

<a href="https://web.archive.org/web/20160424204148/https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/appengine/helloworld/src/main/webapp/WEB-INF/web.xml" class="button" target="_blank" data-track-type="github" data-track-name="gitHubViewButton" data-track-metadata-link-destination="https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/appengine/helloworld/src/main/webapp/WEB-INF/web.xml">View on GitHub</a>

\[This section requires a browser that supports JavaScript and iframes.\]

## Testing the application

Run the following Maven command from within your `helloworld` directory, to compile your app and start the development web server:

```
$ mvn appengine:devserver
```

The web server is now running, listening for requests on port 8080. Test the application by visiting the following URL in your web browser:

-   [http://localhost:8080/](https://web.archive.org/web/20160424204148/http://localhost:8080/)

## Iterative development

During development, if you start your app in the development web server in offline mode using the command `mvn appengine:devserver_start`, you can force the development web server to reload your app by doing the following:

1.  With the development web server running (started using the command `mvn appengine:devserver_start`), edit your code.
2.  Trigger a build. In Maven you can do this using the command `mvn install`. The development server will update with the new build.
3.  View the results in your web browser.
4.  When you are finished iterating, shut down the development web server using the command `mvn appengine:devserver_stop`.

Try it now: start the development web server using the command `mvn appengine:devserver_start`, edit `HelloServlet.java` to change `Hello, world!` to something else, then rebuild the project using `mvn install`. Reload [http://localhost:8080/](https://web.archive.org/web/20160424204148/http://localhost:8080/) to see the changes.

Note that to shut down a web server started using `mvn appengine:devserver_start`, you must use the command `mvn appengine:devserver_stop`. Using Control-C won't work.

Leave the web server running for the rest of this tutorial. If you need to stop it, you can restart it again by running the command above.

**Note:** You don't have to use Maven commands to force the development server to reload the app. If you use the NetBeans IDE, a rebuild is triggered on each file Save; this will cause the development server to reload the app. If you use IntelliJ, clicking on **Build** will also work.

## Uploading the application

1.  If you haven't already done so, create a project for your App Engine app as follows:

    1.  Visit the [Google Cloud Platform Console](https://web.archive.org/web/20160424204148/https://console.cloud.google.com/) and click **Create Project**.
    2.  Supply the desired project name in the New Project form. It doesn't have to match your app name, but using the same name as your app might make administration easier.
    3.  Accept the generated project ID or supply your own ID. *This project ID is used as the App Engine application ID.* Note that this ID can only be used once: if you subsequently delete your project, you won't be able to re-use the ID in a new project.

2.  Note the application ID (project ID) you created above, and enter it in `src/main/webapp/WEB-INF/appengine-web.xml`. While you're in there, set the version to something that suits you.

3.  Upload your finished application to Google App Engine by invoking the following command.

    ```
    mvn appengine:update
    ```

4.  Your app is now deployed and ready for users!

## Congratulations!

You have completed this tutorial.

The full URL for your application is `http://_your-app-id_.appspot.com/`. Optionally, you can instead purchase and use a top-level domain name for your app, or use one that you have already registered.

## Expanding on Hello, World!

In the "[Creating a Guestbook](https://web.archive.org/web/20160424204148/https://cloud.google.com/appengine/docs/java/gettingstarted/)" tutorial, you expand this simple application to become a fully-fledged Guestbook application that lets authenticated Google accounts post messages to a public page.

<a href="https://web.archive.org/web/20160424204148/https://cloud.google.com/appengine/docs/java/gettingstarted/introduction" class="button">Creating a Guestbook &gt;&gt;</a>
