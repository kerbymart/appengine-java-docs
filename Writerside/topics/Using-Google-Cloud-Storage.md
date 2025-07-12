# Using Google Cloud Storage

  

[<img src="https://web.archive.org/web/20160424230936im_/https://cloud.google.com/cloud/images/stack_overflow_questions.png" title="Stack Overflow Questions" data-border="0" width="240" height="53" />](https://web.archive.org/web/20160424230936/http://stackoverflow.com/questions/tagged/google-app-engine+google-cloud-storage)

[](https://web.archive.org/web/20160424230936/http://stackoverflow.com/feeds/tag?sort=votes&tagnames=google-app-engine%2Bgoogle-cloud-storage)

<span class="link gc-analytics-event" category="Sidebar" data-action="Stack Overflow question click"></span>

<a href="https://web.archive.org/web/20160424230936/http://stackoverflow.com/questions/tagged/google-app-engine+google-cloud-storage?sort=votes" class="gc-analytics-event" data-category="Sidebar" data-action="Stack Overflow See more link">See more...</a>

You can use a [Cloud Storage](https://web.archive.org/web/20160424230936/https://cloud.google.com/storage/) bucket to store and serve files, such as movies or images or other static content.

This document describes how to store and retrieve data using Cloud Storage in an App Engine app using the Google Cloud Storage client library. This client library provides error handling, retries, read buffering with prefetch, and uses HTTPS so your data is encrypted during transfer.

For more information about Cloud Storage, see the Cloud Storage [documentation](https://web.archive.org/web/20160424230936/https://cloud.google.com/storage/docs).

## Contents

1.  [Prerequisites and setup](#prerequisites_and_setup)
2.  [Activating a Cloud Storage bucket](#activating_a_cloud_storage_bucket)
    1.  [Setting bucket and object permissions](#setting_bucket_and_object_permissions)
3.  [Downloading the client library](#downloading_the_client_library)
4.  [Reading and writing to a bucket](#reading_and_writing_to_a_bucket)
5.  [Using the client library with the development app server](#using_the_client_library_with_the_development_app_server)
6.  [Pricing, quotas, and limits](#pricing_quotas_and_limits)
7.  [Alternatives ways to access Cloud Storage](#alternatives_ways_to_access_cloud_storage)

## Prerequisites and setup

You need to complete the following:

1.  Follow the instructions in ["Hello, World!" for Java on App Engine](https://web.archive.org/web/20160424230936/https://cloud.google.com/appengine/docs/java/) to set up your environment and project, and to understand how Java apps are structured in App Engine. Write down and save your project ID for use with your application.
2.  Activate a Cloud Storage bucket for your app to use, and set permissions if neccessary .
3.  Download the client library.

## Activating a Cloud Storage bucket

To use Cloud Storage, you'll need to activate at least one bucket. Typically, you might want to use the 5GB free default bucket, and upgrade this into a paid bucket if you need more storage. You can always activate and use other paid bucket if you want, but there is only one free default bucket per project.

To activate the free, default Cloud Storage bucket for your app:

1.  Click **Create** under *Default Cloud Storage Bucket* in the [App Engine settings](https://web.archive.org/web/20160424230936/https://console.cloud.google.com/appengine/settings) page for your project. Notice the name of this bucket: it is in the form `<project-id>.appspot.com`.

2.  If you need more storage than the 5GB limit, you can increase this by [enabling billing](https://web.archive.org/web/20160424230936/https://console.cloud.google.com/billing) for your project, making this a paid bucket. You will be charged for storage over the 5GB limit.

If you want to activate one or more paid buckets, follow the instructions under [Creating a bucket](https://web.archive.org/web/20160424230936/https://cloud.google.com/storage/docs/cloud-console#_creatingbuckets) to activate them.

### Setting bucket and object permissions

By default, when you create a bucket for your project, your app has all the permissions required to read and write to it.

If you want to set permissions to allow other users to access the bucket and its contents, see [Setting bucket permissions](https://web.archive.org/web/20160424230936/https://cloud.google.com/storage/docs/cloud-console#_bucketpermission) and [Setting object permissions](https://web.archive.org/web/20160424230936/https://cloud.google.com/storage/docs/cloud-console#_permissions).

## Downloading the client library

You can download the library using popular tools like <a href="https://web.archive.org/web/20160424230936/https://maven.apache.org/" target="_blank">Apache Maven</a>, <a href="https://web.archive.org/web/20160424230936/http://ant.apache.org/ivy/" target="_blank">Apache Ivy</a>, or <a href="https://web.archive.org/web/20160424230936/http://git-scm.com/" target="_blank">Git</a>, or you can download the library manually from the Maven repository. Choose your preferred method:

### Maven

Maven users should include the following in their application's `pom.xml` file:

```
<dependency>
    <groupId>com.google.appengine.tools</groupId>
    <artifactId>appengine-gcs-client</artifactId>
    <version>RELEASE</version>
</dependency>
```

Note that `RELEASE` is a literal value that results in the download of the latest client library for your project.

### Ivy

Ivy users should include the following in their application's `ivy.xml` file:

```
<dependency org="com.google.appengine.tools"
            name="appengine-gcs-client"
            rev="latest.integration" />
```

### Git

If you have Git installed, you can clone the Google Cloud Storage client library's GitHub repository as follows:

```
git clone https://github.com/GoogleCloudPlatform/appengine-gcs-client.git appengine-gcs-client-read-only
```

### Manual download

Visit the library's Maven repository and download the latest class, source, and JavaDoc JAR files:

-   [Google Cloud Storage client library](https://web.archive.org/web/20160424230936/http://search.maven.org/#search%7Cga%7C1%7Cg%3A%22com.google.appengine.tools%22%20a%3A%22appengine-gcs-client%22)

In addition, you will need to download the following dependencies and include them in your application:

-   [Guava library](https://web.archive.org/web/20160424230936/https://github.com/google/guava)
-   [Joda-Time library](https://web.archive.org/web/20160424230936/http://www.joda.org/joda-time/)
-   [Cloud Storage API client library](https://web.archive.org/web/20160424230936/https://developers.google.com/api-client-library/java/apis/storage/v1)

**Note:** If you use Maven or Ivy, the dependencies are automatically retrieved for you.

## Reading and writing to a bucket

The following sample shows how to write a file to a Cloud Storage bucket, and how to retrieve a file:

<a href="https://web.archive.org/web/20160424230936/https://github.com/GoogleCloudPlatform/appengine-gcs-client/blob/master/java/example/src/com/google/appengine/demos/GcsExampleServlet.java" target="_blank" style="color: white;">java/example/src/com/google/appengine/demos/GcsExampleServlet.java</a>

<a href="https://web.archive.org/web/20160424230936/https://github.com/GoogleCloudPlatform/appengine-gcs-client/blob/master/java/example/src/com/google/appengine/demos/GcsExampleServlet.java" class="button" target="_blank" data-track-type="github" data-track-name="gitHubViewButton" data-track-metadata-link-destination="https://github.com/GoogleCloudPlatform/appengine-gcs-client/blob/master/java/example/src/com/google/appengine/demos/GcsExampleServlet.java">View on GitHub</a>

\[This section requires a browser that supports JavaScript and iframes.\]

Notice the sample shows the optional use of the Blobstore API for serving files stored in Cloud Storage. This can be a more cost effective and faster way to return files to the caller because it returns the file to the caller without your app having to read the file. For more information, see [Using the Blobstore API with Google Cloud Storage](https://web.archive.org/web/20160424230936/https://cloud.google.com/appengine/docs/java/blobstore/#Java_Using_the_Blobstore_API_with_Google_Cloud_Storage).

Also, notice that the code above shows how to set the retry parameters, which control the retries that occur in case of network or resource issues. Normally, you should not have to set the retry parameters, because the default values will work for the majority of cases.

## Using the client library with the development app server

You can use the client library with the development server. This provides Cloud Storage emulation using the local disk.

**Note:** Files saved locally are subject to the file size and naming conventions imposed by the local filesystem.

## Pricing, quotas, and limits

There are no bandwidth charges associated with making Google Cloud Storage client library calls to Cloud Storage. However, there are operations charges; because the calls count against your [URL fetch quota](https://web.archive.org/web/20160424230936/https://cloud.google.com/appengine/docs/quotas#UrlFetch), as the library uses the URL Fetch service to interact with Cloud Storage. And There are operations and storage charges, however,

Notice that Google Cloud Storage is a pay-to-use service; you will be charged according to the Cloud Storage [price sheet](https://web.archive.org/web/20160424230936/https://cloud.google.com/storage/pricing).

## Alternatives ways to access Cloud Storage

Instead of using the client library, you could use the following:

-   [Cloud Storage REST API](https://web.archive.org/web/20160424230936/https://cloud.google.com/appengine/docs/go/googlecloudstorageclient/#Go_Cloud_Storage_REST_API). This could be a good choice if you need a specific feature not provided in the client library. However, this isn't compatible with the development server.
-   [Cloud Storage Browser](https://web.archive.org/web/20160424230936/https://console.cloud.google.com/storage/browser) in the Google Cloud Platform Console. Useful for uploading objects quickly.
-   [gsutil](https://web.archive.org/web/20160424230936/https://cloud.google.com/storage/docs/gsutil). A command line tool.
