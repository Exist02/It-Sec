# Was ist das? 

Die Möglichkeit, Dateien auf einen Server hochzuladen, ist zu einem festen Bestandteil unserer Interaktion mit Webanwendungen geworden. Sei es ein Profilbild für eine Social-Media-Website, ein Bericht, der in einen Cloud-Speicher hochgeladen wird, oder das Speichern eines Projekts auf Github; die Anwendungsmöglichkeiten für Datei-Upload-Funktionen sind grenzenlos.

Leider können Datei-Uploads, wenn sie schlecht gehandhabt werden, auch schwerwiegende Sicherheitslücken auf dem Server öffnen. Dies kann von relativ kleinen, lästigen Problemen bis hin zur vollständigen Remote Code Execution (RCE) führen, wenn es einem Angreifer gelingt, eine Shell hochzuladen und auszuführen. Mit uneingeschränktem Upload-Zugriff auf einen Server (und der Möglichkeit, Daten nach Belieben abzurufen) kann ein Angreifer vorhandene Inhalte verunstalten oder anderweitig verändern - bis hin zur Einschleusung bösartiger Webseiten, die zu weiteren Schwachstellen wie XSS oder CSRF führen. Durch das Hochladen beliebiger Dateien könnte ein Angreifer den Server möglicherweise auch dazu nutzen, illegale Inhalte zu hosten und/oder zu verbreiten oder vertrauliche Informationen weiterzugeben. Realistisch betrachtet ist ein Angreifer, der eine beliebige Datei auf Ihren Server hochladen kann - ohne jegliche Einschränkungen - in der Tat sehr gefährlich.

---
# Generelles Vorgehen
Wir haben also einen Datei-Upload-Punkt auf einer Website. Wie können wir ihn ausnutzen?

Wie bei jeder Art von Hacking ist die Aufzählung der Schlüssel. Je mehr wir über unsere Umgebung wissen, desto mehr können wir mit ihr anfangen. Ein Blick auf den Quellcode der Seite ist gut, um zu sehen, ob irgendeine Art von clientseitiger Filterung angewendet wird. Das Scannen mit einem Verzeichnisbruteforcer wie Gobuster ist normalerweise bei Webangriffen hilfreich und kann aufdecken, wohin Dateien hochgeladen werden; Gobuster ist unter Kali nicht mehr standardmäßig installiert, kann aber mit `sudo apt install gobuster` installiert werden. Das Abfangen von Upload-Anfragen mit **Burpsuite** kann ebenfalls nützlich sein. Browser-Erweiterungen wie **Wappalyser** können auf einen Blick wertvolle Informationen über die Website liefern, die Sie ins Visier nehmen.

Mit einem grundlegenden Verständnis dafür, wie die Website unsere Eingaben behandelt, können wir dann versuchen, herauszufinden, was wir hochladen können und was nicht. Wenn die Website client-seitige Filter einsetzt, können wir uns den Code des Filters ansehen und versuchen, ihn zu umgehen (mehr dazu später!). Wenn die Website über serverseitige Filter verfügt, müssen wir möglicherweise erraten, wonach der Filter sucht, eine Datei hochladen und dann anhand der Fehlermeldung etwas anderes versuchen, wenn der Upload fehlschlägt. Das Hochladen von Dateien, die Fehler provozieren sollen, kann dabei helfen. Tools wie Burpsuite oder **OWASP Zap** können in diesem Stadium sehr hilfreich sein.


---
# Überschreiben von Vorhandenen Dateien 

Wenn Dateien auf den Server hochgeladen werden, sollte eine Reihe von Prüfungen durchgeführt werden, um sicherzustellen, dass die Datei nichts überschreibt, was bereits auf dem Server vorhanden ist. Üblicherweise wird der Datei ein neuer Name zugewiesen - oft entweder zufällig oder mit dem Datum und der Uhrzeit des Uploads, die an den Anfang oder das Ende des ursprünglichen Dateinamens angehängt werden. Alternativ kann geprüft werden, ob der Dateiname bereits auf dem Server existiert; wenn eine Datei mit demselben Namen bereits existiert, gibt der Server eine Fehlermeldung aus und fordert den Benutzer auf, einen anderen Dateinamen zu wählen. Die Dateiberechtigungen spielen auch eine Rolle, wenn es darum geht, bestehende Dateien vor dem Überschreiben zu schützen. Webseiten beispielsweise sollten für den Webbenutzer nicht beschreibbar sein, damit sie nicht durch eine von einem Angreifer hochgeladene bösartige Version überschrieben werden können.

