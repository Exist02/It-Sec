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

## File Length Filtering

Dateilängenfilter werden verwendet, um zu verhindern, dass große Dateien über ein Upload-Formular auf den Server hochgeladen werden (da dies den Server möglicherweise an resourcen aushungern kann). In den meisten Fällen wird dies keine Probleme verursachen, wenn wir Shells hochladen; es ist jedoch zu bedenken, dass, wenn ein Upload-Formular nur das Hochladen einer sehr kleinen Datei erwartet, ein Längenfilter vorhanden sein kann, um sicherzustellen, dass die Dateilängenanforderung eingehalten wird. Unsere vollwertige PHP-Reverse-Shell aus der vorherigen Aufgabe ist beispielsweise 5,4 KB groß - relativ klein, aber wenn das Formular maximal 2 KB erwartet, müssen wir eine andere Shell zum Hochladen finden.

## File Name Filtering

Wie bereits erwähnt, sollten Dateien, die auf einen Server hochgeladen werden, eindeutig sein. Normalerweise würde dies bedeuten, dass dem Dateinamen ein Zufallswert hinzugefügt wird. Eine alternative Strategie wäre jedoch, zu prüfen, ob eine Datei mit demselben Namen bereits auf dem Server existiert, und dem Benutzer in diesem Fall eine Fehlermeldung zu geben. Außerdem sollten Dateinamen beim Hochladen bereinigt werden, um sicherzustellen, dass sie keine „schlechten Zeichen“ enthalten, die beim Hochladen Probleme im Dateisystem verursachen könnten (z. B. Nullbytes oder Schrägstriche unter Linux sowie Steuerzeichen wie ; und möglicherweise Unicode-Zeichen). Für uns bedeutet dies, dass unsere hochgeladenen Dateien auf einem gut verwalteten System wahrscheinlich nicht denselben Namen haben, den wir ihnen vor dem Hochladen gegeben haben. Seien Sie sich also bewusst, dass Sie möglicherweise nach Ihrer Shell suchen müssen, falls es Ihnen gelingt, die Inhaltsfilterung zu umgehen.

## File Content Filtering 

Kompliziertere Filtersysteme können den gesamten Inhalt einer hochgeladenen Datei scannen, um sicherzustellen, dass ihre Erweiterung, ihr MIME-Typ und ihre Magic Number nicht gefälscht sind. Dies ist ein wesentlich komplexerer Prozess als bei den meisten einfachen Filtersystemen.

---

# Bypassing Client Side Filters

Wie bereits erwähnt, lässt sich die clientseitige Filterung in der Regel sehr leicht umgehen, da sie vollständig auf einem von Ihnen kontrollierten Rechner stattfindet. Wenn Sie Zugriff auf den Code haben, ist es sehr einfach, ihn zu ändern.


Es gibt vier einfache Möglichkeiten, den durchschnittlichen clientseitigen Datei-Upload-Filter zu umgehen:
1. Deaktivieren Sie Javascript in Ihrem Browser - dies funktioniert, sofern die Website kein Javascript benötigt, um grundlegende Funktionen bereitzustellen. Wenn die vollständige Deaktivierung von Javascript verhindert, dass die Website überhaupt funktioniert, ist eine der anderen Methoden wünschenswerter; ansonsten kann dies ein effektiver Weg sein, den clientseitigen Filter vollständig zu umgehen.
2. Abfangen und Ändern der eingehenden Seite. Mit Burpsuite können wir die eingehende Webseite abfangen und den Javascript-Filter entfernen, bevor er eine Chance hat, zu laufen. Der Prozess dafür wird weiter unten beschrieben
3. Abfangen und Ändern des Dateiuploads. Während die vorherige Methode funktioniert, bevor die Webseite geladen wird, erlaubt diese Methode, dass die Webseite wie gewohnt geladen wird, fängt aber den Datei-Upload ab, nachdem er bereits erfolgt ist (und vom Filter akzeptiert wurde). Auch diese Methode wird im weiteren Verlauf der Aufgabe behandelt.
4. Senden Sie die Datei direkt an den Upload-Punkt. Warum die Webseite mit dem Filter verwenden, wenn Sie die Datei direkt mit einem Tool wie curl senden können? Das direkte Senden der Daten an die Seite, die den Code für die Verarbeitung des Datei-Uploads enthält, ist eine weitere effektive Methode, um einen clientseitigen Filter vollständig zu umgehen. Wir werden diese Methode in diesem Tutorium nicht wirklich ausführlich behandeln, aber die Syntax für einen solchen Befehl würde etwa so aussehen: `curl -X POST -F „submit:<Wert>“ -F „<Datei-Parameter>:@<Pfad-zur-Datei>“ <Seite>`. Um diese Methode zu verwenden, müssten Sie zunächst einen erfolgreichen Upload abfangen (mit Burpsuite oder der Browserkonsole), um die beim Upload verwendeten Parameter zu sehen, die dann in den obigen Befehl eingefügt werden können.

### Praktisches Beispiel 1 -Entfernen des JS codes

Wieder haben wir ein Seite mit Upload vor uns. 
Bei der Erkundung der Seite und des Source Codes sehen wir das im Sourcecode ein Javascript ist welches anhand des MIME Typs die Dateien Kontrolliert. In dem Beispiel schaut es das nur `image/jpeg` hochgeladen wird. siehe unterhalb
https://imgur.com/8G9RWo3

Unser nächster Schritt ist der Versuch, eine Datei hochzuladen - wie erwartet akzeptiert die Funktion ein JPEG, wenn wir es auswählen. Bei allen anderen Dateien wird der Upload abgelehnt.

