---
Website:
---
go
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
Das sieht dann so aus:
```
gobuster dir -u http://10.10.252.123/myfolder -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x.html,.css,.js
```


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


# WPScan

## Einführung in WPScan

Das WPScan-Framework ist in der Lage, einige Kategorien von Sicherheitslücken in WordPress-Websites aufzuzählen und zu untersuchen - einschließlich, aber nicht beschränkt auf:

- Offenlegung sensibler Informationen (Plugin- und Theme-Installationsversionen für offengelegte Schwachstellen oder CVEs)
- Path Discovery (Suche nach falsch konfigurierten Dateiberechtigungen, z. B. wp-config.php)
- Weak Password Policies (Password Bruteforcing)
- Presence of Default Installation (Suche nach Standarddateien)
- Testing Web Application Firewalls (Common WAF plugins)

**Istallation**

Zum Glück für uns ist WPScan auf den neuesten Versionen von Penetrationstestsystemen wie Kali Linux und Parrot vorinstalliert. Wenn man beispielsweise eine ältere Version von Kali Linux (wie 2019) verwendet, befindet sich WPScan im apt-Repository, kann also durch ein einfaches `sudo apt update && sudo apt install wpscan` installiert werden

### WICHTIG
WPScan verwendet Informationen in einer lokalen Datenbank als primären Bezugspunkt für die Enumeration von Themen und Plugins. Eine Technik, die WPScan bei der Enumeration anwendet, ist die Suche nach gemeinsamen Designs und Plugins, auf die wir später noch näher eingehen werden. Bevor man mit WPScan arbeitet, wird dringend empfohlen, diese Datenbank zu aktualisieren, bevor man irgendwelche Scans durchführt.

Zum Glück ist dies ein einfacher Prozess. Indem man einfach `wpscan --update` durchführt

## Enumeration für installierte Designs

WPScan hat einige Methoden, um das aktive Design einer laufenden WordPress-Installation zu bestimmen. Im Grunde genommen handelt es sich um eine Technik, die wir selbst manuell durchführen können. Wir können uns einfach die Assets ansehen, die unser Webbrowser lädt, und dann nach dem Ort dieser Assets auf dem Webserver suchen. Über die Registerkarte „Netzwerk“ in den Entwicklertools Ihres Webbrowsers kann ermittelt werden, welche Dateien geladen werden, wenn man eine Website besucht.

Beispiel:
https://imgur.com/yW0pcNJ

Wenn man dies mit WPScan machen möchte lautet die Syntax wie folgt:

```
wpscan --url http://cmnatics.playground/ --enumerate t
```

- **wpscan** startet den scan
- **--url http://cmnatics.playground/** Spezifiziert die zu scannende URL
- **--enumerate** sorg dafür das enumeriert wird 
- **t** Spezifiziert das die Themes/Designs Enumeriert werden

Das Tolle an WPScan ist, dass das Tool einem mitteilt, wie es die Ergebnisse ermittelt hat. In diesem Fall wird uns gesagt, dass das „twentytwenty“-Theme durch Scannen von „Known Locations“ bestätigt wurde. Das „twentytwenty“-Theme ist das Standard-WordPress-Theme für WordPress-Versionen im Jahr 2020.

## Enumeration von Installierten Plugins

Eine sehr verbreitete Funktion von Webservern ist das „“Directory Listing„“, welches oft standardmäßig aktiviert ist. Einfach ausgedrückt, ist „Directory Listing“ die Auflistung der Dateien in dem Verzeichnis, zu dem wir navigieren (genau so, als ob wir den Windows Explorer oder den Linux-Befehl ls verwenden würden). URLs sind in diesem Zusammenhang den Dateipfaden sehr ähnlich.