Wenn jedoch keine solchen Vorkehrungen getroffen werden, können wir möglicherweise vorhandene Dateien auf dem Server überschreiben. Realistischerweise stehen die Chancen gut, dass die Dateiberechtigungen auf dem Server verhindern, dass dies eine ernsthafte Sicherheitslücke darstellt. Dennoch könnte es ein ziemliches Ärgernis sein, und es lohnt sich, in einer Pentest- oder Fehlersuchumgebung ein Auge darauf zu haben.

## Praktisches Beispiel: 

- Gegeben ist eine Website mit einem Bild als Hintergrund und einer Upload Möglichkeit für Dateien
- Im Ramen der Enumeration kann man Feststellen das dass Hintergrundbild als "/images/spaniel.jpg" eingebunden ist
	- https://imgur.com/Ltp2FMr
- Dadurch wissen wir jetzt woher das Hintergrund Bild kommt, aber können wir es überschreiben?
- Um das zu Testen brauchen wir erstmal ein zweites B
- Bild welches wir auch "spaniel.jpg" nennen
- Dieses laden wir dann hoch und laden danach die seite neu
	- Erfolg das von uns hochgeladene Bild ist jetzt der Hintergrund

---

# Remote Code Execution

Remote Code Execution (wie der Name schon sagt) würde es uns ermöglichen, beliebigen Code auf dem Webserver auszuführen. Auch wenn dies wahrscheinlich über ein Web-Benutzerkonto mit geringen Privilegien geschieht (z. B. www-data auf Linux-Servern), handelt es sich dennoch um eine äußerst schwerwiegende Sicherheitslücke. Die Remotecodeausführung über eine Upload-Schwachstelle in einer Webanwendung wird in der Regel durch das Hochladen eines Programms ausgenutzt, das in derselben Sprache wie das Backend der Website geschrieben wurde (oder in einer anderen Sprache, die der Server versteht und ausführen kann). Traditionell ist dies PHP, aber in letzter Zeit sind andere Back-End-Sprachen häufiger geworden (Python, Django und Javascript in Form von Node.js sind die besten Beispiele). Es ist erwähnenswert, dass diese Angriffsmethode bei einer gerouteten Anwendung (d. h. einer Anwendung, bei der die Routen programmatisch definiert sind und nicht auf das Dateisystem abgebildet werden) sehr viel komplizierter ist und sehr viel seltener vorkommt. Die meisten modernen Web-Frameworks werden programmatisch geroutet.

Es gibt zwei grundlegende Möglichkeiten, RCE auf einem Webserver zu erreichen, wenn eine Datei-Upload-Schwachstelle ausgenutzt wird: Webshells und Reverse/Bind-Shells. Realistischerweise ist eine voll funktionsfähige Reverse/Bind-Shell das ideale Ziel für einen Angreifer; eine Webshell kann jedoch die einzige verfügbare Option sein (z. B. wenn eine Dateilängenbegrenzung für Uploads eingeführt wurde oder wenn Firewall-Regeln netzwerkbasierte Shells verhindern). Wir werden uns jeden dieser Fälle der Reihe nach ansehen. Als allgemeine Methode würden wir versuchen, eine Shell der einen oder anderen Art hochzuladen und sie dann zu aktivieren, indem wir entweder direkt zur Datei navigieren, wenn der Server dies zulässt (nicht geroutete Anwendungen mit unzureichenden Beschränkungen), oder indem wir die Webanwendung zwingen, das Skript für uns auszuführen (notwendig bei gerouteten Anwendungen).

## Vorbereitung für die Beispiele 

