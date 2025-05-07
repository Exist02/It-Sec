
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