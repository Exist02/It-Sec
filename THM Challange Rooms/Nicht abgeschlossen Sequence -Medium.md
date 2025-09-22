# Target

10.10.58.186 
review.thm

# Scanning 

## Ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Review Shop

## Site Enumeration 

Starting gobuster in directory enumeration mode
===============================================================
/uploads              (Status: 301) [Size: 310] [--> http://review.thm/uploads/]
/mail                 (Status: 301) [Size: 307] [--> http://review.thm/mail/]
/phpmyadmin           (Status: 301) [Size: 313] [--> http://review.thm/phpmyadmin/]
/server-status        (Status: 403) [Size: 275]


# Intressante Findings

#### das /uploads ist Zugänglich


Emailadressen aus /mail/dump.txt
From: software@review.thm
To: product@review.thm

Weitere Sites unter 
/finance.php 
/lottery.php

```
I have successfully updated the code. The Lottery and Finance panels have also been created.
Both features have been placed in a controlled environment to prevent unauthorized access. The Finance panel (`/finance.php`) is hosted on the internal 192.x network, and the Lottery panel (`/lottery.php`) resides on the same segment.
For now, access is protected with a completed 8-character alphanumeric password (S60u}f5j), in order to restrict exposure and safeguard details regarding our potential investors.
I will be away on holiday but will be back soon.
```


# Wie vorgehen? 

Gegeben sind  
Login seite 
PHP Admin seite

2x Mögliche Email adressen
