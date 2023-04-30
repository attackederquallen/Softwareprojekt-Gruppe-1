# Events and Payload

# Client → Server

## Tournament

- Event: tournamt:create → Erstellt ein Tunier mit den spezifizierten Optionen

  - Payload:

    ```json
    // Kein json objekt, sondern ein integer!
    "numBestOfMatches": number  // ungerade zahlen >2

    socket.emit("tournament:create", 5, (data: any) => ...
    ```

  - Response = Acknowledgment:

    - Response Daten:

    ```json
    {
      "success": true,
      "data": {
        "tournamentId": "tournament Id",
        "currentSize": 1,
        "bestOf": 5 // anzahl der best of matches
      },
      "error": null
    }
    ```

    oder wenn z.B. eine falsche größe übergeben wurde

    ```json
    {
      "success": false,
      "data": null,
      "error": {
        "message": "string"
      }
    }
    ```

- tournament:join → Lässt den Spieler das Tuniert mit der gegebenden id joinen

  - Payload:

    ```json
    // Kein json objekt, sondern ein String!
    "tournamentId": string // id of the tournament

    socket.emit("tournament:join", "id", (data: any) => ...
    ```

  - Response = Acknowledgment:

    - Response Daten:

    ```json
    {
      "success": true,
      "data": {
        "tournamentId": "string",
        "currentSize": 2,
        "bestOf": 5, // anzahl der best of matches
        "players": [
          {
            "id": "string",
            "username": "string"
          }
        ]
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
    "data": null,
    "error": null
  }
  ```

  Error

  ```json
  {
    "success": false,
    "data": null,
    "error": null
  }
  ```

# Spiel

- game:makeMove → Spielzug machen (Karte legen/ziehen oder nope)
- Daten die der server zurück schicken muss (put, take, nope)

```json
// put
{
 "type": "put",
 "card1": {
  "type": "number",
  "color": "red",
  "value": 1
 },
 "card2": null,
 "card3": null,
 ...
}

// take
{
 "type": "take",
 "card1": null,
 "card2": null,
 "card3": null,
}

// nope
{
 "type": "nope",
 "card1": null,
 "card2": null,
 "card3": null,
}
```
