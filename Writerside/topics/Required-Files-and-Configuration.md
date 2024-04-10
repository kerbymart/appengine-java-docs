# Required Files and Configuration

  

1.  [The backend API class file](#the_backend_api_class_file)
2.  [appengine-web.xml](#appengine-webxml)
3.  [web.xml](#webxml)

**Note:** In the following sections, we refer to the path/`_ah/spi`. If you have created App Engine apps that are *not* Endpoints, you may be expecting the path `/_ah/api` and not the path `/_ah/spi` as described above. This is not a typo: Endpoints require `/_ah/spi`!

Your project must contain, at a minimum, the following files:

<table>
<thead>
<tr class="header">
<th>File and Location</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>/src/main/java/&lt;packagepath&gt;/&lt;your_api_class&gt;.java</code></td>
<td>The class file (or files, if you implement your API across multiple classes) containing your backend API.</td>
</tr>
<tr class="even">
<td><code>/src/main/webapp/WEB-INF/appengine-web.xml</code></td>
<td>The web app deployment descriptor required for App Engine configuration.</td>
</tr>
<tr class="odd">
<td><code>/src/main/webapp/WEB-INF/web.xml</code></td>
<td>The standard Java web app deployment descriptor mapping URLs to servlets and other information.</td>
</tr>
</tbody>
</table>

The contents of each of these required files is documented in the following sections.

**Important:** If you create your project using Maven and the artifacts described in the [Android Backend Tutorial](https://web.archive.org/web/20160424230315/https://cloud.google.com/appengine/docs/java/endpoints/getstarted/backend/) setup and configuration pages, your project automatically has the `web.xml` and `appengine-web.xml` generated with the proper entries.

## The backend API class file

The required and optional contents of the class file (or files, if you use a [multi-class API](https://web.archive.org/web/20160424230315/https://cloud.google.com/appengine/docs/java/endpoints/multiclass)) are fully described in the topic [Endpoint Annotations](https://web.archive.org/web/20160424230315/https://cloud.google.com/appengine/docs/java/endpoints/annotations).

## appengine-web.xml

The bare minimum contents required for this file are as follows:

```
<?xml version="1.0" encoding="utf-8"?>
<appengine-web-app xmlns="http://appengine.google.com/ns/1.0">
    <application>your-app-id</application>
    <version>version_number</version>
    <threadsafe>true</threadsafe>

    <system-properties>
        <property name="java.util.logging.config.file" value="WEB-INF/logging.properties"/>
    </system-properties>
</appengine-web-app>
```

where:

-   `your-app-id` is replaced with the actual App Engine application ID for your backend API.
-   `version_number` is replaced by the App Engine version number you want to use, with the first version of your app starting at version `1`. (For a full discussion of App Engine versions, see [Deploying to multiple app versions](https://web.archive.org/web/20160424230315/https://cloud.google.com/appengine/docs/java/endpoints/test_deploy#deploying_to_multiple_app_versions).)
-   `threadsafe` is set to true if you want App Engine to send multiple requests in parallel, or set to false, if you want App Engine to send requests serially.

Additional but optional settings are available. See [Java Application Configuration with appengine-web.xml](https://web.archive.org/web/20160424230315/https://cloud.google.com/appengine/docs/java/config/appconfig#Java_appengine_web_xml_About_appengine_web_xml) for more information.

## web.xml

Sample bare minimum contents required for this file are shown as follows:

```
<?xml version="1.0" encoding="utf-8" standalone="no"?><web-app xmlns="http://java.sun.com/xml/ns/javaee" xmlns:web="http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" version="2.5" xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd">
    <servlet>
        <servlet-name>SystemServiceServlet</servlet-name>
        <servlet-class>com.google.api.server.spi.SystemServiceServlet</servlet-class>
        <init-param>
            <param-name>services</param-name>
            <param-value>com.google.devrel.samples.helloendpoints.Greetings</param-value>
        </init-param>
    </servlet>
    <servlet-mapping>
        <servlet-name>SystemServiceServlet</servlet-name>
        <url-pattern>/_ah/spi/*</url-pattern>
    </servlet-mapping>
    <welcome-file-list>
        <welcome-file>index.html</welcome-file>
    </welcome-file-list>
</web-app>
```

where you replace:

-   `<param-value>com.google.devrel.samples.helloendpoints.Greetings</param-value>` with your own API class name. Note that if your API is a [multi-class API](https://web.archive.org/web/20160424230315/https://cloud.google.com/appengine/docs/java/endpoints/multiclass), each class is listed inside this same `<param-value>` each separated by a comma, for example:

    `<param-value>com.google.devrel.samples.helloendpoints.Greetings,com.google.devrel.samples.helloendpoints.AndGoodbyes</param-value>`

-   The value for `<welcome-file>` with the page used as the landing page of the web app, if there is one. You can omit the entire `<welcome-file-list>` block if you don't supply a web client (JavaScript client) for your backend API.

**Important:** The `/_ah/spi/*` url pattern must be explicitly mapped to a servlet or filter in the `web.xml` file. App Engine checks `web.xml` for this URL pattern mapping to enable Cloud Endpoints.

Additional but optional settings are available. For more information, see [The Deployment Descriptor: web.xml](https://web.archive.org/web/20160424230315/https://cloud.google.com/appengine/docs/java/config/webxml). However, note that the information under *Security and Authentication* and *Secure URLs* **do not apply** to backend APIs.
