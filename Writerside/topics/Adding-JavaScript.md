# Adding JavaScript

  

1.  [Adding JavaScript in base.js](#adding_javascript_in_basejs)
2.  [Running and testing your client](#running_and_testing_your_client)
3.  [Next...](#next)

In this part of the tutorial, you'll create the JavaScript to support the UI in the `index.html` page that makes a simple GET request to the backend API. The JavaScript will perform two basic functions:

-   Load the backend API interface.
-   Send the Get Greeting request to the backend when the user clicks **Submit** and display the results.

## Adding JavaScript in `base.js`

To add the JavaScript:

1\. Create the subdirectory `js` in the [backend API project](https://web.archive.org/web/20160424230932/https://cloud.google.com/appengine/docs/java/endpoints/getstarted/backend) subdirectory `src/main/webapp/`.

1.  In the `js` subdirectory, add a file named `base.js`. There is nothing magic about the subdirectory and file name; in your own code you can use whatever you want. Our tutorial expects `/js/base.js`.

2.  In `base.js`, add the following lines to set namespaces expected for this sample:

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
    ```

3.  In `base.js`, append the following lines to load the backend API interface and perform some initialization upon page load:

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
        }
      }

      apisToLoad = 1; // must match number of calls to gapi.client.load()
      gapi.client.load('helloworld', 'v1', callback, apiRoot);
    };
    ```

    In the above code, the `google.devrel.samples.hello.init` function gets passed the backend API's URL (`apiRoot`) from the `index.html` when that page is loaded, through the `init` function defined in `index.html`. We specify the API version we want to load, and provide a callback function to execute after the API is loaded. In our callback, we turn on the buttons, which are disabled otherwise.

    Notice that the callback is executed only after all the APIs are loaded in the calls to `gapi.client.load`. If you wish to use multiple Endpoints APIs or other Google APIs in your JavaScript, increment `apisToLoad` once for each API and add an additional call to `gapi.client.load` for each API you want to use.

4.  In `base.js`, append the following lines that enable the buttons for the above callback when the backend API loads successsfully:

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

    };
    ```

5.  In `base.js`, append the following code:

    ```
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
    ```

    This code handles the user's **Submit** button clicks and displays the results in the browser. The **List Greetings** button gets all the greetings, and the **Get Greeting** button gets the greeting specified in the form.

    Notice how the `getGreeting` function in `base.js` invokes the backend API to send the ID parameter in the GET request:

    ```
    gapi.client.helloWorld.greetings.getGreeting({'id': id})
    ```

6.  When you are finished, your file should look like this:

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
        }
      }

      apisToLoad = 1; // must match number of calls to gapi.client.load()
      gapi.client.load('helloworld', 'v1', callback, apiRoot);
    };
    ```

7.  This completes the coding; you are ready to run and test.

## Running and testing your client

1.  Rebuild your project with the new client:

    ```
    mvn clean install
    ```

2.  Upload and deploy the project and client code:

    ```
    mvn appengine:update
    ```

    When the backend finishes deploying, a message similar to this one is displayed:

    ```
    09:08 AM Completed update of app: your-app-id, version: 1
    ```

3.  Visit the App Engine URL: `https://<your-app-id>.appspot.com/`, to display the UI:

    ![Hello-Endpoints](https://web.archive.org/web/20160424230932im_/https://cloud.google.com/appengine/docs/java/endpoints/getstarted/clients/js/images/hello.png)

    **Important:** You should use `https`, not `http`, to access your web client app. Serving the HTML over `http` will trigger a mixed content warning and may prevent the page from fully loading in browsers with strict security configurations.

4.  Enter a `0` or `1` in the text box and click **Submit** to display a hardcoded Greeting from the backend API. Click **Submit** under **List Greetings** to get all the Greetings.

## Next...

Continue to [Adding a POST Request](https://web.archive.org/web/20160424230932/https://cloud.google.com/appengine/docs/java/endpoints/getstarted/clients/js/add_post).
