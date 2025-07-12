# Blobstore Java API Overview

  

[Python](https://web.archive.org/web/20160424225902/https://cloud.google.com/appengine/docs/python/blobstore/ "View this page in the Python runtime") <span style="color: #ddd; padding: 0em .5em;">|</span><span style="color: black; font-weight:bold">Java</span> <span style="color: #ddd; padding: 0em .5em;">|</span><span style="color:lightgray" title="This page is not available in the PHP runtime">PHP</span> <span style="color: #ddd; padding: 0em .5em;">|</span>[Go](https://web.archive.org/web/20160424225902/https://cloud.google.com/appengine/docs/go/blobstore/ "View this page in the Go runtime")

[<img src="https://web.archive.org/web/20160424225902im_/https://cloud.google.com/cloud/images/stack_overflow_questions.png" title="Stack Overflow Questions" data-border="0" width="240" height="53" />](https://web.archive.org/web/20160424225902/http://stackoverflow.com/questions/tagged/google-app-engine+blobstore)

[](https://web.archive.org/web/20160424225902/http://stackoverflow.com/feeds/tag?sort=votes&tagnames=google-app-engine%2Bblobstore)

<span class="link gc-analytics-event" category="Sidebar" data-action="Stack Overflow question click"></span>

<a href="https://web.archive.org/web/20160424225902/http://stackoverflow.com/questions/tagged/google-app-engine+blobstore?sort=votes" class="gc-analytics-event" data-category="Sidebar" data-action="Stack Overflow See more link">See more...</a>

**Note:** You should consider using [Google Cloud Storage](https://web.archive.org/web/20160424225902/https://cloud.google.com/appengine/docs/java/googlecloudstorageclient/) rather than Blobstore for storing blob data.

The Blobstore API allows your application to serve data objects, called *blobs,* that are much larger than the size allowed for objects in the Datastore service. Blobs are useful for serving large files, such as video or image files, and for allowing users to upload large data files. Blobs are created by uploading a file through an HTTP request. Typically, your applications will do this by presenting a form with a file upload field to the user. When the form is submitted, the Blobstore creates a blob from the file's contents and returns an opaque reference to the blob, called a *blob key,* which you can later use to serve the blob. The application can serve the complete blob value in response to a user request, or it can read the value directly using a streaming file-like interface.

1.  [Introducing the Blobstore](#Java_Introducing_the_Blobstore)
2.  [Using the Blobstore](#Java_Using_the_Blobstore)
3.  [Uploading a blob](#Java_Uploading_a_blob)
4.  [Serving a blob](#Java_Serving_a_blob)
5.  [Complete sample application](#Java_Complete_sample_application)
6.  [Using the Images service with the Blobstore](#Java_Using_the_Images_service_with_the_Blobstore)
7.  [Using the Blobstore API with Google Cloud Storage](#Java_Using_the_Blobstore_API_with_Google_Cloud_Storage)
8.  [Quotas and limits](#Java_Quotas_and_limits)

## Introducing the Blobstore

Google App Engine includes the Blobstore service, which allows applications to serve data objects limited only by the amount of data that can be uploaded or downloaded over a single HTTP connection. These objects are called *Blobstore values,* or *blobs.* Blobstore values are served as responses from request handlers and are created as uploads via web forms. Applications do not create blob data directly; instead, blobs are created indirectly, by a submitted web form or other HTTP `POST` request. Blobstore values can be served to the user, or accessed by the application in a file-like stream, using the Blobstore API.

**Note:** Blobs as defined by the Blobstore service are not related to blob property values used by the datastore.

To prompt a user to upload a Blobstore value, your application presents a web form with a file upload field. The application generates the form's action URL by calling the Blobstore API. The user's browser uploads the file directly to the Blobstore via the generated URL. Blobstore then stores the blob, rewrites the request to contain the blob key, and passes it to a path in your application. A request handler at that path in your application can perform additional form processing.

To serve a blob, your application sets a header on the outgoing response, and App Engine replaces the response with the blob value.

Blobs can't be modified after they're created, though they can be deleted. Each blob has a corresponding *blob info record*, stored in the datastore, that provides details about the blob, such as its creation time and content type. You can use the blob key to fetch blob info records and query their properties.

An application can read a Blobstore value a portion at a time using an API call. The size of the portion can be up to the maximum size of an API return value. This size is a little less than 32 megabytes, represented in Java by the constant `com.google.appengine.api.blobstore.BlobstoreService.MAX_BLOB_FETCH_SIZE`. An application cannot create or modify Blobstore values except through files uploaded by the user.

## Using the Blobstore

Applications can use the Blobstore to accept large files as uploads from users and to serve those files. Files are called blobs once they're uploaded. Applications don't access blobs directly. Instead, applications work with blobs through *blob info entities* (represented by the `BlobInfo` class) in the datastore.

The user creates a blob by submitting an HTML form that includes one or more file input fields. Your application sets `blobstoreService.createUploadUrl()` as the destination (action) of this form, passing the function a URL path of a handler in your application. When the user submits the form, the user's browser uploads the specified files directly to the Blobstore. The Blobstore rewrites the user's request and stores the uploaded file data, replacing the uploaded file data with one or more corresponding blob keys, then passes the rewritten request to the handler at the URL path you provided to `blobstoreService.createUploadUrl()`. This handler can do additional processing based on the blob key.

The application can read portions of a Blobstore value using a file-like streaming interface. See the [`BlobstoreInputStream`](https://web.archive.org/web/20160424225902/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/blobstore/BlobstoreInputStream) class.

## Uploading a blob

To create and upload a blob, follow this procedure:

#### 1. Create an upload URL

Call `blobstoreService.createUploadUrl` to create an upload URL for the form that the user will fill out, passing the application path to load when the `POST` of the form is completed.

```
<body>
    <form action="<%= blobstoreService.createUploadUrl("/upload") %>" method="post" enctype="multipart/form-data">
        <input type="file" name="myFile">
        <input type="submit" value="Submit">
    </form>
</body>
```

Note that this is how the upload form would look if it were created as a JSP.

#### 2. Create an upload form

The form must include a file upload field, and the form's `enctype` must be set to `multipart/form-data`. When the user submits the form, the `POST` is handled by the Blobstore API, which creates the blob. The API also creates an info record for the blob and stores the record in the datastore, and passes the rewritten request to your application on the given path as a blob key.

#### 3. Implement upload handler

In this handler, you can store the blob key with the rest of your application's data model. The blob key itself remains accessible from the blob info entity in the datastore. Note that after the user submits the form and your handler is called, the blob has already been saved and the blob info added to the datastore. If your application doesn't want to keep the blob, you should delete the blob immediately to prevent it from becoming orphaned:

In the following code, `getUploads` returns a set of blobs that have been uploaded. The `Map` object is a list that associates the names of the upload fields to the blobs they contained.

```
Map<String, List<BlobKey>> blobs = blobstoreService.getUploads(req);
List<BlobKey> blobKeys = blobs.get("myFile");

if (blobKeys == null || blobKeys.isEmpty()) {
    res.sendRedirect("/");
} else {
    res.sendRedirect("/serve?blob-key=" + blobKeys.get(0).getKeyString());
}
```

When the Blobstore rewrites the user's request, the MIME parts of the uploaded files have their bodies emptied, and the blob key is added as a MIME part header. All other form fields and parts are preserved and passed to the upload handler. If you don't specify a content type, the Blobstore will try to infer it from the file extension. If no content type can be determined, the newly created blob is assigned content type `application/octet-stream`.

## Serving a blob

**Note:** If you are serving images, a more efficient and potentially less-expensive method is to use [`getServingUrl()`](https://web.archive.org/web/20160424225902/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/images/ImagesService#getServingUrl(com.google.appengine.api.blobstore.BlobKey,%20int,%20boolean)) using the App Engine Images API rather than `blobstoreService.serve()`. The `getServingUrl` method lets you serve the image directly, without having to go through your App Engine instances.

To serve blobs, you must include a blob download handler as a path in your application. This handler should pass the blob key for the desired blob to `blobstoreService.serve(blobKey, res);`. In this example, the blob key is passed to the download handler as the URL argument `(req.getParameter('blob-key'))`. In practice, the download handler can get the blob key by any means you choose, such as through another method or user action.

```
public void doGet(HttpServletRequest req, HttpServletResponse res)
    throws IOException {
        BlobKey blobKey = new BlobKey(req.getParameter("blob-key"));
        blobstoreService.serve(blobKey, res);
```

Blobs can be served from any application URL. To serve a blob in your application, you put a special header in the response containing the blob key. App Engine replaces the body of the response with the content of the blob.

### Blob byte ranges

The Blobstore supports serving part of a large value instead of the full value in response to a request. To serve a partial value, include the `X-AppEngine-BlobRange` header in the outgoing response. Its value is a standard [HTTP byte range](https://web.archive.org/web/20160424225902/http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html). The byte numbering is zero-based. A blank `X-AppEngine-BlobRange` instructs the API to ignore the range header and serve the full blob. Example ranges include:

-   `0-499` serves the first 500 bytes of the value (bytes 0 through 499, inclusive).
-   `500-999` serves 500 bytes starting with the 501st byte.
-   `500-` serves all bytes starting with the 501st byte to the end of the value.
-   `-500` serves the last 500 bytes of the value.

If the byte range is valid for the Blobstore value, the Blobstore sends a `206` `Partial` `Content` status code and the requested byte range to the client. If the range is not valid for the value, the Blobstore sends `416` `Requested` `Range` `Not` `Satisfiable`.

The Blobstore does not support multiple byte ranges in a single request (for example, `100-199,200-299`), whether or not they overlap.

## Complete sample application

In the following sample application, the application's main URL loads the form that asks the user for a file to upload, and the upload handler immediately calls the download handler to serve the data. This is to simplify the sample application. In practice, you would probably not use the main URL to request upload data, nor would you immediately serve a blob you had just uploaded.

```
// file Upload.java

import java.io.IOException;
import java.util.List;
import java.util.Map;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import com.google.appengine.api.blobstore.BlobKey;
import com.google.appengine.api.blobstore.BlobstoreService;
import com.google.appengine.api.blobstore.BlobstoreServiceFactory;

public class Upload extends HttpServlet {
    private BlobstoreService blobstoreService = BlobstoreServiceFactory.getBlobstoreService();

    @Override
    public void doPost(HttpServletRequest req, HttpServletResponse res)
        throws ServletException, IOException {

        Map<String, List<BlobKey>> blobs = blobstoreService.getUploads(req);
        List<BlobKey> blobKeys = blobs.get("myFile");

        if (blobKeys == null || blobKeys.isEmpty()) {
            res.sendRedirect("/");
        } else {
            res.sendRedirect("/serve?blob-key=" + blobKeys.get(0).getKeyString());
        }
    }
}

// file Serve.java

import java.io.IOException;

import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import com.google.appengine.api.blobstore.BlobKey;
import com.google.appengine.api.blobstore.BlobstoreService;
import com.google.appengine.api.blobstore.BlobstoreServiceFactory;

public class Serve extends HttpServlet {
    private BlobstoreService blobstoreService = BlobstoreServiceFactory.getBlobstoreService();

    @Override
    public void doGet(HttpServletRequest req, HttpServletResponse res)
        throws IOException {
            BlobKey blobKey = new BlobKey(req.getParameter("blob-key"));
            blobstoreService.serve(blobKey, res);
        }
}


// file index.jsp

<%@ page import="com.google.appengine.api.blobstore.BlobstoreServiceFactory" %>
<%@ page import="com.google.appengine.api.blobstore.BlobstoreService" %>

<%
    BlobstoreService blobstoreService = BlobstoreServiceFactory.getBlobstoreService();
%>


<html>
    <head>
        <title>Upload Test</title>
    </head>
    <body>
        <form action="<%= blobstoreService.createUploadUrl("/upload") %>" method="post" enctype="multipart/form-data">
            <input type="text" name="foo">
            <input type="file" name="myFile">
            <input type="submit" value="Submit">
        </form>
    </body>
</html>

// web.xml

<?xml version="1.0" encoding="utf-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xmlns="http://java.sun.com/xml/ns/javaee"
   xmlns:web="http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
   xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
   http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd" version="2.5">

  <servlet>
    <servlet-name>Upload</servlet-name>
    <servlet-class>Upload</servlet-class>
  </servlet>
  
  <servlet>
    <servlet-name>Serve</servlet-name>
    <servlet-class>Serve</servlet-class>
  </servlet>
 
  <servlet-mapping>
    <servlet-name>Upload</servlet-name>
    <url-pattern>/upload</url-pattern>
  </servlet-mapping>

  <servlet-mapping>
    <servlet-name>Serve</servlet-name>
    <url-pattern>/serve</url-pattern>
  </servlet-mapping>
  
</web-app>
```

## Using the Images service with the Blobstore

The Images service can use a Blobstore value as the source of a transformation. The source image can be as large as the maximum size for a Blobstore value. The Images service still returns the transformed image to the application, so the transformed image must be smaller than 32 megabytes. This is useful for making thumbnail images of large photographs uploaded by users.

For information on using the Images service with Blobstore values, see the [Images Service documentation](https://web.archive.org/web/20160424225902/https://cloud.google.com/appengine/docs/java/images/overview).

## Using the Blobstore API with Google Cloud Storage

You can use the Blobstore API to store blobs in Cloud Storage instead of storing them in Blobstore. You need to set up a bucket as described in the [Google Cloud Storage](https://web.archive.org/web/20160424225902/https://cloud.google.com/appengine/docs/java/googlecloudstorageclient) documentation and specify the bucket and filename in the `BlobstoreService` [createUploadUrl](https://web.archive.org/web/20160424225902/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/blobstore/BlobstoreService#createUploadUrl(java.lang.String,%20com.google.appengine.api.blobstore.UploadOptions)), specify the bucket name in the `UploadOptions` parameter. In your upload handler, you need to process the returned [FileInfo](https://web.archive.org/web/20160424225902/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/blobstore/FileInfo) metadata and explicitly store the Google Cloud Storage filename needed to retrieve the blob later.

You can also serve Cloud Storage objects using the Blobstore API.

The following code snippets shows how to do this. This sample is in a request handler that gets the bucket name and object name in the request. It creates the Blobstore service and use it to create a blob key for Google Cloud Storage, using the supplied bucket and object name:

```
BlobstoreService blobstoreService = BlobstoreServiceFactory.getBlobstoreService();
BlobKey blobKey = blobstoreService.createGsBlobKey(
    "/gs/" + fileName.getBucketName() + "/" + fileName.getObjectName());
blobstoreService.serve(blobKey, resp);
```
**Note:** Once you obtain a `blobKey` for the Google Cloud Storage object, you can pass it around, serialize it, and otherwise use it interchangeably anywhere you can use a `blobKey` for objects stored in Blobstore. This allows for usage where an app stores some data in blobstore and some in Google Cloud Storage, but treats the data otherwise identically by the rest of the app. (However, `BlobInfo` objects are not available for Google Cloud Storage objects.)

## Quotas and limits

Space used for Blobstore values contributes to the **Stored Data (billable)** quota. Blob info entities in the datastore count towards datastore-related limits.

For more information on system-wide safety quotas, see [Quotas](https://web.archive.org/web/20160424225902/https://cloud.google.com/appengine/docs/quotas), and the **Quota Details** section of [the Cloud Platform Console](https://web.archive.org/web/20160424225902/https://cloud.google.com/appengine/docs/developers-console/#quotas).

In addition to system-wide safety quotas, a limit applies specifically to the use of the Blobstore: the maximum size of Blobstore data that can be read by the application with one API call is 32 megabytes.
