# Creating the Hello Endpoints Complete Sample Backend API

  

1.  [Hello Endpoints Description](#hello_endpoints_description)
2.  [Creating the project](#creating_the_project)
3.  [Running and testing the API locally](#running_and_testing_the_api_locally)
4.  [Deploying to and running on App Engine](#deploying_to_and_running_on_app_engine)
5.  [Congratulations!](#congratulations)

In this part of the tutorial, we'll use the App Engine Maven artifact `hello-endpoints-archetype` to create a complete sample backend API.

## Hello Endpoints Description

The Hello Endpoints sample backend API supports:

-   A simple GET method that retrieves either of two canned responses based on user choice.
-   A simple GET method that retrieves all of the canned responses.
-   A POST method that provides a user-supplied greeting and multiplier to the backend API which then returns the greeting repeated the number of times specified by the multiplier.
-   An OAuth protected method requiring user sign-in, which returns the user's email address.

The UI provided by the included sample web client looks like this:

![Hello Endpoints UI](https://web.archive.org/web/20160424225606im_/https://cloud.google.com/appengine/docs/java/endpoints/images/webclient_ui.png)

## Creating the project

To create the `HelloEndpoints` backend API:

1.  In a terminal window, change to the directory where you want to build the project.

2.  Invoke the following Maven command:

    ```
    mvn archetype:generate -Dappengine-version=1.9.34 -Dfilter=com.google.appengine.archetypes:
    ```

3.  Enter the number `4` to select the `remote -> com.google.appengine.archetypes:hello-endpoints-archetype` from the list of App Engine archetypes.

4.  Select the most recent version from the displayed list of available archetype versions by accepting the default.

5.  When prompted to `Define value for property 'groupId'`, you can supply a namespace of your choosing, but if you want to keep this tutorial in sync with the source files on GitHub, use `com.example.helloendpoints`. (The typical convention is to use a namespace starting with the reversed form of your domain name.)

6.  When prompted to `Define value for property 'artifactId'`, supply the project name `helloendpoints`.

7.  Accept the defaults for the remaining prompts.

8.  Wait for the project to finish generating, then take a look at the resulting project layout:

    ![Maven Project Layout](https://web.archive.org/web/20160424225606im_/https://cloud.google.com/appengine/docs/java/endpoints/images/helloendpoints.png)

    The layout looks very similar to the the layout already described in [Creating a Simple Hello World Backend API](https://web.archive.org/web/20160424225606/https://cloud.google.com/appengine/docs/java/endpoints/getstarted/backend/hello_world). The main difference is that the backend API is already defined in the the Java source. `HelloGreetings.java` is the [JavaBean](https://web.archive.org/web/20160424225606/http://stackoverflow.com/questions/3295496/what-is-a-javabean-exactly) class used for the responses from the backend API and `Greetings.java` contains the backend API definition. We'll describe the code in `Greetings.java` later in the [code walkthrough](https://web.archive.org/web/20160424225606/https://cloud.google.com/appengine/docs/java/endpoints/getstarted/backend/code_walkthrough).

    Also, note the complete web client defined in `src/main/webapp/js/base.js` and `src/main/webapp/index.html`, along with the related CSS files. We won't go into this web client in this tutorial, but you can find a full description of the web client in the [web client tutorial](https://web.archive.org/web/20160424225606/https://cloud.google.com/appengine/docs/java/endpoints/getstarted/clients/js/).

9.  Edit the file `src/main/webapp/WEB-INF/appengine-web.xml` to set `<application>` to the project ID you obtained earlier during [Setup](https://web.archive.org/web/20160424225606/https://cloud.google.com/appengine/docs/java/endpoints/getstarted/backend/setup#creating_an_app_engine_app_for_deployment). (Or, look up the Client ID for web application in the [Google Cloud Platform Console](https://web.archive.org/web/20160424225606/https://console.cloud.google.com/).)

10. Edit the file `src/main/java/com/example/helloendpoints/Constants.java` as follows:

    1.  Set `WEB_CLIENT_ID` to the web client ID you obtained earlier during [Setup](https://web.archive.org/web/20160424225606/https://cloud.google.com/appengine/docs/java/endpoints/getstarted/backend/setup#creating_an_app_engine_app_for_deployment). (Or, look up the web client ID in the [Google Cloud Platform Console](https://web.archive.org/web/20160424225606/https://console.cloud.google.com/).)

    2.  Add the following line to the `Constants` class:

        ```
        public static final String API_EXPLORER_CLIENT_ID = com.google.api.server.spi.Constant.API_EXPLORER_CLIENT_ID;
        ```

        This line is needed for testing of OAuth protected methods in the APIs Explorer.

11. Edit the file `src/main/java/com/example/helloendpoints/Greetings.java` to add `Constants.API_EXPLORER_CLIENT_ID` to the list of `clientIDs` in the `@Api` annotation.

12. Edit the file `src/main/webapp/js/base.js` to set `google.devrel.samples.hello.CLIENT_ID` to the web client ID you obtained earlier during [Setup](https://web.archive.org/web/20160424225606/https://cloud.google.com/appengine/docs/java/endpoints/getstarted/backend/setup#creating_an_app_engine_app_for_deployment).

13. Build the project by invoking

    ```
    mvn clean install
    ```

    Wait for the project to build. When the project successfully finishes, you will see a message similar to this one:

    ```
    [INFO] BUILD SUCCESS
    [INFO] ------------------------------------------------------------------------
    [INFO] Total time: 17.169s
    [INFO] Finished at: Thu Jun 05 13:56:32 PDT 2014
    [INFO] Final Memory: 23M/234M
    ```

## Running and testing the API locally

To test the backend API locally:

1.  Start a new Chrome session as described in [How do I use Explorer with a local HTTP API](https://web.archive.org/web/20160424225606/https://developer.google.com/explorer-help/#hitting_local_api) and specify `8080` as the `localhost` port (`localhost:8080)`.

2.  From the main directory `helloendpoints/`, start the API in the development server as follows:

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

3.  Visit the backend API via the built-in APIs Explorer at this URL:

    ```
    http://localhost:8080/_ah/api/explorer
    ```

    Alternatively visit the web client at `http://localhost:8080`.

    **Note:** If you see a red warning banner warning about using HTTP, which is insecure, locate the shield icon (which, in Chrome, is located in the address bar), and click it, then click **Load unsafe scripts** to allow APIs Explorer to run. None of these HTTPS issues apply since you are running the APIs Explorer locally in a test environment.

4.  To send requests to the backend API, visit the APIs Explorer in your browser at this URL:

    ```
    http://localhost:8080/_ah/api/explorer
    ```

    This opens up the APIs Explorer for your backend. Scroll down if necessary to see the list of APIs displayed with **helloworld API** in that list.

5.  Click **helloworld API** to display the available methods. Notice how the name specified in your `@Api` annotation (`helloworld`) is prepended to the API class and method name.

6.  Click **helloworld.greetings.getGreetings** to display the Explorer for it:

    ![greetings.GetGreetings](https://web.archive.org/web/20160424225606im_/https://cloud.google.com/appengine/docs/java/endpoints/images/explorer.png)

7.  Our simple backend has two stored messages in an array. Get the first message by entering a value of `0` in the **Id** text box, then click **Execute without OAuth**.

    Notice the display of the request and response. In the Response, you'll see the OK result and the returned message, *hello world!*.

8.  Enter a value of `1` in the **Id** text box, then click **Authorize and execute**; this time the message is *goodbye world!*.

9.  Try out the other methods. The `listGreeting` method returns the stored list of greetings, `multiply` repeats a user-posted greeting the specified number of times, and `authed` is an OAuth protected method.

10. To try the "authed" method:

    1.  Click **helloworld.greetings.authed**.
    2.  Click **Authorize and execute**.
    3.  When prompted to supply additional scopes, use the default scope provided by clicking **Authorize and execute** again.
    4.  Observe the results.

## Deploying to and running on App Engine

After you finish testing, you can deploy to App Engine. To deploy to App Engine:

1.  Make sure you are logged into the Google account you used to create the Google Cloud Platform Console project.

2.  Open the [OAuth 2.0 client ID form](https://web.archive.org/web/20160424225606/https://console.cloud.google.com/project/_/apiui/credential/oauthclient) for your project in the Cloud Platform Console.

3.  If you see the **Configure consent screen** button at the bottom of the form, click it, supply a *Product name*, then click **Save**.

4.  Fill out the form:

    1.  Select **Web application** as the *Application Type*.

    2.  If you are [testing the backend locally](https://web.archive.org/web/20160424225606/https://cloud.google.com/appengine/docs/java/endpoints/test_deploy#running_and_testing_api_backends_locally), specify `http://localhost:8080` in the textbox labeled *Authorized JavaScript origins*. If you are deploying your backend API to production App Engine, specify the App Engine URL of your backend API in the textbox labeled *Authorized JavaScript origins*; for example, `https://your_project_id.appspot.com`, replacing `your_project_id` with your actual App Engine project ID.

        **Important:** You must specify the site or host name under *Authorized JavaScript origins* using `https` for this to work with Endpoints, unless you are testing with `localhost`, in which case you must use `http`.

    3.  Click **Create**.

    4.  You might want to keep this tab open so you can look up the client and project IDs later.

5.  From the main project directory, `helloendpoints/`, invoke the command

    ```
    mvn appengine:update
    ```

6.  Follow the prompts: when you are presented with a browser window containing a code, copy it to the terminal window.

7.  Wait for the upload to finish, then visit the URL you specified above for the Authorized Javascript Origins (`https://your_project_id.appspot.com`).

    If you don't see the web client app, or if the client doesn't behave as expected, [check for a successful deployment](https://web.archive.org/web/20160424225606/https://cloud.google.com/appengine/docs/java/endpoints/test_deploy#checking_for_a_successful_deployment).

## Congratulations!

You've just built and deployed a backend API and a web client to production App Engine!

We haven't looked at the backend API code yet. So let's walk through the code next to get an understanding of how a backend API is implemented.

<a href="https://web.archive.org/web/20160424225606/https://cloud.google.com/appengine/docs/java/endpoints/getstarted/backend/code_walkthrough" class="button">Hello Endpoints Code Walkthrough &gt;&gt;</a>
