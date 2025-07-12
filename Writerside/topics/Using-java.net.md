# Using java.net

  

You can use [java.net.URLConnection](https://web.archive.org/web/20160424230419/http://java.sun.com/javase/6/docs/api/java/net/URLConnection.html) from the Java standard library to make HTTP and HTTPS connections from your Java application. App Engine implements this interface using the URL Fetch service.

For more information on java.net, see [Oracle®'s java.net.URLConnection documentation](https://web.archive.org/web/20160424230419/http://java.sun.com/javase/6/docs/api/java/net/URLConnection.html).

1.  [Simple Requests With URLs](#Simple_Requests_With_URLs)
2.  [Using HttpURLConnection](#Using_HttpURLConnection)
3.  [Setting Request Headers](#Setting_Request_Headers)
4.  [Redirects](#Redirects)
5.  [java.net Features Not Supported](#Java_Net_Features_Not_Supported)
6.  [Features of the Low-level API](#Features_of_the_Low_level_API)

## Simple Requests With URLs

A simple way to get the contents of a page at a URL is to create a [java.net.URL](https://web.archive.org/web/20160424230419/http://java.sun.com/javase/6/docs/api/java/net/URL.html) object, then call its `openStream()` method. The method handles the details of creating the connection, issuing an HTTP GET request, and retrieving the response data.

```
import java.net.MalformedURLException;
import java.net.URL;
import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.io.IOException;

// ...
        try {
            URL url = new URL("http://www.example.com/atom.xml");
            BufferedReader reader = new BufferedReader(new InputStreamReader(url.openStream()));
            String line;

            while ((line = reader.readLine()) != null) {
                // ...
            }
            reader.close();

        } catch (MalformedURLException e) {
            // ...
        } catch (IOException e) {
            // ...
        }
```

## Using HttpURLConnection

For more sophisticated requests, you can call the URL object's `openConnection()` method to obtain a HttpURLConnection object. You can prepare this object with additional information before issuing the request. To make the request, you call a method on the URLConnection such as `getInputStream()` or `getOutputStream()`.

App Engine's implementation of URLConnection does not maintain a persistent connection with the remote host. When the app sets request data or writes to the output stream, the request data is kept in memory. When the app accesses any data about the response, such as getting the input stream (or calling the `connect()` method), App Engine calls the URL Fetch service with the request data, gets the response, closes the connection and returns the response data.

The following example makes an HTTP POST request to a URL with some form data:

```
import java.net.HttpURLConnection;
import java.net.MalformedURLException;
import java.net.URL;
import java.net.URLEncoder;
import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.io.IOException;
import java.io.OutputStreamWriter;

// ...
        String message = URLEncoder.encode("my message", "UTF-8");

        try {
            URL url = new URL("http://www.example.com/comment");
            HttpURLConnection connection = (HttpURLConnection) url.openConnection();
            connection.setDoOutput(true);
            connection.setRequestMethod("POST");

            OutputStreamWriter writer = new OutputStreamWriter(connection.getOutputStream());
            writer.write("message=" + message);
            writer.close();
    
            if (connection.getResponseCode() == HttpURLConnection.HTTP_OK) {
                // OK
            } else {
                // Server returned HTTP error code.
            }
        } catch (MalformedURLException e) {
            // ...
        } catch (IOException e) {
            // ...
        }
```

## Setting Request Headers

To set an HTTP header on the outgoing request, call the HttpURLConnection's `setRequestProperty()` method.

```
        connection.setRequestProperty("X-MyApp-Version", "2.7.3");
```

## Redirects

By default, HttpURLConnection will follow HTTP redirects. The URL Fetch service will follow up to 5 redirects.

To disable following redirects, call the HttpURLConnection's `setInstanceFollowRedirects()` method:

```
        connection.setInstanceFollowRedirects(false);
```

## java.net Features Not Supported

The URL Fetch service does not support persistent HTTP connections. When the app accesses response data using the URLConnection object, App Engine calls the URL Fetch service to complete the request. After the response data has been accessed, the request data cannot be modified.

The app cannot set explicit connection timeouts for the request.

## Features of the Low-level API

The URL Fetch service limits the size of the data for an outgoing request, and for an incoming response. When using the java.net API, data larger than the limit is silently truncated. The low-level URL Fetch API lets you specify whether truncation happens silently, or whether exceeding a limit throws an exception.