- Angenommene Lage ist das man auf eine Website gestoßen ist welche es einem Ermöglicht Dateien hochzuladen
- Enumeration via Gobuster im dir modus
```
gobuster dir -u "ZielURL" -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

- Resultat aus Gobuster, es wurden zwei Verzeichnisse gefunden "/assets" und "/resources" 
- nun kann getestet werden ob die Dateien in einem der beiden Verzeichnissen landet nach dem Upload, hier kann wieder ein Beispielbild genutzt werden 
	- Resultat Bild in "/resources" nach Upload gefunden


## Beispiel 1 Webshell

Anhand der Resultate der Vorbereitung können wir jetzt versuchen eine Webshell zu starten. Da wir wissen, dass dieser Webserver mit einem PHP-Backend läuft, können wir direkt mit dem Erstellen und Hochladen der Shell beginnen. Im wirklichen Leben müssen wir vielleicht ein wenig mehr aufzählen, aber PHP ist trotzdem ein guter Ausgangspunkt.

Eine einfache Webshell funktioniert, indem sie einen Parameter entgegennimmt und ihn als Systembefehl ausführt. In PHP würde die Syntax dafür lauten:
```
<?php
echo system($_GET["cmd"]);   
?>
```

Dieser Code nimmt einen GET-Parameter und führt ihn als Systembefehl aus. Die Ausgabe wird dann als Echo auf dem Bildschirm ausgegeben.

Versuchen wir, ihn auf die Website hochzuladen und ihn dann zu verwenden, um unseren aktuellen Benutzer und den Inhalt des aktuellen Verzeichnisses anzuzeigen:
https://imgur.com/nxZY6Hl

Wir könnten nun diese Shell verwenden, um Dateien aus dem System zu lesen, oder von hier aus zu einer Reverse Shell wechseln. Jetzt, wo wir RCE haben, sind die Möglichkeiten grenzenlos. Beachten Sie, dass es bei der Verwendung von Webshells in der Regel einfacher ist, die Ausgabe zu betrachten, indem Sie sich den Quellcode der Seite ansehen. Dadurch wird die Formatierung der Ausgabe drastisch verbessert.


## Beispiel 2 Reverse Shell

Der Prozess zum Hochladen einer Reverse Shell ist fast identisch mit dem Hochladen einer Webshell, daher wird dieser Abschnitt kürzer sein. Wir verwenden die allgegenwärtige Pentest Monkey Reverse Shell, die standardmäßig in Kali Linux enthalten ist, aber auch hier heruntergeladen werden kann. Sie müssen die Zeile 49 der Shell bearbeiten. Sie lautet derzeit `$ip = ‚127.0.0.1‘; // CHANGE THIS`

link zur Rev Shell von Pentestmonkey
https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/master/php-reverse-shell.php

-- wie angewiesen, ändern Sie 127.0.0.1 in Ihre IP-Adresse. Sie können die folgende Zeile ignorieren, die ebenfalls eine Änderung verlangt. Nachdem die Shell bearbeitet wurde, müssen wir als nächstes einen Netcat-Listener starten, um die Verbindung zu empfangen. `nc -lvnp 1234`

Nun laden wir die Shell hoch und aktivieren sie, indem wir zu http://demo.uploadvulns.thm/uploads/shell.php navigieren. Der Name der Shell ist natürlich der, den Sie ihr gegeben haben (standardmäßig php-reverse-shell.php).Die Website sollte hängen und nicht richtig geladen werden - wenn wir jedoch zu unserem Terminal zurückwechseln, haben wir einen Treffer! Wieder einmal haben wir RCE auf diesem Webserver erhalten. Von hier aus würden wir unsere Shell stabilisieren und unsere Privilegien erweitern wollen, aber das sind Aufgaben für ein anderes Mal 

Siehe Stage3 Exploitation > NetCat.md 

---


# Verteidigungs Mechanismen Filtering

Bis jetzt haben wir die Gegenmaßnahmen, die von Webentwicklern zur Abwehr von Datei-Upload-Schwachstellen eingesetzt werden, weitgehend ignoriert. Jetzt werden wir uns einige der Verteidigungsmechanismen ansehen, die verwendet werden, um bösartige Dateiuploads zu verhindern, und wie man sie umgehen kann.

---

Beginnen wir zunächst die Unterschiede zwischen der clientseitigen Filterung und der serverseitigen Filterung zu erörtern.

Wenn wir im Zusammenhang mit Webanwendungen von einem „clientseitigen“ Skript sprechen, meinen wir damit, dass es im Browser des Benutzers und nicht auf dem Webserver selbst ausgeführt wird. JavaScript ist als clientseitige Skriptsprache so gut wie allgegenwärtig, obwohl es auch Alternativen gibt.  Unabhängig von der verwendeten Sprache wird ein clientseitiges Skript in Ihrem Webbrowser ausgeführt. Im Zusammenhang mit dem Hochladen von Dateien bedeutet dies, dass die Filterung erfolgt, bevor die Datei überhaupt auf den Server hochgeladen wird. Theoretisch scheint das eine gute Sache zu sein, oder? In einer idealen Welt wäre es das auch, aber da die Filterung auf unserem Computer stattfindet, ist sie sehr leicht zu umgehen. Daher ist die clientseitige Filterung an sich eine äußerst unsichere Methode, um zu überprüfen, ob eine hochgeladene Datei nicht bösartig ist.

