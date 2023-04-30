# JSON Web Token (JWT)

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

```js
const socket = io("url", {
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

```java
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

```js
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
