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
