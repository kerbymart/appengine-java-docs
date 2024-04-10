# Creating a Simple Hello World Backend API

  

1.  [Creating and building the project](#creating_and_building_the_project)
2.  [Running and testing the API locally](#running_and_testing_the_api_locally)
3.  [Congratulations!](#congratulations)

In this part of the tutorial, we'll use the App Engine Maven artifact `endpoints-skeleton-archetype` to create a new Endpoints project. We'll then show you how to add code.

## Creating and building the project

To create the Hello World backend API:

1.  In a terminal window, change to the directory where you want to build the project.

2.  Invoke the following Maven command:

    ```
    mvn archetype:generate -Dappengine-version=1.9.34 -Dfilter=com.google.appengine.archetypes:
    ```

3.  Enter the number `2` to select the archetype `remote -> com.google.appengine.archetypes:endpoints-skeleton-archetype` from the list of App Engine archetypes.

4.  Accept the default to use the most recent version.

5.  When prompted to `Define value for property 'groupId'`, enter the namespace `com.example.helloworld` to keep this tutorial in sync with the source files at GitHub. (The typical convention is to use a namespace starting with the reversed form of your domain name.)

6.  When prompted to `Define value for property 'artifactId'`, supply the project name `helloworld`.

7.  When prompted to `Define value for property 'version'`, accept the default value.

8.  When prompted to `Define value for property 'package'`, accept the default value.

9.  When prompted to confirm your choices, accept the default value (`Y`).

10. Wait for the project to finish generating, then take a look at the resulting project layout:

    ![Maven Project Layout](https://web.archive.org/web/20160424230950im_/https://cloud.google.com/appengine/docs/java/endpoints/images/helloworld.png)

    The `pom.xml` file contains all the configuration and dependencies needed for your backend API.

11. Change directories to the project's Java source directory: `/src/main/java/com/example/helloworld`.

12. Create a [JavaBean](https://web.archive.org/web/20160424230950/http://stackoverflow.com/questions/3295496/what-is-a-javabean-exactly) class file named `MyBean.java` copy-paste the following code into it:

    <a href="https://web.archive.org/web/20160424230950/https://github.com/GoogleCloudPlatform/appengine-endpoints-helloworld-java-maven/blob/master/src/main/java/com/example/helloworld/MyBean.java" target="_blank" style="color: white;">src/main/java/com/example/helloworld/MyBean.java</a>

    <a href="https://web.archive.org/web/20160424230950/https://github.com/GoogleCloudPlatform/appengine-endpoints-helloworld-java-maven/blob/master/src/main/java/com/example/helloworld/MyBean.java" class="button" target="_blank" data-track-type="github" data-track-name="gitHubViewButton" data-track-metadata-link-destination="https://github.com/GoogleCloudPlatform/appengine-endpoints-helloworld-java-maven/blob/master/src/main/java/com/example/helloworld/MyBean.java">View on GitHub</a>

    \[This section requires a browser that supports JavaScript and iframes.\]

    We'll use this JavaBean object to send data through the backend API.

13. Edit the file `YourFirstAPI.java`, replacing all of the contents with the following code:

    <a href="https://web.archive.org/web/20160424230950/https://github.com/GoogleCloudPlatform/appengine-endpoints-helloworld-java-maven/blob/master/src/main/java/com/example/helloworld/YourFirstAPI.java" target="_blank" style="color: white;">src/main/java/com/example/helloworld/YourFirstAPI.java</a>

    <a href="https://web.archive.org/web/20160424230950/https://github.com/GoogleCloudPlatform/appengine-endpoints-helloworld-java-maven/blob/master/src/main/java/com/example/helloworld/YourFirstAPI.java" class="button" target="_blank" data-track-type="github" data-track-name="gitHubViewButton" data-track-metadata-link-destination="https://github.com/GoogleCloudPlatform/appengine-endpoints-helloworld-java-maven/blob/master/src/main/java/com/example/helloworld/YourFirstAPI.java">View on GitHub</a>

    \[This section requires a browser that supports JavaScript and iframes.\]

    Notice the annotation [@Api](https://web.archive.org/web/20160424230950/https://cloud.google.com/appengine/docs/java/endpoints/javadoc/com/google/api/server/spi/config/Api); this is where we set the configuration of the backend API. (See [Endpoint Annotations](https://web.archive.org/web/20160424230950/https://cloud.google.com/appengine/docs/java/endpoints/annotations) for all of the available attributes of this annotation.)

    Notice the [@ApiMethod](https://web.archive.org/web/20160424230950/https://cloud.google.com/appengine/docs/java/endpoints/javadoc/com/google/api/server/spi/config/ApiMethod) annotation; it allows you to configure your methods.

    Finally, notice the use of the `javax.inject.Named` annotation (`@Named`) in the `sayHi` method definition. You must supply this annotation for all parameters passed to server-side methods, unless the parameter is an [entity type](https://web.archive.org/web/20160424230950/https://cloud.google.com/appengine/docs/java/endpoints/paramreturn_types#entity_types).

14. Build the project by invoking

    ```
    mvn clean install
    ```

    Wait for the project to build. When the project successfully finishes, you will see a message similar to this one:

    ```
    [INFO] BUILD SUCCESS
    [INFO] ------------------------------------------------------------------------
    [INFO] Total time: 14.846s
    [INFO] Finished at: Tue Jun 03 09:43:09 PDT 2014
    [INFO] Final Memory: 24M/331M
    ```

## Running and testing the API locally

To test the backend API locally:

1.  Start a new Chrome session as described in [How do I use Explorer with a local HTTP API](https://web.archive.org/web/20160424230950/https://developer.google.com/explorer-help/#hitting_local_api) and specify `8080` as the `localhost` port (`localhost:8080)`.

2.  From the main directory `helloworld/`, start the API in the development server as follows:

    ```
    mvn appengine:devserver
    ```

    Wait for the dev server to start up. When the server is ready, you will see a message similar to this one:

    ```
    [INFO] INFO: Module instance default is running at http://localhost:8080/
    [INFO] Jun 03, 2014 9:44:47 AM com.google.appengine.tools.development.AbstractModule startup
    [INFO] INFO: The admin console is running at http://localhost:8080/_ah/admin
    [INFO] Jun 03, 2014 9:44:47 AM com.google.appengine.tools.development.DevAppServerImpl doStart
    [INFO] INFO: Dev App Server is now running
    ```

3.  Visit the APIs Explorer at this URL:

    ```
    http://localhost:8080/_ah/api/explorer
    ```

    (The Google APIs Explorer is a built-in tool that lets you send requests to a backend service without using a client app.)

4.  Click **myApi API** to display the methods available from this API.

5.  Click **myApi.sayHi** to display the Explorer for this method:

    ![SayHi form](https://web.archive.org/web/20160424230950im_/https://cloud.google.com/appengine/docs/java/endpoints/images/sayHi.png)

6.  In the **name** field, supply your name, and click **Authorize and execute** and check the Response for the greeting.

## Congratulations!

You've just built and tested your first backend API!

Next, let's build a more complete backend API with GET and POST methods that write to and fetch from the datastore.

<a href="https://web.archive.org/web/20160424230950/https://cloud.google.com/appengine/docs/java/endpoints/getstarted/backend/helloendpoints" class="button">Creating the HelloEndpoints Complete Sample Backend API &gt;&gt;</a>