Beispiel:
Die URL http://cmnatics.playground/a/directory ist eigentlich das konfigurierte Root-Verzeichnis des Webservers/a/directory.
Ein „Directory Listing“ tritt auf, wenn keine Datei vorhanden ist, die der Webserver verarbeiten soll. Eine sehr häufige Datei ist „index.html“ und „index.php“. Da sich diese Dateien nicht im Verzeichnis /a/directory befinden, wird stattdessen der Inhalt angezeigt.
WPScan kann diese Funktion als eine Technik nutzen, um nach installierten Plugins zu suchen. Da sie sich alle in /wp-content/plugins/pluginname befinden, kann WPScan nach gemeinsamen/bekannten Plugins enumieren. Im folgenden Screenshot wurde „easy-table-of-contents“ entdeckt. Großartig! Dies könnte verwundbar sein. Um das festzustellen, müssen wir die Versionsnummer kennen. Glücklicherweise bekommen wir diese von WordPress auf einem Teller serviert. Wenn wir die WordPress-Entwicklerdokumentation lesen, können wir etwas über "Plugin Readme's" erfahren, um herauszufinden, wie WPScan die Versionsnummer ermittelt hat. Plugins müssen einfach eine "README.txt"-Datei haben. Diese Datei enthält Meta-Informationen wie den Namen des Plugins, die WordPress-Versionen, mit denen es kompatibel ist, und eine Beschreibung.

Kontext mit Bildern:
https://imgur.com/LAW0mUo

Wenn man dies mit WPScan machen möchte lautet die Syntax wie folgt:

```
wpscan --url http://cmnatics.playground/ --enumerate p
```

- **wpscan** startet den scan
- **--url http://cmnatics.playground/** Spezifiziert die zu scannende URL
- **--enumerate** sorg dafür das enumeriert wird 
- **p** Spezifiziert das die Plugins Enumeriert werden

## Enumeration von Usern

Wir haben hervorgehoben, dass WPScan in der Lage ist, Brute-Forcing-Angriffe durchzuführen. Während wir eine Passwortliste wie rockyou.txt zur Verfügung stellen müssen, ist die Art und Weise, wie WPScan die Benutzer enumeriert, interessant und einfach. WordPress-Seiten verwenden Autoren für Beiträge. Autoren sind in der Tat eine Art von Benutzer
Und tatsächlich, diese Autoren werden von unserem WPScan erfasst: 

https://imgur.com/bdySd0X

Syntax
```
wpscan --url http://cmnatics.playground/ --enumerate u
```

- **wpscan** startet den scan
- **--url http://cmnatics.playground/** Spezifiziert die zu scannende URL
- **--enumerate** sorg dafür das enumeriert wird 
- **u** Spezifiziert das die User Enumeriert werden

## Die „Vulnerable“ Flagge

In den bisherigen Befehlen haben wir nur WordPress enumeriert, um herauszufinden, welche Themes, Plugins und Benutzer vorhanden sind. Im Moment müssten wir uns die Ausgabe ansehen und Websites wie MITRE, NVD und CVEDetails nutzen, um die Namen dieser Plugins und die Versionsnummern zu ermitteln, um eventuelle Schwachstellen festzustellen.

WICHTIG 
Damit das machbar ist muss WPScan Konfiguriert sein um mir der WPVuln-DB zu Verbinden guide unterhalb:
https://wpscan.com/api/

Syntax:
```
wpscan --url http://cmnatics.playground/ --enumerate vp
```

- **wpscan** startet den scan
- **--url http://cmnatics.playground/** Spezifiziert die zu scannende URL
- **--enumerate** sorg dafür das enumeriert wird 
- **vp** Spezifiziert das nach schwachstellen geschaut werden soll

## Passwort Bruteforcing

Nachdem wir eine Liste möglicher Benutzernamen auf der WordPress-Installation ermittelt haben, können wir WPScan verwenden, um eine Brute-Forcing-Technik gegen den von uns angegebenen Benutzernamen und eine von uns bereitgestellte Passwortliste durchzuführen

```
wpscan –-url http://cmnatics.playground –-passwords rockyou.txt –-usernames cmnatic
```

- **wpscan** startet WPScan
- **--url http://cmnatics.playground/** Spezifiziert die zu Ziel URL
- **--passwords rockyou.txt** Spezifiziert welche Passwortliste genutzt wird
- **--usernames cmnatic** Spezifiziert das der User cmnatic geburteforced werden soll


## Einstellen der Aggressivität von WPScan (WAF)

Wenn nicht anders angegeben, versucht WPScan, so wenig "Lärm" wie möglich zu machen. Viele Anfragen an einen Webserver können Dinge wie Firewalls auslösen und letztendlich dazu führen, dass Sie vom Server blockiert werden.

