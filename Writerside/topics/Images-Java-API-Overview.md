# Images Java API Overview

  

[Python](https://web.archive.org/web/20160424225706/https://cloud.google.com/appengine/docs/python/images/ "View this page in the Python runtime") <span style="color: #ddd; padding: 0em .5em;">|</span><span style="color: black; font-weight:bold">Java</span> <span style="color: #ddd; padding: 0em .5em;">|</span><span style="color:lightgray" title="This page is not available in the PHP runtime">PHP</span> <span style="color: #ddd; padding: 0em .5em;">|</span>[Go](https://web.archive.org/web/20160424225706/https://cloud.google.com/appengine/docs/go/images/ "View this page in the Go runtime")

[<img src="https://web.archive.org/web/20160424225706im_/https://cloud.google.com/cloud/images/stack_overflow_questions.png" title="Stack Overflow Questions" data-border="0" width="240" height="53" />](https://web.archive.org/web/20160424225706/http://stackoverflow.com/questions/tagged/google-app-engine+image)

[](https://web.archive.org/web/20160424225706/http://stackoverflow.com/feeds/tag?sort=votes&tagnames=google-app-engine%2Bimage)

<span class="link gc-analytics-event" category="Sidebar" data-action="Stack Overflow question click"></span>

<a href="https://web.archive.org/web/20160424225706/http://stackoverflow.com/questions/tagged/google-app-engine+image?sort=votes" class="gc-analytics-event" data-category="Sidebar" data-action="Stack Overflow See more link">See more...</a>

App Engine provides the ability to manipulate image data using a dedicated Images service. The Images service can resize, rotate, flip, and crop images; it can composite multiple images into a single image; and it can convert image data between several formats. It can also enhance photographs using a predefined algorithm. The API can also provide information about an image, such as its format, width, height, and a histogram of color values.

The Images service can accept image data directly from the app, or it can use a [Google Cloud Storage](https://web.archive.org/web/20160424225706/https://cloud.google.com/appengine/docs/java/googlestorage) value. (The Images service can also use a [Blobstore](https://web.archive.org/web/20160424225706/https://cloud.google.com/appengine/docs/java/blobstore) value, but we recommend the use of Cloud Storage instead.) When the source is Cloud Storage (or Blobstore), the size of the image to transform can be up to the maximum size of a Cloud Storage (or Blobstore) value. However, the transformed image is returned directly to the app, and so must be no larger than 32 megabytes. This is potentially useful for making thumbnail images of photographs uploaded to the Blobstore or Cloud Storage by users.

**Important:** If you serve images from Google Cloud Storage, you cannot serve an image from two separate apps. Only the first app that calls `getServingUrl` on the image can get the URL to serve it because that app has obtained ownership of the image. Any other app that subsequently calls `getServingUrl` on the image will therefore be unsuccessful. If a second app needs to serve the image, the app needs to first copy the image and then invoke `getServingUrl` on the copy.

1.  [Transforming images in Java](#Java_Transforming_images_in_Java)
2.  [Available image transformations](#Java_Available_image_transformations)
3.  [Image formats](#Java_Image_formats)
4.  [Transforming images from the Blobstore](#Java_Transforming_images_from_the_Blobstore)
5.  [Images and the development server](#Java_Images_and_the_development_server)
6.  [A note about deletion](#Java_A_note_about_deletion)
7.  [Quotas and limits](#Java_Quotas_and_limits)

## Transforming images in Java

The Image service Java API lets you apply transformations to images, using a service instead of performing image processing on the application server. The app prepares an `Image` object with the image data to transform, and a `Transform` object with instructions on how to transform the image. The app gets an `ImagesService` object, then calls its `applyTransform()` method with the `Image` and the `Transform` objects. The method returns an `Image` object of the transformed image.

The app gets `ImagesService`, `Image` and `Transform` instances using the `ImagesServiceFactory`.

```
import com.google.appengine.api.images.Image;
import com.google.appengine.api.images.ImagesService;
import com.google.appengine.api.images.ImagesServiceFactory;
import com.google.appengine.api.images.Transform;

// ...
        byte[] oldImageData;  // ...

        ImagesService imagesService = ImagesServiceFactory.getImagesService();

        Image oldImage = ImagesServiceFactory.makeImage(oldImageData);
        Transform resize = ImagesServiceFactory.makeResize(200, 300);

        Image newImage = imagesService.applyTransform(resize, oldImage);

        byte[] newImageData = newImage.getImageData();
```

Multiple transforms can be combined into a single action using a `CompositeTransform` instance. See [the images API reference](https://web.archive.org/web/20160424225706/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/images/package-summary).

## Available image transformations

The Images service can resize, rotate, flip, and crop images, and enhance photographs. It can also composite multiple images into a single image.

#### Resize

You can resize the image while maintaining the same aspect ratio. Neither the width nor the height of the resized image can exceed 4000 pixels.

![](https://web.archive.org/web/20160424225706im_/https://cloud.google.com/appengine/docs/python/images/transform_before.jpg) ![](https://web.archive.org/web/20160424225706im_/https://cloud.google.com/appengine/docs/python/images/transform_resize_after.jpg)

#### Rotate

You can rotate the image in 90 degree increments.

![](https://web.archive.org/web/20160424225706im_/https://cloud.google.com/appengine/docs/python/images/transform_before.jpg) ![](https://web.archive.org/web/20160424225706im_/https://cloud.google.com/appengine/docs/python/images/transform_rotate_after.jpg)

#### Flip horizontally

You can flip the image horizontally.

![](https://web.archive.org/web/20160424225706im_/https://cloud.google.com/appengine/docs/python/images/transform_before.jpg) ![](https://web.archive.org/web/20160424225706im_/https://cloud.google.com/appengine/docs/python/images/transform_fliph_after.jpg)

#### Flip vertically

You can flip the image vertically.

![](https://web.archive.org/web/20160424225706im_/https://cloud.google.com/appengine/docs/python/images/transform_before.jpg) ![](https://web.archive.org/web/20160424225706im_/https://cloud.google.com/appengine/docs/python/images/transform_flipv_after.jpg)

#### Crop

You can crop the image with a given bounding box.

![](https://web.archive.org/web/20160424225706im_/https://cloud.google.com/appengine/docs/python/images/transform_before.jpg) ![](https://web.archive.org/web/20160424225706im_/https://cloud.google.com/appengine/docs/python/images/transform_crop_after.png)

#### I'm Feeling Lucky

The "I'm Feeling Lucky" transform enhances dark and bright colors in an image and adjusts both color and contrast to optimal levels.

![](https://web.archive.org/web/20160424225706im_/https://cloud.google.com/appengine/docs/python/images/transform_before.jpg) ![](https://web.archive.org/web/20160424225706im_/https://cloud.google.com/appengine/docs/python/images/transform_lucky_after.png)

## Image formats

The service accepts image data in the JPEG, PNG, WEBP, GIF (including animated GIF), BMP, TIFF and ICO formats.

It can return transformed images in the JPEG, WEBP and PNG formats. If the input format and the output format are different, the service converts the input data to the output format before performing the transformation.

**Note:** The Images service does not support TIFF images that have multiple layers.

## Transforming images from the Blobstore

The Images service can use a value from the [Blobstore](https://web.archive.org/web/20160424225706/https://cloud.google.com/appengine/docs/java/blobstore) as the source for a transformation. You have two ways to transform images from the Blobstore:

1.  Using the [ImageServiceFactory()](https://web.archive.org/web/20160424225706/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/images/ImagesServiceFactory) class allows you to perform simple image transformations, such as crop, flip, and rotate.
2.  Using [getServingUrl()](https://web.archive.org/web/20160424225706/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/images/ImagesService#getServingUrl(com.google.appengine.api.blobstore.BlobKey,%20int,%20boolean)) allows you to dynamically resize and crop images, so you don't need to store different image sizes on the server. This method returns a URL that serves the image, and transformations to the image are encoded in this URL.

### Using the ImageServiceFactory() Class

You can transform images from the Blobstore as long as the image size is smaller than the maximum Blobstore value size. Note, however, that the result of the transformation is returned directly to the app, and must therefore not exceed the API response limit of 32 megabytes. You can use this to make thumbnail images of photographs uploaded by users.

To transform an image from the Blobstore in Java, you create the `Image` object by calling the static method `ImageServiceFactory.makeImageFromBlob()`, passing it a `blobstore.BlobKey` value. The rest of the API behaves as expected. The `applyTransform()` method returns the result of the transforms, or throws an `ImageServiceFailureException` if the result is larger than the maximum size of 32 megabytes.

```
import com.google.appengine.api.images.Image;
import com.google.appengine.api.images.ImagesService;
import com.google.appengine.api.images.ImagesServiceFactory;
import com.google.appengine.api.images.Transform;

// ...
        BlobKey blobKey;  // ...

        ImagesService imagesService = ImagesServiceFactory.getImagesService();

        Image oldImage = ImagesServiceFactory.makeImageFromBlob(blobKey);
        Transform resize = ImagesServiceFactory.makeResize(200, 300);

        Image newImage = imagesService.applyTransform(resize, oldImage);

        byte[] newImageData = newImage.getImageData();
```

### Using getServingUrl()

The [getServingUrl()](https://web.archive.org/web/20160424225706/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/images/ImagesService#getServingUrl(com.google.appengine.api.blobstore.BlobKey,%20int,%20boolean)) method allows you to generate a fixed, dedicated URL for an image that is stored in Blobstore.

The generated URL uses highly-optimized image serving infrastructure that is separate from your application. Because it's served separately the serving URL does not incure any dynamic load on your application which can be very cost effective. The URL returned by this method is always publicly accessable but not guessable.

If you wish to stop serving the URL, delete it using the [deleteServingUrl()](https://web.archive.org/web/20160424225706/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/images/ImagesService#deleteServingUrl(com.google.appengine.api.blobstore.BlobKey)) method.

The method returns a URL encoded with the specified size and crop arguments. If you do not specify any arguments, the method returns the default URL for the image, for example:

```
http://lhx.ggpht.com/randomStringImageId
```

You can resize and crop the image dynamically by specifying the arguments in the URL. The available arguments are:

-   `=sxx` where `xx` is an integer from 0–1600 representing the length, in pixels, of the image's longest side. For example, adding `=s32` resizes the image so its longest dimension is 32 pixels.
-   `=sxx-c` where **xx** is an integer from 0–1600 representing the cropped image size in pixels, and `-c` tells the system to crop the image.

```
# Resize the image to 32 pixels (aspect-ratio preserved)
http://lhx.ggpht.com/randomStringImageId=s32

# Crop the image to 32 pixels
http://lhx.ggpht.com/randomStringImageId=s32-c
```

## Images and the development server

The development server uses your local machine to perform the capabilities of the Images service.

The Java development server uses the [ImageIO framework](https://web.archive.org/web/20160424225706/http://download.oracle.com/javase/1.4.2/docs/api/javax/imageio/ImageIO.html) to simulate the Image service. The "I'm Feeling Lucky" photo enhancement feature is not supported. The WEBP image format is only supported if a suitable decoder plugin has been installed. The [Java VP8 decoder](https://web.archive.org/web/20160424225706/http://sourceforge.net/projects/javavp8decoder/) plugin can be used, for example.

## A note about deletion

Whether you store your images in Cloud Storage or Blobstore, the right way to stop an image from being publicly accessible through the serving URL is to call the [deleteServingUrl()](https://web.archive.org/web/20160424225706/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/images/ImagesService#deleteServingUrl(com.google.appengine.api.blobstore.BlobKey)) method.

If you merely delete the underlying stored image from Cloud Storage or Blobstore, under some circumstances the image may remain accessible through the serving URL.

## Quotas, limits, and pricing

There is currently no additional charge incurred by using the Images API. See the App Engine [pricing page](https://web.archive.org/web/20160424225706/https://cloud.google.com/appengine/pricing#Billable_Resource_Unit_Costs).

Each Images service request counts toward the **Image Manipulation API Calls** quota. An app can perform multiple transformations of an image in a single API call.

Data sent to the Images service counts toward the **Data Sent to (Images) API** quota. Data received from the Images service counts toward the **Data Received from (Images) API** quota.

Each transformation of an image counts toward the **Transformations Executed** quota.

For more information, see [Quotas](https://web.archive.org/web/20160424225706/https://cloud.google.com/appengine/docs/quotas) and [Quota Details](https://web.archive.org/web/20160424225706/https://cloud.google.com/appengine/docs/developers-console/#quotas). You can see the current quota usage of your app by visiting the [Google Cloud Platform Console Quota Details](https://web.archive.org/web/20160424225706/https://console.cloud.google.com//project/_/appengine/quotadetails) tab.

In addition to quotas, the following limits apply to the use of the Images service:

<table>
<thead>
<tr class="header">
<th>Limit</th>
<th>Amount</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>maximum data size of image sent to service</td>
<td>32 megabytes</td>
</tr>
<tr class="even">
<td>maximum data size of image received from service</td>
<td>32 megabytes</td>
</tr>
<tr class="odd">
<td>maximum size of image sent or received from service</td>
<td>50 megapixels</td>
</tr>
</tbody>
</table>
