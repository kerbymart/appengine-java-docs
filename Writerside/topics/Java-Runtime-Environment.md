# Java Runtime Environment

  

[Python](https://web.archive.org/web/20160424230608/https://cloud.google.com/appengine/docs/python/runtime "View this page in the Python runtime") <span style="color: #ddd; padding: 0em .5em;">|</span><span style="color: black; font-weight:bold">Java</span> <span style="color: #ddd; padding: 0em .5em;">|</span>[PHP](https://web.archive.org/web/20160424230608/https://cloud.google.com/appengine/docs/php/runtime "View this page in the PHP runtime") <span style="color: #ddd; padding: 0em .5em;">|</span>[Go](https://web.archive.org/web/20160424230608/https://cloud.google.com/appengine/docs/go/runtime "View this page in the Go runtime")

Welcome to Google App Engine for Java! With App Engine, you can build Java web applications that use Google's scalable infrastructure and services.

The Java runtime is also available in the [flexible](https://web.archive.org/web/20160424230608/https://cloud.google.com/appengine/docs/flexible/java/) environment.

1.  [Introduction](#Java_Introduction)
2.  [Selecting the Java runtime](#Java_Selecting_the_Java_runtime)
3.  [The sandbox](#Java_The_sandbox)
4.  [Class loader JAR ordering](#Java_Class_loader_JAR_ordering)
5.  [The JRE white list](#Java_The_JRE_white_list)
6.  [No signed JAR files](#Java_No_signed_JAR_files)
7.  [The Java SDK and tools](#Java_tools)

## Introduction

App Engine runs your Java web application using a Java 7 JVM in a safe "sandboxed" environment. App Engine invokes your app's servlet classes to handle requests and prepare responses in this environment.

The [Google Plugin for Eclipse](https://web.archive.org/web/20160424230608/https://developers.google.com/eclipse/docs/appengine) adds new project wizards and debug configurations to your [Eclipse](https://web.archive.org/web/20160424230608/http://www.eclipse.org/) IDE for App Engine projects. App Engine for Java makes it especially easy to develop and deploy world-class web applications using [Google Web Toolkit](https://web.archive.org/web/20160424230608/https://cloud.google.com/web-toolkit/) (GWT). The Eclipse plugin comes bundled with the App Engine and GWT SDKs.

Third-party plugins are available for other Java IDEs as well. For example, for NetBeans see [NetBeans plugin for Gaelyk Framework](https://web.archive.org/web/20160424230608/https://code.google.com/p/nb-gaelyk-plugin/downloads/list). For IntelliJ, see [Google App Engine Integration for IntelliJ](https://web.archive.org/web/20160424230608/http://plugins.intellij.net/plugin/?id=4254). (These links take you to third-party resources.)

**Warning:** Support for Java 6 has been deprecated and is going to be removed. See the [Java 6 Runtime Support Turndown](https://web.archive.org/web/20160424230608/https://cloud.google.com/appengine/docs/deprecations/java6) document for details and timetable.

App Engine uses [the Java Servlet 2.5 standard](https://web.archive.org/web/20160424230608/https://jcp.org/aboutJava/communityprocess/mrel/jsr154/index.html) for web applications. You provide your app's servlet classes, JavaServer Pages (JSPs), static files and data files, along with the deployment descriptor (the `web.xml` file) and other configuration files, in a standard WAR directory structure. App Engine serves requests by invoking servlets according to the deployment descriptor.

The secured "sandbox" environment isolates your application for service and security. It ensures that apps can only perform actions that do not interfere with the performance and scalability of other apps. For instance, an app cannot spawn threads in some ways, write data to the local file system or make arbitrary network connections. An app also cannot use JNI or other native code. The JVM can execute any Java bytecode that operates within the sandbox restrictions.

The App Engine platform provides many [services](https://web.archive.org/web/20160424230608/https://cloud.google.com/appengine/docs/java/apis) that your code can call. Your application can also configure [scheduled tasks](https://web.archive.org/web/20160424230608/https://cloud.google.com/appengine/docs/java/config/cron) that run at specified intervals.

If you haven't already, see [the Java Getting Started Guide](https://web.archive.org/web/20160424230608/https://cloud.google.com/appengine/docs/java/gettingstarted) for an interactive introduction to developing web applications with Java technologies and Google App Engine.

## Selecting the Java runtime

App Engine knows to use the Java runtime environment for your application when you use the AppCfg tool from the Java SDK to upload the app.

The App Engine Java API is represented by the `appengine-api-*.jar` included with the SDK (where `*` represents the version of the API and the SDK). You select the version of the API your application uses by including this JAR in the application's `WEB-INF/lib/` directory. If a new version of the Java runtime environment is released that introduces changes that are not compatible with existing apps, that environment will have a new version number. Your application will continue to use the previous version until you replace the JAR with the new version (from a newer SDK) and re-upload the app.

## The sandbox

To allow App Engine to distribute requests for applications across multiple web servers, and to prevent one application from interfering with another, the application runs in a restricted "sandbox" environment. In this environment, the application can execute code, store and query data in the App Engine datastore, use the App Engine mail, URL fetch and users services, and examine the user's web request and prepare the response.

An App Engine application cannot:

-   write to the filesystem. Applications must use [the App Engine datastore](https://web.archive.org/web/20160424230608/https://cloud.google.com/appengine/docs/java/datastore) for storing persistent data. Reading from the filesystem is allowed, and all application files uploaded with the application are available.

-   respond slowly. A web request to an application must be handled within a few seconds. Processes that take a very long time to respond are terminated to avoid overloading the web server.

-   make other kinds of system calls.

### Threads

A Java application can create a new thread, but there are some restrictions on how to do it. These threads can't "outlive" the request that creates them. (On a backend server, an application can spawn a [background thread](https://web.archive.org/web/20160424230608/https://cloud.google.com/appengine/docs/java/backends/#Java_Background_threads), a thread that can "outlive" the request that creates it.)

An application can

-   Implement `java.lang.Runnable`; and
-   Create a thread factory by calling [`com.google.appengine.api.ThreadManager.currentRequestThreadFactory()`](https://web.archive.org/web/20160424230608/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/ThreadManager#currentRequestThreadFactory())
-   call the factory's `newRequestThread` method, passing in the `Runnable`, `newRequestThread(runnable)`

or use the factory object returned by [`com.google.appengine.api.ThreadManager.currentRequestThreadFactory()`](https://web.archive.org/web/20160424230608/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/ThreadManager#currentRequestThreadFactory()) with an `ExecutorService` (e.g., call `Executors.newCachedThreadPool(factory)`).

However, you must use one of the methods on `ThreadManager` to create your threads. You cannot invoke `new Thread()` yourself or use the [default thread factory](https://web.archive.org/web/20160424230608/http://docs.oracle.com/javase/6/docs/api/java/util/concurrent/Executors.html#defaultThreadFactory).

An application can perform operations against the current thread, such as `thread.interrupt()`.

Each request is limited to 50 concurrent request threads. The Java runtime will throw a `java.lang.IllegalStateException` if you try to create more than 50 threads in a single request.

**Caution:** Threads are a powerful feature that are full of surprises. If you aren't comfortable with using threads with Java, we recommend Goetz, *Java Concurrency in Practice*.

### The Filesystem

A Java application cannot use any classes used to write to the filesystem, such as `java.io.FileWriter`. An application can read its own files from the filesystem using classes such as `java.io.FileReader`. An application can also access its own files as "resources", such as with `Class.getResource()` or `ServletContext.getResource()`.

Only files that are considered "resource files" are accessible to the application via the filesystem. By default, all files in the WAR are "resource files." You can exclude files from this set using the [appengine-web.xml](https://web.archive.org/web/20160424230608/https://cloud.google.com/appengine/docs/java/config/appconfig) file.

### java.lang.System

Features of the `java.lang.System` class that do not apply to App Engine are disabled.

The following `System` methods do nothing in App Engine: `exit()`, `gc()`, `runFinalization()`, `runFinalizersOnExit()`

The following `System` methods return `null`: `inheritedChannel()`, `console()`

An app cannot provide or directly invoke any native JNI code. The following `System` methods raise a `java.lang.SecurityException`: `load()`, `loadLibrary()`, `setSecurityManager()`

### Reflection

An application is allowed full, unrestricted, reflective access to its own classes.

It may query any private members, call the method `java.lang.reflect.AccessibleObject.setAccessible()`, and read/set private members.

An application can also reflect on JRE and API classes, such as `java.lang.String` and `javax.servlet.http.HttpServletRequest`. However, it can only access public members of these classes, not protected or private.

An application cannot reflect against any other classes not belonging to itself, and it can not use the `setAccessible()` method to circumvent these restrictions.

### Custom class loading

Custom class loading is fully supported under App Engine. An application is allowed to define its own subclass of ClassLoader that implements application-specific class loading logic. Please be aware, though, that App Engine overrides all ClassLoaders to assign the same permissions to all classes loaded by your application. If you perform custom class loading, be cautious when loading untrusted third-party code.

## Class loader JAR ordering

Sometimes, it may be necessary to redefine the order in which JAR files are scanned for classes in order to resolve collisions between class names. In these cases, loading priority can be granted to specific JAR files by adding a `<class-loader-config>` element containing `<priority-specifier>` elements in the `appengine-web.xml` file. For example:

```
<class-loader-config>
    <priority-specifier filename="mailapi.jar"/>
</class-loader-config>
```

This places "`mailapi.jar`" as the first JAR file to be searched for classes, barring those in the directory `war/WEB-INF/classes/`.

If multiple JAR files are prioritized, their original loading order (with respect to each other) will be used. In other words, the order of the `<priority-specifier>` elements themselves does not matter.

## The JRE white list

Access to the classes in the Java standard library (the Java Runtime Environment, or JRE) is limited to the classes in [the App Engine JRE White List](https://web.archive.org/web/20160424230608/https://cloud.google.com/appengine/docs/java/jrewhitelist).

## No signed JAR files

App Engine's precompilation isn't compatible with signed JAR files. If your application is precompiled (the default), it can't load signed JAR files. If the application tries to load a signed JAR, at runtime App Engine will generate an exception like

```
java.lang.SecurityException: SHA1 digest error for com/example/SomeClass.class
    at com.google.appengine.runtime.Request.process-d36f818a24b8cf1d(Request.java)
    at sun.security.util.ManifestEntryVerifier.verify(ManifestEntryVerifier.java:210)
    at java.util.jar.JarVerifier.processEntry(JarVerifier.java:218)
    at java.util.jar.JarVerifier.update(JarVerifier.java:205)
    at java.util.jar.JarVerifier$VerifierStream.read(JarVerifier.java:428)
    at sun.misc.Resource.getBytes(Resource.java:124)
    at java.net.URLClassLoader.defineClass(URLClassLoader.java:273)
    at sun.reflect.GeneratedMethodAccessor5.invoke(Unknown Source)
    at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
    at java.lang.reflect.Method.invoke(Method.java:616)
    at java.lang.ClassLoader.loadClass(ClassLoader.java:266)
```

There are two ways to work around this:

-   [Strip the JAR's signature](https://web.archive.org/web/20160424230608/https://www.google.com/search?q=java+unsign+jar&nfpr=1); **or**
-   [Disable precompilation for the whole application](https://web.archive.org/web/20160424230608/https://cloud.google.com/appengine/docs/java/config/appconfig#Java_appengine_web_xml_Disabling_precompilation).

## The Java SDK and tools

The [App Engine Java SDK](https://web.archive.org/web/20160424230608/https://cloud.google.com/appengine/downloads#Google_App_Engine_SDK_for_Java) includes tools for testing your application, uploading your application files, and downloading log data. The SDK also includes a component for [Apache Ant](https://web.archive.org/web/20160424230608/https://cloud.google.com/appengine/docs/java/tools/ant) to simplify tasks common to App Engine projects. The [Google Plugin for Eclipse](https://web.archive.org/web/20160424230608/https://developers.google.com/eclipse/docs/appengine) adds features to the Eclipse IDE for App Engine development, testing and deployment, and includes the complete App Engine SDK. The Eclipse plugin also makes it easy to develop [Google Web Toolkit](https://web.archive.org/web/20160424230608/https://cloud.google.com/web-toolkit/) applications and run them on App Engine.

The App Engine Java SDK also has a plugin for supporting development with [Apache Maven](https://web.archive.org/web/20160424230608/https://cloud.google.com/appengine/docs/java/tools/maven).

The [development server](https://web.archive.org/web/20160424230608/https://cloud.google.com/appengine/docs/java/tools/devserver) runs your application on your local computer for development and testing. The server simulates the App Engine datastore, services and sandbox restrictions. The development server can also generate configuration for datastore indexes based on the queries the app performs during testing.

A multipurpose tool called [AppCfg](https://web.archive.org/web/20160424230608/https://cloud.google.com/appengine/docs/java/tools/uploadinganapp) handles all command-line interaction with your application running on App Engine. AppCfg can upload your application to App Engine, or just update the datastore index configuration so you can build new indexes before updating the code. It can also download the app's log data, so you can analyze your app's performance using your own tools.