Das bedeutet, dass einige Plugins und Themen von unserem WPScan übersehen werden können. Glücklicherweise können wir Argumente wie `--plugins-detection` und ein Aggressionsprofil (passiv/aggressiv) verwenden, um dies zu spezifizieren. Zum Beispiel: `--plugins-detection aggressive` oder `--plugins-detection-passive`


Summary - Cheatsheet 

|   |   |   |
|---|---|---|
|Flag|Description|Full Example|
|p|Enumerate Plugins|--enumerate p|
|t|Enumerate Themes|--enumerate t|
|u|Enumerate Usernames|--enumerate -u|
|v|Use WPVulnDB to cross-reference for vulnerabilities. Example command looks for vulnerable plugins (p)|--enumerate vp|
|aggressive|This is an aggressiveness profile for WPScan to use.|--plugins-detection aggressive|


# Nikto

## Was das
Nikto wurde ursprünglich im Jahr 2001 veröffentlicht und hat sich im Laufe der Jahre zu einem sehr beliebten Schwachstellen-Scanner entwickelt, da es sich um ein Open-Source-Programm mit vielen Funktionen handelt. Nikto ist in der Lage, alle Arten von Webservern zu überprüfen (und ist nicht anwendungsspezifisch wie z.B. WPScan.). Nikto kann verwendet werden, um mögliche Schwachstellen zu entdecken, darunter:

- Sensible Dateien
- Veraltete Server und Programme (z.B. anfällige Webserver-Installationen)
- Übliche Server- und Software-Fehlkonfigurationen (Verzeichnisindizierung, cgi-Skripte, x-ss-Schutz)
## Installation

Zum Glück für uns ist Nikto auf den neuesten Versionen von Penetrationstestsystemen wie Kali Linux und Parrot vorinstalliert. Wenn man zum Beispiel eine ältere Version von Kali Linux (wie 2019) verwendet, ist Nikto im apt-Repository enthalten und kann durch ein einfaches `sudo apt update && sudo apt install nikto` installiert werden

## Basic Scanning

Der einfachste Scan kann mit dem Flag `-h` und der Angabe einer IP-Adresse oder eines Domänennamens als Argument durchgeführt werden. Bei diesem Scantyp werden die vom Webserver oder der Anwendung (z. B. Apache2, Apache Tomcat, Jenkins oder JBoss) angekündigten Header abgerufen und es wird nach sensiblen Dateien oder Verzeichnissen gesucht (z. B. login.php, /admin/ usw.)

### Syntax 

```
 nikto -h vulnerable_ip
```

- **nikto** startet nikto
- **-h** Spezifiziert das die Header abgegriffen werden
- **vunerable_ip** spezifiziert das Ziel des Scans

Beispiel: 
https://imgur.com/wqOoAPe

## Scannen von Mehreren Hosts & Ports

Nikto ist in dem Sinne umfangreich, dass wir mehrere Argumente angeben können, ähnlich wie bei Tools wie Nmap. Wir können sogar Eingaben direkt von einem Nmap-Scan übernehmen, um einen Hostbereich zu scannen. Indem wir ein Subnetz scannen, können wir nach Hosts in einem ganzen Netzwerkbereich suchen. Wir müssen Nmap anweisen, einen Scan in einem Format auszugeben, das von Nikto gelesen werden kann, indem wir Nmaps -oG-Flags verwenden

  
Wir können zum Beispiel 172.16.0.0/24 (Subnetzmaske 255.255.255.0, was 254 mögliche Hosts ergibt) mit Nmap scannen (unter Verwendung des Standard-Webports 80) und die Ausgabe für Nikto wie folgt parsen: `nmap -p80 172.16.0.0/24 -oG - | nikto -h -`

Es gibt nicht viele Umstände, unter denen man diese Funktion verwenden würde, außer wenn man sich Zugang zu einem Netzwerk verschafft hat. Ein viel häufigeres Szenario ist das Scannen mehrerer Ports auf einem bestimmten Host. Dies kann mit der Option `-p` und der Angabe einer durch ein Komma getrennten Liste von Portnummern geschehen, z. B.: `nikto -h 10.10.10.1 -p 80,8000,8080`

## Plugins
Plugins erweitern die Möglichkeiten von Nikto. Anhand der Informationen, die wir bei unseren Basis-Scans gesammelt haben, können wir die Plugins auswählen, die für unser Ziel geeignet sind. Sie können Nikto mit dem Flag `--list-plugins` verwenden, um die Plugins aufzulisten oder die gesamte Liste in einem leichter zu lesenden Format online anzusehen.

  https://github.com/sullo/nikto/wiki/Plugin-list

