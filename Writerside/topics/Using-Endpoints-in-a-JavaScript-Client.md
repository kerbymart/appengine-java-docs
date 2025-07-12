# Using Endpoints in a JavaScript Client

  

1.  [Enabling a JavaScript client for Endpoints](#enabling_a_javascript_client_for_endpoints)
2.  [Calling an Endpoint API](#calling_an_endpoint_api)
3.  [Adding authentication support with OAuth 2.0](#adding_authentication_support_with_oauth_20)
    1.  [Sample auth flow for a JavaScript client](#sample_auth_flow_for_a_javascript_client)
    2.  [Signing out](#signing_out)
4.  [Testing a JavaScript client against a local development server](#testing_a_javascript_client_against_a_local_development_server)
5.  [Sample client source code](#sample_client_source_code)

To use a Google Cloud Endpoint, you'll need to use the [Google JavaScript client library](https://web.archive.org/web/20160424225905/https://developers.google.com/api-client-library/javascript/reference/referencedocs). The code you need to add to use Endpoints is minimal, as shown in the following sections.

## Enabling a JavaScript client for Endpoints

To enable a JavaScript client to use an Endpoint:

1.  Add the script that includes the Google-hosted JavaScript client library to your HTML:

    ```
    <script src="https://apis.google.com/js/client.js?onload=init">
    </script>
    ```

2.  Notice that the JavaScript line above includes an `onload` property specifying the function to be used as a callback after the library loads. Replace "init" with the name of your page initialization function if you did not name it `init`.

3.  Inside your callback function, load your Endpoint:

    ```
    var ROOT = 'https://your_app_id.appspot.com/_ah/api';
    gapi.client.load('your_api_name', 'v1', function() {
      doSomethingAfterLoading();
    }, ROOT);
    ```

    This example loads `your_api_name v1` in the JavaScript client library and calls `doSomethingAfterLoading` once the API is loaded.

4.  Replace `ROOT` in the above code with the name of your application to load your own APIs. Replace `your_api_name` and `v1` with the name and version of your Endpoint.

## Calling an Endpoint API

Once the JavaScript client is enabled for an Endpoint, you can make calls to that Endpoint using the Google JavaScript client object `gapi.client`. (For more information, see [Getting Started](https://web.archive.org/web/20160424225905/https://developers.google.com/api-client-library/javascript/start/start-js) for the google-api-javascript-client.)

The following two calls show an insert and a list request made to the Tic Tac Toe API:

```
// Insert a score
gapi.client.tictactoe.scores.insert({'outcome':
    'WON'}).execute(function(resp) {
  console.log(resp);
});

// Get the list of previous scores
gapi.client.tictactoe.scores.list().execute(function(resp) {
  console.log(resp);
});
```

During development you may want to use the Chrome developer tools JavaScript console, which automatically suggests available method names when you are manipulating `gapi.client`. With the above example Endpoint, typing `gapi.client.tictactoe.` into that console results in the Chrome developer tools suggesting `scores` as a possible completion.

## Adding authentication support with OAuth 2.0

Built-in support for OAuth is provided by the JavaScript client library for use with Google APIs and Endpoints.

Authorization is provided by the built-in `authorize` method. When you want to require a user sign-in, you call the following:

```
gapi.auth.authorize({client_id: CLIENT_ID, scope: SCOPES,
    immediate: mode}, callback);
```

where

-   `CLIENT_ID` is the client ID of your application, specified in the `clientIds` attribute of the `@Api` (or `@ApiMethod`) annotation. For more information on getting client IDs, see [Adding Authentication to Endpoints](https://web.archive.org/web/20160424225905/https://cloud.google.com/appengine/docs/java/endpoints/auth).
-   `SCOPES` are the list of OAuth 2.0 scopes that must be granted to tokens sent to your backend. At minimum, you should request access to the userinfo.email scope: `https://www.googleapis.com/auth/userinfo.email` which grants access to the user’s email address.
-   `mode` is either `false` or `true`, depending on whether or not you’d like to present the explicit authorization window to the user. If the user has never authorized your application before, calling authorize with `mode: true` (no authorization window) will fail. The correct situation to use each modes is described below.
-   `callback` is the callback function to call once `authorize` completes.

### Sample auth flow for a JavaScript client

Here’s an example flow for authorizing a user in a page using the JS client:

```
function init() {
  var apisToLoad;
  var loadCallback = function() {
    if (--apisToLoad == 0) {
      signin(true, userAuthed);
    }
  };

  apisToLoad = 2; // must match number of calls to gapi.client.load()
  apiRoot = '//' + window.location.host + '/_ah/api';
  gapi.client.load('tictactoe', 'v1', loadCallback, apiRoot);
  gapi.client.load('oauth2', 'v2', loadCallback);
}

function signin(mode, authorizeCallback) {
  gapi.auth.authorize({client_id: CLIENT_ID,
    scope: SCOPES, immediate: mode},
    authorizeCallback);
}

function userAuthed() {
  var request =
      gapi.client.oauth2.userinfo.get().execute(function(resp) {
    if (!resp.code) {
      // User is signed in, call my Endpoint
    }
  });
}
```

Note that the code sample assumes `init()` to be the callback function you provided when loading the JS client library. Inside `init()`, your API and the OAuth 2.0 API are loaded. After the OAuth 2.0 API is loaded, the sample attempts to authorize the user using immediate mode (no authorization prompt). If immediate mode succeeds, the sample sets the token for future requests to the ID token required by your Endpoint's backend.

If your user has previously authorized your application, the user is signed in, and your application can proceed normally. Authorization headers are automatically supplied with any requests.

If the user has not authorized your application previously (or there is no signed in user in the browser session), you’ll need to call authorize again, without using immediate mode. Because this method triggers a popup, it must be called as a result of a user action in the browser, such as clicking a link or button:

```
<a href="#" onclick="auth();" id="signinButton">Sign in!</a>

function auth() {
  signin(false, userAuthed);
};
```

### Signing out

There is no concept of signing out using the JS client library, but you can emulate the behavior if you wish by any of these methods:

-   Resetting the UI to the pre signed-in state.
-   Setting any signed-in variables in your application logic to false
-   Calling `gapi.auth.setToken(null);`

## Testing a JavaScript client against a local development server

You can test your client against a backend API running in production App Engine at any time without making any changes. However, if you want to test your client against a backend API running on the local development server, you'll need to change the javascript origin in the backend API's web client ID.

To make the required changes and test using the local dev server:

1.  Open the [Credentials page](https://web.archive.org/web/20160424225905/https://console.cloud.google.com/project/_/apiui/credential) for your project.

2.  Double-click on your OAuth 2.0 Web client in the client ID list.

3.  In the web client ID form, change the **Authorized JavaScript origins** field to `http://localhost:8080`.

4.  Start the local development server, as described in [Running and testing API backends locally](https://web.archive.org/web/20160424225905/https://cloud.google.com/appengine/docs/java/endpoints/test_deploy#running_and_testing_api_backends_locally).

    Make sure you don't change the default port, which is `8080`, because this is what you used to set the authorized JavaScript origins previously. (You can change the port if you wish, so long as the authorized JavaScript origins and the local dev server are both set to the same port.)

5.  Test your client as desired.

**Important:** If the backend API is Auth-protected, your client must provide support as described above under [Adding Authentication Support with OAuth 2.0](#adding_authentication_support_with_oauth_20). However, when running against the local dev server, no credentials page is displayed in response to a user's attempt to sign in. Instead, the sign-on simply occurs, granting the user access to the protected API, *and the user returned is always `example@example.com`*.

## Sample client source code

For a sample backend API and JavaScript client that illustrates the concepts described above, see the [Hello Endpoints](https://web.archive.org/web/20160424225905/https://github.com/GoogleCloudPlatform/appengine-endpoints-helloendpoints-java-maven) sample.
