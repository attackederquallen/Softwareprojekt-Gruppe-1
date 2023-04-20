# Events & Payload

# Client → Server

## Tournament

- Event: tournamt:create → Erstellt ein Tunier mit den spezifizierten Optionen

  - Payload:
    ```json
    {
    	"numBestOfMatches": number  // ungerade zahlen
    }
    ```
  - Response = Acknowledgment:
    - Response Daten:
    ```json
    {
    	"success": true,
    	"data": {
        "tournamentId": string,
        "currentSize": 1,
        "bestOf": 5, // anzahl der best of matches
      },
    	"error": null
    }
    ```
    oder wenn z.B. eine falsche größe übergeben wurde
    ```json
    {
    	"success": false,
    	"data": null
    	"error": {
    		"message": string
      },
    }
    ```

- tournament:join → Lässt den Spieler das Tuniert mit der gegebenden id joinen

  - Payload:
    ```json
    {
    	"tournamentId": string // id of the tournament
    }
    ```
  - Response = Acknowledgment:
    - Response Daten:
    ```json
    {
    	"success": true,
    	"data": {
        "tournamentId": string,
        "currentSize": number,
        "bestOf": number, // anzahl der best of matches
        "players": [{
    	    "id": string,
    			"username": string,
    		}]
      },
    	"error": null
    }
    ```

- tournament:leave → Lässt den Spieler das Tuniert verlassen
  - Response Daten:
  ```json
  {
  	"success": true,
  	"data": {
      "tournamentId": string,
      "currentSize": number,
      "bestOf": number, // anzahl der best of matches
      "players": [{
  	    "id": string,
  			"username": string,
  		}]
    },
  	"error": null
  }
  ```
- tournament:start → Host startet Tunier
- disconnect

# Spiel

- game:placeCards → Karte legen
- game:drawCards → Karte ziehen

# Server → Client

- availableTournaments → Schickt den Spieler alle verrfügbaren Tuniere denen er joinen kann
  - Payload and Daten die zu dem client kommen
    ```json
    {
    	"tournaments": [
    		{
    			"id": string,
          "createdAt": string, // 2023-04-14T19:16:17.907Z
          "status": string,
          "currentSize": int,  // derzeitige anzahl an spielern
          "players": [{
    				"username": string
    			}]
    		},
    		{
    			"id": string,
          "createdAt": string,
          "status": string,
          "currentSize": int,
          "players": [{
    				"username": string
    			}]
    		},
    		.....
    		.....
    	]
    }
    ```
- tournament:joined → Wird gesendet wenn ein neuer spieler dem Tunier beitritt
  - Response Emit
    ```json
    {
    	"success": true,
    	"data": {
        "tournamentId": string,
        "currentSize": number,
        "bestOf": number, // anzahl der best of matches
        "players": [{
    	    "id": string,
    			"username": string,
    		}]
      },
    	"error": null
    }
    ```
- tournament:started

  ```json
  {
  	 "message": "Tournament started!!!!!!!"
     "tournamentId": string,
     "currentSize": number,
     "players": [{
  		 "id": string,
  		 "username": string,
  	 }]

  }
  ```

- game:started → Infos zum spiel start für alle spieler im spiel

  ```json
  {
  	 "message": "Game started!!!!!!!"
     "gameId": string,
  	 "matchNumber": number
     "players": [{
  		 "id": string,
  		 "username": string,
  		 "points": number // Nummer an gewonnen spielen in den match
  	 }]

  }
  ```

- game:state → Senden des Spielstandes an die Spieler
  ```json
  {
     "gameId": string,
  	 "topCard": Card, // oberste karte vom ablege stapel
  	 "lastTopCard": Card | null, Karte unter oberste Karte
     "drawPileSize": number,
     "players": [
  	   {
  			"username": string,
  			"id": string,
  			"handsize": number
  		 }
     ]
  	 "hand": [Card],
  	 "handSize": number,
  	 "currentPlayer": {
  			"username": string,
  			"id": string,
  		},
  		"currentPlayerIdx": number,
      "prevPlayer": {
  			"username": string,
  			"id": string,
  		},
  		"prevPlayerIdx": number,
  		"prevTurnCards": [Card] // Karten die zuletzt abgelegt wurden
   }
  ```
