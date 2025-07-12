# Adding a Simple POST

  

1.  [Adding UI and JavaScript for a Simple POST](#adding_ui_and_javascript_for_a_simple_post)
2.  [Next...](#next)

In this part of the tutorial, you'll add UI components and JavaScript to support a simple POST request to the backend API.

## Adding UI and JavaScript for a Simple POST

To add a POST request to the client:

1.  In the `index.html` file you added previously in the tutorial, under the lines for the `getGreeting` form, add the following lines for the `multiplyGreeting` UI:

    ```
    <form onsubmit="return false;">
        <h2>Multiply Greetings</h2>
        <div><label for="greeting">Greeting:</label><input id="greeting" /></div>
        <div><label for="count">Count:</label><input id="count" /></div>
        <div><input id="multiplyGreetings" type="submit" class="btn btn-small" value="Submit"></div>
    </form>
    ```

2.  When you are finished, your `index.html` file should look like this:

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

3.  In the `base.js` file you added previously in the tutorial, under the lines for the `getGreeting` function, add the following lines to handle the user clicking on the `multiplyGreetings` **Submit** button:

    ```
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
    ```

4.  In `base.js`, inside the curly braces for the `enableButtons` function, append the following lines to enable the new UI buttons.

    ```
    var multiplyGreetings = document.querySelector('#multiplyGreetings');
    multiplyGreetings.addEventListener('click', function(e) {
      google.appengine.samples.hello.multiplyGreeting(
          document.querySelector('#greeting').value,
          document.querySelector('#count').value);
    });
    ```

5.  When you are finished, your `base.js` file should look like this:

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

6.  Rebuild your project with the new client:

    ```
    mvn clean install
    ```

7.  Upload and deploy the project and client code:

    ```
    mvn appengine:update
    ```

8.  Visit the App Engine URL: `https://<your-app-id>.appspot.com/`, to display the UI:

    ![Hello-Endpoints](https://web.archive.org/web/20160424230704im_/https://cloud.google.com/appengine/docs/java/endpoints/getstarted/clients/js/images/hello2.png)

9.  Enter some text in **Greeting**, supply the number of times to repeat the greeting in **Count**, then click **Submit** to display your Greeting the specified number of times.

## Next...

Continue to [Adding Auth](https://web.archive.org/web/20160424230704/https://cloud.google.com/appengine/docs/java/endpoints/getstarted/clients/js/add_auth).
