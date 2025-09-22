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

