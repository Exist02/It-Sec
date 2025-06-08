# Enumeration
Die Enumeration ist der erste Schritt, den man tun muss, wenn man sich Zugang zu einem beliebigen System verschafft hat. Möglicherweise hat man sich Zugang zum System verschafft, indem man eine kritische Schwachstelle ausgenutzt hat, die zu Root-Zugang führte, oder man hat einfach einen Weg gefunden, Befehle über ein Konto mit geringen Privilegien zu senden. Im Gegensatz zu CTF-Maschinen enden Penetrationstests nicht, sobald man Zugriff auf ein bestimmtes System oder eine bestimmte Benutzerebene erhalten hat.

Enumeration ist nach dem Exploiting immernoch genauso wichtig wie vorher!

Befehle zur Enumeration innerhalb eines Systems:


| Befehl/Verzeichniss | Infos die der Ausgibt                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| hostname            | gibt den Hostname des Geräts an.Obwohl dieser Wert leicht geändert werden kann oder eine relativ bedeutungslose Zeichenfolge ist (z. B. Ubuntu-3487340239), kann er in einigen Fällen Informationen über die Rolle des Zielsystems innerhalb des Unternehmensnetzwerks liefern (z. B. SQL-PROD-01 für einen SQL-Produktionsserver)                                                                                                                                                          |
| uname -a            | Gibt Systeminformationen aus, die uns zusätzliche Details über den vom System verwendeten Kernel liefern. Dies ist nützlich bei der Suche nach möglichen Kernel-Schwachstellen, die zu einer Privilegienausweitung führen könnten.                                                                                                                                                                                                                                                          |
| /proc/version       | Das proc-Dateisystem (procfs) liefert Informationen über die Prozesse des Zielsystems.  Man findet proc in vielen verschiedenen Linux-Varianten, was es zu einem unverzichtbaren Werkzeug in Ihrem Arsenal macht.<br><br>Ein Blick auf /proc/version kann Informationen über die Kernelversion und zusätzliche Daten liefern, z. B. ob ein Compiler (z. B. GCC) installiert ist.                                                                                                            |
| /etc/issue          | Systeme können auch durch einen Blick in die Datei /etc/issue identifiziert werden. Diese Datei enthält normalerweise einige Informationen über das Betriebssystem, kann aber leicht angepasst oder geändert werden. Apropos, jede Datei mit Systeminformationen kann angepasst oder geändert werden. Für ein besseres Verständnis des Systems ist es immer gut, sich alle diese Dateien anzusehen.                                                                                         |
| env                 | Zeigt die Variabeln des enviorments                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| sudo -l             | Das Zielsystem kann so konfiguriert sein, dass die Benutzer einige (oder alle) Befehle mit Root-Rechten ausführen können. Der Befehl sudo -l kann verwendet werden, um alle Befehle aufzulisten, die Ihr Benutzer mit sudo ausführen kann.                                                                                                                                                                                                                                                  |
| Id                  | Ermöglicht einen Generellen Überblick über die Privilegien des aktuellen Benutzers                                                                                                                                                                                                                                                                                                                                                                                                          |
| /etc/passwd         | Das Lesen der Datei /etc/passwd kann ein einfacher Weg sein, um Benutzer auf dem System zu entdecken. Die Ausgabe kann zwar lang und etwas einschüchternd sein, aber man kann sie leicht kürzen und in eine nützliche Liste für Brute-Force-Angriffe umwandeln. Bedenken muss man jedoch, dass die Datei /etc/passwd nicht sehr umfangreich ist. <br>Ein anderer Ansatz könnte sein, nach „home“ zu suchen, da echte Benutzer ihre Ordner höchstwahrscheinlich im „home“-Verzeichnis haben. |
| history             | Wenn man sich frühere Befehle mit dem Befehl history ansieht, kann man sich ein Bild über das Zielsystem machen und, wenn auch selten, Informationen wie Kennwörter oder Benutzernamen speichern.                                                                                                                                                                                                                                                                                           |
| ifconfig            | Das Zielsystem kann ein Dreh- und Angelpunkt zu einem anderen Netzwerk sein. Der Befehl ifconfig liefert uns Informationen über die Netzwerkschnittstellen des Systems.                                                                                                                                                                                                                                                                                                                     |

