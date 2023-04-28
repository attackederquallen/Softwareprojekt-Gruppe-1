# Events & Payload

# Client → Server

## Tournament

- Event: tournamt:create → Erstellt ein Tunier mit den spezifizierten Optionen
    - Payload:
        
        ```json
        {
        	"numBestOfMatches": number  // ungerade zahlen >2
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
    	"data": null // braucht keine infos
    	"error": null
    }
    ```
    
- tournament:start → Host startet Tunier
    - Response = Acknowledgment:
    
    Success:
    
    ```json
    {
    	"success": true,
    	"data": null
    	"error": null
    }
    ```
    
    Error
    
    ```json
    {
    	"success": false,
    	"data": null
    	"error": null
    }
    ```
    
- disconnect

# Spiel

- game:makeMove → Spielzug machen (Karte legeb/ziehen oder nope)
- Daten die der server zurück schicken muss (put, take, n)

```json
// put
{
	"type": "put",
	"card1": {
		type: "number"
		color: "red",
		value: 1
	}
	"card2": null;
	"card3": null;
	...
}

// take
{
	"type": "take",
	"card1": null;
	"card2": null;
	"card3": null;
}

// nope
{
	"type": "nope",
	"card1": null;
	"card2": null;
	"card3": null;
}
```

# Server → Client

- list:tournaments → Schickt den Spieler alle verrfügbaren Tuniere denen er joinen kann
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
        
- tournament:playerInfo→ Wird gesendet wenn ein neuer spieler dem Tunier beitritt oder aus dem tunier rausgeht
    - Response Emit
        
        ```json
        {
        	"message": "..."
        	"tournamentId": string,
          "currentSize": number,
          "bestOf": number, // anzahl der best of matches
          "players": [{
        		"id": string,
        		"username": string,
        	}]
        }
        ```
        
- tournament:info Schickt infos an alle teilnehmer eines tuniers wenn zum beispiel eine match endet oder man selber in ein match eingefügt wird

```json
{
	"message": "...."
}
```

- tournament:status
    
    ```json
    {
    	 "message": "Tournament started!!!!!!!"
       "tournamentId": string,
       "currentSize": number,
       "players": [{
    		 "id": string,
    		 "username": string,
    		 "points": number, // won matches
    	 }]
       "winner": {
    		 "id": string,
    		 "username": string,
    	 } // ist null wenn spiel start übertragen wird
    }
    ```
    
- game:status → Infos zum spiel start oder ende für alle spieler im spiel
    
    ```json
    {
    	 "message": "..."
       "gameId": string,
    	 "matchNumber": number
       "players": [{
    		 "id": string,
    		 "username": string,
    		 "points": number // Nummer an gewonnen spielen in den match
    	 }]
       "winner": {
    		 "id": string,
    		 "username": string,
    	 } // ist null wenn spiel start übertragen wird
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
    			"handSize": number		
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
    

## Acknowledgment Beispiel

### Server (JavaScript)

```jsx
const io = require('socket.io')(3000);

io.on('connection', (socket) => {
  console.log('New user connected');

  socket.on('message', (data, callback) => {
    console.log(`Received message: ${data}`);

    // Send an acknowledgment message back to the client
    callback(`Message received: ${data}`);
  });
});
```

### Client (JavaScript)

```jsx
const socket = io('http://localhost:3000');

socket.on('connect', () => {
  console.log('Connected to server');
});

socket.emit('message', 'Hello server', (data) => {
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
const io = require('socket.io')(3000);

io.on('connection', (socket) => {
  console.log('New user connected');

  socket.on('message', (data) => {
    console.log(`Received message: ${data}`);

    // Send a response message back to the client
    if (data === 'error') {
      socket.emit('error', 'There was an error processing your message');
    } else {
      socket.emit('response', `Server received your message: ${data}`);
    }
  });
});
```

### Client mit Basic Emit (JavaScript)

```jsx
const socket = io('http://localhost:3000');

socket.on('connect', () => {
  console.log('Connected to server');
});

socket.on('response', (data) => {
  console.log(`Server response: ${data}`);
});

socket.on('error', (data) => {
  console.error(`Server error: ${data}`);
});

socket.emit('message', 'Hello server');
socket.emit('message', 'error');
```

### Server mit Acknowledgment (JavaScript)

```jsx
const io = require('socket.io')(3000);

io.on('connection', (socket) => {
  console.log('New user connected');

  socket.on('message', (data, callback) => {
		console.log(`Received message: ${data}`);
		try{

			// store data in database
			if (data === 'error')
			  callback(null, { success: false, data: null, error: { message: '' }});
	    else 
	      callback(null, { success: true, data: data, error: null });
		  
		} catch (error) {
			callback(error, 
					{ success: false, data: null, error: { message: error.message }})
		}
  });
});
```

### Client mit Acknowledgment (JavaScript)

```jsx
const socket = io('http://localhost:3000');

socket.on('connect', () => {
  console.log('Connected to server');
});

socket.emit('message', "Hello Server", (data) => {
  if(data !== null && data.success === true){
		// do something
	}else {
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
    token:
      "euer-jwt",
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
const url = 'https://example.com/api/data';
const token = 'YOUR_JWT_TOKEN_HERE';

fetch(url, {
  method: 'GET',
  headers: {
    'Authorization': `Bearer ${token}`
  }
})
.then(response => response.json())
.then(data => console.log(data))
.catch(error => console.error(error));
```

# Vordefinierte Typen

```tsx
type Card = {
  type: 'number' | 'joker' | 'reboot' | 'see-through' | 'selection';
  color:
     'red'
    | 'blue'
    | 'green'
    | 'yellow'
    | 'red-yellow'
    | 'blue-green'
    | 'yellow-blue'
		| 'red-blue'
		| 'red-green'
		| 'yellow-green'
		| "multi"
    | null; // null for action cards or joker
  value: 1 | 2 | 3 | null; // null for action cards or joker
	select: 1 | -1 | null; // is only not null if the selection card is played 
	selectValue: 1 | 2 | 3 | null;
	selectedColor: 'red' | 'blue' | 'green' |'yellow' | null;
}
```

MovePayload Type

```tsx
type MovePayload {
	type: "put" | "take" | "nope";
	card1: Card | null
	card2: Card | null
	card3: Card | null
}
```

```json
// put
{
	"type": "put",
	"card1": {
		type: "number"
		color: "red",
		value: 1
	}
	"card2": null;
	"card3": null;
	...
}

// take
{
	"type": "take",
	"card1": null;
	"card2": null;
	"card3": null;
}

// nope
{
	"type": "nope",
	"card1": null;
	"card2": null;
	"card3": null;
}

\\ selection card

{
	"type": "put",
	"card1": {
		type: "selection"
		color: "multi",
		selectValue: 1 ;
		selectColor: "red"
	}
	"card2": null;
	"card3": null;
	...
}

```