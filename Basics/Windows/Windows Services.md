## Auflistung an Services, wo sie abliegen und wie sie Aussehen sollten

### **System > smss.exe**

Ist der Windows Session Manager Ist der Erste User Mode Prozess der vom Kernel Gestartet wird This Subsystem includes win32k.sys (kernel Mode), winsrv.dll (user Mode), and csrss.exe (user Mode). Startet u.A Winlogon

Image Path: %SystemRoot%\System32\smss.exe 
Parent Process: System 
Number of Instances: One master instance and child instance per session. The child instance exits after creating the session. 
User Account: Local 
System Start Time: Within seconds of boot time for the master instance

==**What is unusual?**== A different parent process other than System (4) The image path is different from C:\Windows\System32 More than one running process. (children self-terminate and exit after each new session) The running User is not the SYSTEM user Unexpected registry entries for Subsystem



### **csrss.exe**

Bedeutet Client Server Runtime Process Ist Kritisch das es immer läuft Ist für Win32 console und Erstellung und Löschung von Prozess Threats zuständig Zudem verantwortlich für die Windows API, Laufwerke Mappen und das Herunterfahren zu managen Note: Recall that csrss.exe and winlogon.exe are called from smss.exe at Startup for Session 1.

Image Path: %SystemRoot%\System32\csrss.exe 
Parent Process: Created by an instance of smss.exe 
Number of Instances: Two or more User Account: Local System 
Start Time: Within seconds of Boot time for the First Two instances (for Session 0 and 1). Start times for additional instances occur as new sessions are created, although only Sessions 0 and 1 are often created.

==**What is unusual?**== An actual parent Process. (smss.exe calls this Process and self-terminates) Image file path other than C:\Windows\System32 Subtle misspellings to hide rogue processes masquerading as csrss.exe in plain sight The user is not the SYSTEM user.



### **wininit.exe**

Windows Initialisierungs Prozess Kümmert sich das "services.exe", lsass.exe (Local Security Autority) und lsaiso.exe gestartet werden mit Session ID 0 Note: lsaiso.exe is a process associated with Credential Guard and KeyGuard. You will only see this process if Credential Guard is enabled.

Image Path: %SystemRoot%\System32\wininit.exe 
Parent Process: Created by an instance of smss.exe 
Number of Instances: One User Account: 
Local System Start Time: Within seconds of boot time

==**What is unusual?**== An actual parent process. (smss.exe calls this process and self-terminates) Image file path other than C:\Windows\System32 Subtle misspellings to hide rogue processes in plain sight Multiple running instances Not running as SYSTEM



### **wininit.exe > Services.exe**

Ist der SCM (Service Control Manager) Hauptaufgabe Laden, interagieren starten und Stoppen von Servicen in cmd "sc.exe" Daten in der Registry: HKLM\System\CurrentControlSet\Services. Parent Process zu svchost.exe, spoolsv.exe, msmpeng.exe, und dllhost.exe

Image Path: %SystemRoot%\System32\services.exe 
Parent Process: wininit.exe 
Number of Instances: One User Account
Local System Start Time: Within seconds of boot time

==What is unusual?== A parent process other than wininit.exe Image file path other than C:\Windows\System32 Subtle misspellings to hide rogue processes in plain sight Multiple running instances Not running as SYSTEM



### **wininit.exe > services.exe > svchost.exe**

Steht für "Service Host Prozess" Ist für das Hosten und Managen der Services zuständig 
Die in diesem Prozess ausgeführten Dienste werden als DLLs implementiert. Die zu implementierende DLL wird in der Registrierung für den Dienst unter dem Unterschlüssel Parameters in ServiceDLL gespeichert 
HKLM\SYSTEM\CurrentControlSet\Services\SERVICE NAME\
Parameters haben den -k Parameter in dem Call Pfad 
bsp.: C:\Windows\system32\svchost.exe -k DcomLaunch -p 
Wird sehr gerne als Ziel genutzt da der Service immer recht viele Prozesse hat

Image Path: %SystemRoot%\System32\svchost.exe 
Parent Process: services.exe 
Number of Instances: Many 
User Account: Varies (SYSTEM, Network Service, Local Service) depending on the svchost.exe instance. In Windows 10, some instances run as the logged-in user. 
Start Time: Typically within seconds of boot time. Other instances of svchost.exe can be started after boot.

==What is unusual?== 
A parent process other than services.exe Image file path other than C:\Windows\System32 Subtle misspellings to hide rogue processes in plain sight The absence of the -k parameter



### **Lsass.exe**

Local Security Authority Subsystem Service (LSASS) Setzt die Sicherheitsrichtlinien um, Verifiziert Benutzer beim Login auf den PC/Server, handhabt PW änderungen und erstellt access token Schreib in den Security log Erstellt sicheheitstokens für SAM( Security Account Manager, AD und NETLOGON HKLM\System\CurrentControlSet\Control\Lsa Wird auch häufig als ziel genutzt bsp.: via Mimikatz

Image Path: %SystemRoot%\System32\lsass.exe 
Parent Process: wininit.exe 
Number of Instances: One 
User Account: Local 
System Start Time: Within seconds of boot time

==What is unusual?== 
A parent process other than wininit.exe Image file path other than C:\Windows\System32 Subtle misspellings to hide rogue processes in plain sight Multiple running instances Not running as SYSTEM



### **winlogon.exe 

Windows Logon kümmert sich um die Secure Attention Sequence (SAS) Zudem lät er die Nutzerdaten via NTUSER:DAT und userinit.exe Außerdem kümmert er sich um das Sperren des PCs nach zeit

**Image Path**: %SystemRoot%\System32\winlogon.exe 
Parent Process: Created by an instance of smss.exe that exits, so analysis tools usually do not provide the parent process name. 
Number of Instances: One or more 
User Account: Local 
System Start Time: Within seconds of boot time for the first instance (for Session 1). Additional instances occur as new sessions are created, typically through Remote Desktop or Fast User Switching logons.

==**What is unusual?**== 
An actual parent process. (smss.exe calls this process and self-terminates) Image file path other than C:\Windows\System32 Subtle misspellings to hide rogue processes in plain sight Not running as SYSTEM Shell value in the registry other than explorer.exe

### **explorer.exe 

ermöglicht den Zugriff auf Datein und Ordner zudem stellt er z.B. das Start menü und die Taskbar zur verfügung wird von der Userinit.exe gestartet bevor sich diese wieder schließt deshalb kein Parent Process

Image Path: %SystemRoot%\explorer.exe 
Parent Process: Created by userinit.exe and exits 
Number of Instances: One or more per interactively 
logged-in user User Account: Logged-in user(s) 
Start Time: First instance when the first interactive user logon session begins

==What is unusual?== An actual parent process. (userinit.exe calls this process and exits) Image file path other than C:\Windows Running as an unknown user Subtle misspellings to hide rogue processes in plain sight Outbound TCP/IP connections