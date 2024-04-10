# Converting Backends to Modules

  

The original App Engine architecture is based on a single frontend instance with optional backend instances. If you have an application that uses backends, you may want to convert the backends to module format to take full advantage of the additional functionality that modules provide (such as the ability to version backends).

App Engine automatically runs an existing backend as a new, non-default version of the default module. Resident backends are assigned manual scaling and dynamic backends are assigned basic scaling.

You can convert backend instances to named modules that are versioned and have explicit scaling type and instance classes. You must replace the original `backends.xml` file with multiple WAR files, one for each module. The procedure is as follows:

1.  Create a top level `EAR` directory.
2.  Create a `META-INF` subdirectory in the `EAR` directory that contains an `appengine-application.xml` file and an `application.xml` file.
3.  Add module declaration elements to the `appengine-application.xml` file.
4.  Create a subdirectory in the `EAR` directory for each module in the app including the default module. By convention, each subdirectory has the same name as the module it defines.
5.  Copy the contents of the original `WAR` directory into each subdirectory. The original `WAR` should have a subdirectory named `WEB-INF`. Each new subdirectory must have its own copy of that directory.
6.  Add module configuration elements (to define module name, scaling, instance class, etc.) to the `appengine-web.xml` files in each subdirectory.
7.  Application configuration files (`cron.xml`, `dispatch.xml`, `dos.xml`, etc.) should only be included in the default module subdirectory. Remove these files from all the other module subdirectories.

For example, here is a file `example/backends.xml` that defines three backends (`memdb`, `worker`, and `cmdline`):

```
<backends>
 <backend name="memdb">
   <class>B8</class>
   <instances>5</instances>
 </backend>
 <backend name="worker">
   <options>
     <fail-fast>true</fail-fast>
   </options>
 </backend>
 <backend name="cmdline">
   <options>
     <dynamic>true</dynamic>
   </options>
 </backend>
</backends>
```

To convert these backends to modules, first assume you've created a new EAR directory called "ear." Declare the backends as modules in the file `ear/META-INF/application.xml`. Notice that this file also declares the default module as well:

```
<?xml version="1.0" encoding="UTF-8"?>
<application
  xmlns="http://java.sun.com/xml/ns/javaee"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
                      http://java.sun.com/xml/ns/javaee/application_5.xsd"
  version="5">

  <description>Demo Application</description>
  <display-name>My Java App</display-name>

  <!-- Default Module -->
  <module>
    <web>
      <web-uri>default</web-uri>
      <context-root>defaultcontextroot</context-root>
    </web>
  </module>
  <!-- Other Modules -->
  <module>
    <web>
      <web-uri>memdb</web-uri>
      <context-root>memdb</context-root>
    </web>
  </module>
  <module>
    <web>
      <web-uri>worker</web-uri>
      <context-root>worker</context-root>
    </web>
  </module>
  <module>
    <web>
      <web-uri>cmdline</web-uri>
      <context-root>cmdline</context-root>
    </web>
  </module>
</application>
```

The configuration for each module is written in a separate `appengine-web.xml` file, under the corresponding subdirectory. In `ear/memdb/WEB-INF/appengine-web.xml`:

```
<appengine-web-app xmlns="http://appengine.google.com/ns/1.0">
  <application>my-java-app</application>
  <module>memdb</module>
  <version>uno</version>
  <threadsafe>true</true>
  <instance-class>F4</instance-class>
  <manual-scaling>
    <instances>5</instances>
  </manual-scaling>
</appengine-web-app>
```

In `ear/worker/WEB-INF/appengine-web.xml`:

```
// ear/worker/WEB-INF/appengine-web.xml
<appengine-web-app xmlns="http://appengine.google.com/ns/1.0">
  <application>my-java-app</application>
  <module>worker</module>
  <version>uno</version>
  <threadsafe>true</threadsafe>
   <!-- For failfast functionality, please use the X-AppEngine-FailFast header on requests made to this module. -->
  <instance-class>F2</instance-class>
  <manual-scaling>
    <instances>5</instances>
  </manual-scaling>
</appengine-web-app>
```

In `ear/cmdline/WEB-INF/appengine-web.xml`:

```
<appengine-web-app xmlns="http://appengine.google.com/ns/1.0">
  <application>my-java-app</application>
  <module>cmdline</module>
  <version>uno</version>
  <threadsafe>true</threadsafe>
  <instance-class>F2</instance-class>
  <basic-scaling>
    <max-instances>10?</max-instances>
  </basic-scaling>
</appengine-web-app>
```
