# Hello Endpoints Code Walkthrough

  

1.  [Imports](#imports)
2.  [API definition (configuration) using @Api](#api_definition_configuration_using_api)
3.  [A simple GET](#a_simple_get)
4.  [A simple POST](#a_simple_post)
5.  [OAuth Protecting a method](#oauth_protecting_a_method)
6.  [Done with the tutorial: what's next?](#done_with_the_tutorial_whats_next)

In this part of the tutorial, we'll look at the code you'll need to add in your own backend API, using the *Hello Endpoints* code to illustrate. The backend API is defined in the class file `helloendpoints/src/main/java/com/google/appengine/samples/helloendpoints/Greetings.java`.

In this sample, the backend API is implemented as a plain Java object, inheriting from `Object`. However, it can be part of a more complex class hierarchy if you want. (For backend APIs implemented in multiple classes, see [Multiclass APIs](https://web.archive.org/web/20160424230434/https://cloud.google.com/appengine/docs/java/endpoints/multiclass)).

## Imports

Here are the imports for `Greetings.java`:

<a href="https://web.archive.org/web/20160424230434/https://github.com/GoogleCloudPlatform/appengine-endpoints-helloendpoints-java-maven/blob/master/src/main/java/com/example/helloendpoints/Greetings.java" target="_blank" style="color: white;">src/main/java/com/example/helloendpoints/Greetings.java</a>

<a href="https://web.archive.org/web/20160424230434/https://github.com/GoogleCloudPlatform/appengine-endpoints-helloendpoints-java-maven/blob/master/src/main/java/com/example/helloendpoints/Greetings.java" class="button" target="_blank" data-track-type="github" data-track-name="gitHubViewButton" data-track-metadata-link-destination="https://github.com/GoogleCloudPlatform/appengine-endpoints-helloendpoints-java-maven/blob/master/src/main/java/com/example/helloendpoints/Greetings.java">View on GitHub</a>

\[This section requires a browser that supports JavaScript and iframes.\]

You must always import `com.google.api.server.spi.config.Api` because you must always annotate your API class with [@Api](https://web.archive.org/web/20160424230434/https://cloud.google.com/appengine/docs/java/endpoints/javadoc/com/google/api/server/spi/config/Api), as shown in the snippet. In this sample, we also import `com.google.api.server.spi.config.ApiMethod` because we want to illustrate its use in changing the name of a method, but this is optional; all public methods of the class with the `@Api` annotation are automatically exposed in the backend API.

We import `com.google.appengine.api.users.User` because we have a method protected by OAuth, and this import is required for that. We also import `import javax.inject.Named` because we need it for our request parameters.

## API definition (configuration) using `@Api`

Let's look at the `@Api` annotation in `Greetings.java`:

<a href="https://web.archive.org/web/20160424230434/https://github.com/GoogleCloudPlatform/appengine-endpoints-helloendpoints-java-maven/blob/master/src/main/java/com/example/helloendpoints/Greetings.java" target="_blank" style="color: white;">src/main/java/com/example/helloendpoints/Greetings.java</a>

<a href="https://web.archive.org/web/20160424230434/https://github.com/GoogleCloudPlatform/appengine-endpoints-helloendpoints-java-maven/blob/master/src/main/java/com/example/helloendpoints/Greetings.java" class="button" target="_blank" data-track-type="github" data-track-name="gitHubViewButton" data-track-metadata-link-destination="https://github.com/GoogleCloudPlatform/appengine-endpoints-helloendpoints-java-maven/blob/master/src/main/java/com/example/helloendpoints/Greetings.java">View on GitHub</a>

\[This section requires a browser that supports JavaScript and iframes.\]

This API is implemented in the single class called `Greetings`, with the API `name` and `version` always required. If you need to create an API that exposes multiple classes, you must use the same `@Api` annotation for each class. (The `name` and `version` would have to be identical for each class). For details on the attributes available, see [@Api: API-Scoped Annotations](https://web.archive.org/web/20160424230434/https://cloud.google.com/appengine/docs/java/endpoints/annotations#api_api-scoped_annotations) or the Javadoc for [@Api](https://web.archive.org/web/20160424230434/https://cloud.google.com/appengine/docs/java/endpoints/javadoc/com/google/api/server/spi/config/Api).

The sample protects a method by OAuth, which means that the `scopes` attribute is required; this must be set to the value `https://www.googleapis.com/auth/userinfo.email`, which is what `Constants.EMAIL_SCOPE` resolves to. (This scope lets OAuth work with Google Accounts.)

Also, because of the OAuth protection, we must supply a list of clients allowed to access the protected method in the `clientIDs` attribute. The sample suggests one way to do this, with lists of client IDs in the `Constants.java` file. So far, we have supplied a valid web client ID via `Constants.java`, but we have dummy values for Android and iOS clients to show that this client ID list could contain all the supported clients. Only clients in this list can access the protected method.

The `audiences` attribute is set to the backend API's web client. This attribute must be set if there are any Android clients; it is not used for non-Android clients.

Inside the class, notice the lines that set up an `ArrayList` of stored `HelloGreeting` (defined in `HelloGreeting.java`) [JavaBean](https://web.archive.org/web/20160424230434/http://stackoverflow.com/questions/3295496/what-is-a-javabean-exactly) objects that are returned from the methods. In Endpoints, methods can only return Objects (treated as JavaBean objects) or a collection of Objects, which are converted to JSON to form the response.

## A simple GET

The following lines return text greetings. Both `getGreeting` and `listGreeting` are public, so they will be exposed in the backend API:

<a href="https://web.archive.org/web/20160424230434/https://github.com/GoogleCloudPlatform/appengine-endpoints-helloendpoints-java-maven/blob/master/src/main/java/com/example/helloendpoints/Greetings.java" target="_blank" style="color: white;">src/main/java/com/example/helloendpoints/Greetings.java</a>

<a href="https://web.archive.org/web/20160424230434/https://github.com/GoogleCloudPlatform/appengine-endpoints-helloendpoints-java-maven/blob/master/src/main/java/com/example/helloendpoints/Greetings.java" class="button" target="_blank" data-track-type="github" data-track-name="gitHubViewButton" data-track-metadata-link-destination="https://github.com/GoogleCloudPlatform/appengine-endpoints-helloendpoints-java-maven/blob/master/src/main/java/com/example/helloendpoints/Greetings.java">View on GitHub</a>

\[This section requires a browser that supports JavaScript and iframes.\]

The `getGreeting` method serves an incoming GET request that has a numeric value indicating the user's choice of greeting:

-   A value of `0` returning the first value, `hello world!`.
-   A value of `1` returning the second value, `goodbye world!`.

All other values will return an error. Notice that the `@Named` attribute must be used for the incoming parameter since it is not an entity type.

The `listGreeting` method returns all the stored greetings in the array list.

## A simple POST

The following lines of code handle an incoming POST request containing a user-supplied greeting and integer, and returns the greeting repeated the number of times specified by the integer:

<a href="https://web.archive.org/web/20160424230434/https://github.com/GoogleCloudPlatform/appengine-endpoints-helloendpoints-java-maven/blob/master/src/main/java/com/example/helloendpoints/Greetings.java" target="_blank" style="color: white;">src/main/java/com/example/helloendpoints/Greetings.java</a>

<a href="https://web.archive.org/web/20160424230434/https://github.com/GoogleCloudPlatform/appengine-endpoints-helloendpoints-java-maven/blob/master/src/main/java/com/example/helloendpoints/Greetings.java" class="button" target="_blank" data-track-type="github" data-track-name="gitHubViewButton" data-track-metadata-link-destination="https://github.com/GoogleCloudPlatform/appengine-endpoints-helloendpoints-java-maven/blob/master/src/main/java/com/example/helloendpoints/Greetings.java">View on GitHub</a>

\[This section requires a browser that supports JavaScript and iframes.\]

This method is annotated by `@ApiMethod` so we can override the default name that is generated by Endpoints. Notice that Endpoints prepends the class name (`greetings`, lowercase) to the method name when it generates the method name in the backend API: `greetings.insertGreeting`. When we override this value using the method annotation, the prepending does not take place, so we need to add the class prepending manually in our code: `greetings.multiply` to make it consistent with the other backend API method names.

The method annotation can perform other API overrides as well; for more details, see [@ApiMethod: Method-Scoped Annotations](https://web.archive.org/web/20160424230434/https://cloud.google.com/appengine/docs/java/endpoints/annotations#apimethod_method-scoped_annotations).

## OAuth Protecting a method

Any client can access your Endpoints API methods unless you protect them with OAuth 2.0. In some scenarios, you may want to restrict access to some or all of the API methods.

To protect a method in the backend API, you need to do the following:

-   Add required support in the [@Api annotation](#api_definition_configuration_using_api) as described above:
    -   Add `scopes` set to the email scope.
    -   Add `clientIds` containing the client whitelist.
    -   For Android devices only, specify `audiences`.
-   Add a `User` parameter to the method you wish to protect.

The code you copied already has the required annotations, so look at the code with the added `User` parameter:

<a href="https://web.archive.org/web/20160424230434/https://github.com/GoogleCloudPlatform/appengine-endpoints-helloendpoints-java-maven/blob/master/src/main/java/com/example/helloendpoints/Greetings.java" target="_blank" style="color: white;">src/main/java/com/example/helloendpoints/Greetings.java</a>

<a href="https://web.archive.org/web/20160424230434/https://github.com/GoogleCloudPlatform/appengine-endpoints-helloendpoints-java-maven/blob/master/src/main/java/com/example/helloendpoints/Greetings.java" class="button" target="_blank" data-track-type="github" data-track-name="gitHubViewButton" data-track-metadata-link-destination="https://github.com/GoogleCloudPlatform/appengine-endpoints-helloendpoints-java-maven/blob/master/src/main/java/com/example/helloendpoints/Greetings.java">View on GitHub</a>

\[This section requires a browser that supports JavaScript and iframes.\]

When you declare a parameter of type `User` in your API method as shown in the snippet above, the API backend framework automatically authenticates the user and enforces the authorized `clientIds` whitelist, ultimately by supplying the valid `User` or not.

If the request coming in from the client has a valid auth token or is in the list of authorized `clientIDs`, the backend framework supplies a valid User to the parameter. If the incoming request does not have a valid auth token or if the client is not on the `clientIDs` whitelist, the framework sets `User` to `null`. Your own code must handle both the case where `User` is null and the case where there is a valid `User`. If there is no `User`, for example, you could choose to return a not-authenticated error or perform some other desired action.

## Done with the tutorial: what's next?

Now that you've created your first backend APIs, take a deeper dive into backend API features and configurations, and you may want to hook up other App Engine services to your backend API, such as the Datastore. You can learn more by visiting the following resources:

-   [Annotating Your Code](https://web.archive.org/web/20160424230434/https://cloud.google.com/appengine/docs/java/endpoints/annotations); which covers all of the API configuration and features.
-   The [Tic-Tac-Toe](https://web.archive.org/web/20160424230434/https://github.com/GoogleCloudPlatform/appengine-endpoints-tictactoe-java) sample; which shows how to build a backend that uses the Datastore.