Umgekehrt wird, wie Sie vielleicht schon vermutet haben, ein serverseitiges Skript auf dem Server ausgeführt. Traditionell war PHP die vorherrschende serverseitige Sprache (mit Microsofts ASP für IIS an zweiter Stelle); in den letzten Jahren haben sich jedoch andere Optionen (C#, Node.js, Python, Ruby on Rails und eine Vielzahl anderer) immer mehr durchgesetzt. Die serverseitige Filterung ist in der Regel schwieriger zu umgehen, da Sie den Code nicht direkt vor sich haben. Da der Code auf dem Server ausgeführt wird, ist es in den meisten Fällen auch nicht möglich, den Filter vollständig zu umgehen; stattdessen müssen wir einen Payload bilden, der mit den vorhandenen Filtern konform ist, aber dennoch die Ausführung unseres Codes ermöglicht.

## Extention Filtering

File extensions werden (theoretisch) verwendet, um den Inhalt einer Datei zu identifizieren. In der Praxis sind sie sehr leicht zu ändern, so dass sie eigentlich nicht viel bedeuten; MS Windows verwendet sie jedoch immer noch, um Dateitypen zu identifizieren, obwohl Unix-basierte Systeme dazu neigen, sich auf andere Methoden zu verlassen, auf die wir gleich noch eingehen werden. Filter, die nach Erweiterungen suchen, können auf zwei Arten funktionieren. Sie führen entweder eine schwarze Liste von Erweiterungen (d.h. sie haben eine Liste von Erweiterungen, die nicht erlaubt sind) oder sie führen eine weiße Liste von Erweiterungen (d.h. sie haben eine Liste von Erweiterungen, die erlaubt sind, und weisen alles andere zurück).

## File Type Filtering

Ähnlich wie bei der Prüfung von Extensions, aber intensiver, geht es bei der Dateitypfilterung auch hier darum, zu überprüfen, ob der Inhalt einer Datei für das Hochladen geeignet ist. Wir werden uns zwei Arten der Dateitypüberprüfung ansehen:

### MIME Validation
MIME-Typen (Multipurpose Internet Mail Extension) werden als Bezeichner für Dateien verwendet - ursprünglich, wenn sie als Anhänge per E-Mail übertragen werden, aber jetzt auch, wenn Dateien über HTTP(S) übertragen werden. Der MIME-Typ für einen Datei-Upload wird in der Kopfzeile der Anfrage angegeben und sieht in etwa so aus:
https://imgur.com/Z6Zr4CT

MIME-Typen folgen dem Format `<Typ>/<Subtyp>`. In der obigen Anfrage können Sie sehen, dass das Bild „spaniel.jpg“ auf den Server hochgeladen wurde. Da es sich um ein legitimes JPEG-Bild handelt, war der MIME-Typ für diesen Upload „image/jpeg“. Der MIME-Typ einer Datei kann client- und/oder serverseitig überprüft werden; da MIME jedoch auf der Dateierweiterung basiert, kann dies sehr leicht umgangen werden.

## Magic Number Validation

Magic Numbers sind die genauere Art, den Inhalt einer Datei zu bestimmen; allerdings ist es keineswegs unmöglich, sie zu fälschen. Die „magische Zahl“ einer Datei ist eine Zeichenfolge von Bytes ganz am Anfang des Dateiinhalts, die den Inhalt kennzeichnet. Bei einer PNG-Datei zum Beispiel stehen diese Bytes ganz oben in der Datei: `89 50 4E 47 0D 0A 1A 0A`

Beispielbild: https://imgur.com/18vlRlK
Im Gegensatz zu Windows verwenden Unix-Systeme Magic Numbers zur Identifizierung von Dateien; bei Datei-Uploads ist es jedoch möglich, die magische Zahl der hochgeladenen Datei zu überprüfen, um sicherzustellen, dass sie sicher akzeptiert werden kann. Dies ist keineswegs eine garantierte Lösung, aber effektiver als die Überprüfung der Dateierweiterung einer Datei.