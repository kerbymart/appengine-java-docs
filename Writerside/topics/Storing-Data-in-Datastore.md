# Storing Data in Datastore

  

1.  [Setting up Objectify](#setting_up_objectify)
2.  [Creating the data model classes](#creating_the_data_model_classes)
3.  [Adding the greetings and the form to the JSP template](#adding_the_greetings_and_the_form_to_the_jsp_template)
4.  [Creating the form handling servlet](#creating_the_form_handling_servlet)
5.  [Testing the app](#testing_the_app)
6.  [Creating required indexes](#creating_required_indexes)

In this section, you'll extend the simple application created earlier by adding the ability to post greetings and display previously posted greetings. You'll add a new servlet that handles the form submission and save the data.

The database we are using is [Google Cloud Datastore](https://web.archive.org/web/20160424225648/https://cloud.google.com/appengine/docs/java/datastore), a scalable, highly reliable data storage service that's easy to use from App Engine. Unlike a relational database, the Datastore is a schemaless object store, with an index-based query engine and transactional capabilities.

This tutorial uses the [Objectify](https://web.archive.org/web/20160424225648/https://github.com/objectify/objectify) library, a Java data modeling library for the Datastore. There are several alternative APIs you can use: a simple [low-level API](https://web.archive.org/web/20160424225648/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/package-summary), implementations of the [Java Data Objects (JDO)](https://web.archive.org/web/20160424225648/http://db.apache.org/jdo/index) and [Java Persistence API (JPA)](https://web.archive.org/web/20160424225648/http://www.oracle.com/technetwork/java/javaee/tech/persistence-jsp-140049.html) interfaces, or the [Slim3](https://web.archive.org/web/20160424225648/http://sites.google.com/site/slim3appengine/) library.

This is what the app will look like after you add the message submission form:

![Hello\_UI](https://web.archive.org/web/20160424225648im_/https://cloud.google.com/appengine/docs/java/gettingstarted/images/hello-2.png)

You'll make these changes to your existing application:

-   Set up Objectify, and create data model classes for the guestbook.
-   Edit the JSP page to allow users to post greetings to the datastore, and to display all the greetings currently stored.
-   Create a new servlet named `SignGuestbookServlet` that handles the form submission.
-   Add entries in `web.xml` so requests can be routed to the new servlet.

**Note:** If you're following along using [the Github repo for this tutorial](https://web.archive.org/web/20160424225648/https://github.com/GoogleCloudPlatform/appengine-java-guestbook-multiphase), the code in this section is in the **final** subdirectory.

## Setting up Objectify

To use the Objectify library, you must declare the dependency in the `pom.xml` file, so Maven can install it. In the application root directory (`guestbook/`), edit `pom.xml`, locate the `<dependencies>` section, then add the following:

```
<dependency>
    <groupId>com.googlecode.objectify</groupId>
    <artifactId>objectify</artifactId>
    <version>4.0.1</version>
</dependency>
```

Maven will install the library the next time you run a target that needs it (such as the development server).

In order to use Objectify in a JSP, we need a helper class that registers the model classes in the JSP servlet context. In `guestbook/src/main/java/com/example/guestbook/`, create a file named `OfyHelper.java` with the following contents:

\[This section requires a browser that supports JavaScript and iframes.\]

This file refers to the `Greeting` and `Guestbook` model classes that you'll create in the next section.

Finally, in `guestbook/src/main/webapp/WEB-INF/`, edit `web.xml`, and add the following lines inside `<web-app>` to set up the JSP helper:

\[This section requires a browser that supports JavaScript and iframes.\]

## Creating the data model classes

With Objectify, you create classes whose instances will represent *datastore entities* in your code. Objectify does the work of translating these Java objects to datastore entities.

Create a `Greeting` class to represent a message posted by a user. In `guestbook/src/main/java/com/example/guestbook/`, create a file named `Greeting.java` with the following contents:

\[This section requires a browser that supports JavaScript and iframes.\]

We also need a `Guestbook` class to represent an entire guestbook. While this example doesn't explicitly create a `Guestbook` object in the datastore, it uses this class as part of the *datastore key* for the `Greeting` objects. This demonstrates how you might define keys so that all of the greetings in a guestbook could be updated in a single *datastore transaction*.

Create a file named `Guestbook.java` with the following contents:

\[This section requires a browser that supports JavaScript and iframes.\]

## Adding the greetings and the form to the JSP template

In `guestbook/src/main/webapp`, edit `guestbook.jsp`. Add the following lines to the imports at the top:

\[This section requires a browser that supports JavaScript and iframes.\]

Just above the line `<form action="/guestbook.jsp" method="get">`, add the following code:

\[This section requires a browser that supports JavaScript and iframes.\]

When the user loads the page, the template performs a *datastore query* for all `Greeting` entities that use the same `Guestbook` in their key. The query is sorted by date in reverse chronological order (`"-date"`), and results are limited to the five most recent greetings. The rest of the template displays the results as HTML, and also renders the form that the user can use to post a new greeting.

See [Datastore Queries](https://web.archive.org/web/20160424225648/https://cloud.google.com/appengine/docs/java/datastore/queries) to learn more about queries.

## Creating the form handling servlet

The web form sends an HTTP POST request to the `/sign` URL. Let's create a servlet to handle these requests.

In `guestbook/src/main/java/com/example/guestbook`, create a file named `SignGuestbookServlet.java`, and give it the following contents:

\[This section requires a browser that supports JavaScript and iframes.\]

**Note:** `SignGuestbookServlet.java` replaces the functionality provided by `GuestbookServlet.java`, which is now no longer needed.

This servlet is the POST handler for the greetings posted by users. It takes the the greeting (`content`) from the incoming request and stores it in Datastore as a `Greeting` entity. Its properties include the content of the post, the date the post was created, and, if the user is signed in, the author's ID and email address. For more information about entities in App Engine, see [Entities, Properties, and Keys](https://web.archive.org/web/20160424225648/https://cloud.google.com/appengine/docs/java/datastore/entities).

Finally, in `guestbook/src/main/webapp/WEB-INF/`, edit `web.xml` and ensure the new servlets have URLs mapped. The final version should look like this:

\[This section requires a browser that supports JavaScript and iframes.\]

## Testing the app

Restart the development server to build and run the app:

```
mvn appengine:devserver
```

Visit the app in a browser:

[localhost:8080](https://web.archive.org/web/20160424225648/https://cloud.google.com/appengine/docs/java/gettingstarted/localhost:8080)

Try posting a greeting. Also try posting a greeting while signed in, and while not signed in.

## Creating required indexes

If your app uses the Datastore, when you run your app on the development server, the development server automatically creates any Datastore indexes required to run in production App Engine. These indexes are generated in `guestbook/target/guestbook-1.0-SNAPSHOT/WEB-INF/appengine-generated/datastore-indexes-auto.xml`. You'll also notice the file `local_db.bin` at that same location: this is local storage for the development server to persist your app data between development server sessions. The file `datastore-indexes-auto.xml` is automatically uploaded along with your app. `local_db.bin` is not uploaded.

**Important:** Be aware that running `mvn clean install` clears the `datastore-indexes-auto.xml` file; if you run it on your app prior to [uploading to production App Engine](https://web.archive.org/web/20160424225648/https://cloud.google.com/appengine/docs/java/gettingstarted/uploading), you won't get required indexes and you'll experience a runtime error. You only need to include indexes that are new since the last successful upload. Omitted indexes are not automatically removed.

For complete information about indexes, see [Datastore Indexes](https://web.archive.org/web/20160424225648/https://cloud.google.com/appengine/docs/java/datastore/indexes).

------------------------------------------------------------------------

We now have a working guest book application that authenticates users using Google accounts, lets them submit messages, and displays messages other users have left.

The only thing left to do is upload our application...

<a href="https://web.archive.org/web/20160424225648/https://cloud.google.com/appengine/docs/java/gettingstarted/uploading" class="button">Uploading Your Application &gt;&gt;</a>
