
Syntax Beispiele:

Basic Host Discovery
```
nmap -sP 10.0.0.0/24
```
Genereller Port Scan
```
nmap -p- target
```

Optionen/Pararmeter für Nmap

| Pararmeter      |                                                                                         |
| --------------- | --------------------------------------------------------------------------------------- |
| -sn             | aims to discover live hosts without attempting to discover the services running on them |
| -p [RTange]     | Specifies a range of port numbers – -p- scans all the ports                             |
| -PS [PortListe] | for TCP SYN, TCP ACK, and UDP discovery via the given ports                             |
| -PA [PortListe] | -"-                                                                                     |
| -PU [PortListe] | -"-                                                                                     |
| -sL             | Listet nur die möglichen Hosts eines Netzwerks                                          |
| -F              | is for Fast mode, which scans the 100 most common ports (instead of the default 1000).  |
| -sT             | TCP connect scan – complete three-way handshake                                         |
| -sS             | TCP SYN – only first step of the three-way handshake                                    |
| -sU             | UDP scan                                                                                |
| -O              | OS Detection                                                                            |
| -sV             | Service and Version Detection                                                           |
| -A              | OS detection, version detection, and other additions                                    |
| -Pn             | Scan hosts that appear to be down                                                       |
| -PE             | ICMP Echo requests (Checken was up/down ist)                                            |
| -PP             | ICMP Timestamp oder auch ICMP Adress masken Requests (Checken was up/down ist)          |

| Timing                                                        |               |
| ------------------------------------------------------------- | ------------- |
| Da manche IDS systeme nmap kennzeichnen bei schnelleren scans |               |
| T0 (paranoid)                                                 | 9.8 hours     |
| T1 (sneaky)                                                   | 27.53 minutes |
| T2 (polite)                                                   | 40.56 seconds |
| T3 (normal)                                                   | 0.15 seconds  |
| T4 (aggressive)                                               | 0.13 seconds  |

| Saving Scan Report |                                              |
| ------------------ | -------------------------------------------- |
| -oN <filename>     | - Normal output                              |
| -oX <filename>     | - XML output                                 |
| -oG <filename>     | - grep-able output (useful for grep and awk) |
| -oA <basename>     | - Output in all major formats                |

# Optionen zum Scannen von Zielen

|Scan Type|Example Command|
|---|---|
|ARP Scan|`sudo nmap -PR -sn MACHINE_IP/24`|
|ICMP Echo Scan|`sudo nmap -PE -sn MACHINE_IP/24`|
|ICMP Timestamp Scan|`sudo nmap -PP -sn MACHINE_IP/24`|
|ICMP Address Mask Scan|`sudo nmap -PM -sn MACHINE_IP/24`|
|TCP SYN Ping Scan|`sudo nmap -PS22,80,443 -sn MACHINE_IP/30`|
|TCP ACK Ping Scan|`sudo nmap -PA22,80,443 -sn MACHINE_IP/30`|
|UDP Ping Scan|`sudo nmap -PU53,161,162 -sn MACHINE_IP/30`|

Remember to add `-sn` if you are only interested in host discovery without port-scanning. Omitting `-sn` will let Nmap default to port-scanning the live hosts.

|Option|Purpose|
|---|---|
|`-n`|no DNS lookup|
|`-R`|reverse-DNS lookup for all hosts|
|`-sn`|host discovery only|


### Was Das und Warum wird es genutzt?
Nmap (kurz für Network Mapper) ist ein Typischer Netzwerk Scanner. Dieser kann kann gneutzt werden um Fragen wie 
- Welche Systeme sind mit dem Netzwerk Verbunden?
- Welche Services laufen auf dem System? 
- Welche Ports sind offen?

### Erkennen welche Geräte Up sind
Generell kann auf zwei Arten gescannt werden um zu schauen welche Geräte up sind und welche nicht. 
Es gibt
- arp-scan
- masscan

Wie unterscheiden sich diese und Welche Vorgehensweise macht wann sinn? 

## arp-scan - Seperates Tool
https://www.royhills.co.uk/wiki/index.php/Arp-scan_Documentation
- Funktioniert nur innerhalb lokaler netze auf die Man zugriff hat
- Checkt anhand der ARP Response ob Geräte online sind oder nicht

#### TCP Syn Ping
NMap bietet die Möglichkeit via TCP Syn Pakete zu schauen welche Ports offen sind. Dies Funktioniert indem man dem Host TCP Syn Pakete sendet, auf offenen Ports werden diese dann via SYN Acknloedge bestätigt oder wenn Port geschlossen via Deny abgelehnt.

