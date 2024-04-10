# Using Apache Maven

  

1.  [Java Requirements](#java_requirements)
2.  [Maven requirements](#maven_requirements)
3.  [Maven projects in the flexible environment](#maven_projects_in_the_flexible_environment)
4.  [A note about Maven projects and Eclipse](#a_note_about_maven_projects_and_eclipse)
5.  [Maven terms you need to know](#maven_terms_you_need_to_know)
6.  [Installing Maven](#installing_maven)
7.  [Maven App Engine archetypes](#maven_app_engine_archetypes)
8.  [Creating App Engine applications or backend APIs using the archetypes](#creating_app_engine_applications_or_backend_apis_using_the_archetypes)
9.  [Compile and build your project using Maven](#compile_and_build_your_project_using_maven)
10. [Testing your app with the development server](#testing_your_app_with_the_development_server)
11. [Creating required indexes](#creating_required_indexes)
12. [Uploading your app to production App Engine](#uploading_your_app_to_production_app_engine)
    1.  [Adding the App Engine Maven plugin to an existing Maven project](#adding_the_app_engine_maven_plugin_to_an_existing_maven_project)
    2.  [Specifying a port for local testing](#specifying_a_port_for_local_testing)
13. [Managing and running a project with the App Engine Maven plugin](#managing_and_running_a_project_with_the_app_engine_maven_plugin)
    1.  [App Engine Maven plugin goals](#app_engine_maven_plugin_goals)
    2.  [Managing Versions with versions-maven-plugin command options](#managing_versions_with_versions-maven-plugin_command_options)
    3.  [Cloud Endpoints goals](#cloud_endpoints_goals)
14. [Adding JDO or JPA dependencies](#adding_jdo_or_jpa_dependencies)

[Apache Maven](https://web.archive.org/web/20160424230715/http://maven.apache.org/) is a software project management and comprehension tool. It is capable of building WAR files for deployment into App Engine. The App Engine team provides both a plugin and Maven Archetypes for the purpose of speeding up development. You must use Maven 3.1 or greater.

**Note:** When you use Maven, you don't need to download the Java libraries from the Google App Engine SDK. Maven does that for you. You can also use Maven to test your app locally and upload (deploy) it to production App Engine.

## Java Requirements

You need the following:

1.  You must use Java 7. If you don't have Java 7, [download](https://web.archive.org/web/20160424230715/http://www.java.com/en/download/manual.jsp) and install it.

2.  Set your `JAVA_HOME` environment variable. If you are a `bash` user

    1.  For a typical Linux installation, add a line similar to the following to your `.bashrc` file:

        ```
        export JAVA_HOME=/usr/local/tools/java/<jdk_version>
        ```

        where `<jdk_version>` is your JDK version, for example, `jdk1.7.0_45.jdk`.

    2.  If you use Mac OSX and the default Terminal app, your shell session doesn't load `.bashrc` by default. So you may need to add a line similar to the following to your `.bash_profile`:

        ```
        [ -r ~/.bashrc ] && source ~/.bashrc
        ```

    3.  If you use Mac OSX but don't use the default terminal app, for example, you use a terminal management app such as tmux, you may need to add a line similar to the following line to your `.bashrc` file:

        ```
        export JAVA_HOME=/Library/Java/JavaVirtualMachines/<jdk_version>/Contents/Home
        ```

        where `<jdk_version>` is your JDK version, for example, `jdk1.7.0_45.jdk`.

        Alternatively, if your OS supports [/usr/libexec/java\_home](https://web.archive.org/web/20160424230715/http://stackoverflow.com/questions/1348842/what-should-i-set-java-home-to-on-osx), you can request the current JDK version in your export command line, as follows:

        ```
        export JAVA_HOME=$(/usr/libexec/java_home -v 1.7)
        ```

## Maven requirements

You must use Apache Maven 3.1 or greater. To determine whether Maven is installed and which version you have, invoke the following command:

```
 mvn -v
```

This command should display a long string of information beginning with something like `Apache Maven 3.1.0`.

If you don't have the proper version of Maven installed:

1.  [Download](https://web.archive.org/web/20160424230715/http://maven.apache.org/download.cgi) Maven 3.1 or greater from the Maven website.
2.  [Install](https://web.archive.org/web/20160424230715/http://maven.apache.org/install.html) Maven 3.1 or greater on your local machine.

**Note:** You cannot use `apt-get install` to install Maven 3.1 or greater.

## Maven projects in the flexible environment

If you are using the flexible environment, you need to use a new plugin for the Cloud SDK. For more information, refer to the [the flexible environment documentation](https://web.archive.org/web/20160424230715/https://cloud.google.com/appengine/docs/flexible/java/maven).

## A note about Maven projects and Eclipse

A Maven project has a different layout than an Eclipse project. So, if you wish to use a Maven project with Eclipse, you'll need to use Eclipse for Java Enterprise Edition (EE), which has Maven support, and you'll need to use one of the following approaches:

-   Import the Maven project into Eclipse for Java EE.
-   Import a Maven project for App Engine into Eclipse for Java EE as a [Web Tools Platform (WTP) project](https://web.archive.org/web/20160424230715/https://cloud.google.com/appengine/docs/java/webtoolsplatform), as described in [Importing an Existing Maven Project](https://web.archive.org/web/20160424230715/https://cloud.google.com/appengine/docs/java/webtoolsplatform#maven).
-   Set up two debug configurations, one for the Maven project in devserver (`mvn appengine:devserver`), and one for a Remote Java Application that you use to connect the Eclipse debug client to the `devserver jvm`. For details on how to do this, see the response to the StackOverflow question [How do I make Eclipse and mvn appengine:devserver talk to each other?](https://web.archive.org/web/20160424230715/http://stackoverflow.com/questions/13924990/how-do-i-make-eclipse-and-mvn-appenginedevserver-talk-to-each-other).

**Note:** You can also use [IntelliJ](https://web.archive.org/web/20160424230715/http://www.jetbrains.com/idea/), which provides built-in integration support for Maven projects.

## Maven terms you need to know

During project creation, Maven prompts you to supply `groupId`, `artifactId`, `version`, and the `package` for the project. What are these in Maven?

<table>
<thead>
<tr class="header">
<th>Term</th>
<th>Meaning</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>groupId</code></td>
<td>A namespace within Maven to keep track of your artifacts. When people consume your project in their own Maven Project, it will serve as an attribute of the dependency they will end up specifying.</td>
</tr>
<tr class="even">
<td><code>artifactId</code></td>
<td>The name of your project within Maven. It is also specified by consumers of your project when they depend on you in their own Maven projects.</td>
</tr>
<tr class="odd">
<td><code>version</code></td>
<td>The initial Maven version you want to have your project generated with. It's a good idea to have <code>version</code> suffixed by <code>-SNAPSHOT</code> because this will provide support in the Maven release plugin for versions that are under development. For more information, see the <a href="https://web.archive.org/web/20160424230715/http://maven.apache.org/guides/mini/guide-releasing.html">Maven guide</a> to using the release plugin.</td>
</tr>
<tr class="even">
<td><code>package</code></td>
<td>The Java package created during the generation.</td>
</tr>
</tbody>
</table>

## Installing Maven

If you do not already have Maven version 3.1 or greater installed on your system, visit the [Apache Maven](https://web.archive.org/web/20160424230715/http://maven.apache.org/) web site to download and install it.

When Maven is installed and the `mvn` command is on your command path, you can run the following command to verify that it works, and see which version is installed:

```
mvn -v
```

Make sure the version is 3.1 or greater.

## Maven App Engine archetypes

[Maven Archetypes](https://web.archive.org/web/20160424230715/http://maven.apache.org/archetype/maven-archetype-plugin) allow users to create Maven projects using templates that cover common scenarios. App Engine takes advantage of this Maven feature to provide some useful [App Engine archetypes](https://web.archive.org/web/20160424230715/http://search.maven.org/#search%7Cga%7C1%7Ccom.google.appengine.archetypes) at Maven Central. The current App Engine artifacts are listed in the table below:

<table>
<thead>
<tr class="header">
<th>Application Type</th>
<th>Artifact</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>App Engine app*</td>
<td><code>guestbook-archetype</code></td>
<td>Generates the guestbook demo sample, complete and ready to run and test.</td>
</tr>
<tr class="even">
<td>App Engine app*</td>
<td><code>appengine-skeleton-archetype</code></td>
<td>Generates a new, empty App Engine project ready for your own classes and resources, but with required files and directories.</td>
</tr>
<tr class="odd">
<td>Cloud Endpoints API backend</td>
<td><code>hello-endpoints-archetype</code></td>
<td>Generates a simple starter Cloud Endpoints backend API project, ready to build and run.</td>
</tr>
<tr class="even">
<td>Cloud Endpoints API backend</td>
<td><code>endpoints-skeleton-archetype</code></td>
<td>Generates a new, empty Cloud Endpoints backend API project ready for your own classes and resources, with required files and directories.</td>
</tr>
</tbody>
</table>

`*` *App Engine app* in this context means a regular App Engine app, *not* an app serving as a Cloud Endpoints backend API.

## Creating App Engine applications or backend APIs using the archetypes

The following table shows how to use App Engine Maven archetypes to create an App Engine app or a Cloud Endpoints backend API. Click on the desired app type to see the instructions.

**Note:** Some of the following commands can take time to execute, and some can potentially download large artifacts. We recommended that you run these with a good internet connection.

### App Engine App

To create an App Engine App:

1.  Get an application ID from the Cloud Platform Console.

2.  Change directory to a directory where you want to build the project.

3.  Invoke the following Maven command:

    ```
         mvn archetype:generate -Dappengine-version=1.9.34 -Dapplication-id=your-app-id -Dfilter=com.google.appengine.archetypes:
    ```

    where `-Dappengine-version` is set to the most recent App Engine Java SDK version, and `application-id` is set to the Cloud Platform Console application ID used for your app.

4.  If you want to create the complete, ready-to-run guestbook sample app, supply the number corresponding to `com.google.appengine.archetypes:guestbook-archetype`.

    If you want to create an empty project that contains the required directory structure and files, ready for your own classes, supply the number corresponding to `com.google.appengine.archetypes:appengine-skeleton-archetype`.

5.  Select the most recent version from the displayed list of available archetype versions by accepting the default.

6.  When prompted to `Define value for property 'groupId'`, supply the desired namespace for your app; for example, `com.mycompany.myapp`.

7.  When prompted to `Define value for property 'artifactId'`, supply the project name; for example, `myapp` or `guestbook`.

8.  When prompted to `Define value for property 'version'`, accept the default value.

9.  When prompted to `Define value for property 'package'`, supply your preferred package name (or accept the default). The generated Java files will have the package name you specify here.

10. When prompted to confirm your choices, accept the default value (`Y`).

11. Wait for the project to finish generating. then change directories to the new project directory, for example `guestbook/`.

12. Build the project by invoking

    ```
    mvn clean install
    ```

13. Wait for the project to build. When the project successfully finishes you will see a message similar to this one:

    ```
    [INFO] --------------------------------------------------
    [INFO] BUILD SUCCESS
    [INFO] --------------------------------------------------
    [INFO] Total time: 1:16.656s
    [INFO] Finished at: Mon Sep 15 11:42:22 PDT 2014
    [INFO] Final Memory: 16M/228M
    [INFO] --------------------------------------------------
    ```

14. If you created the sample Guestbook demo app using the `guestbook-archetype` artifact:

    1.  Test the application locally in the development server as follows:

        ```
        mvn appengine:devserver
        ```

        Wait for the dev server to start up. When it finishes starting up, you will see a message similar to this:

        ```
         [INFO] INFO: Module instance default is running at http://localhost:8080/
         [INFO] Sep 15, 2014 11:44:19 AM com.google.appengine.tools.development.AbstractModule startup
         [INFO] INFO: The admin console is running at http://localhost:8080/_ah/admin
         [INFO] Sep 15, 2014 11:44:19 AM com.google.appengine.tools.development.DevAppServerImpl doStart
         [INFO] INFO: Dev App Server is now running
        ```

        **Note:** If there is a failure indicating that the App Engine Maven plugin is missing, this could be due to the fact that a new version was recently pushed to the Maven repo. To remedy this, invoke Maven with the -U option: `mvn appengine:devserver -U`

    2.  Visit the application at the default URL and port `http://localhost:8080/` used by the development server. You will see the guestbook demo app.

    3.  To shut down the app and the development server, press **Control+C** in the Windows/Linux terminal window you started it in, or **CMD+C** on the Mac.

15. If you created a new, empty app using the `appengine-skeleton-archetype` artifact:

    1.  Before starting to code your own classes for the app, familiarize yourself with the basic project layout and the required project files is complete: inside the directory where you created the project, you'll have a subdirectory named `myapp`, which contains a `pom.xml` file, the `src/main/java` subdirectory, and the `src/main/webapp/WEB-INF` subdirectory:

        ![Maven Project Layout](https://web.archive.org/web/20160424230715im_/https://cloud.google.com/appengine/docs/java/tools/images/maven_layout.png)

        -   You'll add your own application Java classes to `src/main/java/...`
        -   You'll configure your application using the file `src/main/webapp/WEB-INF/appengine.web.xml`
        -   You'll configure your application deployment using the file `src/main/webapp/WEB-INF/web.xml`

    2.  Create your application Java classes and add them to `src/main/java/...`/ For more information, see [Getting Started](https://web.archive.org/web/20160424230715/https://cloud.google.com/appengine/docs/java/gettingstarted/introduction)

    3.  Add the UI that you want to provide to your app users. For more information, see [Adding Application Code and UI](https://web.archive.org/web/20160424230715/https://cloud.google.com/appengine/docs/java/gettingstarted/ui_and_code).

    4.  The artifact you used to create the project has done the basic `src/main/webapp/WEB-INF/appengine.web.xml` configuration for you. However, for more advanced configuration, you may need to edit this file. For more information, see [Configuring with appengine-web.xml](https://web.archive.org/web/20160424230715/https://cloud.google.com/appengine/docs/java/config/appconfig).

    5.  Edit the file `src/main/webapp/WEB-INF/web.xml` to map URLs to your app handlers, specify authentication, filters, and so forth. This is described in detail in [The Deployment Descriptor](https://web.archive.org/web/20160424230715/https://cloud.google.com/appengine/docs/java/config/webxml).

### Cloud Endpoints Backend API

To create a Cloud Endpoints backend API project:

1.  Change directory to a directory where you want to build the project.

2.  Invoke the following Maven command:

    ```
         mvn archetype:generate -Dappengine-version=1.9.34 -Dfilter=com.google.appengine.archetypes:
    ```

    where `-Dappengine-version` is set to the most recent App Engine Java SDK version.

3.  If you want to create the complete, ready-to-run Hello Endpoints backend API, supply the number corresponding to `hello-endpoints-archetype`.

    If you want to create an empty project that contains the required directory structure and files, ready for your own classes, supply the number corresponding to `endpoints-skeleton-archetype`.

4.  Select the most recent version from the displayed list of available archetype versions by accepting the default.

5.  When prompted to `Define value for property 'groupId'`, supply the namespace for your app; for example, `com.mycompany.myapp`. (For the Hello Endpoints sample backend API, supply the value `com.google.appengine.samples.helloendpoints`.)

6.  When prompted to `Define value for property 'artifactId'`, supply the project name; for example, `myapp`. (For the Hello Endpoints sample, supply the value `helloendpoints`.)

7.  When prompted to `Define value for property 'version'`, accept the default value.

8.  When prompted to `Define value for property 'package'`, accept the default value.

9.  When prompted to confirm your choices, accept the default value (`Y`).

10. Wait for the project to finish generating. then change directories to the new project directory, for example `helloendpoints/`.

11. Build the project by invoking

    ```
    mvn clean install
    ```

12. Wait for the project to build. When the project successfully finishes you will see a message similar to this one:

    ```
    [INFO] BUILD SUCCESS
    [INFO] ------------------------------------------------------------------------
    [INFO] Total time: 14.846s
    [INFO] Finished at: Tue Jun 03 09:43:09 PDT 2014
    [INFO] Final Memory: 24M/331M
    ```

13. If you created the sample Hello Endpoints sample backend API:

    1.  Test the application locally in the development server as follows:

        ```
        mvn appengine:devserver
        ```

        Wait for the dev server to start up. When it finishes starting up, you will see a message similar to this one:

        ```
         [INFO] INFO: Module instance default is running at http://localhost:8080/
         [INFO] Jun 03, 2014 9:44:47 AM com.google.appengine.tools.development.AbstractModule startup
         [INFO] INFO: The admin console is running at http://localhost:8080/_ah/admin
         [INFO] Jun 03, 2014 9:44:47 AM com.google.appengine.tools.development.DevAppServerImpl doStart
         [INFO] INFO: Dev App Server is now running
        ```

    2.  Visit the URL `http://localhost:8080` to send requests to the backend API

14. If you created a new, empty app:

    1.  Familiarize yourself with the basic project layout shown here:

        ![Maven Project Layout](https://web.archive.org/web/20160424230715im_/https://cloud.google.com/appengine/docs/java/tools/images/maven_layout-e.png)

        `YourFirstAPI.java` is a starter file for your own API. (You aren't required to use this name.)

    2.  Add your classes as desired.

## Compile and build your project using Maven

To build an app created with the Maven App Engine archetypes:

1.  Change directory to the main directory for your project, for example, `guestbook/` http://maven.apache.org/archetype/maven-archetype-plugin

2.  Invoke Maven as follows:

    ```
    mvn clean install
    ```

3.  Wait for the project to build. When the project successfully finishes you will see a message similar to this one:

    ```
    BUILD SUCCESS
     Total time: 10.724s
     Finished at: Thur Jul 04 14:50:06 PST 2014
     Final Memory: 24M/213M
    ```

4.  Optionally, test the application using the following procedure.

## Testing your app with the development server

During the development phase, you can run and test your app at any time in the development server by invoking the App Engine Maven plugin. The procedure varies slightly depending on the artifact used to create the project, so click on the appropriate tab below:

### App Engine App

To test your app:

1.  If you haven't already done so, build your app (`mvn clean install`).

2.  Change directory to the top level of your project (for example, to `myapp`) and invoke Maven as follows:

    ```
    mvn appengine:devserver
    ```

    Wait for the the server to start. When the server is completely started with your app running, you will see a message similar to this one:

    ```
     Aug 24, 2014 2:56:42 PM com.google.appengine.tools.development.DevAppServerImpl start
     INFO: The server is running at http://localhost:8080/
     Aug 24, 2014 2:56:42 PM com.google.appengine.tools.development.DevAppServerImpl start
     INFO: The admin console is running at http://localhost:8080/_ah/admin
    ```

3.  Use your browser to visit `http://localhost:8080/` to access your app.

4.  Shut down the app and the development server by pressing **Control+C** in the Windows/Linux terminal window where you started it, or **CMD+C** on the Mac.

### Cloud Endpoints Backend API

To test your app:

1.  If you haven't already done so, build your app (`mvn clean install`).

2.  Change directory to your project's main directory `/myapp`) and invoke Maven as follows:

    ```
    mvn appengine:devserver
    ```

    Wait for the the server to start. When the server is completely started with your app running, you will see a message similar to this one:

    ```
     Jul 04, 2014 2:56:42 PM com.google.appengine.tools.development.DevAppServerImpl start
     INFO: The server is running at http://localhost:8080/
     Jul 04, 2013 2:56:42 PM com.google.appengine.tools.development.DevAppServerImpl start
     INFO: The admin console is running at http://localhost:8080/_ah/admin
    ```

3.  Use your browser to visit `http://localhost:8080/` to access your app, or, alternatively, to test the API using the built-in Google API Explorer, visit `http://localhost:8080/_ah/api/explorer`.

4.  Shut down the app and the development server by pressing **Control+C** in the Windows/Linux terminal window where you started it, or **CMD+C** on the Mac.

## Creating required indexes

If your app uses Datastore, you must create indexes before uploading your app to production App Engine. You can do this using the development server. When you run your app on the development server, it automatically creates the Datastore indexes required to run in production App Engine. These indexes are generated in `guestbook/target/guestbook-1.0-SNAPSHOT/WEB-INF/appengine-generated/datastore-indexes-auto.xml`. You'll also notice the file `local_db.bin` at that same location: this is local storage for the development server to persist your app data between development server sessions. The file `datastore-indexes-auto.xml` is automatically uploaded along with your app; `local_db.bin` is not uploaded.

**Note:** Running `mvn clean install` clears the `datastore-indexes-auto.xml` file; if you run it on your app prior to [uploading to production App Engine](https://web.archive.org/web/20160424230715/https://cloud.google.com/appengine/docs/java/gettingstarted/uploading), you won't get required indexes and you'll experience a runtime error.

For complete information about indexes, see [Datastore Indexes](https://web.archive.org/web/20160424230715/https://cloud.google.com/appengine/docs/java/datastore/indexes).

## Uploading your app to production App Engine

**Important:** If your app uses Datastore, you must generate any required Datastore indexes before uploading your app. For information and instructions, see [Testing your app on the development server](#testing_your_app_with_the_development_server) and [Creating required indexes](#creating_required_indexes)

The upload procedure varies slightly depending on the artifact used to create the project, so click on the appropriate tab below:

### App Engine App

To upload an app created with the the `appengine-skeleton-archetype`:

1.  Change directory to the top level of your project (for example, `myapp`) and invoke Maven as follows:

    ```
    mvn appengine:update
    ```

2.  You will be prompted for an authorization code in the terminal window and your web browser will launch with a consent screen which you must accept in order to be authorized. Follow the prompts to copy any codes from the browser to the command line.

### Cloud Endpoints Backend API

To upload an app created with the the `endpoints-skeleton-archetype`:

1.  Change directory to your project's main directory (for example, `/myapp`) and invoke Maven as follows:

    ```
    mvn appengine:update
    ```

2.  You will be prompted for an authorization code in the terminal window and your web browser will launch with a consent screen which you must accept in order to be authorized. Follow the prompts to copy any codes from the browser to the command line.

### Adding the App Engine Maven plugin to an existing Maven project

To add the Google App Engine Maven plugin to an existing Maven project, add the following into the `plugins` section in the project `pom.xml` file:

```
<plugin>
   <groupId>com.google.appengine</groupId>
   <artifactId>appengine-maven-plugin</artifactId>
   <version>1.9.34</version>
</plugin>
```

### Specifying a port for local testing

When you run your app in the local development server, the default port is `8080`. You can change this default by modifying the plugin entry for `appengine-maven-plugin` (or adding it if it doesn't exist). For example, we specify port and address in the following `<plugin>` entry within `<plugins>` inside the main app directory `pom.xml` file (`myapp/pom.xml`):

```
    <plugin>
         <groupId>com.google.appengine</groupId>
         <artifactId>appengine-maven-plugin</artifactId>
         <version>1.9.34</version>
         <configuration>
             <enableJarClasses>false</enableJarClasses>
             <port>8181</port>
             <address>0.0.0.0</address>
         </configuration>
    </plugin>
```

Notice that the `<port>` sets the port here to `8181` as shown, and the address `0.0.0.0` is specified, which means the development server will listen to requests coming in from the local network.

## Managing and running a project with the App Engine Maven plugin

The App Engine Maven plugin enables support for App Engine within Maven. It provides the ability to use the [development server](https://web.archive.org/web/20160424230715/https://cloud.google.com/appengine/docs/java/tools/devserver) and the majority of the functionality of the [appcfg](https://web.archive.org/web/20160424230715/https://cloud.google.com/appengine/docs/java/tools/uploadinganapp) tool.

The plugin also provides [Google Cloud Endpoints](https://web.archive.org/web/20160424230715/https://cloud.google.com/appengine/docs/java/endpoints) goals for discovery doc generation and client library generation.

Once the App Engine Maven plugin is added to the project's `pom.xml` file, several App Engine-specific Maven goals are available. To see all of the available goals, invoke the command:

```
    mvn help:describe -Dplugin=appengine
```

### App Engine Maven plugin goals

The App Engine Maven plugin goals can be categorized as devserver goals, app and project management goals, and Endpoints goals.

#### Development server goals

These are the development server goals:

`appengine:devserver`  
Runs the App Engine development server. When the server is running, it continuously checks to determine whether `appengine-web.xml` has changed. If it has, the server does a hot reload of the application. This means that you do not need to stop and restart your application because of changes to `appengine-web.xml`. The following parameters are available:

-   `<fullScanSeconds>`
-   `<address>`
-   `<disableUpdateCheck>`
-   `<jvmFlags>`
-   `<port>`
-   `<server>`

For example, to enable running the server in debug mode on port 8000 without suspending at start time, you can use the following flags:

```
<jvmFlags>
  <jvmFlag>-Xdebug</jvmFlag>
  <jvmFlag>-agentlib:jdwp=transport=dt_socket,address=8000,server=y,suspend=n</jvmFlag>
</jvmFlags>
```

By default, the `<fullScanSeconds>` flag is set to 5 seconds, which means server is checking every 5 seconds for changes in the web application files, and reloads the application automatically. This is useful with IDEs that support the *compile on save* feature like NetBeans. In order to use this feature, you must configure the `<build>` section as follows:

```
<build>
   <outputDirectory>target/${project.artifactId}-${project.version}/WEB-INF/classes</outputDirectory>
   <plugins>
      ....
   </plugins>
</build>
```

`appengine:devserver_start`  
Performs an asynchronous start for the devserver and then returns to the command line. When this goal runs, the behavior is the same as the `devserver` goal except that Maven continues processing goals and exits after the server is up and running.

`appengine:devserver_stop`  
Stops the development server. Available only if you started the development server with `appengine:devserver_start`.

#### Application management goals

For application and project management, the goals are listed in the following table:

<table>
<thead>
<tr class="header">
<th>Goal</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>appengine:backends_stop</code></td>
<td>This stops any running development server listening on the port as configured in your <code>pom.xml</code> file. This goal can be used in conjunction with the <code>devserver_start</code> command to do integration tests with the Maven plugin.</td>
</tr>
<tr class="even">
<td><code>appengine:backends_configure</code></td>
<td>Configure the specified backend.</td>
</tr>
<tr class="odd">
<td><code>appengine:backends_delete</code></td>
<td>Delete the specified backend.</td>
</tr>
<tr class="even">
<td><code>appengine:backends_rollback</code></td>
<td>Roll back a previously in-progress update.</td>
</tr>
<tr class="odd">
<td><code>appengine:backends_start</code></td>
<td>Start the specified backend.</td>
</tr>
<tr class="even">
<td><code>appengine:backends_update</code></td>
<td>Update the specified backend or (if no backend is specified) all backends.</td>
</tr>
<tr class="odd">
<td><code>appengine:enhance</code></td>
<td>Runs the App Engine Datanucleus JDO enhancer.</td>
</tr>
<tr class="even">
<td><code>appengine:rollback</code></td>
<td>Rollback an in-progress update.</td>
</tr>
<tr class="odd">
<td><code>appengine:set_default_version</code></td>
<td>Set the default application version.</td>
</tr>
<tr class="even">
<td><code>appengine:update</code></td>
<td>Create or update an app version.</td>
</tr>
<tr class="odd">
<td><code>appengine:update_cron</code></td>
<td>Update application cron jobs.</td>
</tr>
<tr class="even">
<td><code>appengine:update_dispatch</code></td>
<td>Update the application dispatch configuration.</td>
</tr>
<tr class="odd">
<td><code>appengine:update_dos</code></td>
<td>Update application DoS protection configuration.</td>
</tr>
<tr class="even">
<td><code>appengine:update_indexes</code></td>
<td>Update application indexes.</td>
</tr>
<tr class="odd">
<td><code>appengine:update_queues</code></td>
<td>Update application task queue definitions.</td>
</tr>
<tr class="even">
<td><code>appengine:vacuum_indexes</code></td>
<td>Delete unused indexes from application.</td>
</tr>
<tr class="odd">
<td><code>appengine:start_module_version</code></td>
<td>Start the specified module version.</td>
</tr>
<tr class="even">
<td><code>appengine:stop_module_version</code></td>
<td>Stop the specified module version.</td>
</tr>
</tbody>
</table>

##### Troubleshooting upload errors

If you use the update goal, your update attempt may fail with a message similar to this one: `404 Not Found This application does not exist (app_id=u'your-app-ID')`. This error will occur if you have multiple Google accounts and are using the wrong account to perform the update.

To solve this issue, change directories to `~`, locate a file named `.appcfg_oauth2_tokens_java`, and rename it. Then try updating again. http://maven.apache.org/archetype/maven-archetype-plugin

### Managing Versions with `versions-maven-plugin` command options

The [versions-maven-plugin](https://web.archive.org/web/20160424230715/http://mojo.codehaus.org/versions-maven-plugin/) provides several useful command options for managing versions in your Maven project. The following table lists useful activities supported by `versions-maven-plugin`:

<table>
<thead>
<tr class="header">
<th>Command</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>mvn versions:display-dependency-updates</code></td>
<td>Shows dependency updates (including app engine libs)</td>
</tr>
<tr class="even">
<td><code>mvn versions:display-plugin-updates</code></td>
<td>Shows plugin updates (including appengine-maven-plugin)</td>
</tr>
<tr class="odd">
<td><code>mvn versions:use-latest-versions</code></td>
<td>Automatically updates to the latest versions and makes a backup of the original <code>pom.xml</code>.</td>
</tr>
<tr class="even">
<td><code>mvn versions:commit</code></td>
<td>Removes the backup and makes the change permanent.</td>
</tr>
<tr class="odd">
<td><code>mvn versions:revert</code></td>
<td>Restores <code>pom.xml</code> with the contents of the backup <code>pom.xml</code>.</td>
</tr>
</tbody>
</table>

### Cloud Endpoints goals

These are the Endpoints goals:

`appengine:endpoints_get_client_lib`  
Generate a zip file in the directory `${project.build.directory}/generated-sources/appengine-endpoints/WEB-INF` for the Java client library for your endpoints.

`appengine:endpoints_get_discovery_doc`  
A combination of both `gen-api-config` and `gen-discovery-doc` commands. The Endpoints API and discovery documents for both REST and RPC are generated in the `${project.build.directory}/generated-sources/appengine-endpoints/WEB-INF` directory. The source `web.xml` is automatically changed with the addition of the Endpoints servlet and the correct Endpoints classes. In order to use the generated artifacts, you need to configure the Maven WAR plugin as follows:

```
<plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-war-plugin</artifactId>
        <version>2.3</version>
        <configuration>
          <webXml>${project.build.directory}/generated-sources/appengine-endpoints/WEB-INF/web.xml</webXml>
          <webResources>
            <resource>
              <directory>${project.build.directory}/generated-sources/appengine-endpoints</directory>
              <includes>
                <include>WEB-INF/*.discovery</include>
                <include>WEB-INF/*.api</include>
              </includes>
            </resource>
          </webResources>
        </configuration>
      </plugin>
```

You can automatically call this goal as part of a Maven build by configuring the App Engine Maven plugin as follows:

```
    <plugin>
            <groupId>com.google.appengine</groupId>
            <artifactId>appengine-maven-plugin</artifactId>
            <version>1.9.34</version>
            <configuration>
              <enableJarClasses>false</enableJarClasses>
            </configuration>
            <executions>
              <execution>
                <goals>
                  <goal>endpoints_get_discovery_doc</goal>
                </goals>
              </execution>
            </executions>
          </plugin>
```

For a reference configuration, see the Endpoint sample application [appengine-tictactoe-java-maven](https://web.archive.org/web/20160424230715/https://github.com/GoogleCloudPlatform/appengine-endpoints-tictactoe-java-maven).

## Adding JDO or JPA dependencies

If you use the Java Persistence API (JPA), or Java Data Objects (JDO), you can add the required dependencies to the `pom.xml` file generated using the App Engine artifacts documented in this page. For information on the entries you must add, see the App Engine documentation for [JDO](https://web.archive.org/web/20160424230715/https://cloud.google.com/appengine/docs/java/datastore/jdo/overview-dn2#build_tools) or [JPA](https://web.archive.org/web/20160424230715/https://cloud.google.com/appengine/docs/java/datastore/jpa/overview-dn2#build_tools).
