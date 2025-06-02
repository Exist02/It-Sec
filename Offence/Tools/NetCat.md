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


### Technik 2: rlwrap
### Technik 3: Socat