Nachdem wir dies festgestellt haben, starten wir Burpsuite und laden die Seite neu. Hier sehen wir dann unsere anfrage an den Server und wählen via Rechtsclick auf die Anfrage "Do Intercept" > "Response to this Request" Bild unterhalb

https://imgur.com/czHlAAk

Jetzt können wir den Request durchlassen und erhalten Antwort vom Server. In dieser Antwort können wir das Javascript/die Javascript Funktion löschen die das Filtern übernimmt.

Im Anschluss lässt sich ohne Probleme ein Script wie die Reverse Shell aus dem RCE Teil hochladen.

Wenn wir nun zu http://demo.uploadvulns.thm/uploads/shell.php navigieren, nachdem wir einen netcat-Listener eingerichtet haben, erhalten wir eine Verbindung von der Shell!
#### WICHTIG
Es ist erwähnenswert, dass Burpsuite standardmäßig keine externen Javascript-Dateien abfängt, die von der Webseite geladen werden. Wenn Sie ein Skript bearbeiten müssen, das sich nicht auf der geladenen Hauptseite befindet, müssen Sie auf die Registerkarte „Optionen“ am oberen Rand des Burpsuite-Fensters gehen und dann unter dem Abschnitt „Abfangen von Client-Anfragen“ die Bedingung der ersten Zeile bearbeiten, um `^js$|` zu Entfernen

### Praktisches Beispiel 2- Anpassen des MIME Typs

Wieder haben wir ein Seite mit Upload vor uns. 
Bei der Erkundung der Seite und des Source Codes sehen wir das im Sourcecode ein Javascript ist welches anhand des MIME Typs die Dateien Kontrolliert. In dem Beispiel schaut es das nur `image/jpeg` hochgeladen wird. siehe unterhalb
https://imgur.com/8G9RWo3

Unser nächster Schritt ist der Versuch, eine Datei hochzuladen - wie erwartet akzeptiert die Funktion ein JPEG, wenn wir es auswählen. Bei allen anderen Dateien wird der Upload abgelehnt.

Jetzt kommt der teil der sich ändert. 

Zuerst gehen wir her und ändern die Payload aus `shell.php` zu `shell.jpg`. Dann aktivieren wir Burp und laden die wählen unsere modifizierte "jpg" aus und gehen auf Upload. Hier greift sich Burp dann den Request und diesen müssen wir anpassen indem wir den filename wieder auf shell.php und den Content Type auf text/x-php setzen, siehe unterhalb

https://imgur.com/X5dbMjw

Wenn wir nun zu http://demo.uploadvulns.thm/uploads/shell.php navigieren, nachdem wir einen netcat-Listener eingerichtet haben, erhalten wir eine Verbindung von der Shell!

--- 

# Bypassing Server-Side Filtering: File Extentions

Client-seitige Filter sind leicht zu umgehen - man kann den Code für sie sehen, auch wenn er verschleiert wurde und verarbeitet werden muss, bevor man ihn lesen kann; aber was passiert, wenn man den Code nicht sehen oder manipulieren kann? Nun, das ist ein serverseitiger Filter. Kurz gesagt, wir müssen viele Tests durchführen, um uns ein Bild davon zu machen, was der Filter zulässt und was nicht, und dann nach und nach eine Payload zusammenstellen, die mit den Einschränkungen konform ist.

### Beispiel 1
Im ersten Teil dieser Aufgabe werden wir uns eine Website ansehen, die eine Blacklist für Dateierweiterungen als serverseitigen Filter verwendet. Es gibt eine Vielzahl von Möglichkeiten, wie dies kodiert werden kann, und die Umgehung, die wir verwenden, hängt davon ab. In der realen Welt würden wir den Code dafür nicht sehen können, aber für dieses Beispiel wird er hier eingefügt:

```
<?php  
    //Get the extension  
    $extension = pathinfo($_FILES["fileToUpload"]["name"])["extension"];  
    //Check the extension against the blacklist -- .php and .phtml  
    switch($extension){  
        case "php":  
        case "phtml":  
        case NULL:  
            $uploadFail = True;  
            break;  
        default:  
            $uploadFail = False;  
    }  
?>
```

In diesem Fall sucht der Code nach dem letzten Punkt (.) im Dateinamen und verwendet diesen, um die Erweiterung zu bestätigen, was wir hier also zu umgehen versuchen. Der Code könnte auch nach dem ersten Punkt im Dateinamen suchen oder den Dateinamen an jedem Punkt aufteilen und prüfen, ob irgendwelche Erweiterungen auf der schwarzen Liste auftauchen. Letzteres werden wir später behandeln, aber bis dahin wollen wir uns auf den Code konzentrieren, den wir hier haben.

Wir können sehen, dass der Code die Erweiterungen `.php` und `.phtml` herausfiltert. Wenn wir also ein PHP-Skript hochladen wollen, müssen wir eine andere Erweiterung finden. Auf der Wikipedia-Seite für PHP finden wir einige gängige Erweiterungen, die wir ausprobieren können; es gibt jedoch noch eine Reihe anderer, seltener verwendeter Erweiterungen, die von Webservern möglicherweise trotzdem erkannt werden. Dazu gehören: `.php3, .php4, .php5, .php7, .phps, .php-s, .pht und .phar.` Viele von ihnen umgehen den Filter (der nur .php und .phtml blockiert), aber es scheint, dass der Server so konfiguriert ist, dass er sie nicht als PHP-Dateien erkennt.

Sobald wir ein Format haben das sich uploaden und ausführen lässt bekommen wir unsere Reverse Shell.

### Beispiel 2

