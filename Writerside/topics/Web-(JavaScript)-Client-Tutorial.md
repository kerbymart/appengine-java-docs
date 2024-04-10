# Web (JavaScript) Client Tutorial

  

1.  [Purpose of the tutorial](#purpose_of_the_tutorial)
2.  [Complete sample code location](#complete_sample_code_location)
3.  [Topics covered](#topics_covered)
4.  [Next...](#next)

Google Cloud Endpoints is a feature that enables developers to easily develop and host APIs on App Engine with OAuth 2.0 support. From this single API source, Endpoints also lets developers generate strongly-typed client libraries for Java (Android) and Objective-C (iOS), along with dynamically-typed libraries for JavaScript.

## Purpose of the tutorial

This tutorial focuses only on building a web (JavaScript) client that uses the API provided by the API backend created in the [API backend tutorial](https://web.archive.org/web/20160425095256/https://cloud.google.com/appengine/docs/java/endpoints/getstarted/backend). You need to complete that backend tutorial first, because this client tutorial adds code to that backend project and requires that backend API to be running in order to successfully make requests against it.

In this tutorial, you will learn how to:

-   Create a simple HTML page that provides client UI to access the backend API.
-   Create the JavaScript that is invoked by the client UI and does the actual work of accessing the backend.
-   Deploy client app and API backend to production App Engine, then test the client.
-   Use OAuth 2.0 in the client to gain access to an API backend method that is restricted to authorized clients.

## Complete sample code location

The tutorial leads you through the addition of all the code required to run the sample. However, if you want to download the entire project, visit the code repository containing the [appengine-endpoints-helloendpoints-java-maven project](https://web.archive.org/web/20160425095256/https://github.com/GoogleCloudPlatform/appengine-endpoints-helloendpoints-java-maven).

## Topics covered

The following topics lead you through the tutorial:

-   [Setup](https://web.archive.org/web/20160425095256/https://cloud.google.com/appengine/docs/java/endpoints/getstarted/clients/js/setup).
-   [Adding the Client UI](https://web.archive.org/web/20160425095256/https://cloud.google.com/appengine/docs/java/endpoints/getstarted/clients/js/client_ui) to invoke a simple GET and display results.
-   [Adding JavaScript](https://web.archive.org/web/20160425095256/https://cloud.google.com/appengine/docs/java/endpoints/getstarted/clients/js/add_javascript) for a simple GET.
-   [Adding a simple POST](https://web.archive.org/web/20160425095256/https://cloud.google.com/appengine/docs/java/endpoints/getstarted/clients/js/add_post).
-   [Adding Auth support to the client](https://web.archive.org/web/20160425095256/https://cloud.google.com/appengine/docs/java/endpoints/getstarted/clients/js/add_auth) to access a backend method Protected by OAuth.

## Next...

Continue to [Setup](https://web.archive.org/web/20160425095256/https://cloud.google.com/appengine/docs/java/endpoints/getstarted/clients/js/setup).
