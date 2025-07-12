# Using Google Cloud SQL

  

[Python](https://web.archive.org/web/20160424230503/https://cloud.google.com/appengine/docs/python/cloud-sql/ "View this page in the Python runtime") <span style="color: #ddd; padding: 0em .5em;">|</span><span style="color: black; font-weight:bold">Java</span> <span style="color: #ddd; padding: 0em .5em;">|</span>[PHP](https://web.archive.org/web/20160424230503/https://cloud.google.com/appengine/docs/php/cloud-sql/ "View this page in the PHP runtime") <span style="color: #ddd; padding: 0em .5em;">|</span>[Go](https://web.archive.org/web/20160424230503/https://cloud.google.com/appengine/docs/go/cloud-sql/ "View this page in the Go runtime")

[<img src="https://web.archive.org/web/20160424230503im_/https://cloud.google.com/cloud/images/stack_overflow_questions.png" title="Stack Overflow Questions" data-border="0" width="240" height="53" />](https://web.archive.org/web/20160424230503/http://stackoverflow.com/questions/tagged/google-app-engine+google-cloud-sql)

[](https://web.archive.org/web/20160424230503/http://stackoverflow.com/feeds/tag?sort=votes&tagnames=google-app-engine%2Bgoogle-cloud-sql)

<span class="link gc-analytics-event" category="Sidebar" data-action="Stack Overflow question click"></span>

<a href="https://web.archive.org/web/20160424230503/http://stackoverflow.com/questions/tagged/google-app-engine+google-cloud-sql?sort=votes" class="gc-analytics-event" data-category="Sidebar" data-action="Stack Overflow See more link">See more...</a>

This document describes how to use Google Cloud SQL instances with the App Engine Java SDK.

1.  [Creating a Cloud SQL instance](#create)
2.  [Build a starter application and database](#Java_Build_a_starter_application_and_database)
3.  [Connect to your database](#Java_Connect_to_your_database)
4.  [Using a local MySQL instance during development](#Java_Using_a_local_MySQL_instance_during_development)
5.  [Size and access limits](#Java_Size_and_access_limits)
6.  [Using Persistence APIs with Cloud SQL](#Java_Persistence_connectors)

To learn more about Google Cloud SQL, see the [Google Cloud SQL](https://web.archive.org/web/20160424230503/https://cloud.google.com/cloud-sql/) documentation.

**Note:** Access to Google Cloud SQL Second Generation instances can be granted only for apps running in the flexible environment. [Learn more.](https://web.archive.org/web/20160424230503/https://cloud.google.com/sql/docs/dev-access)

If you haven't already created a Google Cloud SQL instance, the first thing you need to do is create one.

## Creating a Cloud SQL Instance

A Cloud SQL instance is equivalent to a server. A single Cloud SQL instance can contain multiple databases. Follow these steps to create a Google Cloud SQL instance:

1.  Sign into the <a href="https://web.archive.org/web/20160424230503/https://console.cloud.google.com/" data-track-name="consoleLink" data-track-type="tutorial" data-track-metadata-position="sqlSignUp">Google Cloud Platform Console</a>.
2.  Create a new project, or open an existing project.
3.  From within a project, select **Cloud SQL** to open the Cloud SQL control panel for that project.
4.  Click **New Instance** to create a new Cloud SQL instance in your project, and configure your size, billing and replication options.
    -   [More information on Cloud SQL billing options and instance sizes](https://web.archive.org/web/20160424230503/https://cloud.google.com/cloud-sql/docs/billing)
    -   [More information on Cloud SQL replication options](https://web.archive.org/web/20160424230503/https://cloud.google.com/cloud-sql/faq#data_location)

    You'll notice that the App Engine application associated with your current project is already authorized to access this new instance. For more information on app authorization see the [Access Control](https://web.archive.org/web/20160424230503/https://cloud.google.com/cloud-sql/docs/access_control#appaccess) topic in the Cloud SQL docs.

That's it! You can now connect to your Google Cloud SQL instance from within your app, or any of [these other methods](https://web.archive.org/web/20160424230503/https://cloud.google.com/cloud-sql/docs/before_you_begin#connect).

#### MySQL case sensitivity

When you are creating or using databases and tables, keep in mind that all identifiers in Google Cloud SQL are case-sensitive. This means that all tables and databases are stored with the same name and case that was specified at creation time. When you try to access your databases and tables, make sure that you are using the exact database or table name.

For example, if you create a database named PersonsDatabase, you will not be able to reference the database using any other variation of that name, such as personsDatabase or personsdatabase. For more information about identifier case sensitivity, see the [MySQL documentation.](https://web.archive.org/web/20160424230503/http://dev.mysql.com/doc/refman/5.5/en/identifier-case-sensitivity.html)

## Build a starter application and database

The easiest way to build an App Engine application that accesses Google Cloud SQL is to create a starter application then modify it. This section leads you through the steps of building an application that displays a web form that lets users read and write entries to a guestbook database. The sample application demonstrates how to read and write to a Google Cloud SQL instance.

### Step 1: Create your App Engine sample application

Follow the instructions in [Creating a Project](https://web.archive.org/web/20160424230503/https://cloud.google.com/appengine/docs/java/gettingstarted/creating) to create a simple App Engine application.

### Step 2: Grant your App Engine application access to the Google Cloud SQL instance

You can grant individual Google App Engine applications access to a Google Cloud SQL instance. One application can be granted access to multiple instances, and multiple applications can be granted access to a particular instance. To grant access to a Google App Engine application, you need its application ID which can be found at the [Google App Engine administration console](https://web.archive.org/web/20160424230503/https://appengine.google.com/) under the **Applications** column.

**Note**: An App Engine application must be in the same region (either EU or US) as a Google Cloud SQL instance to be authorized to access that Google Cloud SQL instance.

**To grant an App Engine application access to a Google Cloud SQL instance:**

1.  In the [Google Cloud Platform Console](https://web.archive.org/web/20160424230503/https://console.cloud.google.com/project/_/sql/instances), select a project.

2.  Click the name of the instance to which you want to grant access.

3.  In the instance dashboard, click **Edit**.

4.  In the **Instance settings** window, enter your Google App Engine application ID in the **Authorized App Engine applications** section. You can grant access to multiple applications, by entering them one at a time.

    **Note**: In order to improve performance, the Google Cloud SQL instance will be kept as close as possible to the first App Engine application on the list, so this should be the application whose performance is most important.

5.  Click **Confirm** to apply your changes.

After you have added authorized applications to your Google Cloud SQL instance, you can view a list of these applications in the instance dashboard, in the section titled **Applications**.

### Step 3: Create your database and table

For example, you can use [MySQL Client](https://web.archive.org/web/20160424230503/https://cloud.google.com/cloud-sql/docs/mysql-client) to run the following commands:

1.  Create a new database called `guestbook` using the following SQL statement:

    ```
    CREATE DATABASE guestbook;
    ```

2.  Inside the `guestbook` database create a table called `entries` with columns for the guest name, the message content, and a random ID, using the following statement:

    ```
    CREATE TABLE guestbook.entries (
      entryID INT NOT NULL AUTO_INCREMENT,
      guestName VARCHAR(255),
      content VARCHAR(255),
      PRIMARY KEY(entryID)
    );
    ```

After you have set up a bare-bones application, you can modify it and deploy it.

## Connect to your database

One way to connect your Java App Engine application to a Cloud SQL databases is to use the [Google Plugin for Eclipse](https://web.archive.org/web/20160424230503/https://cloud.google.com/eclipse/). The following steps assume that you created an application following the [Creating a New Web Application](https://web.archive.org/web/20160424230503/https://cloud.google.com/eclipse/docs/creating_new_webapp) instructions. For the sample we are buiding here, we don't need the Google Web Toolkit (GWT) so you can create your web application without that SDK.

Note that the code below assumes that you created a web application project with a **Project name** specified as "Guestbook" and a **Package** specified as "guestbook". If you choose different values, you will have to make the appropriate changes in the servlet and web form.

1.  [Enable MySQL Connector/J](#enable_connector_j)
2.  [Connect and Post to Your Database](#connect_and_post)
3.  [Create Your Webform](#create_webform)
4.  [Map Your Servlet](#map_servlet)

### Enable MySQL Connector/J

The MySQL Connector/J is available in Google App Engine, but it's not included in an app unless the app explicitly asks for it. To do that, add a `<use-google-connector-j>` element as a child of the `<appengine-web-app>` element in the app's `appengine-web.xml` file as shown in the following example:

```
<?xml version="1.0" encoding="utf-8"?>
<appengine-web-app xmlns="http://appengine.google.com/ns/1.0">
  ...
  <use-google-connector-j>true</use-google-connector-j>
</appengine-web-app>
```

### Connect and post to your database

Replace `doGet()` in `GuestbookServlet` with the following code. Replacing `your-instance-name` with your Google Cloud SQL instance name and `your-project-id` with the literal project ID. This code performs the following actions:

-   Initiates the connection by calling `getConnection()`
-   Collects the contents from a web form and posting them to the server
-   Redirects the user to a file called `guestbook.jsp` (we will create it in a later step)

```
import java.io.*;
import java.sql.*;
import javax.servlet.http.*;
import com.google.appengine.api.utils.SystemProperty;

public class GuestbookServlet extends HttpServlet {
  @Override
  public void doPost(HttpServletRequest req, HttpServletResponse resp) throws IOException {
    String url = null;
    try {
      if (SystemProperty.environment.value() ==
          SystemProperty.Environment.Value.Production) {
        // Load the class that provides the new "jdbc:google:mysql://" prefix.
        Class.forName("com.mysql.jdbc.GoogleDriver");
        url = "jdbc:google:mysql://your-project-id:your-instance-name/guestbook?user=root";
      } else {
        // Local MySQL instance to use during development.
        Class.forName("com.mysql.jdbc.Driver");
        url = "jdbc:mysql://127.0.0.1:3306/guestbook?user=root";

        // Alternatively, connect to a Google Cloud SQL instance using:
        // jdbc:mysql://ip-address-of-google-cloud-sql-instance:3306/guestbook?user=root
      }
    } catch (Exception e) {
      e.printStackTrace();
      return;
    }

    PrintWriter out = resp.getWriter();
    try {
      Connection conn = DriverManager.getConnection(url);
      try {
        String fname = req.getParameter("fname");
        String content = req.getParameter("content");
        if (fname == "" || content == "") {
          out.println(
              "<html><head></head><body>You are missing either a message or a name! Try again! " +
              "Redirecting in 3 seconds...</body></html>");
        } else {
          String statement = "INSERT INTO entries (guestName, content) VALUES( ? , ? )";
          PreparedStatement stmt = conn.prepareStatement(statement);
          stmt.setString(1, fname);
          stmt.setString(2, content);
          int success = 2;
          success = stmt.executeUpdate();
          if (success == 1) {
            out.println(
                "<html><head></head><body>Success! Redirecting in 3 seconds...</body></html>");
          } else if (success == 0) {
            out.println(
                "<html><head></head><body>Failure! Please try again! " +
                "Redirecting in 3 seconds...</body></html>");
          }
        }
      } finally {
        conn.close();
      }
    } catch (SQLException e) {
      e.printStackTrace();
    }
    resp.setHeader("Refresh", "3; url=/guestbook.jsp");
  }
}
```

Although the above example connects to the Google Cloud SQL instance as the `root` user without any password, you can also connect to the instance indicating one:

```
Connection conn = DriverManager.getConnection(
    "jdbc:google:mysql://your-project-id:your-instance-name/database",
    "user", "password");
```

For information about creating MySQL users, see [Adding Users](https://web.archive.org/web/20160424230503/http://dev.mysql.com/doc/refman/5.5/en/adding-users.html) in the MySQL documentation.

#### Managing connections

An App Engine application is made up of one or more [modules](https://web.archive.org/web/20160424230503/https://cloud.google.com/appengine/docs/java/modules/#Java_Application_hierarchy). Each module consists of source code and configuration files. An instance instantiates the code which is included in an App Engine module, and a particular version of module will have one or more instances running. The number of instances running depends on the number of incoming requests. You can configure App Engine to scale the number of instances automatically in response to processing volume (see [Instance scaling and class](https://web.archive.org/web/20160424230503/https://cloud.google.com/appengine/docs/java/modules/#Java_Instance_scaling_and_class)).

When App Engine instances talk to Google Cloud SQL, each App Engine instance cannot have more than 12 concurrent connections to a Cloud SQL instance. Always close any established connections before finishing processing a request. Not closing a connection will cause it to leak and may eventually cause new connections to fail. You can exit this state by shutting down the affected App Engine instance.

You should also keep in mind that there is also a maximum number of concurrent connections and queries for each Cloud SQL instance, depending on the tier (see [Cloud SQL pricing](https://web.archive.org/web/20160424230503/https://cloud.google.com/cloud-sql/pricing)). For guidance on managing connections, see [How should I manage connections?](https://web.archive.org/web/20160424230503/https://cloud.google.com/cloud-sql/faq#connections) in the "Google Cloud SQL FAQ" document.

### Create your webform

Next, we'll create the front-facing part of the sample application, which list the entries of your entries table and provides a simple form to post new entries.

In your `war/` directory, create a new file called `guestbook.jsp` with the following code, replacing `instance_name` with your instance name:

```
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ page import="java.util.List" %>
<%@ page import="java.sql.*" %>
<%@ page import="com.google.appengine.api.utils.SystemProperty" %>

<html>
  <body>

<%
String url = null;
if (SystemProperty.environment.value() ==
    SystemProperty.Environment.Value.Production) {
  // Load the class that provides the new "jdbc:google:mysql://" prefix.
  Class.forName("com.mysql.jdbc.GoogleDriver");
  url = "jdbc:google:mysql://your-project-id:your-instance-name/guestbook?user=root";
} else {
  // Local MySQL instance to use during development.
  Class.forName("com.mysql.jdbc.Driver");
  url = "jdbc:mysql://127.0.0.1:3306/guestbook?user=root";
}

Connection conn = DriverManager.getConnection(url);
ResultSet rs = conn.createStatement().executeQuery(
    "SELECT guestName, content, entryID FROM entries");
%>

<table style="border: 1px solid black">
<tbody>
<tr>
<th width="35%" style="background-color: #CCFFCC; margin: 5px">Name</th>
<th style="background-color: #CCFFCC; margin: 5px">Message</th>
<th style="background-color: #CCFFCC; margin: 5px">ID</th>
</tr>

<%
while (rs.next()) {
    String guestName = rs.getString("guestName");
    String content = rs.getString("content");
    int id = rs.getInt("entryID");
 %>
<tr>
<td><%= guestName %></td>
<td><%= content %></td>
<td><%= id %></td>
</tr>
<%
}
conn.close();
%>

</tbody>
</table>
<br />
No more messages!
<p><strong>Sign the guestbook!</strong></p>
<form action="/sign" method="post">
    <div>First Name: <input type="text" name="fname"></input></div>
    <div>Message:
    <br /><textarea name="content" rows="3" cols="60"></textarea>
    </div>
    <div><input type="submit" value="Post Greeting" /></div>
    <input type="hidden" name="guestbookName" />
  </form>
  </body>
</html>
```

### Map your servlet

Finally, override your `web.xml` with the following code to map your servlet correctly:

```
<?xml version="1.0" encoding="utf-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xmlns="http://java.sun.com/xml/ns/javaee"
xmlns:web="http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd" version="2.5">
   <servlet>
        <servlet-name>sign</servlet-name>
        <servlet-class>guestbook.GuestbookServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>sign</servlet-name>
        <url-pattern>/sign</url-pattern>
    </servlet-mapping>
    <welcome-file-list>
        <welcome-file>guestbook.jsp</welcome-file>
    </welcome-file-list>
</web-app>
```

That's it! Now you can [deploy your application](https://web.archive.org/web/20160424230503/https://cloud.google.com/appengine/docs/java/tools/uploadinganapp) and try it out!

## Using a local MySQL instance during development

[The Guestbook example above](#Java_Build_a_starter_application_and_database) shows how your application can connect to a Cloud SQL instance when the code runs in App Engine and connect to a local MySQL server when the code runs in the [Development Server](https://web.archive.org/web/20160424230503/https://cloud.google.com/appengine/docs/java/tools/devserver). We encourage this pattern to minimize confusion and maximize flexibility.

## Size and access limits

The following limits apply to Google Cloud SQL:

**Cloud SQL instances limits**

<table>
<colgroup>
<col style="width: 22%" />
<col style="width: 20%" />
<col style="width: 20%" />
<col style="width: 38%" />
</colgroup>
<tbody>
<tr class="header">
<th>Limit</th>
<th>First Generation instances</th>
<th>Second Generation instances</th>
<th>Notes</th>
</tr>
&#10;<tr class="odd">
<td>Concurrent connections</td>
<td>Determined by <a href="https://web.archive.org/web/20160424230503/https://cloud.google.com/sql/pricing#packages">tier</a></td>
<td>4,000</td>
<td></td>
</tr>
<tr class="even">
<td>Pending connections</td>
<td>100</td>
<td>N/A</td>
<td>If more than 100 clients try to connect simultaneously to a First Generation instance, some of them will fail to connect.</td>
</tr>
<tr class="odd">
<td>Storage size</td>
<td>250 GB</td>
<td>Determined by <a href="https://web.archive.org/web/20160424230503/https://cloud.google.com/sql/pricing#v2-instance-pricing">machine type</a></td>
<td>It is possible to increase individual First Generation instance limits up to 500 GB for customers with a Silver or higher <a href="https://web.archive.org/web/20160424230503/https://support.google.com/cloud/answer/3420040">Google Cloud support package</a>.</td>
</tr>
</tbody>
</table>

The connection limits are in place to protect against accidents and abuse. For questions about increasing these values, contact the [cloud-sql@google.com](https://web.archive.org/web/20160424230503/mailto:cloud-sql@google.com) team.

**Google App Engine Limits**

Requests from Google App Engine applications to Google Cloud SQL are subject to the following time and connection limits:

-   For apps running in the Google App Engine standard environment, all database requests must finish within the [HTTP request timer](https://web.archive.org/web/20160424230503/https://cloud.google.com/appengine/docs/java/requests#Java_The_request_timer), around **60 seconds**. For apps running in the flexible environment, all database requests must finish within **24 hours**.
-   Offline requests like [cron tasks](https://web.archive.org/web/20160424230503/https://cloud.google.com/appengine/docs/java/config/cron) have a time limit of **10 minutes.**
-   Requests from App Engine modules to Google Cloud SQL are subject to the type of [module scaling](https://web.archive.org/web/20160424230503/https://cloud.google.com/appengine/docs/java/modules/#Java_Instance_scaling_and_class) and instance residence time in memory.
-   Each App Engine instance running in a standard environment or using [Standard-compatible APIs](https://web.archive.org/web/20160424230503/https://cloud.google.com/appengine/docs/flexible/python/migrating-an-existing-app) cannot have more than **12 concurrent connections** to a Google Cloud SQL instance.

Google App Engine applications are also subject to additional App Engine quotas and limits as discussed on the [Quotas](https://web.archive.org/web/20160424230503/https://cloud.google.com/appengine/docs/quotas) page.

Learn more about the [App Engine flexible environment](https://web.archive.org/web/20160424230503/https://cloud.google.com/appengine/docs/flexible/).

## Using Persistence APIs with Cloud SQL

There are several ways to connect to MySQL using Java. The most basic one is via JDBC as shown above. You can also connect using [Java Persistence API (JPA)](https://web.archive.org/web/20160424230503/http://www.oracle.com/technetwork/java/javaee/tech/persistence-jsp-140049.html) and [Java Data Objects (JDO)](https://web.archive.org/web/20160424230503/http://www.oracle.com/technetwork/java/index-jsp-135919.html). Some JPAs, such as [Hibernate](https://web.archive.org/web/20160424230503/http://www.hibernate.org/), [DataNucleus](https://web.archive.org/web/20160424230503/http://www.datanucleus.com/), and [EclipseLink](https://web.archive.org/web/20160424230503/http://www.eclipse.org/eclipselink/), have their own custom additions.

**Warning:** Always close any established connections before finishing processing a request (note the `try .. finally` from the example from above). Not closing a connection will cause it to leak and will eventually cause new connections to fail. You can exit this state by shutting down the affected App Engine instance. For more information about connections, see [Managing connections](#Java_Managing_connections).

To use persistence providers to connect to Cloud SQL, make sure you get the right JARs from the upstream distros, configure the `javax.persistence.jdbc.driver` to use the `com.mysql.jdbc.GoogleDriver`, and point the `javax.persistence.jdbc.url` to the desired Cloud SQL instance using a `jdbc:google:mysql://` prefix in the connection string. To allow the same JPA code to run in both production and locally with `dev_appserver`, use the following pattern:

```
import java.util.Map;
import java.util.HashMap;
import com.google.appengine.api.utils.SystemProperty;

...
    // Set the persistence driver and url based on environment, production or local.
    Map<String, String> properties = new HashMap();
    if (SystemProperty.environment.value() ==
          SystemProperty.Environment.Value.Production) {
      properties.put("javax.persistence.jdbc.driver",
          "com.mysql.jdbc.GoogleDriver");
      properties.put("javax.persistence.jdbc.url",
          "jdbc:google:mysql://your-project-id:your-instance-name/demo");
    } else {
      properties.put("javax.persistence.jdbc.driver",
          "com.mysql.jdbc.Driver");
      properties.put("javax.persistence.jdbc.url",
          "jdbc:mysql://127.0.0.1:3306/demo");
    }

    // Create a EntityManager which will perform operations on the database.
    EntityManagerFactory emf = Persistence.createEntityManagerFactory(
        "persistence-unit-name", propertiesMap);
...
```

For JDO, create a `PersistenceManagerFactory` instead an `EntityManagerFactory`:

```
PersistenceManagerFactory pmf = JDOHelper.getPersistenceManagerFactory(propertiesMap, "persistence-unit-name");
```

Complete examples are available at [https://github.com/GoogleCloudPlatform](https://web.archive.org/web/20160424230503/https://github.com/GoogleCloudPlatform) for:

-   [Hibernate JPA](https://web.archive.org/web/20160424230503/https://github.com/GoogleCloudPlatform/appengine-cloudsql-native-mysql-hibernate-jpa-demo-java)
-   [DataNucleus JPA](https://web.archive.org/web/20160424230503/https://github.com/GoogleCloudPlatform/appengine-cloudsql-native-mysql-datanucleus-jpa-demo-java)
-   [DataNucleus JDO](https://web.archive.org/web/20160424230503/https://github.com/GoogleCloudPlatform/appengine-cloudsql-native-mysql-datanucleus-jdo-demo-java)
-   [EclipseLink JPA](https://web.archive.org/web/20160424230503/https://github.com/GoogleCloudPlatform/appengine-cloudsql-native-mysql-eclipselink-jpa-demo-java)
