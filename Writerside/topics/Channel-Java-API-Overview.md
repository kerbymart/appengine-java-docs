# Channel Java API Overview

  

[Python](https://web.archive.org/web/20160425100002/https://cloud.google.com/appengine/docs/python/channel/ "View this page in the Python runtime") <span style="color: #ddd; padding: 0em .5em;">|</span><span style="color: black; font-weight:bold">Java</span> <span style="color: #ddd; padding: 0em .5em;">|</span><span style="color:lightgray" title="This page is not available in the PHP runtime">PHP</span> <span style="color: #ddd; padding: 0em .5em;">|</span>[Go](https://web.archive.org/web/20160425100002/https://cloud.google.com/appengine/docs/go/channel/ "View this page in the Go runtime")

[<img src="https://web.archive.org/web/20160425100002im_/https://cloud.google.com/cloud/images/stack_overflow_questions.png" title="Stack Overflow Questions" data-border="0" width="240" height="53" />](https://web.archive.org/web/20160425100002/http://stackoverflow.com/questions/tagged/google-app-engine+channel)

[](https://web.archive.org/web/20160425100002/http://stackoverflow.com/feeds/tag?sort=votes&tagnames=google-app-engine%2Bchannel)

<span class="link gc-analytics-event" category="Sidebar" data-action="Stack Overflow question click"></span>

<a href="https://web.archive.org/web/20160425100002/http://stackoverflow.com/questions/tagged/google-app-engine+channel?sort=votes" class="gc-analytics-event" data-category="Sidebar" data-action="Stack Overflow See more link">See more...</a>

1.  [Overview](#Java_Overview)
2.  [Elements of the Channel API](#Java_Elements_of_the_Channel_API)
3.  [Life of a typical channel message](#Java_Life_of_a_typical_channel_message)
4.  [Sending messages to different app versions](#Java_Sending_messages_to_different_app_versions)
5.  [Example Tic Tac Toe application](#Java_Example_Tic_Tac_Toe_application)
6.  [Tracking client connections and disconnections](#Java_Tracking_client_connections_and_disconnections)
7.  [Tokens and security](#Java_Tokens_and_security)
8.  [Caveats](#Java_Caveats)

## Overview

The Channel API creates a persistent connection between your application and Google servers, allowing your application to send messages to JavaScript clients in real time without the use of polling. This is useful for applications designed to update users about new information immediately. Some example use-cases include collaborative applications, multi-player games, or chat rooms. In general, using the Channel API is a better choice than polling in situations where updates can't be predicted or scripted, such as when relaying information between human users or from events not generated systematically.

## Elements of the Channel API

### Javascript client

The user interacts with a JavaScript client built into a webpage. The JavaScript client is primarily responsible for three things:

-   Connecting to the channel once it receives the channel’s unique token from the server
-   Listening on the channel for updates regarding other clients and making appropriate use of the data (e.g. updating the interface, etc.)
-   Sending update messages to the server so they may be passed on to remote clients

Refer to the [JavaScript Reference](https://web.archive.org/web/20160425100002/https://cloud.google.com/appengine/docs/java/channel/javascript) page for details on building your client.

### The server

The server is responsible for:

-   Creating a unique channel for individual JavaScript clients
-   Creating and sending a unique token to each JavaScript client so they may connect and listen to their channel
-   Receiving update messages from clients via HTTP requests
-   Sending update messages to clients via their channels
-   Optionally, managing client connection state.

### The client ID

The Client ID is responsible for identifying individual JavaScript clients on the server. The server knows what channel on which to send a particular message because of the Client ID.

A Client ID can be anything that makes sense in the design of your application. For example, you can use something like cookie or login information, randomized numerical ID, or a user-selected name.

You may also create Client IDs in whatever way makes sense in your application. For example, you may choose to create the Client ID on the client and pass it to the server in an explicit request for a token, or create it on the server and inject it into the page’s HTML when the server replies to the browser’s request for the page.

### Tokens

Tokens are responsible for allowing the JavaScript Client to connect and listen to the channel created for it. The server creates one token for each client using information such as the client’s Client ID and expiration time.

Tokens expire after two hours and should also be treated as secret. See the [Tokens and Security](#Java_Tokens_and_security) section for more details.

### The channel

A channel is a one-way communication path through which the server sends updates to a specific JavaScript client identified by its Client ID. The server receives updates from clients via HTTP requests, then sends the messages to relevant clients via their channels.

### The message

Messages are sent via HTTP requests from one client to the server. Once received, the server passes the message to the designated client via the correct channel identified by the Client ID. Messages are limited to 32K.

**Warning:** Avoid sending raw messages, especially when sending URLs. Instead, use [JSON](https://web.archive.org/web/20160425100002/https://en.wikipedia.org/wiki/JSON) encoding to ensure that messages arrive intact.

### Socket

The JavaScript client opens a socket using the token provided by the server. It uses the socket to listen for updates on the channel.

### Presence

The server can register to receive a notification when a client connects to or disconnects from a channel.

## Life of a typical channel message

These two diagrams illustrate the life of a typical example message sent via Channel API between two different clients using one possible implementation of Channel API.

![](https://web.archive.org/web/20160425100002im_/https://cloud.google.com/appengine/images/channel_overview01.png)

This diagram shows the creation of a channel on the server. In this example, it shows the JavaScript client explicitly requests a token and sends its Client ID to the server. In contrast, you could choose to design your application to inject the token into the client before the page loads in the browser, or some other implementation if preferred.

Next, the server uses Client A’s Client ID to create a channel and then sends the token for that channel back to Client A. Client A uses the token to open a socket and listen for updates on the channel.

![](https://web.archive.org/web/20160425100002im_/https://cloud.google.com/appengine/images/channel_overview02.png)

This diagram shows Client B sending a message using `POST` to the server. The server processes the message and sends it to Client A over the channel. Client A receives the message and makes use of the new information.

## Sending messages to different app versions

The [`ChannelMessage`](https://web.archive.org/web/20160425100002/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/channel/package-summary) class can be initialized with a token passed as the `clientId` parameter. This allows applications to send the message object from different versions of the app. For instance you could create the channel on the front end and then send messages from a backend of the app.

## Example Tic Tac Toe application

To better illustrate how to use Channel API, take a look at the following example Tic Tac Toe game application written in Java. The game allows users to create a game, invite another player by sending out a URL, and play the game together in real time. The application updates both players' views of the board in real time as soon as the other player makes a move.

The full source code of the application is available at the [Java Channel Tac Toe Project page](https://web.archive.org/web/20160425100002/http://code.google.com/p/java-channel-tic-tac-toe). We'll go over major highlights that illustrate Channel API below.

### Creating and connecting to a channel

When a user visits the Tic Tac Toe game for the first time, two things happen:

-   The game servlet injects a token into the html page sent to the client. The client uses this token to open a socket and listen for updates on the channel.
-   The game servlet provides the user with a URL they can share with a friend in order to invite him or her to join the game.

In order to initiate the process of creating a channel, Java applications call the [`ChannelServiceFactory.getChannelService()`](https://web.archive.org/web/20160425100002/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/channel/ChannelServiceFactory#getChannelService()) method and get a [`ChannelService()`](https://web.archive.org/web/20160425100002/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/channel/ChannelService) object. Then they use the object's [`createChannel()`](https://web.archive.org/web/20160425100002/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/channel/ChannelService#createChannel(java.lang.String)) method and get a token that the client page can use to connect to the channel. When calling `createChannel()`, they send a Client ID that uniquely identifies the client to the server.

```
public class TicTacToeServlet extends HttpServlet {
  @Override
  public void doGet(HttpServletRequest req, HttpServletResponse resp) throws IOException {
    // Game creation, user sign-in, etc. omitted for brevity.
    String userId = userService.getCurrentUser().getUserId();

    ChannelService channelService = ChannelServiceFactory.getChannelService();

    // The 'Game' object exposes a method which creates a unique string based on the game's key
    // and the user's id.
    String token = channelService.createChannel(game.getChannelKey(userId));

    // Index is the contents of our index.html resource, details omitted for brevity.
    index = index.replaceAll("\\{\\{ token \\}\\}", token);

    resp.setContentType("text/html");
    resp.getWriter().write(index);
  }
}
```

The server returns the token to the client, which injects it into the index.html resource template and creates a new `goog.appengine.Channel()` object.

```
<body>
  <script>
    channel = new goog.appengine.Channel('{{ token }}');
    socket = channel.open();
    socket.onopen = onOpened;
    socket.onmessage = onMessage;
    socket.onerror = onError;
    socket.onclose = onClose;
  </script>
</body>
```

The game client uses the Channel object's `open()` method to create a socket. The client also sets callback functions on the socket to be called when the state of the socket changes.

### Opening the socket

In our example, when the Tic Tac Toe client is ready to receive messages, it calls the `onOpened()` function, which it set to the socket's [`onopen`](https://web.archive.org/web/20160425100002/https://cloud.google.com/appengine/docs/java/channel/javascript#onopen) callback. The `onOpened` function also updates the UI for the user to indicate that the game is ready to play and sends a `POST` message to the server to ask it to send the latest game state.

The following client-side JavaScript code implements this functionality:

```
sendMessage = function(path, opt_param) {
  path += '?g=' + state.game_key;
  if (opt_param) {
    path += '&' + opt_param;
  }
  var xhr = new XMLHttpRequest();
  xhr.open('POST', path, true);
  xhr.send();
};

onOpened = function() {
  connected = true;
  sendMessage('opened');
  updateBoard();
};
```

Note that the application defines `sendMessage()` as a wrapper around `XmlHttpRequest`, which the client uses to send messages to the server.

### Updating the game state

The Tic Tac Toe Javascript client uses an `onClick` handler called `moveInSquare` to handle mouse clicks in the board. When a player makes a move in our Tic Tac Toe game by clicking on a square, the client uses `XmlHttpRequest` to send a `POST` message to the application with the proposed move.

The following client Javascript code snippet sends the message to the server:

```
moveInSquare = function(id) {
  if (isMyMove() && state.board[id] == ' ') {
    sendMessage('/move', 'i=' + id);
  }
}
```

### Validating and sending the new game state

Use an HTTP request to send messages from the client to the server. In our example, when the server receives the client's message via an HTTP request, it first uses its request handler to validate the move. Then, if the move is legal, the server uses the [`ChannelService.sendMessage()`](https://web.archive.org/web/20160425100002/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/channel/ChannelService#sendMessage(com.google.appengine.api.channel.ChannelMessage)) method to send messages indicating the new state of the board to both clients.

```
public class Game {
  // member variables, etc omitted for brevity.

  public String getChannelKey(String user) {
    return user + KeyFactory.keyToString(key);
  }

  private void sendUpdateToUser(String user) {
    if (user != null) {
      ChannelService channelService = ChannelServiceFactory.getChannelService();
      String channelKey = getChannelKey(user);
      channelService.sendMessage(new ChannelMessage(channelKey, getMessageString()));
    }
  }

  public void sendUpdateToClients() {
    sendUpdateToUser(userX);
    sendUpdateToUser(userO);
  }
}

public class MoveServlet extends HttpServlet {
  @Override
  public void doPost(HttpServletRequest req, HttpServletResponse resp) throws IOException {
    String gameId = req.getParameter("g");
    Game game = pm.getObjectById(Game.class, KeyFactory.stringToKey(gameId));

    // Code to retrieve user id, check rules and update game omitted for brevity
    game.sendUpdateToClients();
  }
}
```

## Tracking client connections and disconnections

Applications may register to be notified when a client connects to or disconnects from a channel.

You can [enable this inbound service](https://web.archive.org/web/20160425100002/https://cloud.google.com/appengine/docs/java/config/appconfig#Java_appengine_web_xml_Inbound_services) in `appengine-web.xml`:

```
<inbound-services>
  <service>channel_presence</service>
</inbound-services>
```

When you enable `channel_presence`, your application receives POSTs to the following URL paths:

-   POSTs to `/_ah/channel/connected/` signal that the client has connected to the channel and can receive messages.
-   POSTs to `/_ah/channel/disconnected/` signal that the client has disconnected from the channel.

Your application can register handlers to these paths in order to receive notifications. You can use these notifications to track which clients are currently connected.

You can use the `ChannelService` class's `parsePresence()` to retrieve the `client_id` of the connected or disconnected channel.

```
// In the handler for _ah/channel/connected/
ChannelService channelService = ChannelServiceFactory.getChannelService();
ChannelPresence presence = channelService.parsePresence(req);
```

## Tokens and security

Treat the token returned by `createChannel()` as a secret. If a malicious application gains access to the token, it could listen to messages sent along the channel you are using. Avoid using the token in a URL request because a malicious website could see it in their referrer logs.

By default, tokens expire in two hours. If a client remains connected to a channel for longer than the token duration, the socket’s `onerror()` and `onclose()` callbacks are called. At this point, the client can make an XHR request to the application to request a new token and open a new channel.

## Caveats

### One client per client ID

Only one client at a time can connect to a channel using a given Client ID, so an application cannot use a Client ID for fan-out. In other words, it's not possible to create a central Client ID for connections to multiple clients (For example, you can't create a Client ID for something like a "global-high-scores" channel and use it to broadcast to multiple game clients.)

### One client per channel per page

A client can only connect to one channel per page. If an application needs to send multiple types of data to a client, aggregate it on the server side and send it to appropriate handlers in the client’s [`socket.onmessage`](https://web.archive.org/web/20160425100002/https://cloud.google.com/appengine/docs/java/channel/javascript#onmessage) callback.
