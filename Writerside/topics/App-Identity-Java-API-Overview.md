# App Identity Java API Overview

  

[Python](https://web.archive.org/web/20160424225320/https://cloud.google.com/appengine/docs/python/appidentity/ "View this page in the Python runtime") <span style="color: #ddd; padding: 0em .5em;">|</span><span style="color: black; font-weight:bold">Java</span> <span style="color: #ddd; padding: 0em .5em;">|</span>[PHP](https://web.archive.org/web/20160424225320/https://cloud.google.com/appengine/docs/php/appidentity/ "View this page in the PHP runtime") <span style="color: #ddd; padding: 0em .5em;">|</span>[Go](https://web.archive.org/web/20160424225320/https://cloud.google.com/appengine/docs/go/appidentity/ "View this page in the Go runtime")

The App Identity API lets an application discover its application ID (also called the [project ID](https://web.archive.org/web/20160424225320/https://support.google.com/cloud/answer/6158840)). Using the ID, an App Engine application can assert its identity to other App Engine Apps, Google APIs, and third-party applications and services. The application ID can also be used to generate a URL or email address, or to make a run-time decision.

## Contents

1.  [Getting the application ID](#getting_the_application_id)
2.  [Getting the application hostname](#getting_the_application_hostname)
3.  [Asserting identity to other App Engine apps](#asserting_identity_to_other_app_engine_apps)
4.  [Asserting identity to Google APIs](#asserting_identity_to_google_apis)
5.  [Asserting identity to third-party services](#asserting_identity_to_third-party_services)
6.  [Getting the default Cloud Storage Bucket name](#getting_the_default_cloud_storage_bucket_name)

## Getting the application ID

The application ID can be found using the `ApiProxy.getCurrentEnvironment().getAppId()` method.

## Getting the application hostname

By default, App Engine apps are served from URLs in the form `http://<your_app_id>.appspot.com`, where the app ID is part of the hostname. If an app is served from a custom domain, it may be necessary to retrieve the entire hostname component. You can do this using the `com.google.appengine.runtime.default_version_hostname` attribute of the `CurrentEnvironment`.

<a href="https://web.archive.org/web/20160424225320/https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/appengine/appidentity/src/main/java/com/example/appengine/appidentity/IdentityServlet.java" target="_blank" style="color: white;">appengine/appidentity/src/main/java/com/example/appengine/appidentity/IdentityServlet.java</a>

<a href="https://web.archive.org/web/20160424225320/https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/appengine/appidentity/src/main/java/com/example/appengine/appidentity/IdentityServlet.java" class="button" target="_blank" data-track-type="github" data-track-name="gitHubViewButton" data-track-metadata-link-destination="https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/appengine/appidentity/src/main/java/com/example/appengine/appidentity/IdentityServlet.java">View on GitHub</a>

\[This section requires a browser that supports JavaScript and iframes.\]

## Asserting identity to other App Engine apps

If you want to determine the identity of the App Engine app that is making a request to your App Engine app, you can use the request header `X-Appengine-Inbound-Appid`. This header is added to the request by the URLFetch service and is not user modifiable, so it safely indicates the requesting application's ID, if present.

In order for this header to be added to the request, the app making the request must tell the URLFetch service to not follow redirects. That is, your app must specify [doNotFollowRedirect](https://web.archive.org/web/20160424225320/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/urlfetch/FetchOptions#doNotFollowRedirects()) if it uses the [URLFetchService](https://web.archive.org/web/20160424225320/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/urlfetch/URLFetchService) class. If your app uses [`java.net`](https://web.archive.org/web/20160424225320/https://cloud.google.com/appengine/docs/java/urlfetch/usingjavanet), it must set the connection as follows: `connection.setInstanceFollowRedirects(false);` App Engine will then automatically add the header to the HTTP response.

In your application handler, you can check the incoming ID by reading the `X-Appengine-Inbound-Appid` header and comparing it to a list of IDs allowed to make requests.

**Note:** The `X-Appengine-Inbound-Appid` header is only set if the call is made to the `appspot.com` domain. If the app has a custom domain, this header **will not be set.**

## Asserting identity to Google APIs

Google APIs use the OAuth 2.0 protocol for [authentication and authorization](https://web.archive.org/web/20160424225320/https://developers.google.com/identity/protocols/OAuth2). The App Identity API can create OAuth tokens that can be used to assert that the source of a request is the application itself. The `getAccessToken()` method returns an access token for a scope, or list of scopes. This token can then be set in the HTTP headers of a call to identify the calling application.

The following example shows a REST call to the Google URL Shortener API. Note that the [Google API Client Libraries](https://web.archive.org/web/20160424225320/https://developers.google.com/url-shortener/libraries) can also manage much of this for you automatically.

<a href="https://web.archive.org/web/20160424225320/https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/appengine/appidentity/src/main/java/com/example/appengine/appidentity/UrlShortener.java" target="_blank" style="color: white;">appengine/appidentity/src/main/java/com/example/appengine/appidentity/UrlShortener.java</a>

<a href="https://web.archive.org/web/20160424225320/https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/appengine/appidentity/src/main/java/com/example/appengine/appidentity/UrlShortener.java" class="button" target="_blank" data-track-type="github" data-track-name="gitHubViewButton" data-track-metadata-link-destination="https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/appengine/appidentity/src/main/java/com/example/appengine/appidentity/UrlShortener.java">View on GitHub</a>

\[This section requires a browser that supports JavaScript and iframes.\]

Note that the application's identity is represented by the service account name, which is typically *applicationid@appspot.gserviceaccount.com*. You can get the exact value by using the `getServiceAccountName()` method. For services which offer ACLs, you can grant the application access by granting this account access.

## Asserting identity to third-party services

The token generated by `getAccessToken()` only works against Google services. However you can use the underlying signing technology to assert the identity of your application to other services. The `signForApp()` method will sign bytes using a private key unique to your application, and the `getPublicCertificatesForApp()` method will return certificates which can be used to validate the signature.

**Note:** The certificates may be rotated from time to time, and the method may return multiple certificates. Only certificates that are currently valid are returned; if you store signed messages you will need additional key management in order to verify signatures later.

Here is an example showing how to sign a blob and validate its signature:

<a href="https://web.archive.org/web/20160424225320/https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/appengine/appidentity/src/main/java/com/example/appengine/appidentity/SignForAppServlet.java" target="_blank" style="color: white;">appengine/appidentity/src/main/java/com/example/appengine/appidentity/SignForAppServlet.java</a>

<a href="https://web.archive.org/web/20160424225320/https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/appengine/appidentity/src/main/java/com/example/appengine/appidentity/SignForAppServlet.java" class="button" target="_blank" data-track-type="github" data-track-name="gitHubViewButton" data-track-metadata-link-destination="https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/appengine/appidentity/src/main/java/com/example/appengine/appidentity/SignForAppServlet.java">View on GitHub</a>

\[This section requires a browser that supports JavaScript and iframes.\]

## Getting the default Cloud Storage Bucket name

Each application can have up to one default Cloud Storage bucket, which has free quota and doesn't require billing to be enabled. The maximum storage of this bucket is 5GB, which you can increase by [enabling billing](https://web.archive.org/web/20160424225320/https://cloud.google.com/appengine/docs/java/console/#billing) for your app and paying for the extra storage making this a paid bucket.

To get the name of the default bucket, you can use the App Identity API. Call [AppIdentityService.getDefaultGcsBucketName](https://web.archive.org/web/20160424225320/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/appidentity/AppIdentityService.html#getDefaultGcsBucketName--).
