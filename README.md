# Webbit EasyRemote

Webbit EasyRemote is a thin bidirectional RPC layer on top of Webbit's WebSockets.
It allows you to invoke Javascript functions in the browwser from server-side Java objects
and java methods on the server from Javascript functions.

## On the server

Start by creating a Java interface that represents the Javascript functions you want to invoke from the server:

    @Remote
    interface ChatClient {
      void say(String username, String message);
      void leave(String username);
      void join(String username);
    }

The `@Remote` annotation will cause the interface to be implemented by a dynamic proxy that the server can use to talk to the browser.

Now, implement the object you would like to invoke methods on from the browser Javascript:

    public class ChatServer implements Server<ChatClient> {
      @Override
      public void onOpen(WebSocketConnection connection, ChatClient client) throws Exception {
      }

      @Override
      public void onClose(WebSocketConnection connection, ChatClient client) throws Exception {
      }

      @Remote
      public void login(DataHolder connection, String username) {
      }

      @Remote
      public void say(DataHolder connection, String message) {
      }
    }

The `@Remote` annotation on the methods exposes methods so they can be invoked by the browser.

Now hook it all up in Webbit:

    WebServer webServer = createWebServer(9877)
      .add("/chatsocket", magic(ChatClient.class, new ChatServer()))   // Mount your client proxy and server instance
      .add(new EmbeddedResourceHandler("org/webbitserver/easyremote")) // Serves webbit.easyremote.js
      .start();

## On the client

Start by including the Javascript library (2k) in your HTML page:

    <script type="text/javascript" src="webbit.easyremote.js"></script>

And hook up to the server:

    var chatServer = new WebbitSocket('/chatsocket', {
      onopen: function() {
      },
      onclose: function() {
      },
      say: function(username, message) {
      },
      join: function(username) {
      },
      leave: function(username) {
      }
    });

Now you can talk to the server:

    chatServer.login("Brian");
    chatServer.say("I am NOT the Messiah!");

## Using CSV format for server->client communication

By default, Webbit EasyRemote will send a small JSON document to the browser over the WebSocket to invoke a method:

    {
      "action": "say",
      "args": ["Brian", "I am NOT the Messiah!"]
    }

If the volume of messages going from the server to the client is high, this might clog up the browser since parsing JSON is relatively
slow, even for short messages. Parsing this is faster:

    say,Brian,I am NOT the Messiah!

To use the CSV format, just construct your WebbitSocket like this:

   new WebbitSocket("/websocket", myClient, "csv");

That's it!

## Error handling

If either side tries to invoke a method/function that doesn't exist on the other side, an error will be logged on the server.
If a method/function is invoked with a different number of arguments than the other side accepts an error will also be logged on the server.