# Adding Auth Support to the Client

  

1.  [Creating a web client ID](#creating_a_web_client_id)
2.  [Adding the web client ID to the backend API](#adding_the_web_client_id_to_the_backend_api)
3.  [Adding UI components to support auth](#adding_ui_components_to_support_auth)
4.  [Adding auth support in JavaScript](#adding_auth_support_in_javascript)

In this part of the tutorial, you'll add UI components and JavaScript to support a simple GET request to a backend API method that is protected by OAuth. To do this, we need to:

-   Register your client app with [Google Cloud Platform Console](https://web.archive.org/web/20160424225449/https://console.cloud.google.com/) so it works with your project.
-   Add the client ID to your backend API's whitelist
-   Add the UI sign-in component and the UI component to send the "auth-ed" request
-   Add Javascript to `base.js`: add the client ID, support sign-in and to support the UI in making an "auth-ed" GET, .

## Creating a web client ID

The web client ID must be added to the backend API's whitelist in order for your client to access a method protected by OAuth.

To create a web client ID, follow the instructions provided under [Creating an OAuth 2.0 web client ID](https://web.archive.org/web/20160424225449/https://cloud.google.com/appengine/docs/java/endpoints/auth#Creating_a_web_client_ID).

## Adding the web client ID to the backend API

In order for your web client to be recognized by the backend API, you must add the web client ID obtained above to the backend's list of client IDs. This is described in [Adding an OAuth 2.0-protected method](https://web.archive.org/web/20160424225449/https://cloud.google.com/appengine/docs/java/endpoints/getstarted/backend/auth#adding_an_oauth_20-protected_method).

## Adding UI components to support auth

1.  We'll need to add a sign-in button for user sign-in. In the `index.html` file you added previously in the tutorial, immediately under the `Hello Endpoints!` line, add the following lines:

    ```
    <a class="brand" href="#">Hello Endpoints!</a>
    <div class="nav-collapse collapse pull-right">
        <a href="javascript:void(0);" class="btn" id="signinButton">Sign in</a>
    </div>
    ```

2.  We'll need a button to send a request to the auth-protected backend API method. Immediately under the **Multiply Greetings** form, add the following lines:

    ```
    <form onsubmit="return false;">
        <h2>Authenticated Greeting</h2>
        <div><input id="authedGreeting" type="submit" class="btn btn-small" disabled value="Submit"></div>
    </form>
    ```

3.  When you are finished, your `index.html` file should look like this:

    ```
    <!DOCTYPE html>
    <html>
    <head>
        <title>Hello Endpoints!</title>
        <script async src="/js/base.js"></script>
        <link rel="stylesheet" href="/bootstrap/css/bootstrap.css">
        <link rel="stylesheet" href="/bootstrap/css/bootstrap-responsive.css">
        <style>
            body {
                padding-top: 40px;
                padding-bottom: 40px;
                background-color: #f5f5f5;
            }
            blockquote {
                margin-bottom: 10px;
                border-left-color: #bbb;
            }
            form {
                margin-top: 10px;
            }
            form label {
              width: 90px;
              display: inline-block;
            }
            .form-signin input[type="text"] {
                font-size: 16px;
                height: auto;
                margin-bottom: 15px;
                padding: 7px 9px;
            }
            .row {
                margin-left: 0px;
                margin-top: 10px;
                overflow: scroll;
            }
        </style>
    </head>
    <body>
    <div class="navbar navbar-inverse navbar-fixed-top">
        <div class="navbar-inner">
            <div class="container">
                <button type="button" class="btn btn-navbar" data-toggle="collapse" data-target=".nav-collapse">
                    <span class="icon-bar"></span>
                    <span class="icon-bar"></span>
                    <span class="icon-bar"></span>
                </button>
                <a class="brand" href="#">Hello Endpoints!</a>
                <div class="nav-collapse collapse pull-right">
                    <a href="javascript:void(0);" class="btn" id="signinButton">Sign in</a>
                </div>
            </div>
        </div>
    </div>
    <div class="container">

        <output id="outputLog"></output>

        <form onsubmit="return false;">
            <h2>Get Greeting</h2>
            <div><label for="id">Greeting ID:</label><input id="id" /></div>
            <div><input id="getGreeting" type="submit" class="btn btn-small" value="Submit"></div>
        </form>

        <form onsubmit="return false;">
            <h2>List Greetings</h2>
            <div><input id="listGreeting" type="submit" class="btn btn-small" value="Submit"></div>
        </form>

        <form onsubmit="return false;">
            <h2>Multiply Greetings</h2>
            <div><label for="greeting">Greeting:</label><input id="greeting" /></div>
            <div><label for="count">Count:</label><input id="count" /></div>
            <div><input id="multiplyGreetings" type="submit" class="btn btn-small" value="Submit"></div>
        </form>
        <form onsubmit="return false;">
            <h2>Authenticated Greeting</h2>
            <div><input id="authedGreeting" type="submit" class="btn btn-small" disabled value="Submit"></div>
        </form>
        <script>
            function init() {
                google.appengine.samples.hello.init('//' + window.location.host + '/_ah/api');
            }
        </script>
        <script src="https://apis.google.com/js/client.js?onload=init"></script>
    </div>
    </body>
    </html>
    ```

## Adding auth support in JavaScript

1.  In the `base.js` file you added previously in the tutorial, under the lines that set the namespace, add the following lines, replacing the value `your-client-ID` with the value obtained above when you [registered](#registering_a_web_client) your client app:

    ```
    /**
     * Client ID of the application (from the APIs Console).
     * @type {string}
     */
    google.appengine.samples.hello.CLIENT_ID =
        'your-client-ID';

    /**
     * Scopes used by the application.
     * @type {string}
     */
    google.appengine.samples.hello.SCOPES =
        'https://www.googleapis.com/auth/userinfo.email';
    ```

    The email scopes value as shown is needed by OAuth.

2.  In `base.js`, add the sign-in button and auth'ed request button to the list of buttons to be enabled:

    ```
    /**
     * Enables the button callbacks in the UI.
     */
    google.appengine.samples.hello.enableButtons = function() {
      var getGreeting = document.querySelector('#getGreeting');
      getGreeting.addEventListener('click', function(e) {
        google.appengine.samples.hello.getGreeting(
            document.querySelector('#id').value);
      });

      var listGreeting = document.querySelector('#listGreeting');
      listGreeting.addEventListener('click',
          google.appengine.samples.hello.listGreeting);

      var multiplyGreetings = document.querySelector('#multiplyGreetings');
      multiplyGreetings.addEventListener('click', function(e) {
        google.appengine.samples.hello.multiplyGreeting(
            document.querySelector('#greeting').value,
            document.querySelector('#count').value);
      });

      var authedGreeting = document.querySelector('#authedGreeting');
      authedGreeting.addEventListener('click',
          google.appengine.samples.hello.authedGreeting);

      var signinButton = document.querySelector('#signinButton');
      signinButton.addEventListener('click', google.appengine.samples.hello.auth);
    };
    ```

3.  In `base.js`, modify the `init` function to load the OAuth library, and add user sign-in to the callback function to be executed after libraries are successfully loaded:

    ```
    /**
     * Initializes the application.
     * @param {string} apiRoot Root of the API's path.
     */
    google.appengine.samples.hello.init = function(apiRoot) {
      // Loads the OAuth and helloworld APIs asynchronously, and triggers login
      // when they have completed.
      var apisToLoad;
      var callback = function() {
        if (--apisToLoad == 0) {
          google.appengine.samples.hello.enableButtons();
          google.appengine.samples.hello.signin(true,
              google.appengine.samples.hello.userAuthed);
        }
      }

      apisToLoad = 2; // must match number of calls to gapi.client.load()
      gapi.client.load('helloworld', 'v1', callback, apiRoot);
      gapi.client.load('oauth2', 'v2', callback);
    };
    ```

    Notice that all of the libraries you want to load are loaded before your callback is invoked. Notice also how the `apisToLoad` value is incremented to reflect the count of libraries to load.

4.  The `signin` function shown above, which invokes the OAuth library's `authorize` function, is executed upon page load. We'll define this next by adding the following lines above the `getGreeting` function:

    ```
    /**
     * Handles the auth flow, with the given value for immediate mode.
     * @param {boolean} mode Whether or not to use immediate mode.
     * @param {Function} callback Callback to call on completion.
     */
    google.appengine.samples.hello.signin = function(mode, callback) {
      gapi.auth.authorize({client_id: google.appengine.samples.hello.CLIENT_ID,
          scope: google.appengine.samples.hello.SCOPES, immediate: mode},
          callback);
    };
    ```

5.  Add the logic for the authorization form that is invoked when the user clicks the sign-in button:

    ```
    /**
     * Presents the user with the authorization popup.
     */
    google.appengine.samples.hello.auth = function() {
      if (!google.appengine.samples.hello.signedIn) {
        google.appengine.samples.hello.signin(false,
            google.appengine.samples.hello.userAuthed);
      } else {
        google.appengine.samples.hello.signedIn = false;
        document.querySelector('#signinButton').textContent = 'Sign in';
        document.querySelector('#authedGreeting').disabled = true;
      }
    };
    ```

    The `signedIn` boolean is initially set to false because our client logic happens to need it.

6.  Add the logic that loads the auth-related application UI. The `signedIn`boolean is initially set to false for our client logic.This function is called during page load:

    ```
    /**
     * Whether or not the user is signed in.
     * @type {boolean}
     */
    google.appengine.samples.hello.signedIn = false;

    /**
     * Loads the application UI after the user has completed auth.
     */
    google.appengine.samples.hello.userAuthed = function() {
      var request = gapi.client.oauth2.userinfo.get().execute(function(resp) {
        if (!resp.code) {
          google.appengine.samples.hello.signedIn = true;
          document.querySelector('#signinButton').textContent = 'Sign out';
          document.querySelector('#authedGreeting').disabled = false;
        }
      });
    };
    ```

7.  Finally, add the function that is invoked when the user clicks on the Authenticated Greeting button to send a request to the backend API:

    ```
    /**
     * Greets the current user via the API.
     */
    google.appengine.samples.hello.authedGreeting = function(id) {
      gapi.client.helloworld.greetings.authed().execute(
          function(resp) {
            google.appengine.samples.hello.print(resp);
          });
    };
    ```

8.  When you are finished, your `base.js` file should look like this:

    ```
    /**
     * @fileoverview
     * Provides methods for the Hello Endpoints sample UI and interaction with the
     * Hello Endpoints API.
     */

    /** google global namespace for Google projects. */
    var google = google || {};

    /** appengine namespace for Google Developer Relations projects. */
    google.appengine = google.appengine || {};

    /** samples namespace for App Engine sample code. */
    google.appengine.samples = google.appengine.samples || {};

    /** hello namespace for this sample. */
    google.appengine.samples.hello = google.appengine.samples.hello || {};
    /**
     * Client ID of the application (from the APIs Console).
     * @type {string}
     */
    google.appengine.samples.hello.CLIENT_ID =
        'your-client-ID';

    /**
     * Scopes used by the application.
     * @type {string}
     */
    google.appengine.samples.hello.SCOPES =
        'https://www.googleapis.com/auth/userinfo.email';
    /**
     * Whether or not the user is signed in.
     * @type {boolean}
     */
    google.appengine.samples.hello.signedIn = false;

    /**
     * Loads the application UI after the user has completed auth.
     */
    google.appengine.samples.hello.userAuthed = function() {
      var request = gapi.client.oauth2.userinfo.get().execute(function(resp) {
        if (!resp.code) {
          google.appengine.samples.hello.signedIn = true;
          document.querySelector('#signinButton').textContent = 'Sign out';
          document.querySelector('#authedGreeting').disabled = false;
        }
      });
    };
    /**
     * Handles the auth flow, with the given value for immediate mode.
     * @param {boolean} mode Whether or not to use immediate mode.
     * @param {Function} callback Callback to call on completion.
     */
    google.appengine.samples.hello.signin = function(mode, callback) {
      gapi.auth.authorize({client_id: google.appengine.samples.hello.CLIENT_ID,
          scope: google.appengine.samples.hello.SCOPES, immediate: mode},
          callback);
    };
    /**
     * Presents the user with the authorization popup.
     */
    google.appengine.samples.hello.auth = function() {
      if (!google.appengine.samples.hello.signedIn) {
        google.appengine.samples.hello.signin(false,
            google.appengine.samples.hello.userAuthed);
      } else {
        google.appengine.samples.hello.signedIn = false;
        document.querySelector('#signinButton').textContent = 'Sign in';
        document.querySelector('#authedGreeting').disabled = true;
      }
    };
    /**
     * Prints a greeting to the greeting log.
     * param {Object} greeting Greeting to print.
     */
    google.appengine.samples.hello.print = function(greeting) {
      var element = document.createElement('div');
      element.classList.add('row');
      element.innerHTML = greeting.message;
      document.querySelector('#outputLog').appendChild(element);
    };

    /**
     * Gets a numbered greeting via the API.
     * @param {string} id ID of the greeting.
     */
    google.appengine.samples.hello.getGreeting = function(id) {
      gapi.client.helloworld.greetings.getGreeting({'id': id}).execute(
          function(resp) {
            if (!resp.code) {
              google.appengine.samples.hello.print(resp);
            }
          });
    };

    /**
     * Lists greetings via the API.
     */
    google.appengine.samples.hello.listGreeting = function() {
      gapi.client.helloworld.greetings.listGreeting().execute(
          function(resp) {
            if (!resp.code) {
              resp.items = resp.items || [];
              for (var i = 0; i < resp.items.length; i++) {
                google.appengine.samples.hello.print(resp.items[i]);
              }
            }
          });
    };

    /**
     * Gets a greeting a specified number of times.
     * @param {string} greeting Greeting to repeat.
     * @param {string} count Number of times to repeat it.
     */
    google.appengine.samples.hello.multiplyGreeting = function(
        greeting, times) {
      gapi.client.helloworld.greetings.multiply({
          'message': greeting,
          'times': times
        }).execute(function(resp) {
          if (!resp.code) {
            google.appengine.samples.hello.print(resp);
          }
        });
    };
    /**
     * Greets the current user via the API.
     */
    google.appengine.samples.hello.authedGreeting = function(id) {
      gapi.client.helloworld.greetings.authed().execute(
          function(resp) {
            google.appengine.samples.hello.print(resp);
          });
    };
    /**
     * Enables the button callbacks in the UI.
     */
    google.appengine.samples.hello.enableButtons = function() {
      var getGreeting = document.querySelector('#getGreeting');
      getGreeting.addEventListener('click', function(e) {
        google.appengine.samples.hello.getGreeting(
            document.querySelector('#id').value);
      });

      var listGreeting = document.querySelector('#listGreeting');
      listGreeting.addEventListener('click',
          google.appengine.samples.hello.listGreeting);

      var multiplyGreetings = document.querySelector('#multiplyGreetings');
      multiplyGreetings.addEventListener('click', function(e) {
        google.appengine.samples.hello.multiplyGreeting(
            document.querySelector('#greeting').value,
            document.querySelector('#count').value);
      });

      var authedGreeting = document.querySelector('#authedGreeting');
      authedGreeting.addEventListener('click',
          google.appengine.samples.hello.authedGreeting);

      var signinButton = document.querySelector('#signinButton');
      signinButton.addEventListener('click', google.appengine.samples.hello.auth);
    };
    /**
     * Initializes the application.
     * @param {string} apiRoot Root of the API's path.
     */
    google.appengine.samples.hello.init = function(apiRoot) {
      // Loads the OAuth and helloworld APIs asynchronously, and triggers login
      // when they have completed.
      var apisToLoad;
      var callback = function() {
        if (--apisToLoad == 0) {
          google.appengine.samples.hello.enableButtons();
          google.appengine.samples.hello.signin(true,
              google.appengine.samples.hello.userAuthed);
        }
      }

      apisToLoad = 2; // must match number of calls to gapi.client.load()
      gapi.client.load('helloworld', 'v1', callback, apiRoot);
      gapi.client.load('oauth2', 'v2', callback);
    };
    ```

9.  Rebuild your project with the new client:

    ```
    mvn clean install
    ```

10. Upload and deploy the project and client code:

    ```
    mvn appengine:update
    ```

11. Visit the App Engine URL: `https://<your-app-id>.appspot.com/` to display the UI:

    ![Hello-Endpoints](https://web.archive.org/web/20160424225449im_/https://cloud.google.com/appengine/docs/java/endpoints/getstarted/clients/js/images/hello3.png)

12. Click **Sign In**.

13. Under **Authenticated Greeting**, click **Submit** to display your login name.

And that's it! You've successfully accessed a backend API protected by OAuth.

This completes the Getting Started for web clients.
