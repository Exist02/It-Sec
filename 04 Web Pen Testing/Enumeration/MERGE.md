

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
