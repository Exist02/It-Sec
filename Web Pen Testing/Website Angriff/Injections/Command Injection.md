### Was Das?
 Command Injection ist der Missbrauch des Verhaltens einer Anwendung zur Ausführung von Befehlen auf dem Betriebssystem unter Verwendung der gleichen Berechtigungen, mit denen die Anwendung auf einem Gerät ausgeführt wird. Wird beispielsweise eine Befehlsinjektion auf einem Webserver durchgeführt, der unter einem Benutzer namens joe läuft, werden Befehle unter diesem joe-Benutzer ausgeführt - und somit alle Berechtigungen erlangt, die joe hat.

### Erkennen/Verstehen von Command Injection
Diese Schwachstelle besteht, weil Anwendungen häufig Funktionen in Programmiersprachen wie PHP, Python und NodeJS verwenden, um Daten an das Betriebssystem des Rechners zu übergeben und Systemaufrufe zu tätigen

Beispiel zur Funktionsweise: 
https://imgur.com/try56WI
Erklärung: 
1.  Die Anwendung speichert MP3-Dateien in einem Verzeichnis, das sich auf dem Betriebssystem befindet. / /var/www/html/songs
2.  Der Nutzer kann nach einem Song suchen das wird in $Titel Gespeichert
3.  Die Daten aus der $Titel Variable wird direkt via Grep weiterverarbeitet ohne irgendwelche Validierung
4.  Der Output des Suchens in der Songtitel.txt sorg für Correspondierenden Output
Was ist hier Das Problem?
In schritt 3 wird der Inhalt der $Titel Variable einfach mit Grep Verarbeitet ohne weitere Validierung. Dies kann ausgenutzt werden um andere Datein zu Grepen und auszulesen.

Schwachstellen wie diese können auch unabhängig der Programiersprache IMMER ausgeführt werden solange die Befehle direkt im OS ausgeführt werden.

### Exploiting Command Injections

| **Method** | **Description**                                                                                                                                                                                                                                                               |
| ---------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Blind      | This type of injection is where there is no direct output from the application when testing payloads. You will have to investigate the behaviours of the application to determine whether or not your payload was successful.                                                 |
| Verbose    | This type of injection is where there is direct feedback from the application once you have tested a payload. For example, running the `whoami` command to see what user the application is running under. The web application will output the username on the page directly. |
##### Erkennen von Blind Injections

Für diese Art der Befehlsinjektion müssen wir Nutzdaten verwenden, die eine gewisse Zeitverzögerung verursachen. Die Befehle „ping“ und „sleep“ sind zum Beispiel wichtige Nutzdaten, mit denen man testen kann. Mit ping als Beispiel bleibt die Anwendung für x Sekunden hängen, je nachdem, wie viele Pings Sie angegeben haben.

Alternativ kann man auch Testen indem man einen Output Forced. Dies geht mit Operatron wie >. Zum Beispiel können wir die Webanwendung anweisen, Befehle wie whoami auszuführen und diese in eine Datei umzuleiten. Wir können dann einen Befehl wie cat verwenden, um den Inhalt dieser neu erstellten Datei zu lesen.

Der Befehl curl eignet sich hervorragend zum Testen auf Command Injection. Der Grund dafür ist, dass Sie curl verwenden können, um Daten zu und von einer Anwendung in Ihrem Payload zu übermitteln. Das folgende Code-Snippet ist ein Beispiel dafür, dass eine einfache Curl-Payload an eine Anwendung für Command Injection möglich ist.

```
curl http://vulnerable.app/process.php%3Fsearch%3DThe%20Beatles%3B%20whoami
```

Zu Beachten gibt es auch noch das für diese art an schwachstelle meistens viel Rum experimentieren bedarf, da die Syntax eines Linux Hosts zu dem eines Windows Hosts absolut unterschiedlich ist.


#### Erkennen von Verbose Injections

Die Erkennung von Command Injection auf diese Weise ist wohl die einfachste der beiden Methoden. Eine ausführliche Befehlsinjektion liegt vor, wenn die Anwendung eine Rückmeldung oder Ausgabe darüber gibt, was passiert oder ausgeführt wird.
Zum Beispiel wird die Ausgabe von Befehlen wie ping oder whoami direkt in der Webanwendung angezeigt.


#### Bypassing Filters

Anwendungen verwenden zahlreiche Techniken zum Filtern und Bereinigen von Daten, die von den Eingaben eines Benutzers stammen. Diese Filter beschränken Sie auf bestimmte Nutzdaten; wir können jedoch die Logik hinter einer Anwendung missbrauchen, um diese Filter zu umgehen. Beispielsweise kann eine Anwendung Anführungszeichen entfernen; wir können stattdessen den Hexadezimalwert dieser Zeichen verwenden, um das gleiche Ergebnis zu erzielen.

https://imgur.com/nZtKHWP


#### Generell Praktische Payloads für Command Injections;

Beispiel Befehl
&cat /home/etc/flag

hat funktioniert da die & Funktion vulnerable ist und damit Input zulässt 

https://github.com/payloadbox/command-injection-payload-list

Linux

|   |   |
|---|---|
|**Payload**|**Description**|
|whoami|See what user the application is running under.|
|ls|List the contents of the current directory. You may be able to find files such as configuration files, environment files (tokens and application keys), and many more valuable things.|
|ping|This command will invoke the application to hang. This will be useful in testing an application for blind command injection.|
|sleep|This is another useful payload in testing an application for blind command injection, where the machine does not have `ping` installed.|
|nc|Netcat can be used to spawn a reverse shell onto the vulnerable application. You can use this foothold to navigate around the target machine for other services, files, or potential means of escalating privileges.|
Windows

|   |   |
|---|---|
|**Payload**|**Description**|
|whoami|See what user the application is running under.|
|dir|List the contents of the current directory. You may be able to find files such as configuration files, environment files (tokens and application keys), and many more valuable things.|
|ping|This command will invoke the application to hang. This will be useful in testing an application for blind command injection.|
|timeout|This command will also invoke the application to hang. It is also useful for testing an application for blind command injection if the `ping` command is not installed.|