Syntax
```
nmap -PA -sn *ZielIP*
```

-PS gefolgt von der Portnummer wird genutzt  um Syn Pakete auf einen Speziellen Port zu senden

#### UDP Ping
Wird genutzt um zu schauen welche Hosts Online sind. Dies Funktioniert indem man einen UDP ping an einen Geschlossenen port schickt und dann ein ICMP Paket "unreachable" bekommt. Das ist dann der Hinweis das der Host up and Running ist


Syntax 
```
nmap -PU -sn *ZielIP*
```


#### Masscan
Nebenbei bemerkt, verwendet Masscan einen ähnlichen Ansatz, um die verfügbaren Systeme zu finden. Um den Netzwerkscan schnell zu beenden, ist Masscan jedoch ziemlich aggressiv mit der Rate der erzeugten Pakete. Die Syntax ist recht ähnlich: -p kann von einer Portnummer, einer Liste oder einem Bereich gefolgt werden. 

Beispiele:
```
- masscan MACHINE_IP/24 -p443
- masscan MACHINE_IP/24 -p80,443
- masscan MACHINE_IP/24 -p22-25
- masscan MACHINE_IP/24 ‐‐top-ports 100
```


# Basic Port Scans

Welche Status kann Technisch gesehen ein Port haben
1. Open -zeigt an, dass ein Dienst auf dem angegebenen Port lauscht.
2. Closed -Zeigt an das kein Service auf dem Port Lauscht. Der Port aber Technisch erreihcbar ist nud nicht geflitert wird
3. Filterd - bedeutet, dass Nmap nicht feststellen kann, ob der Port offen oder geschlossen ist, weil der Port nicht erreichbar ist. Dieser Zustand ist normalerweise darauf zurückzuführen, dass eine Firewall Nmap daran hindert, den Port zu erreichen. Es kann sein, dass die Pakete von Nmap den Port nicht erreichen, oder dass die Antworten den Host von Nmap nicht erreichen.
4. Ungefiltert -.edeutet, dass Nmap nicht feststellen kann, ob der Port offen oder geschlossen ist, obwohl der Port zugänglich ist. Dieser Zustand tritt auf, wenn ein ACK-Scan -sA verwendet wird.
5. Offen|Gefiltert -Dies bedeutet, dass Nmap nicht feststellen kann, ob der Port offen oder gefiltert ist.
6. Geschlossen|Gefiltert  -Dies bedeutet, dass Nmap nicht entscheiden kann, ob ein Port geschlossen oder gefiltert ist.+


#### TCP connect port scan
Funktioniert auf Basis des TCP handshakes. Da wir aber beim Scannen nicht mit dem Offenen Port verbunden bleiben wollen wird nach Herstellung der Verbindung diese wieder mit einem RST/Ack Paket gekappt.
Beispiel für den scan mit Syntax:

```
nmap -sT *ZielIP*
```

Wichtig: 
Wenn man keine Admin/root bzw sudo rechte hat kann der Scan nur offene ports erkennen

Zu Beachten ist, dass die Option -F verwendet werden kann, um den Schnellmodus zu aktivieren und die Anzahl der gescannten Ports von 1000 auf 100 häufigste Ports zu reduzieren.

Es ist erwähnenswert, dass die Option -r auch hinzugefügt werden kann, um die Ports in fortlaufender Reihenfolge statt in zufälliger Reihenfolge zu scannen. Diese Option ist nützlich, wenn getestet werden soll, ob die Ports in einer konsistenten Weise geöffnet werden, z. B. beim Hochfahren eines Ziels.



##### TCP SYN Port Scan
Nutzer ohne Admin/sudo rechte sind Beschränkt auf einen Connect Scan. Dieser hat den großen unterschied das er den TCP Handshake nicht vollendet sondern nach der SYN/ACK Antwort des Servers die Verbindung wieder mit einem RST abbricht. 

Das hat den großen Vorteil das weniger Pakete in Generell versendet werden und damit der Scan leiser ist.

```
nmap -sS *Ziel ip*
```

##### UDP port scan

Da UDP ein Protokoll ist das keine Feste Verbindung benötigt braucht es auch keine handshakes etc. Aber meistens bekommt man auf dem UDP dann aber eine Antwort. Wenn ein UDP Paket aber einen geschlossenen Port trifft bekommt der Client eine ICMP Port unreachable meldung. 


```
nmap -sU *Target IP*
```

# Advanced Port Scans 

##### NULL Scan

