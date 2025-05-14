
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







8. TCP connect port scan
9. TCP SYN port scan
10. UDP port scan
