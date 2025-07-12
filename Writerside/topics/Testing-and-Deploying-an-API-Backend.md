# Testing and Deploying an API Backend

  

1.  [Running and testing API backends locally](#running_and_testing_api_backends_locally)
2.  [Testing clients against the backend](#testing_clients_against_the_backend)
3.  [Deploying an API backend](#deploying_an_api_backend)
4.  [Checking for a Successful Deployment](#checking_for_a_successful_deployment)
5.  [Troubleshooting a deployment failure](#troubleshooting_a_deployment_failure)
6.  [Deploying to multiple app versions](#deploying_to_multiple_app_versions)
    1.  [Accessing backend API versions deployed to an application default version](#accessing_backend_api_versions_deployed_to_an_application_default_version)
    2.  [Accessing backend API versions deployed to non-default application versions](#accessing_backend_api_versions_deployed_to_non-default_application_versions)
    3.  [Accessing backend API versions deployed to non-default modules](#accessing_backend_api_versions_deployed_to_non-default_modules)
    4.  [Managing your backend API versions](#managing_your_backend_api_versions)

Testing an API backend follows the typical App Engine pattern of using the development server to run the web app locally during development and incremental testing, followed by deployment to production when you are ready for real-world testing.

## Running and testing API backends locally

To test the backend, you'll use the API Explorer, which is a tool that lets you test APIs interactively without using a client app. This tool is automatically run for you when you navigate to your backend API's subdirectory `/_ah/api/explorer` as we'll describe in the instructions below.

To test the API backend locally on the development server:

1.  Start a new Chrome session as described in [How do I use Explorer with a local HTTP API](https://web.archive.org/web/20160424230904/https://developer.google.com/explorer-help/#hitting_local_api) and specify `8080` as the `localhost` port (`localhost:8080)`. You can use a different port but it must match whatever value you use in the next steps.

2.  The default port for testing is 8080. If you wish, you can test using a port other than the default 8080 and the default address. (For example, if you want the app to listen to the local network, you may need to change the address to `0.0.0.0`.) To change the port and/or address, see [Specifying a port for local testing](https://web.archive.org/web/20160424230904/https://cloud.google.com/appengine/docs/java/tools/maven#specifying_a_port_for_local_testing).

3.  Start the API backend by invoking the development app server. Using Maven, invoke the command from the project's `<app>-ear` subdirectory (not from the `<app>-war`):

    ```
    mvn appengine:devserver
    ```

    Wait til the backend loads; if it loads successfully, this message is displayed:

    ```
    INFO: Dev App Server is now running.
    ```

4.  In your browser, visit the URL

    ```
    http://localhost:8080/_ah/api/explorer
    ```

5.  Double-click the API you created to display the list of its methods.

6.  Click the method you wish to test, fill in any request fields as needed, then click **Execute** to test the method: the request will be displayed along with the response.

If you want to use `curl` instead of the API Explorer, you can alternatively test the API backend by sending an test request using [curl](https://web.archive.org/web/20160424230904/http://curl.haxx.se/download.html). The following request makes a call to a sample Board API backend to send a request to our sample tictactoe game:

```
    curl --header "Content-Type: application/json" -X POST -d '{"state": "----X----"}' http://localhost:8888/_ah/api/tictactoe/v1/board
```

## Testing clients against the backend

You can test your client against a backend API running in production App Engine at any time without making any changes. However, if you want to test your client against a backend API running on the local development server, you'll need to make changes as described below:

-   For more information on testing JavaScript clients locally, see [Testing a JavaScript client against a local development server](https://web.archive.org/web/20160424230904/https://cloud.google.com/appengine/docs/java/endpoints/consume_js#testing_a_javascript_client_against_a_local_development_server).
-   For more information on testing Android clients locally, see [Testing an Android client against a local development server](https://web.archive.org/web/20160424230904/https://cloud.google.com/appengine/docs/java/endpoints/consume_android#testing_an_android_client_against_a_local_development_server).

## Deploying an API backend

When you are satisfied that your API backend works as expected, you can deploy it to App Engine. Deploying an API backend is no different from deploying a regular App Engine app.

To deploy:

If you use [Maven](https://web.archive.org/web/20160424230904/https://cloud.google.com/appengine/docs/java/tools/maven) for your project, you deploy the app as follows:

1.  Change directory to your project main directory containing the `pom.xml` file.

2.  Invoke Maven as follows:

    `mvn appengine:update`

    The first time you deploy to production App Engine, you may be prompted to supply login information; supply it as directed.

    If the deployment fails with the error `403 Forbidden You do not have permission to modify this app (app_id=u'your-app-id')`, check your source file `your-app/src/main/webapp/WEB-INF/appengine-web.xml` and make sure the value for the `<application>` element is set to a valid [application ID](https://web.archive.org/web/20160424230904/https://cloud.google.com/appengine/docs/java/endpoints/getstarted/backend/setup#creating_an_app_engine_app_for_deployment).

If you don't use Maven, follow the instructions for deploying provided in [Uploading and Managing a Java App](https://web.archive.org/web/20160424230904/https://cloud.google.com/appengine/docs/java/tools/uploadinganapp).

After you deploy, you can use the API Explorer to experiment with your API without using a client app by visiting

```
    https://your_app_id.appspot.com/_ah/api/explorer
```
**Important:** It may take several moments for the API to load. If the API **loading** message does not go away, and your API does not appear, clear your cookies and refresh the API Explorer. If problems persist, follow the procedures provided under [Troubleshooting a deployment failure](#troubleshooting_a_deployment_failure).

Alternatively, use the discovery service to view your API by visiting

```
    https://your_app_id.appspot.com/_ah/api/discovery/v1/apis
```

## Checking for a Successful Deployment

After you upload the API, you will see a success message if the files upload correctly. However, *after* the files are uploaded, App Engine does more work unpacking the API and deploying it. If there is a failure during this process, you are not informed about it. You must check the logs to determine success.

To check the logs after you deploy your API,

1.  Open the [logs viewer](https://web.archive.org/web/20160424230904/https://console.cloud.google.com/project/_/logs) for your project.

2.  Look at the logs nearest in time to the time you deployed. A successful API deployment results in a log that looks something like this:

    ```
    013-09-30 18:39:09.781 Endpoints: https://2-dot-test-pont.appspot.com/_ah/api/helloWorld@v2 Saved
    ```

    or

    ```
    2013-09-30 16:07:17.987 Endpoints: https://7-dot-test2.appspot.com/_ah/api/guestbook@v1 OK
    ```

    An unsuccessful deployment has this kind of log:

    ```
     https://7-dot-test2.appspot.com/_ah/api/Guestbook@v2 ERROR
    ```

If the logs indicate a deployment failure, refer the the following Troubleshooting section to see how to handle the failure.

## Troubleshooting a deployment failure

**Note:** In the following table on backend configuration and deployment issues, we refer to the path `/_ah/spi`, which you might think is a typo, because applications send their requests to `/_ah/api`. However, the Endpoints backend actually handles these requests at the path `/_ah/spi`.

Here are some common deployment error messages you might find in your application log, along with suggested remedies:

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Error</th>
<th>Remedy</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Endpoints configuration not updated. Check the app's AppEngine logs for errors.</td>
<td>There is a some type of exception being thrown by your app that is preventing your app from starting.<br />
<br />
Check your app's logs to narrow down the exception being thrown. Check for errors logged from the <code>/_ah/spi/BackendService.getApiConfigs</code> handler, which the Google Cloud Endpoints server uses to retrieve your API configuration. Here is what such a log might look like:<br />
<br />
<code>2014-09-04 13:03:46.531 </code><strong><code>/_ah/spi/BackendService.getApiConfigs</code></strong><code> 200 4 ms 28kb module=default version=1</code><br />
<code>10.1.0.41 - - [04 Sep/2014:13:03:46 -0700] "POST" /_ah/spi/BackendService.getApiConfigs HTTP</code><br />
<code>app_engine_release=1.9.10 apps</code><br />
. . .</td>
</tr>
<tr class="even">
<td>Endpoints configuration not updated. There are authentication requirements on the <code>/_ah/spi</code> handler that are preventing the Google Cloud Endpoints server from accessing the app.</td>
<td><strong>Remove</strong> the authentication requirements on that handler from your app's <code>web.xml</code> file (see <a href="https://web.archive.org/web/20160424230904/https://cloud.google.com/appengine/docs/java/config/webxml#Security_and_Authentication">Security and Authentication</a> for the lines to remove). Note: this error is <em>not</em> caused by setting the file to require HTTPS, as described in <a href="https://web.archive.org/web/20160424230904/https://cloud.google.com/appengine/docs/java/config/webxml#Secure_URLs">Secure URLs</a>, because this does work with Endpoints.<br />
<br />
If you want to require authentication for your Endpoints API, follow the process described in <a href="https://web.archive.org/web/20160424230904/https://cloud.google.com/appengine/docs/java/endpoints/auth">Using Auth with Endpoints</a>.</td>
</tr>
<tr class="odd">
<td></td>
<td></td>
</tr>
</tbody>
</table>

## Deploying to multiple app versions

When you deploy your backend API, you deploy it to the Cloud project ID you created for your API. This ID is the same one used by App Engine for your backend API. When you deploy, in addition to the App Engine/Cloud Project ID, you must also specify the App Engine version you deploy to. You specify the App Engine/Cloud Project ID in the `<application>` field in the `/WEB-INF/appengine-web.xml`file; you specify the application version in the `<version>` field. Notice that the App Engine app version is *not* the same thing as the backend API version number, which you specify in the `version` attribute of the `@Api` annotation.

![apiversions](https://web.archive.org/web/20160424230904im_/https://cloud.google.com/appengine/docs/java/endpoints/images/apiversions.png)

As shown in the figure above, you can deploy multiple API versions to the same App Engine App version. And, you can have many App Engine App versions.

### Accessing backend API versions deployed to an application default version

The first App Engine version that you deploy your backend API to is the default version. This version runs at the URL `http://your_app_id.appspot.com`. All of the backend API versions deployed to that App Engine app version can be accessed using that URL.

The default version remains the default version until you explicitly change it. To change the default version:

1.  Open the [versions page](https://web.archive.org/web/20160424230904/https://console.cloud.google.com/project/_/appengine/versions) for your project.
2.  Select the version you want as the default version and click **Make Default**.

### Accessing backend API versions deployed to non-default application versions

If you need to access API versions that are *not* deployed to the default App Engine app version, you use a *version-specific* URL.

To access backend API versions that are deployed to non-default App Engine versions of your app, you must include the version specifier in the URL, like this: `https://version-dot-your_app_id.appspot.com`. For example, suppose your default app version is 1, but you want to access a backend API version deployed to App Engine app version 2; you would use this URL: `https://2-dot-your_app_id.appspot.com`.

**Important:** For backend APIs, only host names formed as shown above using `-dot-` in place of literal periods will work for versioned apps.

### Accessing backend API versions deployed to non-default modules

If you need to access API versions that are *not* deployed to the default App Engine module, you would use a *version-specific* URL using the dot syntax as follows:

```
`https://<module-name>-dot-<app-name>.appspot.com/_ah/spi/...`
```

### Managing your backend API versions

In managing your API versions, consider these recommendations:

-   When you want to introduce an incremental, but non-breaking change, keep the API version constant and deploy over the existing API.
-   When you introduce a breaking change to your API, increment the API version.
-   For additional protection, increment the App Engine app version as well and then deploy the new API version to that new App Engine app version. This lets you use the built-in flexibility of App Engine to quickly switch between App Engine versions and serve from the old working versions if you run into unexpected problems.

The following table is an illustration of cascading backend API versions to different App Engine app versions:

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>App Version</th>
<th>backend API version</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>1</td>
<td><ul>
<li>v1 --&gt; v1 (2) --&gt; v1 (n)</li>
</ul></td>
</tr>
<tr class="even">
<td>2</td>
<td><ul>
<li>v1 (n)</li>
<li>v2 --&gt; v2 (2) --&gt; v2 (n)</li>
</ul></td>
</tr>
<tr class="odd">
<td>3</td>
<td><ul>
<li>v1 (n)</li>
<li>v2 (n)</li>
<li>v3 --&gt; v3 (2) --&gt; v3 (n)</li>
</ul></td>
</tr>
</tbody>
</table>

As shown in the table, incremental, non-breaking updates to v1 of the API are rolled out, each overwriting the previous version. When breaking changes are introduced, the API version is incremented to v2 and it is deployed to a new App Engine app version. That allows you to switch back to the previous app version if needed.

In the table, notice that app version 2 supports both the latest v1 version and the new v2 version of your API. If you don't delete the existing v1 code from your project, deploying the project will result in both `v2` and `vl` (n) of your API being deployed to App version 2.
