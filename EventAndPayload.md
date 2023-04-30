# Events and Payload

## Client → Server

### Tournament

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
    "data": null, // braucht keine infos
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

## Spiel

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
 }
 "card2": null,
 "card3": null,
}
```

```json
// take
{
  "type": "take",
  "card1": null,
  "card2": null,
  "card3": null
}
```

```json
// nope
{
  "type": "nope",
  "card1": null,
  "card2": null,
  "card3": null
}
```

## Vordefinierte Typen

```ts
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
    | "multi"
    | null; // null for action cards or joker
  value: 1 | 2 | 3 | null; // null for action cards or joker
  select: 1 | -1 | null; // is only not null if the selection card is played
  selectValue: 1 | 2 | 3 | null;
  selectedColor: "red" | "blue" | "green" | "yellow" | null;
};
```

MovePayload Type

```ts
type MovePayload {
  type: "put" | "take" | "nope";
  card1: Card | null;
  card2: Card | null;
  card3: Card | null;
}
```

```json
// put
{
 "type": "put",
 "card1": {
  "type": "number",
  "color": "red",
  "value": 1
 }
 "card2": null,
 "card3": null,
}
```

```json
// take
{
  "type": "take",
  "card1": null,
  "card2": null,
  "card3": null
}
```

```json
// nope
{
  "type": "nope",
  "card1": null,
  "card2": null,
  "card3": null
}
```

```json
// selection card

{
  "type": "put",
  "card1": {
    "type": "selection",
    "color": "multi",
    "selectValue": 1,
    "selectColor": "red"
  },
  "card2": null,
  "card3": null
}
```
