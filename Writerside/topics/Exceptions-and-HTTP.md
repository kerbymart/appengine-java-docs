# Exceptions and HTTP

  

In many situations, you may want to use common HTTP status codes to indicate the success or failure of a user's API request. For example, if a user is attempting to retrieve an entity which does not exist, you may want to send an HTTP 404 status code saying No entity exists with ID: `entityId`.

You can send such common HTTP status codes by throwing an exception provided by the endpoints library as follows:

```
String message = "No entity exists with ID: " + entityId;
throw new NotFoundException(message);
```

## Exceptions Provided by Endpoints

The endpoints library provides the following exceptions, corresponding to specific HTTP status codes:

<table>
<thead>
<tr class="header">
<th>Exception</th>
<th>Corresponding HTTP Status Code</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>com.google.api.server.spi.response.BadRequestException</code></td>
<td>HTTP 400</td>
</tr>
<tr class="even">
<td><code>com.google.api.server.spi.response.UnauthorizedException</code></td>
<td>HTTP 401</td>
</tr>
<tr class="odd">
<td><code>com.google.api.server.spi.response.ForbiddenException</code></td>
<td>HTTP 403</td>
</tr>
<tr class="even">
<td><code>com.google.api.server.spi.response.NotFoundException</code></td>
<td>HTTP 404</td>
</tr>
<tr class="odd">
<td><code>com.google.api.server.spi.response.ConflictException</code></td>
<td>HTTP 409</td>
</tr>
<tr class="even">
<td><code>com.google.api.server.spi.response.InternalServerErrorException</code></td>
<td>HTTP 500</td>
</tr>
<tr class="odd">
<td><code>com.google.api.server.spi.response.ServiceUnavailableException</code></td>
<td>HTTP 503</td>
</tr>
</tbody>
</table>

## Supported HTTP Status Codes

Endpoints supports a subset of HTTP status codes in API responses. The following table describes the supported codes.

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>HTTP Status Codes</th>
<th>Support</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>HTTP 2xx</td>
<td>HTTP 200 is typically assumed by Endpoints if the API method returns successfully.<br />
If the API method response type is <code>void</code> or the return value of the API method is <code>null</code> , HTTP 204 will be set instead.<br />
<em>HTTP 2xx codes should not be used in custom exception classes</em>.</td>
</tr>
<tr class="even">
<td>HTTP 3xx</td>
<td>HTTP 3xx codes are not supported. Use of any HTTP 3xx codes will result in an HTTP 404 response.</td>
</tr>
<tr class="odd">
<td>HTTP 4xx</td>
<td>Only the HTTP 4xx codes listed below are supported:
<ul>
<li>400</li>
<li>401</li>
<li>403</li>
<li>404</li>
<li>409</li>
<li>410</li>
<li>412</li>
<li>413</li>
</ul>
Any other HTTP 4xx codes will be returned as error 404, except for the following:
<ul>
<li>405 is returned as 501</li>
<li>408 is returned as 503</li>
</ul></td>
</tr>
<tr class="even">
<td>HTTP 5xx</td>
<td>All HTTP 5xx status codes are converted to be HTTP 503 in the client response.</td>
</tr>
</tbody>
</table>

## Creating Your Own Exception Classes

If you want to create other exception classes for other HTTP status codes, you can do so by subclassing `com.google.api.server.spi.ServiceException`. The following snippet shows how to create an exception class that represents an HTTP 408 status code:

```
import com.google.api.server.spi.ServiceException;

public class RequestTimeoutException extends ServiceException {
    public RequestTimeoutException(String message) {
        super(408, message);
    }
}
```
**Important:** An uncaught exception in your application will result in an HTTP 503 error from your Endpoints API, unless it extends `com.google.api.server.spi.ServiceException`.
