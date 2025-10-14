# Ziel 10.10.126.91

# Scanning
## Nmap 

```
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 68:ed:7b:19:7f:ed:14:e6:18:98:6d:c5:88:30:aa:e9 (RSA)
|   256 5c:d6:82:da:b2:19:e3:37:99:fb:96:82:08:70:ee:9d (ECDSA)
|_  256 d2:a9:75:cf:2f:1e:f5:44:4f:0b:13:c2:0f:d7:37:cc (ED25519)
80/tcp   open  http    Apache httpd 2.4.6 ((CentOS) PHP/5.6.40)
|_http-generator: Joomla! - Open Source Content Management
| http-robots.txt: 15 disallowed entries 
| /joomla/administrator/ /administrator/ /bin/ /cache/ 
| /cli/ /components/ /includes/ /installation/ /language/ 
|_/layouts/ /libraries/ /logs/ /modules/ /plugins/ /tmp/
|_http-server-header: Apache/2.4.6 (CentOS) PHP/5.6.40
|_http-title: Home
3306/tcp open  mysql   MariaDB (unauthorized)
MAC Address: 02:4F:86:41:03:09 (Unknown)

```
-> Website Vorhanden 
-> Maria db offen verfügbar?
# Enumeration
# Gobuster 

```
[+] Url:                     http://10.10.126.91
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/images               (Status: 301) [Size: 235] [--> http://10.10.126.91/images/]
/media                (Status: 301) [Size: 234] [--> http://10.10.126.91/media/]
/templates            (Status: 301) [Size: 238] [--> http://10.10.126.91/templates/]
/modules              (Status: 301) [Size: 236] [--> http://10.10.126.91/modules/]
/bin                  (Status: 301) [Size: 232] [--> http://10.10.126.91/bin/]
/plugins              (Status: 301) [Size: 236] [--> http://10.10.126.91/plugins/]
/includes             (Status: 301) [Size: 237] [--> http://10.10.126.91/includes/]
/language             (Status: 301) [Size: 237] [--> http://10.10.126.91/language/]
/components           (Status: 301) [Size: 239] [--> http://10.10.126.91/components/]
/cache                (Status: 301) [Size: 234] [--> http://10.10.126.91/cache/]
/libraries            (Status: 301) [Size: 238] [--> http://10.10.126.91/libraries/]
/tmp                  (Status: 301) [Size: 232] [--> http://10.10.126.91/tmp/]
/layouts              (Status: 301) [Size: 236] [--> http://10.10.126.91/layouts/]
/administrator        (Status: 301) [Size: 242] [--> http://10.10.126.91/administrator/]
/cli                  (Status: 301) [Size: 232] [--> http://10.10.126.91/cli/]
Progress: 218275 / 218276 (100.00%)

```

-> Adminportal? -> Joomla CMS

# SQLI versuch 

Aufgrund dessen das ich ein Admin Portal, eine Login maske auf der Normalen seite ud bekannterweise eine Maria db habe habe ich versucht via SQLi zugriff zu erhalten 

ohne Erfolg

# Weitere Enumeration 

Um vielleicht einen Exploit zu Finden habe ich in Google geschaut ob ich irgendwie die Version des CMS herausbekomme ohne angemeldet zu sein, und siehe da eine der Antworten beinhaltete das es über den Pfad "/administrator/manifests/files/joomla.xml" möglich ist und siehe da auf dem Ziel läuft Joomla CMS Version 3.7.0

