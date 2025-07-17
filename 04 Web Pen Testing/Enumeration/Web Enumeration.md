
# Manuelle Enumeration

Wir müssen nicht von Anfang an ein ausgefallenes Toolkit auspacken. In den meisten Fällen bringt es mehr Ergebnisse, wenn wir unsere eigene Initiative gegenüber automatischen Scans einsetzen. So können wir beispielsweise das „goldene Ticket“ finden, ohne viel Lärm zu machen.

## Verwendung der Browserkonsole für Entwickler
Moderne Browser wie Chrome und Firefox verfügen über eine Reihe von Tools, die sich in der "Developer Tools/Console" befinden. Wir werden uns mit Firefox befassen, aber auch Chrome hat eine sehr ähnliche Suite. Dieses Paket umfasst eine Reihe von Tools, darunter:
- Anzeigen des Seitenquellcodes (alternativ via "view-source:*URL"
- Auffinden von Assets
- Debuggen und Ausführen von Code wie Javascript auf der Client-Seite (unserem Browser)

 Mit der Tastenkombination "F12" auf unserer Tastatur können Sie diese Tools starten.

## Kontrolle des Zertifikats
In Zertifikaten von Webistes können manchmal aufschlussreiche Informationen oder Hinweise versteckt sein.  

# GoBuster

## Was das?
- Ist ein Opensource tool        
- Genutzt für Enumeration und Bruteforce

Enumeration von 
- web directories, DNS subdomains, vhosts, Amazon S3 buckets, and Google Cloud Storage by brute force, using specific wordlists and handling the incoming responses    

nach MITRE Gesetzt zwischen reconnaissance and scanning phases
## Globale Flags 

Es gibt einige nützliche globale Flaggen, die ebenfalls verwendet werden können. Ich habe sie in der Tabelle unten aufgeführt.
Haupt github Doku: 
https://github.com/OJ/gobuster


| Flag | Long Flag                | Beschreibung                                               |
| ---- | ------------------------ | ---------------------------------------------------------- |
| -c   | --cookies                | Zu verwendende Cookies für Anfragen                        |
| -x   | --extentions             | Zu suchende Dateierweiterung(en)                           |
| -H   | --headers                | HTTP-Header angeben, -H ‚Header1: val1‘ -H 'Header2: val2' |
| -k   | --no-tls-validation      | TLS-Zertifikatsprüfung überspringen                        |
| -n   | --no-status              | Statuscodes nicht ausdrucken                               |
| -P   | --password               | Password for Basic Auth                                    |
| -s   | --status-codes           | Positive Statuscodes                                       |
| -b   | --status-codes-blacklist | --status-codes-blacklist                                   |
| -U   | --username               | Benutzername für Basic Auth                                |
## die -k Flagge

Die -k Flag ist etwas Besonderes, da es bei Penetrationstests eine wichtige Rolle spielt und die Flag-Ereignisse aufzeichnet. In einem Capture-the-Flag-Raum auf TryHackMe zum Beispiel, wenn HTTPS aktiviert ist, werden mit großer Wahrscheinlichkeit ungültige Zertifikatsfehler wie der folgende auftreten
https://imgur.com/wWH2XJC
Wenn man in solchen Fällen versucht, Gobuster ohne die -k Flag auszuführen, wird es nichts herausfinden und höchstwahrscheinlich einen Fehler ausgeben. Aber keine Sorge, das lässt sich leicht beheben! Es reicht, wenn man dem Scan die -k Flag hinzufügt, um die ungültige Zertifizierung zu umgehen und mit dem Scannen fortzufahren.
## Hinweis: Dieses Flag kann mit den Modi „dir“ und „vhost“ verwendet werden.


## "dir" Mode

Dirbuster verfügt über einen „dir“-Modus, der es dem Benutzer ermöglicht, die Verzeichnisse einer Website aufzuzählen. Dies ist nützlich, wenn man einen Penetrationstest durchführt und sehen möchte, wie die Verzeichnisstruktur einer Website aussieht. Oft folgen die Verzeichnisstrukturen von Websites und Webanwendungen einer bestimmten Konvention, was sie anfällig für Brute-Forcing mit Wortlisten macht. z.B. WordPress verwendet eine sehr spezifische Verzeichnisstruktur für seine Websites.

Gobuster ist sehr leistungsfähig, weil es einem nicht nur erlaubt, die Website zu scannen, sondern auch die Statuscodes zurückgibt. So weiß man sofort, ob man als Außenstehender dieses Verzeichnis abrufen kann oder nicht. Eine weitere Funktionalität von Gobuster ist die Möglichkeit, mit einem einfachen Flag nach Dateien zu suchen!

### Syntax

```
gobuster dir -u http://10.10.10.10 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

Befehl erklärung: 

- gobuster dir -> Spezifiziert das Gobuster im Directory Modus läuft
- -u "url" -> Spezifiziert Ziel URL
- - w "Path to Wordlist" -> Spezifiziert welche Wordlist genutzt werden soll

Häufig wird der "dir" modus auch mit der `-k` Flagge oder in der langen version `--extentions` flagge genutzt um nach dem Inhalt von Verzeichnissen zu suchen, die zuvor mit einer Liste von Dateierweiterungen aufgezählt wurden. Die Dateierweiterungen sind im Allgemeinen repräsentativ für die Daten, die sie enthalten können. So enthalten beispielsweise `.conf`- oder `.config`-Dateien in der Regel Konfigurationen für die Anwendung - einschließlich sensibler Informationen wie Datenbankanmeldeinformationen.

Einige andere Dateien, nach denen Sie vielleicht suchen möchten, sind `.txt`-Dateien oder andere Webanwendungsseiten wie `.html` oder `.php` . Stellen wir nun einen Befehl zusammen, mit dem wir das Verzeichnis „myfolder“ auf einem Webserver nach den folgenden drei Dateien durchsuchen könnten:
1. html
2. js
3. css

## "dns" Mode

Der nächste Modus, auf den wir uns konzentrieren werden, ist der „dns“-Modus. Dieser Modus ermöglicht es Gobuster, Subdomains zu brute-forcen. Während eines Penetrationstests (oder Capture the Flag) ist es wichtig, die Subdomänen der Hauptdomäne des Ziels zu überprüfen. Nur weil etwas in der regulären Domain gepatcht ist, bedeutet das nicht, dass es auch in der Subdomain gepatcht ist. Möglicherweise gibt es in einer dieser Unterdomänen eine Sicherheitslücke, die Sie ausnutzen können. Wenn State Farm beispielsweise statefarm.com und mobile.statefarm.com besitzt, kann es in mobile.statefarm.com eine Lücke geben, die in statefarm.com nicht vorhanden ist. Aus diesem Grund ist es wichtig, auch nach Subdomains zu suchen!

### Syntax

```
gobuster dns -d mydomain.thm -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt
```

- gobuster dns -> Spezifiziert das Gobuster im dns Modus läuft
- -u "url" -> Spezifiziert Ziel URL
- - w "Path to Wordlist" -> Spezifiziert welche Wordlist genutzt werden soll

### Zusatz Flags

| Flag | Long Flag    | Beschreibung                                                                      |
| ---- | ------------ | --------------------------------------------------------------------------------- |
| -c   | --show-cname | CNAME-Einträge anzeigen (kann nicht mit der Option ‚-i‘ verwendet werden)         |
| -i   | --show-ip    | Show IP Addresses                                                                 |
| -r   | --resolver   | Benutzerdefinierten DNS-Server verwenden (Format server.com oder server.com:port) |

## "vhost" Mode

Der letzte und abschließende Modus, auf den wir uns konzentrieren werden, ist der „vhost“-Modus. Er ermöglicht es Gobuster, virtuelle Hosts zu brute-forcen. Virtuelle Hosts sind verschiedene Websites auf demselben Rechner laufen. In manchen Fällen können sie wie Subdomains aussehen! Virtuelle Hosts sind IP-basiert und laufen auf demselben Server. Dies ist für den Endbenutzer in der Regel nicht ersichtlich. Es kann sich jedoch lohnen, Gobuster in diesem Modus zu starten, um zu sehen, ob es etwas findet. Man kann nie wissen, vielleicht findet es ja etwas! Bei der Teilnahme an Räumen auf TryHackMe sind virtuelle Hosts eine gute Möglichkeit, eine völlig andere Website zu verstecken, wenn bei Ihrem Hauptscan auf Port 80/443 nichts gefunden wird.

### Syntax

```
gobuster vhost -u http://example.com -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt
```

- gobuster vhost -> Spezifiziert das Gobuster im vhost Modus läuft
- -u "url" -> Spezifiziert Ziel URL
- - w "Path to Wordlist" -> Spezifiziert welche Wordlist genutzt werden soll

## Wordlists

Kali Linux Standardlisten
Nachfolgend finden Sie eine nützliche Liste von Wortlisten, die unter Kali Linux standardmäßig installiert sind. Diese Liste entspricht der aktuellen Version 2020.3, die zum Zeitpunkt der Erstellung dieses Artikels vorliegt. Alles, was einen Platzhalter (*) enthält, bedeutet, dass es mehr als eine passende Liste gibt. Denken Sie daran, dass viele davon zwischen den Modi ausgetauscht werden können. Zum Beispiel enthalten Wortlisten im "dir"-Modus (wie die aus dem dirbuster-Verzeichnis) Wörter wie "admin", "index", "about", "events", usw. Viele dieser Wörter könnten auch Subdomains sein. Probieren Sie sie mit den verschiedenen Modi aus!


/usr/share/wordlists/dirbuster/directory-list-2.3-*.txt
/usr/share/wordlists/dirbuster/directory-list-1.0.txt
/usr/share/wordlists/dirb/big. txt
/usr/share/wordlists/dirb/common.txt
/usr/share/wordlists/dirb/small.txt
/usr/share/wordlists/dirb/extensions_common.txt - Nützlich beim Fuzzing nach Dateien!

Nicht-Standard-Listen

Zusätzlich zu den oben genannten hat Daniel Miessler ein erstaunliches GitHub-Repositorium namens SecLists erstellt. Es stellt viele verschiedene Listen zusammen, die für viele verschiedene Dinge verwendet werden. Das Beste daran ist, dass es in apt enthalten ist! Du kannst `sudo apt install seclists` ausführen und erhältst das gesamte Repo! Wir werden nicht in andere Listen eintauchen, da es viele davon gibt. Allerdings bezweifle ich, dass du zwischen dem, was standardmäßig auf Kali installiert ist, und dem SecLists-Repository etwas anderes brauchst.