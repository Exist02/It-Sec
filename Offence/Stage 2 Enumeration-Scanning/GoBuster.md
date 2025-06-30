Was das?        
-          Ist ein Opensource tool        
-          Genutzt für Enumeration und Bruteforce
Enumeration von       

-          web directories, DNS subdomains, vhosts, Amazon S3 buckets, and Google Cloud Storage by brute force, using specific wordlists and handling the incoming responses    

nach MITRE Gesetzt zwischen reconnaissance and scanning phases

## Syntax und Befehle: 

generell einsehbar via `gobuster -h`

| Short Flag | Long Flag  | Description                                                                                                                                                                                                                                                                                                                           |
| ---------- | ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| -t         | --threads  | This flag configures the number of threads to use for the scan.  <br>Each of these threads sends out requests with a slight delay.  <br>The default number of threads is 10. This number may be slow when using large wordlists.  <br>You can increase or decrease the number of threads depending on the available system resources. |
| -w         | --wordlist | The flag configures a wordlist to use for iterating.  <br>Each wordlist entry is attached to the URL you included in the command.                                                                                                                                                                                                     |
|            | --delay    | This flag defines the amount of time to wait between sending requests.  <br>Some web servers include mechanisms to detect enumeration  <br>by looking at how many requests are received in a certain period of time.  <br>We can increase the delay between subsequent requests to make it look like normal web traffic.              |
|            | --debug    | This flag helps us to troubleshoot when our command gives unexpected errors.                                                                                                                                                                                                                                                          |
| -o         | --output   | This flag writes the enumeration results to a file we choose.                                                                                                                                                                                                                                                                         |
Beispiel: 
```
gobuster dir -u "http://www.example.thm/" -w /usr/share/wordlists/dirb/small.txt -t 64
```

- gobuster dir -> Spezifiziert das Gobuster im Directory Modus läuft
- -u "url" -> Spezifiziert Ziel URL
- - w "Path to Wordlist" -> Spezifiziert welche Wordlist genutzt werden soll
- -t 64 -> Sagt an wie viele Threats genutzt werden dürfen, was die Performance Verbessert

## DNS Mode

Wird Genutzt um Subdomains zu Enumerieren, dass kann nützlich sein da Schwachstellen die in der Top Level Domain Gefixt sind nicht unbedingt auch in der Subdomain gefixt sind

Hilfe Syntax `gobuster dns -help`

| Flag | Long Flag    | Description                                                                       |
| ---- | ------------ | --------------------------------------------------------------------------------- |
| -c   | --show-cname | Show CNAME Records (cannot be used with the -i flag).                             |
| -i   | --show-ips   | Including this flag shows IP addresses that the domain and subdomains resolve to. |
| -r   | --resolver   | This flag configures a custom DNS server to use for resolving.                    |
| -d   | --domain     | This flag configures the domain you want to enumerate.                            |
Syntax und Beispiel:

```
gobuster dns -d *Beispiel url* -w *path/to/wordlist*

gobuster dns -d example.thm -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt
```

| gobuster dns   | Starten von Gobuster in DNS mode        |
| -------------- | --------------------------------------- |
| -d example.thm | setzt die Ziel Domain auf "example.thm" |
| -w *wordlist*  | verweist mit Pfad auf die Wordlist      |


## VHOST Mode

Bruteforce Virtual Hosts       

Virtual Hosts können mehrere Websites auf einem Host haben die dann alle Gebruteforced werden          

gobuster vhost --help

| Short Flag | Long Flag         | Description                                                                                           |
| ---------- | ----------------- | ----------------------------------------------------------------------------------------------------- |
| -u         | --url             | Specifies the base URL (target domain) for brute-forcing virtual hostnames.                           |
|            | --append-domain   | Appends the base domain to each word in the wordlist (e.g., word.example.com).                        |
| -m         | --method          | Specifies the HTTP method to use for the requests (e.g., GET, POST).                                  |
|            | --domain          | Appends a domain to each wordlist entry to form a valid hostname (useful if not provided explicitly). |
|            | --exclude-length  | Excludes results based on the length of the response body (useful to filter out unwanted responses).  |
| -r         | --follow-redirect | Follows HTTP redirects (useful for cases where subdomains may redirect).                              |
### Syntax               

```
gobuster vhost -u "http://example.thm" -w /path/to/wordlist                     
```
-u "*URL*"        Verweist auf die URL
-w *wordlist* verweist mit Pfad auf die Wordlist 


Beispiel mit Zusatz Flags                      

```
gobuster vhost -u "http://10.10.169.142" --domain example.thm -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt --append-domain --exclude-length 250-320                       
```

gobuster vhost                                      startet Gobuster im vhost mode
-u "domain"                                           setzt target domain
-w /word/list                                          setzt wordlist
--domain example.thm                          setzt die Top- und Second-Level-Domains im Teil Hostname: der Anfrage auf example.thm.
--append-domain                                  fügt die konfigurierte Domäne an jeden Eintrag in der Wortliste an. Wenn dieses Flag nicht konfiguriert ist, würde der eingestellte Hostname www, blog usw. lauten. Dies führt dazu, dass der Befehl nicht korrekt funktioniert und falsch positive Ergebnisse anzeigt.
--exclude-length                                   entfernt False Positives
