# Uploading Your Application

  

If you use Datastore, you'll need to generate indexes before uploading.

1.  [Generating indexes](#generating_indexes)
2.  [Uploading (deploying) your app](#uploading_deploying_your_app)
3.  [Troubleshooting deployment failures](#troubleshooting_deployment_failures)
4.  [What's next](#whats_next)

## Generating indexes

To generate indexes so you can deploy your app:

1.  Generate any required indexes before uploading your app, as described in [Testing your app on the development server](https://web.archive.org/web/20160424230158/https://cloud.google.com/appengine/docs/java/gettingstarted/usingdatastore#testing_your_app_on_the_development_server) and [Creating required indexes](https://web.archive.org/web/20160424230158/https://cloud.google.com/appengine/docs/java/gettingstarted/usingdatastore#creating_required_indexes).

2.  Open the file `guestbook/target/guestbook-1.0-SNAPSHOT/WEB-INF/appengine-generated/datastore-indexes-auto.xml` in an editor and make sure that it contains the following:

    ```
    <datastore-indexes>
      <datastore-index kind="Greeting" ancestor="true" source="manual">
          <property name="date" direction="desc"/>
      </datastore-index>
    </datastore-indexes>
    ```

    Typically, you have to change `source="auto"` to `source="manual"`.

    **Note:** If needed indexes are not uploaded, your app may fail at runtime. However, in this case, you can address this by examining the error message: the needed indexes are included in the error message. Copy those index entries from the error message into `datastore-indexes-auto.xml` and run the update command again.

## Uploading (deploying) your app

To deploy your app:

1.  Change directory to `guestbook`, and invoke Maven as follows:

    ```
    mvn appengine:update
    ```

    The first time you upload, you'll be prompted to supply login information and you will be led through the login process via the browser. Follow the prompts. You may need to copy the success code from the browser into the command line at the end of the login process. (This information is remembered after you do it once.)

2.  Wait for the upload to complete. If the upload succeeds, you'll see a message similar to this one:

    ```
    ...
    98% Uploading index definitions.

    Update for module default completed successfully.
    Success.
    Cleaning up temporary files for module default...
    ```

3.  In your browser, visit the url `https://<project-ID>.appspot.com` to run the application hosted on App Engine, replacing `<project-ID>` with your project ID.

    **Note:** You may not be able to access your app in your browser immediately after a successful deployment. In some cases, it might take a few minutes before your app begins serving, even after a successful deployment.

## Troubleshooting deployment failures

Your update attempt may fail with a message similar to this one: `404 Not Found This application does not exist (app_id=u'your-app-ID').` This error will occur if you have multiple Google accounts and are using the wrong account to perform the update.

To solve this issue, change directories to ~, locate a file named `.appcfg_oauth2_tokens_java`, and rename it. Then try updating again; you will be prompted to authorize the upload again.

------------------------------------------------------------------------

That's it! You have completed this tutorial.

## What's next

You might want to check out the following features:

-   Authenticating users with [Google Accounts](https://web.archive.org/web/20160424230158/https://cloud.google.com/appengine/docs/java/users/) or [OAuth](https://web.archive.org/web/20160424230158/https://cloud.google.com/appengine/docs/java/oauth/)
-   [Reading and writing logs](https://web.archive.org/web/20160424230158/https://cloud.google.com/appengine/docs/java/logs), which shows how to write application logs and how to interpret the system logs
-   Using [Task queues](https://web.archive.org/web/20160424230158/https://cloud.google.com/appengine/docs/java/taskqueue/), which shows how to use task queues to do background work to be run after the request

For a deeper dive into how App Engine works, see the documentation on the [Java Runtime Environment](https://web.archive.org/web/20160424230158/https://cloud.google.com/appengine/docs/java/runtime), and [request handling](https://web.archive.org/web/20160424230158/https://cloud.google.com/appengine/docs/java/requests) and [routing](https://web.archive.org/web/20160424230158/https://cloud.google.com/appengine/docs/java/modules/routing).
