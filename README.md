# socketio-over-websocket protocol definition

The socket IO protocol is ostensibly defined in
https://github.com/socketio/socket.io-protocol, but as is the case with
virtually all nodejs packages, it is insanely underspecified.  I
reverse-engineered the socketio-over-websocket protocol below.

# WebSocket frame format

One socket.io message is sent in one websocket frame. Each message is a text by
default.

A message consists of a mandatory `ws_op`, followed by an optional socketio
message. The socketio message itself consists of a `sockio_op` and an optional
json. There is no space between these components.

    ws_op (sockio_op? json?)?

Below are some examples

    0{"sid":"RJ8AGvKTEzlZ5SzQAAAA","upgrades":[],"pingInterval":25000,"pingTimeout":60000}

This is the socket.io greet message sent from the server to the client.

    42["welcome",{"message":"Welcome!"}]

This is an example of an application message. 4 = message, 2 = EVENT, the rest is the json payload.

`ws_op` is one of the following. They are defined in `engine.io-parser`.

    "0": open
    "1": close
    "2": ping
    "3": pong
    "4": message
    "5": upgrade
    "6": noop

`sockio_op` is one of the following. They are defined in `socket.io-parser`.

    "0": CONNECT
    "1": DISCONNECT
    "2": EVENT (text)
    "3": ACK
    "4": ERROR
    "5": BINARY_EVENT
    "6": BINARY_ACK

# Protocol

In the following descriptions, "message" means a websocket frame.

1. First, the client and the server does a websocket handshake, as described
   in [RFC 6455](https://tools.ietf.org/html/rfc6455). Then, the server sends
   the following two messages to complete the handshake.

        0{sid:"sessionid", upgrades: [], pingInterval: 25000, pingTimeout: 60000}
        40

    `sid` is a random base64 string (`engine.io/lib/server.js`). It should be
    generated anew for every websocket.  `upgrades` is empty for websocket.
    `pingInterval` and `pingTimeout` are in milliseconds.

2. The client and server exchanges messages of the following form.

        42["welcome",{"message":"Welcome!"}]

   The above message is sent with JS code `io.emit("welcome", {message: "Welcome!"})`.

3. Every `pingInterval` as defined in the "open" handshake message, the client
   periodically sends a ping message:

        2

   and the server responds with pong.

        3

4. For some unknown reason, the client sometimes sends message "5" (upgrade),
   even though we are using websocket. The server shall drop such a message.

# TODO


Figure out what the `EIO=3` means in the URI.