Einige interessante Plugins sind:

| Plugin Name   | Description                                                                                                                                                                           |
| ------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| apacheusers   | Attempt to enumerate Apache HTTP Authentication Users                                                                                                                                 |
| cgi           | Look for CGI scripts that we may be able to exploit                                                                                                                                   |
| robots        | Analyse the robots.txt file which dictates what files/folders we are able to navigate to                                                                                              |
| dir_traversal | Attempt to use a directory traversal attack (i.e. LFI) to look for system files such as /etc/passwd on Linux (http://ip_address/application.php?view=../../../../../../../etc/passwd) |

Syntax

```
nikto -h 10.10.10.1 -Plugin apacheuser
```
- **nikto** startet nikto
- **-h** Spezifiziert das die Header abgegriffen werden (notwendig)
- **10.10.10.1** spezifiziert das Ziel des Scans
- **-Plugin** Spezifiziert das ein Plugin genutzt wird
- **apacheuser** Beispiel Plugin

## Verbosing our Scan/Mehr Infos


Wir können die Ausführlichkeit unseres Nikto-Scans erhöhen, indem wir die folgenden Argumente mit dem Flag `-Display` angeben. Wenn nicht angegeben, ist die von Nikto ausgegebene Ausgabe nicht die gesamte Ausgabe, da sie manchmal irrelevant sein kann (was aber nicht immer der Fall ist!)

| Argument | Description                                          | Reasons for Use                                                                                                                                                                                                         |
| -------- | ---------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1        | Show any redirects that are given by the web server. | Web servers may want to relocate us to a specific file or directory, so we will need to adjust our scan accordingly for this.                                                                                           |
| 2        | Show any cookies received                            | Applications often use cookies as a means of storing data. For example, web servers use sessions, where e-commerce sites may store products in your basket as these cookies. Credentials can also be stored in cookies. |
| E        | Output any errors                                    | This will be useful for debugging if your scan is not returning the results that you expect!                                                                                                                            |
## Abstimmung des Scans auf die Suche nach Sicherheitslücken

Nikto verfügt über mehrere Kategorien von Schwachstellen, die wir in unserem Scan auflisten und auf die wir testen können. Die folgende Liste ist nicht sehr umfangreich und enthält nur die Kategorien, die Sie möglicherweise häufig verwenden. Wir können das `-Tuning`-Flag verwenden und einen Wert in unserem Nikto-Scan angeben:

| Category Name                     | Description                                                                                                                                                        | Tuning Option |
| --------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------- |
| File Upload                       | Search for anything on the web server that may permit us to upload a file. This could be used to upload a reverse shell for an application to execute.             | 0             |
| Misconfigurations / Default Files | Search for common files that are sensitive (and shouldn't be accessible such as configuration files) on the web server.                                            | 2             |
| Information Disclosure            | Gather information about the web server or application (i.e. verison numbers, HTTP headers, or any information that may be useful to leverage in our attack later) | 3             |
| Injection                         | Search for possible locations in which we can perform some kind of injection attack such as XSS or HTML                                                            | 4             |
| Command Execution                 | Search for anything that permits us to execute OS commands (such as to spawn a shell)                                                                              | 8             |
| SQL Injection                     | Look for applications that have URL parameters that are vulnerable to SQL Injection                                                                                | 9             |
##  Speichern der Ergebnisse

Anstatt mit der Ausgabe auf dem Terminal zu arbeiten, können wir sie zur weiteren Analyse direkt in eine Datei ausgeben - das macht unser Leben viel einfacher! Nikto ist in der Lage, eine Reihe von Dateiformaten zu speichern, darunter:

- Textdatei
- HTML-Bericht

Wir können das Argument `-o` (kurz für -Output) verwenden und sowohl einen Dateinamen als auch eine kompatible Erweiterung angeben. Wir können das Format (-f) spezifisch angeben, aber Nikto ist intelligent genug, um die Erweiterung zu verwenden, die wir im Argument `-o` angeben, um die Ausgabe entsprechend anzupassen.

  

Ein Beispiel: Wir scannen einen Webserver und geben dies in "report.html" aus: 

```
nikto -h http://ip_address -o report.html
```
