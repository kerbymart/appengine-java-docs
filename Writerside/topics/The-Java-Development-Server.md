# The Java Development Server

  

The [App Engine Java SDK](https://web.archive.org/web/20160424230254/https://cloud.google.com/appengine/downloads#Google_App_Engine_SDK_for_Java) includes a development web server for testing your application on your computer. The development web server simulates the App Engine Java runtime environment and all of its services, including Datastore. The [Google Plugin for Eclipse](https://web.archive.org/web/20160424230254/https://developers.google.com/eclipse/docs/appengine) can run the server in the Eclipse debugger. You can also run the development server from the command line.

1.  [Running the Development Web Server](#Running_the_Development_Web_Server)
2.  [Using the Datastore](#Using_the_Datastore)
3.  [Using Users](#Using_Users)
4.  [Using URL Fetch](#Using_URL_Fetch)
5.  [The Development Console](#The_Development_Console)
6.  [Command-Line Arguments](#Command_Line_Arguments)

## Running the Development Web Server

If you are using Eclipse and the Google Plugin, you can run the development web server in the Eclipse debugger. To run the server with the default configuration, select the **Run** menu, **Debug As &gt; Web Application**. For more control over how the server is started, such as which port the server uses, create a new debug configuration using the configuration type "Web Application". For more information, see [Google Plugin for Eclipse](https://web.archive.org/web/20160424230254/https://developers.google.com/eclipse/docs/appengine).

**Note:** When you start the development server from within Eclipse using the Google Plugin for Eclipse (discussed later), the server uses the port `8888` by default. When you start the server using the `dev_appserver` command described below, it uses the port `8080` by default. You can change the port number used by the `dev_appserver` command by providing the `--port=...` argument.

You can also run the development web server from a command prompt. The command to run is in the SDK's `appengine-java-sdk/bin/` directory.

If you are using Windows, the command is as follows:

```
appengine-java-sdk\bin\dev_appserver.cmd [options] war-directory-location
```

If you are using Mac OS X or Linux, the command is as follows:

```
appengine-java-sdk/bin/dev_appserver.sh [options] war-directory-location
```

The command takes the location of your application's WAR directory as an argument.

To stop the web server, press Control-C (on Windows, Mac or Linux).

These commands are OS-specific wrapper scripts that run the Java class `com.google.appengine.tools.KickStart` in `appengine-java-sdk/lib/appengine-tools-api.jar`.

## Using the Datastore

The development web server simulates the App Engine Datastore using a local file-backed Datastore on your computer. The Datastore is named `local_db.bin`, and it is created in your application's WAR directory, in the `WEB-INF/appengine-generated/` directory. (It is not uploaded with your application.)

This Datastore persists between invocations of the web server, so data you store will still be available the next time you run the web server. To clear the contents of the Datastore, shut down the server, then delete this file.

As described in [Datastore Index Configuration](https://web.archive.org/web/20160424230254/https://cloud.google.com/appengine/docs/java/config/indexconfig), the development server can generate configuration for Datastore indexes needed by your application, determined from the queries it performs while you are testing it. This generates a file named `datastore-indexes-auto.xml` in the directory `WEB-INF/appengine-generated/` in the WAR. To disable automatic index configuration, create or edit the `datastore-indexes.xml` file in the `WEB-INF/` directory, using the attribute `autoGenerate="false"` for the `<datastore-indexes>` element. See [Datastore Index Configuration](https://web.archive.org/web/20160424230254/https://cloud.google.com/appengine/docs/java/config/indexconfig) for more information.

### Browsing the local Datastore

To browse your local Datastore using the development web server:

1.  Start the development server as described previously.
2.  Go to the [Development Console](#The_Development_Console).
3.  Click **Datastore Viewer** in the left navigation pane to view your local Datastore contents.

### The High Replication Datastore Consistency Model

By default, the local Datastore is configured to simulate the consistency model of the [High Replication Datastore](https://web.archive.org/web/20160424230254/https://cloud.google.com/appengine/docs/java/datastore/hr/overview), with the percentage of Datastore writes that are not immediately visible in global queries set to 10%.

To adjust this level of consistency, set the `datastore.default_high_rep_job_policy_unapplied_job_pct` system property with a value corresponding to the amount of eventual consistency you want your application to see.

```
-Ddatastore.default_high_rep_job_policy_unapplied_job_pct=20
```
If you are setting this property using the command prompt `dev_appserver.sh`, you need to use `--jvm_flag=...` to set the property:
```
appengine-java-sdk/bin/dev_appserver.sh  --jvm_flag=-Ddatastore.default_high_rep_job_policy_unapplied_job_pct=20
```

The valid range for `datastore.default_high_rep_job_policy_unapplied_job_pct` is between 0 and 100. If you use numbers outside of this range, you will receive an error.

**Note:** If you require strong consistency for your query results, you need to use an ancestor query limiting the results to a single entity group. See [Structuring Data for Strong Consistency](https://web.archive.org/web/20160424230254/https://cloud.google.com/appengine/docs/java/datastore/structuring_for_strong_consistency) for more information.

If you're using the Google Plugin for Eclipse, adjust this setting by selecting **Run &gt; Run Configurations**. Select your app under **Web Application**. Select the **App Engine** tab. Then make changes to **Local Datastore Settings**:

![](https://web.archive.org/web/20160424230254im_/https://cloud.google.com/appengine/images/adjustLocalHRDEclipse.png)

If you're using [Maven](https://web.archive.org/web/20160424230254/https://cloud.google.com/appengine/docs/java/tools/maven#app_engine_maven_plugin_goals), you can pass this flag as an argument to `appengine:devserver` using the `jvmFlags`:

```
  <jvmFlags>
    <jvmFlag>-Ddatastore.default_high_rep_job_policy_unapplied_job_pct=20</jvmFlag>
  </jvmFlags>
```

### Specifying the Automatic ID Allocation Policy

You can configure how the local Datastore assigns [automatic entity IDs](https://web.archive.org/web/20160424230254/https://cloud.google.com/appengine/docs/java/datastore/entities#Java_Assigning_identifiers). The following automatic ID allocation policies are supported in the development server:

-   `sequential`: IDs are assigned from the sequence of consecutive integers.
-   `scattered`: IDs are assigned from a non-repeating sequence of approximately uniformly distributed integers.

**Note:** The auto ID assignment policies for the production server are completely different than those used by the development server. The `default` production server policy is similar to the `scattered` policy but not the same. There is no policy that corresponds to `sequential`. Your app should make no assumptions about the sequence of automatic IDs assigned in production.

The default policy in the local Datastore is `scattered`.

To specify the automatic ID policy, set the `datastore.auto_id_allocation_policy` system property to either `sequential` or `scattered`.

```
-Ddatastore.auto_id_allocation_policy=scattered
```

To set this system property through a flag passed to the dev\_appserver macro:

```
dev_appserver --jvm_flag=-Ddatastore.auto_id_allocation_policy=scattered
```

## Using Users

The development web server simulates Google Accounts with its own sign-in and sign-out pages. While running under the development web server, the methods that generate sign-in and sign-out URLs return URLs for `/_ah/login` and `/_ah/logout` on the local server.

The development sign-in page includes a form where you can enter an email address. Your session uses whatever email address you enter as the active user.

To have the application believe that the logged-in user is an administrator, check the "Sign in as Administrator" checkbox on the form.

## Using URL Fetch

When your application uses the URL Fetch API to make an HTTP request, the development web server makes the request directly from your computer. The behavior may differ from when your application runs on App Engine if you use a proxy server for accessing websites.

## The Development Console

The development web server includes a console web application. With the console you can browse the local Datastore.

To access the console, visit the URL `/_ah/admin` on your server: [http://localhost:8080/\_ah/admin](https://web.archive.org/web/20160424230254/http://localhost:8080/_ah/admin)

## Command-Line Arguments

The development server command supports the following command-line arguments:

`--address=`*`...`*  
The host address to use for the server. You may need to set this to be able to access the development server from another computer on your network. An address of `0.0.0.0` allows both localhost access and hostname access. Default is `localhost`.

`--default_gcs_bucket=`*`...`*  
Set the default Google Cloud Storage bucket name.

`--disable_update_check`  
If given, the development server will not contact App Engine to check for the availability of a new release of the SDK. By default, the server checks for a new version on start-up, and prints a message if a new version is available.

`--generated_dir=`*`...`*  
Set the directory where generated files are created.

`--help`  
Prints a helpful message then quits.

`--jvm_flag=`*`...`*  
Pass the given flag as a JVM argument. May be repeated to supply multiple flags.

`--port=`*`...`*  
The port number to use for the server. Default is `8080`.

`--sdk_root=`*`...`*  
A path to the App Engine Java SDK, if different from the location of the tool.

`--server=`*`...`*  
The server to use to determine the latest SDK version.
