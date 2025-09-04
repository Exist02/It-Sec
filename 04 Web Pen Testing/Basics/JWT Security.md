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

# JWTs 

## JWT Struktur

Ein JWT besteht aus drei Komponenten, die jeweils Base64Url-kodiert und durch Punkte getrennt sind:

**Header** – Der Header gibt in der Regel den Typ des Tokens an, in diesem Fall JWT, sowie den verwendeten Signaturalgorithmus.

**Payload** – Die Payload ist der Hauptteil des Tokens, der die Claims enthält. Ein Claim ist eine Information, die für eine bestimmte Entität bereitgestellt wird. In JWTs gibt es registrierte Ansprüche, die durch den JWT-Standard vordefiniert sind, sowie öffentliche oder private Ansprüche. Die öffentlichen und privaten Ansprüche werden vom Entwickler definiert. Es ist wichtig, den Unterschied zwischen öffentlichen und privaten Ansprüchen zu kennen, jedoch nicht aus Sicherheitsgründen.

**Signatur** – Die Signatur ist der Teil des Tokens, der eine Methode zur Überprüfung der Authentizität des Tokens bereitstellt. Die Signatur wird unter Verwendung des im Header des JWT angegebenen Algorithmus erstellt.

## Signing Algorithms

Obwohl im JWT-Standard mehrere verschiedene Algorithmen definiert sind, interessieren uns eigentlich nur drei Hauptalgorithmen:

**None** – Der None-Algorithmus bedeutet, dass für die Signatur kein Algorithmus verwendet wird. Tatsächlich handelt es sich hierbei um ein JWT ohne Signatur, was bedeutet, dass die im JWT enthaltenen Angaben nicht anhand der Signatur überprüft werden können.
**Symmetrische Signatur** – Ein symmetrischer Signaturalgorithmus wie HS256 erstellt die Signatur, indem er einen geheimen Wert an den Header und den Body des JWT anhängt, bevor er einen Hashwert generiert. Die Überprüfung der Signatur kann von jedem System durchgeführt werden, das den geheimen Schlüssel kennt.
**Asymmetrische Signatur** – Ein asymmetrischer Signaturalgorithmus wie RS256 erstellt die Signatur, indem er einen privaten Schlüssel verwendet, um den Header und den Body des JWT zu signieren. Diese wird erstellt, indem der Hash generiert und dann mit dem privaten Schlüssel verschlüsselt wird. Die Überprüfung der Signatur kann von jedem System durchgeführt werden, das den öffentlichen Schlüssel kennt, der mit dem privaten Schlüssel verknüpft ist, der zur Erstellung der Signatur verwendet wurde.

## Sicherheit in der Signatur

JWTs können verschlüsselt werden (sogenannte JWEs), aber die eigentliche Stärke von JWTs liegt in der Signatur. Sobald ein JWT signiert ist, kann er an den Client gesendet werden, der diesen JWT bei Bedarf verwenden kann. Wir können einen zentralen Authentifizierungsserver einrichten, der die JWTs erstellt, die in mehreren Anwendungen verwendet werden. Jede Anwendung kann dann die Signatur des JWT überprüfen. Wenn die Signatur bestätigt wird, können die im JWT enthaltenen Angaben als vertrauenswürdig angesehen und entsprechend verarbeitet werden.

# Sensetive Information Disclosure

Das erste häufige Problem, mit dem wir uns befassen werden, ist die Offenlegung sensibler Informationen innerhalb des JWT. Bei Tokens werden die Ansprüche jedoch offengelegt, da das gesamte JWT clientseitig gesendet wird. Wenn dieselbe Entwicklungspraxis befolgt wird, können sensible Informationen offengelegt werden. Einige Beispiele sind in realen Anwendungen zu sehen:
- Offenlegung von Anmeldedaten mit dem Passwort-Hash oder, noch schlimmer, dem Klartext-Passwort, das als Anspruch gesendet wird.
- Offenlegung interner Netzwerkinformationen wie der privaten IP-Adresse oder des Hostnamens des Authentifizierungsservers.

## Praktisches Beispiel

Sehen wir uns ein praktisches Beispiel an. Authentifizieren wir uns mit der folgenden cURL-Anfrage bei unserer API:

`curl -H 'Content-Type: application/json' -X POST -d '{ "username" : "user", "password" : "password1" }' http://10.10.82.44/api/v1.0/example1`

Dadurch erhalten Sie ein JWT-Token. Entschlüsseln Sie nach der Wiederherstellung den Textkörper des JWT, um sensible Informationen aufzudecken. Sie können den Textkörper manuell entschlüsseln oder hierfür eine Website wie `JWT.io` verwenden. https://www.jwt.io/

### Was ist hier der Fehler
In diesem Beispiel wurden sensible Informationen zum Anspruch hinzugefügt, wie unten dargestellt:

```
payload = { 
"username" : username, 
"password" : password, 
"admin" : 0, 
"flag" : "[redacted]" 
} 
access_token = jwt.encode(payload, self.secret, algorithm="HS256")
```

### Der Fix 
Werte wie das Passwort oder die Flagge sollten nicht als Ansprüche hinzugefügt werden, da das JWT auf der Clientseite gesendet wird. Stattdessen sollten diese Werte sicher auf der Serverseite im Backend gespeichert werden. Bei Bedarf kann der Benutzername aus einem verifizierten JWT gelesen und zum Nachschlagen dieser Werte verwendet werden, wie im folgenden Beispiel gezeigt:

```
payload = jwt.decode(token, self.secret, algorithms="HS256") 

username = payload['username'] 
flag = self.db_lookup(username, "flag")
```