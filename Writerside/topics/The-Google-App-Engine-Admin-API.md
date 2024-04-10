# The Google App Engine Admin API

  

**Beta**

This is a Beta release of Google App Engine Admin API. This API is not covered by any SLA or deprecation policy and may be subject to backward-incompatible changes.

The Google App Engine Admin API is a RESTful API for provisioning and managing App Engine applications. It provides programmatic access to many of the App Engine administrative operations found in the [Google Cloud Platform Console](https://web.archive.org/web/20160424224718/https://cloud.google.com/appengine/docs/developers-console).

The current release of the Google App Engine Admin API is `v1beta5`.

1.  [API setup](#api_setup)
2.  [API calls](#api_calls)
3.  [Deployments](#deployments)
4.  [Sample cURL request](#sample_curl_request)
5.  [Client libraries](#client_libraries)

## API setup

You first need to enable access to the Google App Engine Admin API for a Cloud project.

1.  <a href="https://web.archive.org/web/20160424224718/https://console.cloud.google.com/flows/enableapi?apiid=appengine,storage_component" target="console" data-track-type="commonIncludes" data-track-name="consoleLink" data-track-metadata-end-goal="enableAPI">Enable the Google App Engine Admin API and Cloud Storage APIs</a>.

2.  Obtain an access token that grants access to these APIs by following the [OAuth 2.0](https://web.archive.org/web/20160424224718/https://developers.google.com/accounts/docs/OAuth2) setup instructions. The scope parameter should contain:

    ```
     https://www.googleapis.com/auth/cloud-platform
    ```

## API calls

The Google App Engine Admin API follows a RESTful design, which means that standard HTTP methods are used to retrieve and manipulate resources. For example, to list of all the versions in the default service of an application named "guestbook," you might send an HTTP request like:

```
 GET https://appengine.googleapis.com/v1beta5/apps/guestbook/services/default/versions
```

## Deployments

The Google App Engine Admin API deploys application resources that are staged in [Google Cloud Storage](https://web.archive.org/web/20160424224718/https://cloud.google.com/storage/). Before making a deployment, you need to upload all relevant code, static files, and dockerfiles to Cloud Storage using one of the available [tools](https://web.archive.org/web/20160424224718/https://cloud.google.com/storage/docs/overview#using). A deployment request consists of a version definition and a manifest of Cloud Storage resources to be copied and packaged with the app. The version definition is a JSON representation of app.yaml and the manifest is a mapping from the filesystem name to the Cloud Storage URL.

## Sample cURL request

Before getting started, follow the [API setup](#api_setup) to authenticate your Cloud project. Save the access token in an environment variable.

```
 export TOKEN=<token>
```

The sample command below deploys a new version to the default service of the "guestbook" appplication, with a configuration as specified in app.json. See the [Quickstart](https://web.archive.org/web/20160424224718/https://cloud.google.com/appengine/docs/admin-api/quickstart) for an example of a JSON configuration.

```
 curl -vv -T app.json -H "Content-Type: application/json" -H "Authorization: Bearer $TOKEN" https://appengine.googleapis.com/v1beta5/apps/guestbook/services/default/versions
```

## Client libraries

You can build client libraries for the Google App Engine Admin API using the [Google API Discovery Service](https://web.archive.org/web/20160424224718/https://developers.google.com/discovery/v1/using) and the Google App Engine Admin API discovery document, which lives at:

```
https://www.googleapis.com/discovery/v1/apis/appengine/v1beta5/rest
```
