# Understanding Storage Features

  

[<img src="https://web.archive.org/web/20160424231015im_/https://cloud.google.com/cloud/images/stack_overflow_questions.png" title="Stack Overflow Questions" data-border="0" width="240" height="53" />](https://web.archive.org/web/20160424231015/http://stackoverflow.com/questions/tagged/google-app-engine+google-cloud-storage)

[](https://web.archive.org/web/20160424231015/http://stackoverflow.com/feeds/tag?sort=votes&tagnames=google-app-engine%2Bgoogle-cloud-storage)

<span class="link gc-analytics-event" category="Sidebar" data-action="Stack Overflow question click"></span>

<a href="https://web.archive.org/web/20160424231015/http://stackoverflow.com/questions/tagged/google-app-engine+google-cloud-storage?sort=votes" class="gc-analytics-event" data-category="Sidebar" data-action="Stack Overflow See more link">See more...</a>

## Contents

1.  [Buckets, objects, and ACLs](#buckets_objects_and_acls)
2.  [ACLs and the client library](#acls_and_the_client_library)
3.  [Modifying Cloud Storage objects](#modifying_cloud_storage_objects)
4.  [Cloud Storage and subdirectories](#cloud_storage_and_subdirectories)

## Buckets, objects, and ACLs

A bucket is the storage location you read files from and write files to. You must always specify a bucket when using the Google Cloud Storage client library. Your project can access multiple buckets. Note that the client library doesn't support [bucket creation](https://web.archive.org/web/20160424231015/https://cloud.google.com/appengine/docs/java/googlecloudstorageclient/using-cloud-storage#activating_a_cloud_storage_bucket).

Access control lists (ACLs) control access to the buckets and to the objects contained in them. Your project and your App Engine app are automatically added to the ACL that permits bucket access when your create a bucket in your project.

Note that the ACL that permits bucket access is distinct from the potentially many ACLs governing the objects in that bucket. Thus, your app has read and write privileges to the bucket(s) it is activated for, but it only has full rights to the objects it creates in the bucket. Your app's access to objects created by other apps or persons is limited to the rights given to your app by the objects' creator.

If an object is created in the bucket without an ACL explicitly defined for it, it uses the default object ACL assigned to the bucket by the bucket owner. If the bucket owner has not specified a default object ACL, the object default is `public-read`, which means that anyone allowed bucket access can read the object.

## ACLs and the client library

An app using the client library cannot change the bucket ACL, but it can specify an ACL that controls access to the objects it creates.

The available ACL settings are described under documentation for the [`GcsService.FcsFileOptions` object](https://web.archive.org/web/20160424231015/https://cloud.google.com/appengine/docs/java/googlecloudstorageclient/javadoc/com/google/appengine/tools/cloudstorage/GcsFileOptions.html) .

## Modifying Cloud Storage objects

Once you create an object in a bucket, you cannot modify or append to it. Instead, you must overwrite the object with a new object of the same name that contains your desired changes.

## Cloud Storage and subdirectories

The Cloud Storage client library lets you supply subdirectory delimiters when you create an object, but there are no true subdirectories in Cloud Storage. Instead, a subdirectory in Cloud Storage is a part of the object filename.

For example, you might assume that creating an object `somewhere/over/the/rainbow.mp3` would store the file `rainbow.mp3` in the subdirectory `somewhere/over/the/`. Instead, the object name is set to `somewhere/over/the/rainbow.mp3`.

**Note:** The Google Cloud Storage client library provides a configurable mechanism for automatic request retries in event of timeout failures when accessing Cloud Storage. The same mechanism also provides [exponential backoff](https://web.archive.org/web/20160424231015/https://cloud.google.com/storage/docs/exponential-backoff) to determine an optimal processing rate.  
  
To change the default values for retries and backoff, use the [RetryParams](https://web.archive.org/web/20160424231015/https://cloud.google.com/appengine/docs/java/googlecloudstorageclient/javadoc/com/google/appengine/tools/cloudstorage/RetryParams) class.
