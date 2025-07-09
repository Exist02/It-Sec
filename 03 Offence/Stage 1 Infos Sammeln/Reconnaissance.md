Warum ist das wichtig und was hat es damit auf Sich? 
 
Vor den Anfängen der Computersysteme und -netze lehrte Sun Tzu in der Kunst des Krieges: „Wenn du den Feind kennst und dich selbst kennst, wird dein Sieg nicht in Zweifel stehen.“ Wenn du die Rolle eines Angreifers spielst, musst du Informationen über deine Zielsysteme sammeln. Wenn du die Rolle des Verteidigers spielst, musst du wissen, was dein Gegner über deine Systeme und Netzwerke herausfinden wird.

Generell wird die Reconnaissance in zwei Teile Aufgeteilt
	1.Active Reconnaissance 
	2. Passive Reconnaissance

Was ist hier der Unterschied? 
- Passive Reconnaissance
	- Hier werden alle Infos gesammelt die Öffentlich zugänglich sind ohne das man direkt mit dem Ziel Interagieren muss 
	- Kann man sich vorstellen wie Mit einem Fernglas in das Feindgebiet zu spähen ohne aber dieses zu Betreten
	- Beinhaltet vieles wie: 
		- DNS Record Lookups von Öffentlichen DNS Einträgen
		- Durchschauen der Website des Ziels 
		- Zeitungsartikel über das ziel lesen um generell ein Gefühl für das Ziel zu bekommen

- Active Reconnaissance
	- Hier kann nicht mehr so diskret vorgegangen werden wie bei Passive 
	- Man kann es sich vorstellen wie das Kontrollieren der Schlösser an Fenstern und Türen und das Suchen nach weiteren Eintritts punkten in ein Gebäude
	- Beinhaltet Sachen wie:
		- Verbinden mit einem Ziel Server via z.B. HTTP, FTP, SMTP
		- Anrufen der Zielfirma um Informationen zu erhalten via z.B. Social Engineering
		- Eindringen in die Zielfirma versteckt als Service Techniker 
	- WICHTIG Wenn man die Invasive Natur der Active Reconnaissance bedenkt fällt auf das es hier auch sehr schnell zu Rechtlichen Problemen kommen kann


### Tooling
##### Passive Reconnissance
- Whois
	- Terminal syntax 

```
whois Domainname.com
```

- nslookup und dig
nslookup kurz für "Name Server Look up" sucht förmlich die passende IP zu der Domain raus. Kann auch auf mehrere IPs verweisen -> Vllt größeres Angriffs Scope
nslookup Syntax und Query Types

```
nslookup TryHackMe.com
```

| Query type | Result             |
| ---------- | ------------------ |
| A          | IPv4 Addresses     |
| AAAA       | IPv6 Addresses     |
| CNAME      | Canonical Name     |
| MX         | Mail Servers       |
| SOA        | Start of Authority |
| TXT        | TXT Records        |
Syntax mit QueryType

```
nslookup -type=TXT tryhackme.com
```

Dig kurz für Domain Information Groper
Beispiel Syntax 
```
dig tryhackme.com MX
```
Hier wird der MX Record abgefragt
Funktioniert aber sonst wie nslookup 


- DNS Dumpster
Online Tool das zusätzlich zu den infos die nslookup oder dig geben noch Subdomains finden kann 

- Shodan.io
Shodan.io versucht, sich mit jedem online erreichbaren Gerät zu verbinden, um im Gegensatz zu einer Suchmaschine für Webseiten eine Suchmaschine für vernetzte „Dinge“ aufzubauen. Sobald es eine Antwort erhält, sammelt es alle mit dem Dienst verbundenen Informationen und speichert sie in der Datenbank, um sie durchsuchbar zu machen

##### Active Reconnaissance

Tools/Addons

- Der Webbrowser selbst 
Klingt doof ist aber so. Zumbeispiel via den Site Inspector (F12) lassen sich diverse weitere infos finden zur Connection aber auch zur seite selber.
Dann gibt es da auch noch addons hier ein paar beispiele die sich lohnen:
	- Foxy Proxy: 
	Mit FoxyProxy kann man rasch den Proxyserver wechseln, den man für den Zugriff auf die Ziel-Website verwendet. Diese Browsererweiterung ist praktisch, wenn man ein Tool wie Burp Suite verwendet oder wenn man regelmäßig den Proxy-Server wechseln muss. 
	-User-Agent Switcher and Manager:
	gibt einem die Möglichkeit, so zu tun, als würde man von einem anderen Betriebssystem oder einem anderen Webbrowser auf die Webseite zugreifen. Mit anderen Worten: Man kann so tun, als würde man eine Website mit einem iPhone besuchen, während man in Wirklichkeit mit Mozilla Firefox darauf zugreift.
	- Wappalyzer:
	 gibt Aufschluss über die auf den besuchten Websites verwendeten Technologien. Eine solche Erweiterung ist vor allem dann praktisch, wenn man all diese Informationen sammelt, während man wie jeder andere Benutzer auf der Website surft.


- Ping
Obvious nur dann sinnvoll wenn ICMP an ist. Aber dann generell Praktisch um zu sehen ob Target up

- Traceroute
Rausfinden der Hops/Router auf dem Weg zum Ziel


- Telnet 
Der von telnet verwendete Standard-Port ist 23. Aus Sicht der IT-Sicherheit sendet Telnet alle Daten, einschließlich Benutzernamen und Kennwörter, im Klartext. Das Senden im Klartext macht es jedem, der Zugang zum Kommunikationskanal hat, leicht, die Anmeldedaten zu stehlen. 
Der Telnet-Client kann jedoch aufgrund seiner Einfachheit auch für andere Zwecke verwendet werden. Da der Telnet-Client auf dem TCP-Protokoll basiert, kann man mit Telnet eine Verbindung zu einem beliebigen Dienst herstellen und dessen Banner abgreifen. Mit telnet MACHINE_IP PORT können Sie eine Verbindung zu jedem Dienst herstellen, der über TCP läuft, und sogar ein paar Nachrichten austauschen, sofern er keine Verschlüsselung verwendet.

- Netcat
Netcat oder einfach nc hat verschiedene Anwendungen, die für einen Pentester von großem Wert sein können. Netcat unterstützt sowohl TCP- als auch UDP-Protokolle. Es kann als Client fungieren, der sich mit einem Listening Port verbindet, oder als Server, der auf einem Port Ihrer Wahl lauscht. Somit ist es ein praktisches Tool, das Sie als einfachen Client oder Server über TCP oder UDP verwenden können.

Syntax 

```
nc *ZIELIP* *ZIEL PORT*
```

generelle Optionen für die Syntax 

|option|meaning|
|---|---|
|-l|Listen mode|
|-p|Specify the Port number|
|-n|Numeric only; no resolution of hostnames via DNS|
|-v|Verbose output (optional, yet useful to discover any bugs)|
|-vv|Very Verbose (optional)|
|-k|Keep listening after client disconnects|