- game:ended → Spiel ist zuende
  ```tsx
  {
  	 "message": "game ended"
     "gameId": string,
     "matchNumber": number,
     "players": [{
  		 "id": string,
  		 "username": string,
  		 "points": number // Nummer an gewonnen spielen in den match
  	 }]
     "winner": {
  		 "id": string,
  		 "username": string,
  	 }
   }
  ```

## Acknowledgment Beispiel

### Server (JavaScript)

```jsx
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

```jsx
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

```java
const io = require('socket.io')(3000);

io.on('connection', (socket) => {
  console.log('New user connected');

  socket.on('message', (data) => {
    console.log(`Received message: ${data}`);
  });
});
```

### Client (JavaScript)

```java
const socket = io('http://localhost:3000');

socket.on('connect', () => {
  console.log('Connected to server');
});

socket.emit('message', 'Hello server');
```

## Warum Acknowledgments ?

Stellt euch vor ihr schickt etwas zum Server aber es tritt ein Fehler auf

### Server mit Basic Emit (JavaScript)

```jsx
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

```jsx
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

```jsx
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

```jsx
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

Es kann auch sein das ein normaler error geworfen wird dann würde data so aussehen

```json
// Error object
{
	"status": number
	"message": string
}
```

sonst eigentlich so

```json
// good
{
	"success": true,
	"data": ...,
	"error": null
}
// error aber ohne throw error
{
	"success": false,
	"data": null,
	"error": {
		"message": "..."
	};
}
```

### Callback Type

```tsx
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

# Jwt → JSON Web Token

## Wofür brauchen wir JWT’s?

Damit nicht jeder auf alles zugreifen kann. Heißt nicht jeder soll z.B. an einen Tunier teilnehmen können wenn dieser keinen Account hat

## Mehr Infos

[https://jwt.io/introduction](https://jwt.io/introduction)

## Wie nutzt ihr euer JWT wenn ihr euch eingeloggt habt?

### Speichert es irgendwo in einer variable.

```java
public class Constants {

	private String access_token;

	// setter und getter
	// ....
	// ....
	// ....
	// ....
}

public class MyMain {

	public static void main(Sring[] args) {
			// login sachen und so

			Constants c = new Constants();
			c.setAccessToken(jwt);
	}

}
```

### Zum authenifizieren mit den Websockets

JavaScript code

```jsx
const socket = io("http://localhost:4040/game-room", {
  auth: {
    token: "euer-jwt",
  },
});
```

Super schöner Java code

```java
IO.Options options = IO.Options.builder()
        .setAuth(singletonMap("token", "jwt"))
        .build();

Socket socket = IO.socket(URI.create("https://example.com"), options);
```

### Authentifizierung mit JWT’s in HTTP Requests

```jsx
import java.net.*;
import java.io.*;
import java.util.*;

public class HttpGetRequest {
    public static void main(String[] args) throws Exception {
        URL url = new URL("https://example.com/api/data");
        HttpURLConnection con = (HttpURLConnection) url.openConnection();
        con.setRequestMethod("GET");
        con.setRequestProperty("Authorization", "Bearer " + "YOUR_JWT_TOKEN_HERE");

        BufferedReader in = new BufferedReader(
            new InputStreamReader(con.getInputStream()));
        String inputLine;
        StringBuffer content = new StringBuffer();
        while ((inputLine = in.readLine()) != null) {
            content.append(inputLine);
        }
        in.close();

        System.out.println(content.toString());
    }
}
```

```jsx
const url = "https://example.com/api/data";
const token = "YOUR_JWT_TOKEN_HERE";

fetch(url, {
  method: "GET",
  headers: {
    Authorization: `Bearer ${token}`,
  },
})
  .then((response) => response.json())
  .then((data) => console.log(data))
  .catch((error) => console.error(error));
```

# Vordefinierte Typen

```tsx
type Card = {
  type: "number" | "joker" | "reboot" | "see-through" | "selection";
  color:
    | "red"
    | "blue"
    | "green"
    | "yellow"
    | "red-yellow"
    | "blue-green"
    | "yellow-blue"
    | "red-blue"
    | "red-green"
    | "yellow-green"
    | null; // null for action cards or joker
  value: 1 | 2 | 3 | null; // null for action cards or joker
};
```