Ein NULL Scan setzt wie der Name Sagt NULL Flags bits. Ein TCP-Paket, bei dem keine Flags gesetzt sind, löst keine Antwort aus, wenn es einen offenen Port erreicht. Aus der Sicht von Nmap bedeutet das Ausbleiben einer Antwort bei einem Null-Scan also, dass entweder der Port offen ist oder eine Firewall das Paket blockiert.
Wenn aber das Ziel mit einem RST/ACK Paket antwortet ist klar das der Port zu ist. Sprich keine Antwort = Offener Port oder Firewall block, Eine Antwort = Geschlossener Port 

```
sudo nmap -sN *Ziel IP*
```


##### FIN Scan

Der FIN-Scan sendet ein TCP-Paket mit gesetztem FIN-Flag. Ebenso wird hier keine Antwort gesendet, wenn der TCP-Port offen ist. Auch hier kann Nmap nicht sicher sein, ob der Port offen ist oder ob eine Firewall den Verkehr zu diesem TCP-Port blockiert. Folglich können wir wissen, welche Ports geschlossen sind und dieses Wissen nutzen, um auf die offenen oder gefilterten Ports zu schließen. Es sei darauf hingewiesen, dass einige Firewalls den Datenverkehr „stillschweigend“ verwerfen, ohne ein RST zu senden.

```
sudo nmap -sF *Ziel IP*
```

##### Xmas Scan

Der Xmas-Scan hat seinen Namen von der Weihnachtsbaumbeleuchtung. Bei einem Xmas-Scan werden die Flaggen FIN, PSH und URG gleichzeitig gesetzt. 
Wie beim Null-Scan und FIN-Scan bedeutet der Empfang eines RST-Pakets, dass der Anschluss geschlossen ist. Andernfalls wird er als offen|gefiltert gemeldet.

```
sudo nmap -sX *Ziel Ip*
```

##### TCP ACK Scan

Wie der Name schon sagt, wird bei einem ACK-Scan ein TCP-Paket mit gesetztem ACK-Flag gesendet. Logischer weise würde das Ziel auf das ACK mit RST antworten, unabhängig vom Zustand des Ports. Dieses Verhalten kommt zustande, weil ein TCP-Paket mit gesetztem ACK-Flag nur als Antwort auf ein empfangenes TCP-Paket gesendet werden sollte, um den Empfang von Daten zu bestätigen, anders als in unserem Fall. Daher wird uns dieser Scan nicht sagen, ob der Zielport in einem einfachen Setup offen ist.

Diese Art von Scan wäre hilfreich, wenn sich vor dem Ziel eine Firewall befindet. Anhand der ACK-Pakete, die zu Antworten führten, erfährt man, welche Ports nicht von der Firewall blockiert wurden. Mit anderen Worten, diese Art von Scan ist besser geeignet, um Firewall-Regelsätze und Konfigurationen zu ermitteln.

```
sudo nmap -sA *IP Adresse*
```


##### TCP Window Scan

Ein weiterer ähnlicher Scan ist der TCP Window Scan. Der TCP- Window-Scan ist fast dasselbe wie der ACK-Scan, untersucht jedoch das TCP-Fensterfeld der zurückgesendeten RST-Pakete. Auf bestimmten Systemen kann dadurch festgestellt werden, dass der Anschluss offen ist. 

Auch ein TCP-Fenster-Scan gegen ein Linux-System ohne Firewall wird nicht viel Aufschluss geben. Wenn wir jedoch unseren TCP-Fenster-Scan gegen einen Server hinter einer Firewall wiederholen, erhalten wir erwartungsgemäß zufriedenstellendere Ergebnisse

```
sudo nmap -sW *Ziel IP*
```

##### Custom Flag Scan

Wenn man mit einer neuen TCP-Flag-Kombination experimentieren möchte, die über die eingebauten TCP-Scan-Typen hinausgeht, kann man dies mit --scanflags tun. Wenn man zum Beispiel SYN, RST und FIN gleichzeitig setzen möchte, kann man dies mit --scanflags RSTSYNFIN tun.

Abschließend ist anzumerken, dass der ACK-Scan und der Fensterscan sehr effizient bei der Erstellung der Firewall-Regeln waren. Es ist jedoch wichtig, daran zu denken, dass nur weil eine Firewall einen bestimmten Port nicht blockiert, dies nicht unbedingt bedeutet, dass ein Dienst diesen Port abhört.

Es besteht zum Beispiel die Möglichkeit, dass die Firewall-Regeln aktualisiert werden müssen, um die jüngsten Dienständerungen zu berücksichtigen. ACK- und Fensterscans decken daher die Firewall-Regeln auf, nicht die Dienste.

```
sudo nmap --scanflags *Flag auf die Gescannt wird* *Ziel IP *
```