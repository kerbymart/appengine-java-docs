# Getting Started

  

Before running through the steps below, make sure that:

-   You have [activated](https://web.archive.org/web/20160424231044/https://cloud.google.com/appengine/docs/java/googlecloudstorageclient/using-cloud-storage#activating_a_cloud_storage_bucket) your project for Google Cloud Storage and App Engine, including creating at least one Cloud Storage bucket.
-   You have [downloaded](https://web.archive.org/web/20160424231044/https://cloud.google.com/appengine/docs/java/googlecloudstorageclient/using-cloud-storage#downloading_the_client_library) the client lib distribution and unzipped it.
-   You have [installed and configured](https://web.archive.org/web/20160424231044/https://cloud.google.com/appengine/docs/java/googlecloudstorageclient/using-cloud-storage#downloading_the_client_library) the latest App Engine Java SDK.

## Running the LocalExample.java Sample (Eclipse)

[LocalExample.java](https://web.archive.org/web/20160424231044/https://github.com/GoogleCloudPlatform/appengine-gcs-client/blob/master/java/example/src/com/google/appengine/demos/LocalExample.java) is a non-deployable sample useful for quick testing and investigation of the Cloud Storage functionality. It has no UI components other than Eclipse console output. ([Cloud Storage client library deployable samples](#deployable_gcs_client_library_samples_with_ui) with UI are also available.)

To run LocalExample.java in Eclipse:

1.  Start Eclipse.

2.  In Eclipse, click *Windows &gt; Preferences &gt; Google &gt; App Engine* and then click *Add*. (On Mac OS X: *Eclipse &gt; Preferences &gt; Google &gt; App Engine*.)

3.  Follow the prompts to supply the install path of the App Engine SDK, then click *OK*.

4.  In the *Files* menu, click *Files &gt; New &gt; Java Project* and create a project named `LocalExample` with a package name of `com.google.appengine.demos`.

5.  Select the project in the Package Explorer pane and click *Files &gt; New &gt; Class* and give the class the name `LocalExample` with a package name of `com.google.appengine.demos`.

6.  Copy the contents of the [LocalExampleJava source](https://web.archive.org/web/20160424231044/https://github.com/GoogleCloudPlatform/appengine-gcs-client/blob/master/java/example/src/com/google/appengine/demos/LocalExample.java) into that class file.

7.  Select the project again in the Package Explorer pane and right-click, then click *Properties &gt; Java Build Path*.

8.  In the *Libraries* tab, click *Add External Jars*. You must add the following JARs:
    -   `appengine-gcs-client.jar` from wherever you installed the Cloud Storage client library
    -   `guava-15.0.jar` from wherever you installed the Cloud Storage client library
    -   `joda-time-2.3.jar` from wherever you installed the Cloud Storage client library
    -   `appengine-testing.jar` from the App Engine install subdirectory `/lib/testing`.
    -   `appengine-api.jar` from the App Engine install subdirectory `/lib/impl`.
    -   `appengine-api-stubs.jar` from the App Engine install subdirectory `/lib/impl`.

9.  Build, then *Run As &gt; Java Application*.

10. You should see the following output in the Eclipse Console pane:

    ![](https://web.archive.org/web/20160424231044im_/https://cloud.google.com/appengine/docs/java/googlecloudstorageclient/images/local_example.png)

## Investigating the LocalExample.java Sample

[LocalExample.java](https://web.archive.org/web/20160424231044/https://github.com/GoogleCloudPlatform/appengine-gcs-client/blob/master/java/example/src/com/google/appengine/demos/LocalExample.java) is described in detail below, to illustrate the use of the Cloud Storage client library.

### Imports

There are a number of imports used that you might not need, or that are used to enable local testing. The followng snippet lists this sample's imports:

```
import com.google.appengine.tools.cloudstorage.GcsFileOptions;
import com.google.appengine.tools.cloudstorage.GcsFilename;
import com.google.appengine.tools.cloudstorage.GcsInputChannel;
import com.google.appengine.tools.cloudstorage.GcsOutputChannel;
import com.google.appengine.tools.cloudstorage.GcsService;
import com.google.appengine.tools.cloudstorage.GcsServiceFactory;
import com.google.appengine.tools.cloudstorage.RetryParams;
import com.google.appengine.tools.development.testing.LocalBlobstoreServiceTestConfig;
import com.google.appengine.tools.development.testing.LocalDatastoreServiceTestConfig;
import com.google.appengine.tools.development.testing.LocalServiceTestHelper;

import java.io.IOException;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.nio.ByteBuffer;
import java.nio.channels.Channels;
import java.util.Arrays;
import java.util.HashMap;
import java.util.Map;
```

These imports are briefly described below:

-   `com.google.appengine.tools.cloudstorage.*` this lets us use the Cloud Storage client library.
-   `com.google.appengine.tools.development.testing.*` Only needed for [local unit testing](https://web.archive.org/web/20160424231044/https://cloud.google.com/appengine/docs/java/tools/localunittesting#Java_Introducing_the_Java_testing_utilities) of certain App Engine features while running your app.
-   `java.io.ObjectInputStream` and `java.io.ObjectOutputStream` are used for reading and writing objects.
-   `java.nio.ByteBuffer` is used for unbuffered reads and writes.
-   Although not shown in the snippets,`java.io.IOException` is needed for error handling.
-   `java.nio.channels.Channels` is used to convert the input or output channel to a stream.
-   `java.nio.channels.ReadableByteChannel` is used for the data being read in from Cloud Storage.

### Creating a GcsService To Send Requests

To send and receive requests to Cloud Storage via this library, you need a [GcsService](https://web.archive.org/web/20160424231044/https://cloud.google.com/appengine/docs/java/googlecloudstorageclient/javadoc/com/google/appengine/tools/cloudstorage/GcsService.html) instance:

```
private final GcsService gcsService =
    GcsServiceFactory.createGcsService(RetryParams.getDefaultInstance());
```

In the snippet, notice the use of [RetryParams](https://web.archive.org/web/20160424231044/https://cloud.google.com/appengine/docs/java/googlecloudstorageclient/javadoc/com/google/appengine/tools/cloudstorage/RetryParams.html) in the call to `createGcsService`. Supplied as shown, `createGcsService(RetryParams.getDefaultInstance())`, this sets the default retry settings to be used for retries in the event of request failures due to timeouts or other unexpected failures in accessing Cloud Storage. To specify different values, such as for maximum number of retries, you use [RetryParams.Builder](https://web.archive.org/web/20160424231044/https://cloud.google.com/appengine/docs/java/googlecloudstorageclient/javadoc/com/google/appengine/tools/cloudstorage/RetryParams.Builder.html) to change settings in a new [RetryParams](https://web.archive.org/web/20160424231044/https://cloud.google.com/appengine/docs/java/googlecloudstorageclient/javadoc/com/google/appengine/tools/cloudstorage/RetryParams.html) object and supply it when you create `GcsService`. Note that once a [GcsService](https://web.archive.org/web/20160424231044/https://cloud.google.com/appengine/docs/java/googlecloudstorageclient/javadoc/com/google/appengine/tools/cloudstorage/GcsService.html) object is created, its retry parameters cannot be changed.

You can create as many `GcsService` instances as you need. Each instance is independent, immutable (and therefore threadsafe), and reusable. For example, one can be used to write a file with one set of parameters, and another can read a different file with different retry parameters at the same time.

A recommended practice is to use a separate (possibly static) instance in each of your classes that does I/O; this will help keep your classes independent and has very little overhead.

### Writing Data to Cloud Storage

The following samples show how to write data to a file stored in Cloud Storage. Separate snippets are provided for writing serialized object data and byte arrays.

Cloud Storage files are not fully created until `close` is invoked.

In the snippets below, `close` does not appear in a `finally` block, as is often done in Java. This is so that in the event of an exception, any partly written data will automatically be cleaned up.

#### Writing an Object to Cloud Storage

Here is how to serialize an object to a Cloud Storage file. First, get a writable byte channel:

```
GcsOutputChannel outputChannel =
    gcsService.createOrReplace(fileName, GcsFileOptions.getDefaultInstance());
```

In the snippet, `GcsService.createOrReplace` is invoked with the [GcsFilename](https://web.archive.org/web/20160424231044/https://cloud.google.com/appengine/docs/java/googlecloudstorageclient/javadoc/com/google/appengine/tools/cloudstorage/GcsFilename.html) as the first parameter. This object contains the name of the bucket to be used and name of the object. If there already is an object with that same name in that bucket, and your app has write permissions to it, this call will overwrite the existing file. (Cloud Storage does not support appending.) If there is no file of that name, this call results in the creation of a new file.

The second parameter shown is [GcsFileOptions](https://web.archive.org/web/20160424231044/https://cloud.google.com/appengine/docs/java/googlecloudstorageclient/javadoc/com/google/appengine/tools/cloudstorage/GcsFileOptions.html). To supply default options, use `GcsFileOptions.getDefaultInstance`. To supply your own settings, use [GcsFileOptions.Builder](https://web.archive.org/web/20160424231044/https://cloud.google.com/appengine/docs/java/googlecloudstorageclient/javadoc/com/google/appengine/tools/cloudstorage/GcsFileOptions.Builder) to set options and create a `GcsFileOptions` to supply as the second parameter.

The file options are passed through to Cloud Storage to tell it the content type of the file, any user metadata you want passed in the headers, the ACL to govern access to the file, and so forth. If you don't supply a content type (`mimeType`), the file will be served from Cloud Storage in the default MIME type, which is currently `binary/octet-stream`. If you don't specify an ACL, the access assigned to the object will be the current [default object ACL](https://web.archive.org/web/20160424231044/https://cloud.google.com/storage/docs/accesscontrol#default).

Note that all of the file options information can be obtained from an object after a `close` without having to download the object itself, by calling [GcsService.getMetadata(fileName)](https://web.archive.org/web/20160424231044/https://cloud.google.com/appengine/docs/java/googlecloudstorageclient/javadoc/com/google/appengine/tools/cloudstorage/GcsFileMetadata.html).

For more information on the above settings, visit the Cloud Storage documentation on [ACLs](https://web.archive.org/web/20160424231044/https://cloud.google.com/storage/docs/accesscontrol) and [file options](https://web.archive.org/web/20160424231044/https://cloud.google.com/storage/docs/reference-methods#putbucket).

Now write the data using an output stream:

```
@SuppressWarnings("resource")
ObjectOutputStream oout =
    new ObjectOutputStream(Channels.newOutputStream(outputChannel));
oout.writeObject(content);
oout.close();
```

#### Writing a Byte Array to a Cloud Storage File

Here is how to write a byte array to a Cloud Storage file:

```
@SuppressWarnings("resource")
GcsOutputChannel outputChannel =
    gcsService.createOrReplace(fileName, GcsFileOptions.getDefaultInstance());
outputChannel.write(ByteBuffer.wrap(content));
outputChannel.close();
```

For a description of the parameters used in the `createOrReplace` call, see the description under [Writing an Object to Cloud Storage](#writing_an_object_to_a_gcs_file).

### Reading Data from Cloud Storage

The following samples show how to read data from a file stored in Cloud Storage. Separate snippets are provided for reading the Cloud Storage file into an object (serialize) and into a byte array.

#### Reading Cloud Storage Data into an Object

This approach is useful for streaming larger files into a buffer:

```
GcsInputChannel readChannel = gcsService.openPrefetchingReadChannel(fileName, 0, 1024 * 1024);
```

The call to [GcsService.openPrefetchingReadChannel](https://web.archive.org/web/20160424231044/https://cloud.google.com/appengine/docs/java/googlecloudstorageclient/javadoc/com/google/appengine/tools/cloudstorage/GcsService.html#openPrefetchingReadChannel(com.google.appengine.tools.cloudstorage.GcsFilename,long,int)) takes a [GcsFilename](https://web.archive.org/web/20160424231044/https://cloud.google.com/appengine/docs/java/googlecloudstorageclient/javadoc/com/google/appengine/tools/cloudstorage/GcsFilename.html), which contains the name of the Cloud Storage bucket to be read from and name of the object to be read. The second parameter is the number of bytes from the start of the file, with 0 as shown causing the read to start at the beginning of the file. If you supply some other starting point within the file, say, at byte 300, the read operation begins at byte 300 and continues to the end of the file, or until you stop reading. (The prefetch buffer will read ahead, though, so it will contain bytes past the point where you stop.)

Prefetching provides is a major performance advantage for most applications, because it allows processing part of the file while more data is being downloaded in the background in parallel.

The third parameter is the size of the prefetch buffer, set in the example to 1 megabyte.

Now, read the file from the channel:

```
try (ObjectInputStream oin = new ObjectInputStream(Channels.newInputStream(readChannel))) {
  return oin.readObject();
}
```

#### Reading Data into a Byte Array

For smaller files, you could read the file all at once into a byte array:

```
int fileSize = (int) gcsService.getMetadata(fileName).getLength();
ByteBuffer result = ByteBuffer.allocate(fileSize);
try (GcsInputChannel readChannel = gcsService.openReadChannel(fileName, 0)) {
  readChannel.read(result);
}
```

In the snippet above, note the use of `java.nio.ByteBuffer`, especially the setting of the buffer size to be equal to the size of the file being read in from the channel.

Note, however, that for most applications, the prefered method is to stream the file (see [Reading Data into an Object](#reading_gcs_data_into_an_object)) because that doesn't require holding the all of the data in memory at once.

## Deployable Cloud Storage Client Library Samples with UI

For deployable samples with UI, visit the source code for:

-   [GcsExampleServlet.java](https://web.archive.org/web/20160424231044/https://github.com/GoogleCloudPlatform/appengine-gcs-client/blob/master/java/example/src/com/google/appengine/demos/GcsExampleServlet.java), which uploads/downloads files to/from Cloud Storage.

You'll need to build the samples before you can run them against the development server or deploy them. To build the samples:

1.  Checkout the source code by invoking the following command from a terminal window:

    ```
    svn checkout http://appengine-gcs-client.googlecode.com/svn/trunk/ appengine-gcs-client-read-only
    ```

2.  Change directory to `appengine-gcs-client-read-only/java`.

3.  Invoke `ant compile_example`, which will build the samples using the `build.xml` in that directory. For more information about using Apache Ant, see [Using Apache Ant](https://web.archive.org/web/20160424231044/https://cloud.google.com/appengine/docs/java/tools/ant).

4.  Run the dev server by invoking

    ```
    /path/to/AppEngSDK/dev_appserver.sh  /path/to/example/war
    ```

5.  In your browser, visit `localhost:8080`. For `GCSExampleServlet` you should see something like this:

    ![](https://web.archive.org/web/20160424231044im_/https://cloud.google.com/appengine/docs/java/googlecloudstorageclient/images/java_sample.png)
