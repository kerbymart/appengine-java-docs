# Uploading and Managing a Java App

  

The App Engine Java SDK includes a command for interacting with App Engine. You can use this command to upload new versions of the code, configuration, and static files for your app to App Engine. You can also use the command to manage datastore indexes and download log data.

**Note:** If you created your project using the Google Cloud Platform Console, your project has a title and an ID. In the instructions that follow, the *project* title and ID can be used wherever an *application* title and ID are mentioned. They are the same thing.

1.  [The appcfg command](#appcfg_command)
2.  [Uploading the App](#Uploading_the_App)
3.  [Updating Indexes](#Updating_Indexes)
4.  [Deleting Unused Indexes](#Deleting_Unused_Indexes)
5.  [Managing Task Queues](#Managing_Task_Queues)
6.  [Updating the dispatch file](#Java_Updating_the_dispatch_file)
7.  [Managing DoS Protection](#Managing_DoS_Protection)
8.  [Managing Scheduled Tasks](#Managing_Scheduled_Tasks)
9.  [Downloading an Application](#Downloading_an_Application)
10. [Downloading Logs](#Downloading_Logs)
11. [Using an HTTP Proxy](#Using_an_HTTP_Proxy)
12. [Staging yaml files](#staging_yaml_files)
13. [Command Line Arguments](#Command_Line_Arguments)

## The appcfg command

The `appcfg` command provides a command line interface for uploading your app. It also supports other features such as downloading logs. The command is located in the SDK's `appengine-java-sdk/bin/` directory. It is available in two OS-specific wrapper scripts that run the Java class `com.google.appengine.tools.admin.AppCfg` in `appengine-java-sdk/lib/appengine-tools-api.jar`:

If you are using Windows, the command is as follows:

```
appengine-java-sdk\bin\appcfg.cmd [options] <action> <war-location>
```

If you are using Mac OS X or Linux, the command is as follows:

```
./appengine-java-sdk/bin/appcfg.sh [options] <action> <war-location>
```

The command takes the name of an action to perform and the location of your application's WAR directory as arguments.

The `appcfg` command uses your Google account to authenticate. The default authentication scheme is [OAuth 2.0](https://web.archive.org/web/20160424230850/https://developers.google.com/identity/protocols/OAuth2), which asks the user to login and authorize `appcfg` through a browser. After a successful authorization, `appcfg` stores the access token so that the user is not prompted again for subsequent attempts. The token can be deleted by manually removing `.appcfg_oauth2_tokens_java` found in the user's home directory. You can also specify the `--no_cookies` option to prevent the token from being stored at all.

## Uploading the App

To upload an application, use the `update` action, as follows:

```
./appengine-java-sdk/bin/appcfg.sh update myapp/war
```

You can [upload one or more modules at a time](https://web.archive.org/web/20160424230850/https://cloud.google.com/appengine/docs/java/modules/#Java_Uploading_modules) with the `update` command.

You can also upload your application from Eclipse, if using the [Google Plugin for Eclipse](https://web.archive.org/web/20160424230850/https://developers.google.com/eclipse/docs/appengine).

**Note:** Currently, App Engine has a limit of 10 deployed versions per application. If you try to deploy an 11th version you'll get an error: `Too Many Versions (403).` You can use the Cloud Platform Console to delete an older version and then upload your latest code.

### Troubleshooting Upload Errors

If you use the update feature with [passwordless login](#Passwordless_Login_with_OAuth2), your update attempt may fail with a message similar to this one: `404 Not Found This application does not exist (app_id=u'your-app-ID')`. This error will occur if you have multiple Google accounts and are using the wrong account to perform the update.

To solve this issue, change directories to `~`, locate a file named `.appcfg_oauth2_tokens_java`, and rename it. Then try updating again.

The [App Engine Maven plugin](https://web.archive.org/web/20160424230850/https://cloud.google.com/appengine/docs/java/tools/maven#app_engine_maven_plugin_goals) update feature uses passwordless login by default, so this error can also be experienced when using that plugin.

## Updating Indexes

When you upload an application using the `update` action, the update includes the app's index configuration (the `datastore-indexes.xml` and `generated/datastore-indexes-auto.xml` files). If the index configuration defines an index that doesn't exist yet on App Engine, App Engine creates the new index. Depending on how much data is already in the datastore that needs to be mentioned in the new index, the process of creating the index may take a while. If the app performs a query that requires an index that hasn't finished building yet, the query will raise an exception.

To prevent this, you must ensure that the new version of the app that requires a new index is not the live version of the application until the indexes finish building. One way to do this is to give the app a new version number in `appengine-web.xml` whenever you add or change an index in the configuration. The app is uploaded as a new version, and does not become the default version automatically. When your indexes have finished building, you can [change the default version](https://web.archive.org/web/20160424230850/https://console.cloud.google.com/project/_/appengine/versions) in the Cloud Platform Console.

Another way to ensure that new indexes are built before the new app goes live is to upload the index configuration separately before uploading the app. To upload only the index configuration for an app, use the `update_indexes` action:

```
./appengine-java-sdk/bin/appcfg.sh update_indexes myapp/war
```

The `index.xml` file should be in the default module's WEB-INF directory.

In the Cloud Platform Console, you can [check the status of the app's indexes](https://web.archive.org/web/20160424230850/https://console.cloud.google.com/project/_/datastore/indexes).

## Deleting Unused Indexes

When you change or remove an index from the index configuration, the original index is *not* deleted from App Engine automatically. This gives you the opportunity to leave an older version of the app running while new indexes are being built, or to revert to the older version immediately if a problem is discovered with a newer version.

When you are sure that old indexes are no longer needed, you can delete them from App Engine using the `vacuum_indexes` action:

```
./appengine-java-sdk/bin/appcfg.sh vacuum_indexes myapp/war
```

This command deletes all indexes for the app that are not mentioned in the local versions of `datastore-indexes.xml` and `generated/datastore-indexes-auto.xml`.

## Managing Task Queues

You define and configure task queues in a file called `queue.xml`. You can upload this configuration separately from the rest of the app using the `update_queues` command:

```
./appengine-java-sdk/bin/appcfg.sh update_queues myapp/war
```

`appcfg update` will also upload task queue configuration if the file exists.

The `queue.xml` file should be in the default module's WEB-INF directory. For more on task queues, see the [Task Queues](https://web.archive.org/web/20160424230850/https://cloud.google.com/appengine/docs/java/taskqueue) documentation.

## Updating the dispatch file

To upload a new version of your app's dispatch file, use the `appcfg.sh update_dispatch` command:

```
appcfg.sh update_dispatch myapp/war
```

The `dispatch.xml` file should be in the default module's WEB-INF directory. Be sure that all the modules mentioned in the file have been uploaded before using this command.

You can also upload the dispatch file at the same time you upload modules using the optional `auto_update_dispatch` flag with the [update command](#appcfg_sh).

## Managing DoS Protection

You configure DoS protection for an app in a file called `dos.xml`. You can upload this configuration separately from the rest of the app using the `update_dos` command:

```
./appengine-java-sdk/bin/appcfg.sh update_dos myapp/war
```

The `dos.xml` file should be in the default module's WEB-INF directory.

`appcfg update` will also upload DoS protection configuration if the file exists.

## Managing Scheduled Tasks

App Engine supports scheduled tasks (known as cron jobs). You specify these in a file called `cron.xml`, and upload them using the `update_cron` command:

```
./appengine-java-sdk/bin/appcfg.sh update_cron myapp/war
```

The `cron.xml` file should be in the default module's WEB-INF directory.

`appcfg update` will also upload cron job specifications if the file exists.

You can also verify your cron configuration from the command line using the `cron_info` action:

```
./appengine-java-sdk/bin/appcfg.sh cron_info myapp/war
```

For more on cron jobs, see the [Cron Jobs](https://web.archive.org/web/20160424230850/https://cloud.google.com/appengine/docs/java/config/cron) documentation.

## Downloading an Application

You can download an application by running appcfg.sh with the `download_app` action in the Java SDK command line tool:

```
./appengine-java-sdk/bin/appcfg.sh -A <your_app_id> -V <your_app_version> download_app <output-dir>
```

This command downloads your compiled application in the WAR that you uploaded most recently.

Output like the following results if the command is successful:

```
0% Fetching files...
0% [1/13] WEB-INF/lib/appengine-api-labs-1.2.6.jar
7% [2/13] WEB-INF/appengine-web.xml
14% [3/13] WEB-INF/lib/datanucleus-jpa-1.1.5.jar
21% [4/13] WEB-INF/lib/datanucleus-core-1.1.5.jar
28% [5/13] WEB-INF/lib/datanucleus-appengine-1.0.3.jar
35% [6/13] WEB-INF/lib/jdo2-api-2.3-eb.jar
42% [7/13] WEB-INF/classes/com/example/MyServlet.class
50% [8/13] WEB-INF/lib/geronimo-jpa_3.0_spec-1.1.1.jar
64% [9/13] WEB-INF/lib/geronimo-jta_1.1_spec-1.1.1.jar
71% [10/13] testlogin.html
78% [11/13] WEB-INF/web-bck.xml
85% [12/13] WEB-INF/web.xml
92% [13/13] __static__/testlogin.html

download_app completed successfully.
```

Only the developer who uploaded the app can download it. If anyone other than that developer attempts to download the app, they'll receive an error message.

## Downloading Logs

App Engine maintains a log of messages that your application emits. App Engine also records each request in the log. Each request is assigned a [request ID](https://web.archive.org/web/20160424230850/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/log/RequestLogs#getRequestId()), a globally unique identifier based on the request's start time. Each [log level](#command_line_arguments_appcfg_severity) has a fixed buffer size that controls the amount of log information you can access. Normally, you use logging features more at lower log levels; thus, the time window is smaller for log events at these levels. In the Cloud Platform Console, you can [browse your app's logs of the last 90 days](https://web.archive.org/web/20160424230850/https://console.cloud.google.com/project/_/logs).

If you wish to perform more detailed analysis of your application's logs, you can download the log data to a file on your computer. To download logs to a file named `mylogs.txt`, use the `request_logs` action, as follows:

```
./appengine-java-sdk/bin/appcfg.sh request_logs myapp/war mylogs.txt
```

By default, the command downloads log messages from the current calendar day (since midnight Pacific Time) with a log level of INFO or higher (omitting DEBUG level messages). You can adjust the number of days, the minimum log level, and whether to overwrite or append to the local log file using command line options.

By default, the command overwrites the local log file. You can tell it to append the downloaded data to the file instead by specifying the `--append` argument. This simply appends the requested log data to the file, it doesn't guarantee that the file doesn't contain duplicate log messages.

## Using an HTTP Proxy

If your development environment resides behind an HTTP proxy, the `appcfg` command needs to know about the proxy to connect to App Engine. To tell AppCfg about your proxy, using the `--proxy` option:

```
./appengine-java-sdk/bin/appcfg.sh --proxy=10.1.2.3 update myapp/war
```

If you use a different proxy for HTTPS, you can specify it using the `--proxy_https` option.

## Staging yaml files

The `stage` action is a utility that creates a directory of `yaml` configuration files that correspond to the contents of a WAR directory. This can be helpful if you are deploying your app using the [gcloud](https://web.archive.org/web/20160424230850/https://cloud.google.com/sdk/gcloud/) command line tool, which can only accept `yaml` files.
```
./appengine-java-sdk/bin/appcfg.sh stage <WAR-dir> <yaml-dir>
```

## Command Line Arguments

The `appcfg` command takes a set of options, an action, and arguments for the action.

The following actions are available:

`appcfg.sh `*`[options]`*` update `*`<app-directory>`*`|`*`<files...>`*  
If you specify an app-directory, it must contain of the files required by the app. The update command creates or updates the app version named in the app.yaml file at the top level of the directory. It follows symlinks and recursively uploads all files to the server. Temporary or source control files (e.g. foo~, .svn/\*) are skipped.

You can [update one or more modules at a time](https://web.archive.org/web/20160424230850/https://cloud.google.com/appengine/docs/java/modules/#Java_Uploading_modules) with the `update` command.

You can also [update the dispatch file](#Java_Updating_the_dispatch_file) at the same time you update modules, by adding the optional `auto_update_dispatch` flag, which can be used in two forms:

```
appcfg.sh --auto_update_dispatch update <app-directory>|<files...>
appcfg.sh -D update <app-directory>|<files...>
```

`appcfg.sh `*`[options]`*` cron_info `*`<app-directory>`*  
Verifies and prints the scheduled task (cron) configuration.

`appcfg.sh -A <app_id> -V <version> download_app <output-dir>`  
Retrieves the most current version of your application using the following options:

`-A`  
The application ID (required).

`-V`  
The current application version. May be one of the following:

-   `-V major.minor` – Specifies the exact version specified.
-   `-V major` – Specifies the latest major version.
-   Omitted entirely – Returns the current default version, if one exists

`<output-dir>`  
The directory where you wish to save the files (required).

`appcfg.sh `*`[options]`*` request_logs `*`<war-location>`*` `*`<output-file>`*  
Retrieves log data for the application running on App Engine. *`output-file`* is the name of the file to create or replace. If *`output-file`* is a hyphen (`-`), the log data is printed to the console. The following options apply to `request_logs`:

`--num_days=`*`...`*  
The number of days of log data to retrieve, ending on the current date at midnight UTC. A value of 0 retrieves all available logs. If `--append` is given, then the default is 0, otherwise the default is 1.

`--severity=`*`...`*  
The minimum log level for the log messages to retrieve. The value is a number corresponding to the log level: 4 for CRITICAL, 3 for ERROR, 2 for WARNING, 1 for INFO, 0 for DEBUG. All messages at the given log level and above will be retrieved. Default is 1 (INFO).

`--append`  
Tells AppCfg to append logs to the log output file instead of overwriting the file. This simply appends the requested data, it does not guarantee the file won't contain duplicate error messages. If this argument is not specified, AppCfg will overwrite the log output file.

`appcfg.sh `*`[options]`*` help `*`<action>`*  
Prints a help message about the given action, then quits.

`appcfg.sh `*`[options]`*` version`  
Prints detailed version information about the SDK, Java and the operating system, then quits. Use this when submitting bug reports.

`appcfg.sh `*`[options]`*` rollback `*`<war-location>`*  
Undoes a partially completed update for the given application. You can use this if an update was interrupted, and the command is reporting that the application cannot be updated due to a lock.

`appcfg.sh `*`[options]`*` update `*`<war-location>`*  
Uploads files for an application given the application's root directory. The application ID and version are taken from the `appengine-web.xml` file.

`appcfg.sh `*`[options]`*` set_default_version `*`<war-directory>`*  
Specifies the default version of the given module.

`appcfg.sh `*`[options]`*` update_cron `*`<war-directory>`*  
Updates the schedule task (cron) configuration for the app, based on the `cron.xml` file.

`appcfg.sh `*`[options]`*` update_dos `*`<war-directory>`*  
Updates the DoS protection configuration for the app, based on the `dos.xml` file.

`appcfg.sh `*`[options]`*` update_indexes `*`<war-directory>`*  
Updates datastore indexes in App Engine to include newly added indexes. If a new version of your application requires an additional index definition that was added to the index configuration, you can update your index configuration in App Engine prior to uploading the new version of your application. Running this action a few hours prior to uploading your new application version will give the indexes time to build and be serving when the application is deployed.

`appcfg.sh `*`[options]`*` update_queues `*`<war-directory>`*  
Updates the task queue configuration (`queue.xml`) in App Engine. If the new configuration does not include a named queue (other than `default`) that exists and has tasks, the existing queue is paused.

`appcfg.sh `*`[options]`*` update_dispatch `*`<war-directory>`*  
Updates the dispatch configuration (`dispatch.xml`) in App Engine.

`appcfg.sh `*`[options]`*` start `*`<war-location>`*  
Start the specified module version.

`appcfg.sh `*`[options]`*` stop `*`<war-location>`*  
Stop the specified module version.

`appcfg.sh `*`[options]`*` stage `*`<war-location>`*` `*`<yaml-location>`*  
Copy the contents of the WAR directory into a new directory, translating the configuration data into `yaml` file format.

The `appcfg` command accepts the following options for all actions:

`--server=`*`...`*  
The App Engine server hostname. The default is `appengine.google.com`.

`--host=`*`...`*  
The hostname of the local machine for use with remote procedure calls.

`--sdk_root=`*`...`*  
A path to the App Engine Java SDK, if different from the location of the tool.

`--no_cookies`  
Do not store the user's access token. Go through the OAuth2 flow every time.

`--proxy=...`  
Use the given HTTP proxy to contact App Engine.

`--proxy_https=...`  
Use the given HTTPS proxy to contact App Engine, when using HTTPS. If `--proxy` is given but `--proxy_https` is not, both HTTP and HTTPS requests will use the given proxy.
