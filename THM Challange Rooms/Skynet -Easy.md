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

