# Capabilities Java API Overview

  

[Python](https://web.archive.org/web/20160424231040/https://cloud.google.com/appengine/docs/python/capabilities/ "View this page in the Python runtime") <span style="color: #ddd; padding: 0em .5em;">|</span><span style="color: black; font-weight:bold">Java</span> <span style="color: #ddd; padding: 0em .5em;">|</span><span style="color:lightgray" title="This page is not available in the PHP runtime">PHP</span> <span style="color: #ddd; padding: 0em .5em;">|</span>[Go](https://web.archive.org/web/20160424231040/https://cloud.google.com/appengine/docs/go/capabilities/ "View this page in the Go runtime")

With the Capabilities API, your application can detect outages and scheduled downtime for specific [API capabilities](#Java_Supported_capabilities). You can use this API to reduce downtime in your application by detecting when a capability is unavailable and then bypassing it.

For example, if you use the Images API to resize images, you can use the Capabilities API to detect when the Images API is unavailable and skip the resize:

```
import com.google.appengine.api.capabilities.*;

CapabilitiesService service =
    CapabilitiesServiceFactory.getCapabilitiesService();
CapabilityStatus status = service.getStatus(Capability.IMAGES).getStatus();

if (status == CapabilityStatus.DISABLED) {
    // Images API is not available.
}
```

You can separately query for the availability of Datastore reads and writes. The following sample shows how to detect the availability of Datastore writes and, during downtime, provide a message to users:

```
CapabilityStatus status =
    service.getStatus(Capability.DATASTORE_WRITE).getStatus();

if (status == CapabilityStatus.DISABLED) {
    // Datastore is in read-only mode.
}
```

## Using the Capabilities API in Java

Each Capability is represented as a static constant on the Capability class (such as [`Capability.DATASTORE_WRITE`](https://web.archive.org/web/20160424231040/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/capabilities/Capability#DATASTORE_WRITE)). Each Capability has a state, which you can retrieve from [`CapabilitiesService.getStatus(Capability)`](https://web.archive.org/web/20160424231040/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/capabilities/CapabilitiesService#getStatus(com.google.appengine.api.capabilities.Capability)). Each state has a status, which is an enumeration reflecting a the availability of a capability: either `ENABLED` or `DISABLED`. See below for the [list of services](#Java_Supported_capabilities) currently enabled in this API.

## Supported capabilities

The API currently supports the following capabilities:

<table>
<thead>
<tr class="header">
<th>Capability</th>
<th>Arguments to <code>getStatus</code></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Availability of the blobstore</td>
<td><a href="https://web.archive.org/web/20160424231040/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/capabilities/Capability#BLOBSTORE">Capability.BLOBSTORE</a></td>
</tr>
<tr class="even">
<td>Datastore reads</td>
<td><a href="https://web.archive.org/web/20160424231040/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/capabilities/Capability#DATASTORE">Capability.DATASTORE</a></td>
</tr>
<tr class="odd">
<td>Datastore writes</td>
<td><a href="https://web.archive.org/web/20160424231040/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/capabilities/Capability#DATASTORE_WRITE">Capability.DATASTORE_WRITE</a></td>
</tr>
<tr class="even">
<td>Availability of the Images service</td>
<td><a href="https://web.archive.org/web/20160424231040/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/capabilities/Capability#IMAGES">Capability.IMAGES</a></td>
</tr>
<tr class="odd">
<td>Availability of the Mail service</td>
<td><a href="https://web.archive.org/web/20160424231040/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/capabilities/Capability.html#MAIL">Capability.MAIL</a></td>
</tr>
<tr class="even">
<td>Availability of the Memcache service</td>
<td><a href="https://web.archive.org/web/20160424231040/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/capabilities/Capability.html#MEMCACHE">Capability.MEMCACHE</a></td>
</tr>
<tr class="odd">
<td>Availability of the Task Queue service</td>
<td><a href="https://web.archive.org/web/20160424231040/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/capabilities/Capability.html#TASKQUEUE">Capability.TASKQUEUE</a></td>
</tr>
<tr class="even">
<td>Availability of the URL Fetch service</td>
<td><a href="https://web.archive.org/web/20160424231040/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/capabilities/Capability#URL_FETCH">Capability.URL_FETCH</a></td>
</tr>
<tr class="odd">
<td>Availability of the XMPP service</td>
<td><a href="https://web.archive.org/web/20160424231040/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/capabilities/Capability.html#XMPP">Capability.XMPP</a></td>
</tr>
</tbody>
</table>
