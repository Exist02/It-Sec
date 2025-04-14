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

### LFI

Kann wie Path Traversal die dot dot Slash methode Benutzen. Kann aber auch schon jenachdem einfachere Variante nutzen 


wie z.B. die gesuchte Datei an die URL anheften. Dies funktioniert wenn im Query code kein Direcory Spezifiziert ist. 
Bsp.
http://10.10.208.102//lab1.php?file=/etc/passwd


Wenn ein Directory Spezifiziert ist kann die Dot Dot Slash methode verwendet werden. 
Bsp.:
http://10.10.208.102/lab2.php?file=../../../../etc/passwd

