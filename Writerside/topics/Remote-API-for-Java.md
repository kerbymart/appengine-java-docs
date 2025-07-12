# Remote API for Java

  

1.  [Configuring Remote API on the Server](#configuring_remote_api_on_the_server)
2.  [Configuring Remote API on a Standalone Client](#configuring_remote_api_on_a_standalone_client)
3.  [Configuring Remote API on an App Engine Client](#configuring_remote_api_on_an_app_engine_client)
4.  [Using Remote API with Maven](#using_remote_api_with_maven)

The Java SDK includes a library called Remote API that lets you transparently access App Engine services from any Java application. For example, you can use Remote API to access a production datastore from an app running on your local machine. You can also use Remote API to access the datastore of one App Engine app from a different App Engine app.

**Important:** Using RemoteApi over a network proxy is not supported.

## Configuring Remote API on the Server

The server component of Remote API is a Java servlet that is part of the App Engine runtime for Java. This servlet receives requests from the Remote API client, dispatches them to the appropriate backend service, and then returns the result of the service call to the client. To install the Remote API servlet, add the following to your `web.xml`:

```
<servlet>
    <display-name>Remote API Servlet</display-name>
    <servlet-name>RemoteApiServlet</servlet-name>
    <servlet-class>com.google.apphosting.utils.remoteapi.RemoteApiServlet</servlet-class>
    <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
    <servlet-name>RemoteApiServlet</servlet-name>
    <url-pattern>/remote_api</url-pattern>
</servlet-mapping>
```
**Note:** for more information on using `web.xml`, see the [Deployment Descriptor](https://web.archive.org/web/20160424230746/https://cloud.google.com/appengine/docs/java/config/webxml) page.

The servlet returns an error if there is no authenticated user or the authenticated user is not an admin of your application, so there is no need to configure any additional security. Once you've deployed your app with these settings, any app with the Remote API client installed can use its services. This includes Python clients that are using the Python Remote API.

## Configuring Remote API on a Standalone Client

These instructions show the use of OAuth 2.0. If you have an older application that uses ClientLogin (where you supply username and password), you should migrate the app to use OAuth 2.0 as shown on this page because [ClientLogin is being turned off soon](https://web.archive.org/web/20160424230746/https://cloud.google.com/appengine/docs/deprecations/clientlogin). After making the code changes shown on this page, you must also remove the `<security-constraint>` for your remote-api servlet in your application's `web.xml` file and then redeploy your application. Note that the servlet still performs admin authentication even without this constraint, so your application continues to be protected.

To configure the client component of Remote API for use inside a Java application, add `${SDK_ROOT}/lib/impl/appengine-api.jar` and `${SDK_ROOT}/lib/appengine-remote-api.jar` to your classpath. Then, in your code, configure and install Remote API:

```
import com.google.appengine.tools.remoteapi.RemoteApiInstaller;
import com.google.appengine.tools.remoteapi.RemoteApiOptions;

// ...
RemoteApiOptions options = new RemoteApiOptions()
    .server("your_app_id.appspot.com", 443)
    .useApplicationDefaultCredential();

RemoteApiInstaller installer = new RemoteApiInstaller();
installer.install(options);
// ... all API calls executed remotely
installer.uninstall();
```

The Remote API client will rely on Application Default Credentials that use OAuth 2.0.

In order to get a credential run:

```
$ gcloud auth login
```
**Note:** The Remote API call to `useApplicationDefaultCredential()` can only use credentials provided by the `gcloud` command.

You can just as easily connect to an App Engine app running locally in the [Development Server](https://web.archive.org/web/20160424230746/https://cloud.google.com/appengine/docs/java/tools/devserver):

```
RemoteApiOptions options = new RemoteApiOptions()
    .server("localhost", 8888) // server name must equal "localhost"
    .useDevelopmentServerCredential();
```

Here's a complete Java application that, when run, inserts an Entity into the datastore:

```
package remoteapiexample;

import com.google.appengine.api.datastore.DatastoreService;
import com.google.appengine.api.datastore.DatastoreServiceFactory;
import com.google.appengine.api.datastore.Entity;
import com.google.appengine.tools.remoteapi.RemoteApiInstaller;
import com.google.appengine.tools.remoteapi.RemoteApiOptions;
import java.io.IOException;

public class RemoteApiExample {
    public static void main(String[] args) throws IOException {
        RemoteApiOptions options = new RemoteApiOptions()
            .server("<i>your_app_id</i>.appspot.com", 443)
            .useApplicationDefaultCredential();
        RemoteApiInstaller installer = new RemoteApiInstaller();
        installer.install(options);
        try {
            DatastoreService ds = DatastoreServiceFactory.getDatastoreService();
            System.out.println("Key of new entity is " +
                ds.put(new Entity("Hello Remote API!")));
        } finally {
            installer.uninstall();
        }
    }
}
```

## Configuring Remote API on an App Engine Client

You can also use Remote API to access the services of one App Engine application from a different App Engine application. You need to add `${SDK_ROOT}/lib/appengine-remote-api.jar` to your `WEB-INF/lib` directory and then, in your client App Engine app, configure and install Remote API just as you did in your standalone Java client.

Notice that `RemoteApiInstaller` only installs Remote API on the thread performing the installation, so be careful not to share instances of this class across threads.

## Using Remote API with Maven

To use the Remote API feature in your Maven project, add the following dependency to your project `pom.xml` file:

```
<dependency>
  <groupId>com.google.appengine</groupId>
  <artifactId>appengine-remote-api</artifactId>
  <version>1.9.34 </version>
</dependency>
```