Außerhalb der Tabelle da mehrere Optionen vorhanden

##### ps Command
Der Befehl ps ist eine effektive Methode, um die laufenden Prozesse auf einem Linux-System zu sehen. Wenn man ps in das Terminal eingibt, werden die Prozesse der aktuellen Shell angezeigt.

Die Ausgabe von ps (Prozessstatus) zeigt Folgendes an;
- PID: Die Prozess-ID (eindeutig für den Prozess)
- TTY: Der vom Benutzer verwendete Terminaltyp
- Time: Die vom Prozess verbrauchte CPU-Zeit (dies ist NICHT die Zeit, die der Prozess bereits läuft
- CMD: Der Befehl oder die ausführbare Datei, die gerade ausgeführt wird (es werden KEINE Befehlszeilenparameter angezeigt)

Der Befehl „ps“ bietet ein paar nützliche Optionen.
- ps -A: Alle laufenden Prozesse anzeigen
- ps axjf: Prozessbaum anzeigen (siehe die Baumbildung bis zur Ausführung von ps axjf weiter unten)
- ps aux: Die Option aux zeigt Prozesse für alle Benutzer (a), den Benutzer, der den Prozess gestartet hat (u), und Prozesse, die nicht an ein Terminal angeschlossen sind (x), an. Wenn wir uns die Ausgabe des Befehls ps aux ansehen, können wir ein besseres Verständnis des Systems und möglicher Schwachstellen gewinnen.

##### netstat
Nach einer ersten Überprüfung der vorhandenen Schnittstellen und Netzwerkrouten lohnt es sich, die bestehende Kommunikation zu untersuchen. Der Befehl netstat kann mit verschiedenen Optionen verwendet werden, um Informationen über bestehende Verbindungen zu sammeln.

- netstat -a: zeigt alle lauschenden Ports und bestehenden Verbindungen an.
- netstat -at oder netstat -au können auch verwendet werden, um TCP- bzw. UDP-Protokolle aufzulisten.
- netstat -l: listet Ports im „Abhör“-Modus auf. Diese Ports sind offen und bereit, eingehende Verbindungen zu akzeptieren. Dies kann mit der Option „t“ verwendet werden, um nur Ports aufzulisten, die das TCP-Protokoll abhören (siehe unten)
- netstat -tp: Auflistung von Verbindungen mit dem Namen des Dienstes und PID-Informationen; kann auch mit der Option -l verwendet werden, um lauschende Ports aufzulisten
- netstat -i: Zeigt Schnittstellenstatistiken an. Wir sehen unten, dass „eth0“ und „tun0“ aktiver sind als „tun1“.

Die netstat-Anwendung, die Sie wahrscheinlich am häufigsten in Blog-Beiträgen, Aufsätzen und Kursen sehen werden, ist netstat -ano, die sich wie folgt aufschlüsseln lässt;
-a: Alle Sockets anzeigen
-n: Namen nicht auflösen
-o: Timer anzeigen


##### find
Unterhalb Optionen und Funktionen dieser

Syntax

```
find *Option aus beispiel unterhalb*
```

| Option                                | Was es tut                                                                                                                       |
| ------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| . -name flag1.txt                     | Sucht nach der  "flag1.txt" datei im aktuellen Verzeichniss                                                                      |
| /home -name flag1.txt                 | Sucht nach der "flag1.txt" Datei im /home Verzeichniss                                                                           |
| / -type d -name config                | sucht das Verzeichnis namens config unter „/“                                                                                    |
| / -type f -perm 0777                  | sucht nach Dateien mit der Berechtigung 777 (Dateien, die von allen Benutzern gelesen, geschrieben und ausgeführt werden können) |
| / -perm a=x                           | findet ausführbare Dateien                                                                                                       |
| /home -user frank                     | findet alle Dateien für den Benutzer „frank“ unter „/home“                                                                       |
| / -mtime 10                           | findet Dateien, die in den letzten 10 Tagen geändert wurden                                                                      |
| / -atime 10                           | findet Dateien, auf die in den letzten 10 Tagen zugegriffen wurde                                                                |
| / -cmin -60                           | findet Dateien, die in der letzten Stunde (60 Minuten) geändert wurden                                                           |
| / -amin -60                           | Findet Dateien, auf die innerhalb der letzten Stunde (60 Minuten) zugegriffen wurde                                              |
| / -size 50M                           | findet Dateien mit einer Größe von 50 MB                                                                                         |
| / -writable -type d 2>/dev/null       | Suche nach global beschreibbaren Ordnern                                                                                         |
| / -perm -222 -type d 2>/dev/null      | Suche nach global beschreibbaren Ordnern                                                                                         |
| / -perm -o w -type d 2>/dev/null      | Suche nach global beschreibbaren Ordnern                                                                                         |
| find / -perm -o x -type d 2>/dev/null | Suche nach global aussführbaren Ordnern                                                                                          |
|                                       |                                                                                                                                  |
|                                       |                                                                                                                                  |


# Automatische Enumeration Tools

Verschiedene Tools können dabei helfen, während des Enumeration-Prozesses Zeit zu sparen. Diese Tools sollten nur verwendet werden, um Zeit zu sparen, da sie einige Vektoren zur Privilegienerweiterung übersehen können. Im Folgenden findet sich eine Liste beliebter Linux-Enumeration-Tools mit Links zu den jeweiligen Github-Repositories.

Die Umgebung des Zielsystems hat Einfluss auf das Tool, das man verwenden kann. So kann zum Beispiel ein in Python geschriebenes Tool nicht ausgeführt werden, wenn es nicht auf dem Zielsystem installiert ist. Aus diesem Grund ist es besser, mit einigen wenigen Tools vertraut zu sein, als ein einziges zu haben.

- **LinPeas**: [https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS)
- **LinEnum:** [https://github.com/rebootuser/LinEnum](https://github.com/rebootuser/LinEnum)[](https://github.com/rebootuser/LinEnum)
- **LES (Linux Exploit Suggester):** [https://github.com/mzet-/linux-exploit-suggester](https://github.com/mzet-/linux-exploit-suggester)
- **Linux Smart Enumeration:** [https://github.com/diego-treitos/linux-smart-enumeration](https://github.com/diego-treitos/linux-smart-enumeration)
- **Linux Priv Checker:** [https://github.com/linted/linuxprivchecker](https://github.com/linted/linuxprivchecker)

# Privilegein Eskalierung: Kernel Exploits

Die Eskalation von Privilegien führt im Idealfall zu Root-Rechten. Dies kann manchmal einfach durch das Ausnutzen einer bestehenden Schwachstelle erreicht werden, oder in einigen Fällen durch den Zugriff auf ein anderes Benutzerkonto, das über mehr Privilegien, Informationen oder Zugriff verfügt.

Sofern nicht eine einzige Sicherheitslücke zu einer Root-Shell führt, beruht der Prozess der Privilegienerweiterung auf Fehlkonfigurationen und lockeren Berechtigungen.
Der Kernel auf Linux-Systemen verwaltet die Kommunikation zwischen Komponenten wie dem Speicher des Systems und Anwendungen. Für diese kritische Funktion benötigt der Kernel bestimmte Privilegien; ein erfolgreicher Exploit führt daher potenziell zu Root-Rechten.

Die Methodik zur Ausnutzung des Kernels ist einfach;
- Identifizieren der Kernel-Version
- Suchen und finden von Exploit-Code für die Kernel-Version des Zielsystems
- Ausführen des Exploits

Auch wenn es einfach aussieht, denken Sie bitte daran, dass ein fehlgeschlagener Kernel-Exploit zu einem Systemabsturz führen kann. Vergewissern Sie sich, dass dieses potenzielle Ergebnis im Rahmen Ihres Penetrationstestauftrags akzeptabel ist, bevor Sie einen Kernel-Exploit versuchen.

Mögliche Ressourcen:
1. Based on your findings, you can use Google to search for an existing exploit code.
2. Sources such as https://www.cvedetails.com/ can also be useful.
3. Another alternative would be to use a script like LES (Linux Exploit Suggester) but remember that these tools can generate false positives (report a kernel vulnerability that does not affect the target system) or false negatives (not report any kernel vulnerabilities although the kernel is vulnerable).


# Privilegien Eskalieren: Sudo
