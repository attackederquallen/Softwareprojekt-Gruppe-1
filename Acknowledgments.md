## Acknowledgments

## Dokumentation

Javascript
<https://socket.io/docs/v4/emitting-events/#acknowledgements>

Java
https://socketio.github.io/socket.io-client-java/emitting_events.html#Acknowledgements

### Server (JavaScript)

```js
const io = require("socket.io")(3000);

io.on("connection", (socket) => {
  console.log("New user connected");

  socket.on("message", (data, callback) => {
    console.log(`Received message: ${data}`);

    // Send an acknowledgment message back to the client
    callback(`Message received: ${data}`);
  });
});
```

### Client (JavaScript)

```js
const socket = io("http://localhost:3000");

socket.on("connect", () => {
  console.log("Connected to server");
});

socket.emit("message", "Hello server", (data) => {
  console.log(`Server acknowledged: ${data}`);
});
```

### Client (Java 17)

```java
import io.socket.client.IO;
import io.socket.client.Socket;
import io.socket.emitter.Emitter;

import java.net.URISyntaxException;

public class SocketIOClientExample {

    public static void main(String[] args) throws URISyntaxException {
        Socket socket = IO.socket("http://localhost:3000");

        socket.on(Socket.EVENT_CONNECT, new Emitter.Listener() {
            @Override
            public void call(Object... args) {
                System.out.println("Connected to server");
                socket.emit("message", "Hello server", new Ack() {
                    @Override
                    public void call(Object... args) {
                        String ackMessage = (String) args[0];
                        System.out.println("Server acknowledged: " + ackMessage);
                    }
                });
            }
        });

        socket.connect();
    }
}
```

## Normales Emit → (Senden)

### Server (JavaScript)

```js
const io = require("socket.io")(3000);

io.on("connection", (socket) => {
  console.log("New user connected");

  socket.on("message", (data) => {
    console.log(`Received message: ${data}`);
  });
});
```

### Client (JavaScript)

```js
const socket = io("http://localhost:3000");

socket.on("connect", () => {
  console.log("Connected to server");
});

socket.emit("message", "Hello server");
```

## Warum Acknowledgments ?

Stellt euch vor ihr schickt etwas zum Server aber es tritt ein Fehler auf

### Server mit Basic Emit (JavaScript)

```js
const io = require("socket.io")(3000);

io.on("connection", (socket) => {
  console.log("New user connected");

  socket.on("message", (data) => {
    console.log(`Received message: ${data}`);

    // Send a response message back to the client
    if (data === "error") {
      socket.emit("error", "There was an error processing your message");
    } else {
      socket.emit("response", `Server received your message: ${data}`);
    }
  });
});
```

### Client mit Basic Emit (JavaScript)

```js
const socket = io("http://localhost:3000");

socket.on("connect", () => {
  console.log("Connected to server");
});

socket.on("response", (data) => {
  console.log(`Server response: ${data}`);
});

socket.on("error", (data) => {
  console.error(`Server error: ${data}`);
});

socket.emit("message", "Hello server");
socket.emit("message", "error");
```

### Server mit Acknowledgment (JavaScript)

```js
const io = require("socket.io")(3000);

io.on("connection", (socket) => {
  console.log("New user connected");

  socket.on("message", (data, callback) => {
    console.log(`Received message: ${data}`);
    try {
      // store data in database
      if (data === "error")
        callback(null, { success: false, data: null, error: { message: "" } });
      else callback(null, { success: true, data: data, error: null });
    } catch (error) {
      callback(error, {
        success: false,
        data: null,
        error: { message: error.message },
      });
    }
  });
});
```

### Client mit Acknowledgment (JavaScript)

```js
const socket = io("http://localhost:3000");

socket.on("connect", () => {
  console.log("Connected to server");
});

socket.emit("message", "Hello Server", (data) => {
  if (data !== null && data.success === true) {
    // do something
  } else {
    // error
  }
});
```

```json
// good
{
 "success": true,
 "data": {}, // json objekt,
 "error": null
}
// error aber ohne throw error
{
 "success": false,
 "data": null,
 "error": {
  "message": "..."
 }
}
```

### Callback Type

```ts
export type SocketCallback<T> = (
  error?: Error | null,
  response?: SocketResponse<T>
) => void;

export interface SocketResponse<T> {
  success: boolean;
  data: T;
  error: {
    message: string;
  } | null;
}
```
