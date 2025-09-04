# Token based Authentifizierung

## APIs 
Application Programming Interfaces, kurz APIs, sind heute unglaublich beliebt. Einer der Hauptgründe für diesen Boom ist die Möglichkeit, eine einzige API zu erstellen, die dann mehrere verschiedene Schnittstellen, wie z. B. eine Webanwendung und eine mobile Anwendung, gleichzeitig bedienen kann. Dadurch kann die gleiche serverseitige Logik zentralisiert und für alle Schnittstellen wiederverwendet werden. Aus Sicherheitsperspektive ist dies in der Regel auch von Vorteil, da wir so die serverseitige Sicherheit in einer einzigen API implementieren können, die dann unseren Server unabhängig von der verwendeten Schnittstelle schützt.

Mit dem Aufkommen von APIs wurden jedoch auch neue Methoden zur Session-Verwaltung entwickelt. Da Cookies in der Regel mit Webanwendungen verbunden sind, die über einen Browser genutzt werden, funktioniert die Cookie-basierte Authentifizierung für APIs in der Regel nicht so gut, da die Lösung dann nicht für andere Schnittstellen geeignet ist. Hier kommt die tokenbasierte Session-Verwaltung ins Spiel.

## Token Basiertes Session Management
Die tokenbasierte Sitzungsverwaltung ist ein relativ neues Konzept. Anstelle der automatischen Cookie-Verwaltungsfunktionen des Browsers wird hierfür clientseitiger Code verwendet. Nach der Authentifizierung stellt die Webanwendung ein Token im Request-Body bereit. Mithilfe von clientseitigem JavaScript-Code wird dieses Token dann im LocalStorage des Browsers gespeichert.

Bei einer neuen Anfrage muss der JavaScript-Code das Token aus dem Speicher laden und als Header anhängen. Eine der gängigsten Arten von Tokens sind JSON Web Tokens (JWT), die über den Header „`Authorization: Bearer`“ übergeben werden. Da wir jedoch nicht die integrierten Cookie-Verwaltungsfunktionen des Browsers verwenden, herrscht hier ein wenig Wildwest-Stimmung, in der alles möglich ist. Es gibt zwar Standards, aber nichts zwingt dazu, sich an diese Standards zu halten. Tokens wie JWTs sind eine Möglichkeit, die tokenbasierte Session-Verwaltung zu standardisieren.

## Kommunikation mit der API

Nachfolgend finden sich die zwei mögliche Curl-Anfragen, die man für die Schnittstelle mit der API verwenden kann. Für die Authentifizierung kann die folgende Curl-Anfrage gestellt werden:

```
curl -H 'Content-Type: application/json' -X POST -d '{ "username" : "user", "password" : "passwordX" }' http://10.10.82.44/api/v1.0/exampleX
```

Zur Benutzerüberprüfung kann die folgende Curl-Anfrage gestellt werden:

```
curl -H 'Authorization: Bearer [JWT token]' http://10.10.82.44/api/v1.0/example2?username=Y
```

Die Komponente [JWT-Token] muss durch den JWT ersetzt werden, der bei der ersten Anfrage empfangen wurde. In diesem Fall kann Y je nach Ihren Berechtigungen entweder „user” oder „admin” sein.

