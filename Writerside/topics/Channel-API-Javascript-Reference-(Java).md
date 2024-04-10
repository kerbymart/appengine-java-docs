# Channel API Javascript Reference (Java)

  

Use the following reference when building JavaScript clients for the channel API.

## Building your JavaScript client

Include the following in your html page before any JavaScript code that refers to it:

```
<script type="text/javascript" src="/_ah/channel/jsapi"></script>
```

All other JavaScript code must be between the `<body>` tags of the page. For best performance, place your JavaScript code at the bottom of your HTML markup just before the `</body>`.

## goog.appengine.Channel() Class

### Constructors

##### `goog.appengine.Channel(token)`

Create a channel object using the token returned by the [`createChannel()`](https://web.archive.org/web/20160424230525/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/channel/ChannelService#createChannel(java.lang.String)) call on the server.

### Methods

##### `open(optional_handler)`

Open a socket on this channel. `open()` returns a `goog.appengine.Socket` object. You can set the callback properties directly on the returned socket object or set them using an optional object handler with the following properties:

-   `onopen`
-   `onmessage`
-   `onerror`
-   `onclose`

If the token specified during channel creation is invalid or expired then the `onerror` and `onclose` callbacks will be called. The `code` field for the error object will be 401 (Unauthorized) and the `description` field will be `'Invalid+token.'` or `'Token+timed+out.'` respectively. The `onerror` callback is also called asynchronously whenever the token for the channel expires. An `onerror` call is always followed by an `onclose` call and the channel object will have to be recreated after this event.

## goog.appengine.Socket() Class

### Methods

##### `close()`

Close the socket. The socket cannot be used again after calling close; the server must create a new socket.

### Properties

##### `onopen`

Set this to a function called when the socket is ready to receive messages.

##### `onmessage`

Set this to a function called when the socket receives a message. The function is passed one parameter: a message object. The `data` field of this object is the string passed to the `send_message` method on the server.

##### `onerror`

Set this property to a function called when an error occurs on the socket. The function is passed one parameter: an error object. The `description` field is a description of the error and the `code` field is an HTTP error code indicating the error.

##### `onclose`

Set this property to a function that called when the socket is closed. When the socket is closed, it cannot be reopened. Use the `open()` method on a `goog.appengine.Channel` object to create a new socket.
