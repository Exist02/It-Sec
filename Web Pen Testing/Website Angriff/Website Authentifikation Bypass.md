### Usernames Herausfinden via Error Meldungen

Funktioniert anhand der Unterschiedlichen Meldungen wenn versucht wird sich mit einem Unbekannten Nutzer gegenüber mit einem Bekannten Nutzer mit Falschem PW anzumelden

Beispiel via ffuf
```
ffuf -w /usr/share/wordlists/SecLists/Usernames/Names/names.txt -X POST -d "username=FUZZ&email=x&password=x&cpassword=x" -H "Content-Type: application/x-www-form-urlencoded" -u http://10.10.134.46/customers/signup -mr "username already exists"
```

Befehl erklärt: 
**fuff** ist klar zum starten
**-w**  Spezifiziert die Wordlist mit dem Pfad dahinter 
**-X** Spezifiziert die art des Requests (Default GET) hier geändert auf POST
**-d** Spezifiziert die Daten die wir senden hier die Felder Username, Password, cpassword. in dem beispiel setzen wir die daten für "Username" auf FUZZ damit diese durchgetestet werden
**-H** wird verwendet, um der Anfrage zusätzliche Kopfzeilen hinzuzufügen. In diesem Fall setzen wir den Content-Type, damit der Webserver weiß, dass wir Formulardaten senden
**-u** Spezifiziert die URL des Ziels 
**-mr** Spezifiziert den Text den wir als Reply suchen um zu wissen das es den Username gibt

### Passwort via fuff Brute Forcen
Beispiel Befehl:
```
ffuf -w valid_usernames.txt:W1,/usr/share/wordlists/SecLists/Passwords/Common-Credentials/10-million-password-list-top-100.txt:W2 -X POST -d "username=W1&password=W2" -H "Content-Type: application/x-www-form-urlencoded" -u http://10.10.134.46/customers/login -fc 200
```

Befehl erklärt:
- fuff Startet den Befehl 
- -w spezifiziert die Wordlists (da zwei genutzt werden Spezifizierung auf W1 und W2)
- -X Spezifiziert das es ein PUSH Request sein soll
- -d Spezifiziert welche daten auf wohin sollen 
- -H wird verwendet, um der Anfrage zusätzliche Kopfzeilen hinzuzufügen. In diesem Fall setzen wir den Content-Type, damit der Webserver weiß, dass wir Formulardaten senden
- -u Gibt die Ziel URL vor
- -fc 200 chect für HTTP Status Code 200 


### Logic Fehler 
Manchmal enthalten Authentifizierungsverfahren logische Fehler. Ein Logikfehler liegt vor, wenn der typische logische Pfad einer Anwendung entweder bypassed, circumvented oder von einem Hacker manipuliert wird. Logikfehler können in jedem Bereich einer Website auftreten, aber wir werden uns in diesem Fall auf Beispiele im Zusammenhang mit der Authentifizierung konzentrieren.

Beispiel: 

Wir werden die Funktion „Passwort zurücksetzen“ auf der Website von Acme IT Support (http://10.10.134.46/customers/reset) untersuchen. Wir sehen ein Formular, in dem wir nach der E-Mail-Adresse gefragt werden, die mit dem Konto verbunden ist, für das wir das Passwort zurücksetzen möchten. Wenn eine ungültige E-Mail-Adresse eingegeben wird, erhalten Sie die Fehlermeldung „Account not found from supplied email address“.

Zu Demonstrationszwecken verwenden wir die E-Mail-Adresse robert@acmeitsupport.thm, die akzeptiert wird. Im nächsten Schritt des Formulars werden wir nach dem Benutzernamen gefragt, der mit dieser Anmelde-E-Mail-Adresse verknüpft ist. Wenn wir robert als Benutzernamen eingeben und auf die Schaltfläche Benutzernamen prüfen klicken, erhalten Sie eine Bestätigungsmeldung, dass eine E-Mail zum Zurücksetzen des Passworts an robert@acmeitsupport.thm gesendet wird.

Darauf hin schauen wir uns die Reply auf einen passenden Curl befgehl an: 
```
curl 'http://10.10.134.46/customers/reset?email=robert%40acmeitsupport.thm' -H 'Content-Type: application/x-www-form-urlencoded' -d 'username=robert'
```

Mit dem Flag -H fügen wir der Anfrage eine zusätzliche Kopfzeile hinzu. In diesem Fall setzen wir den Content-Type auf application/x-www-form-urlencoded, wodurch der Webserver weiß, dass wir Formulardaten senden, damit er unsere Anfrage richtig versteht.
In der Anwendung wird das Benutzerkonto mit Hilfe des Query-Strings abgefragt, aber später, in der Anwendungslogik, wird die E-Mail zum Zurücksetzen des Passworts mit Hilfe der Daten in der PHP-Variablen $_REQUEST gesendet.

Die PHP-Variable $_REQUEST ist ein Array, das Daten aus dem Query-String und POST-Daten enthält. Wenn derselbe Schlüsselname sowohl für den Query-String als auch für die POST-Daten verwendet wird, bevorzugt die Anwendungslogik für diese Variable die POST-Datenfelder gegenüber dem Query-String. Wenn wir also dem POST-Formular einen weiteren Parameter hinzufügen, können wir steuern, wohin die E-Mail zum Zurücksetzen des Passworts zugestellt wird.

Im nächsten Schritt müssen Sie ein Konto im Kundenbereich von Acme IT Support erstellen. Dadurch erhalten Sie eine eindeutige E-Mail-Adresse, mit der Sie Support-Tickets erstellen können. Die E-Mail-Adresse hat das Format {username}@customer.acmeitsupport.thm

Curl Befehl mit angepasstem Ziel im Post

```
curl 'http://10.10.134.46/customers/reset?email=robert@acmeitsupport.thm' -H 'Content-Type: application/x-www-form-urlencoded' -d 'username=robert&email=**{username}**@customer.acmeitsupport.thm'
```


### Cookie Manipulation

Manchmal gibt es Cookies in Plain Text wodurch es sehr einfach wird diese zu manipulieren 
Bsp.:
für einen erfolgreichen login 

Set-Cookie: logged_in=true; Max-Age=3600; Path=/
Set-Cookie: admin=false; Max-Age=3600; Path=/

um diese einzuspielen hier ein Beispiel Befehl: 

```
curl -H "Cookie: logged_in=true; admin=true" http://10.10.134.46/cookie-test
```

Manchmal können Cookie-Werte wie eine lange Kette zufälliger Zeichen aussehen; diese werden als Hashes bezeichnet, die eine unumkehrbare Darstellung des ursprünglichen Textes sind. Hier sind einige Beispiele, auf die man häufiger Stößt

|                     |                 |                                                                                                                                  |
| ------------------- | --------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| **Original String** | **Hash Method** | **Output**                                                                                                                       |
| 1                   | md5             | c4ca4238a0b923820dcc509a6f75849b                                                                                                 |
| 1                   | sha-256         | 6b86b273ff34fce19d6b804eff5a3f5747ada4eaa22f1d49c01e52ddb7875b4b                                                                 |
| 1                   | sha-512         | 4dff4ea340f0a823f15d3f4f01ab62eae0e5da579ccb851f8db9dfe84c58b2b37b89903a740e1ee172da793a6e79d560e5f7f9bd058a12a280433ed6fa46510a |
| 1                   | sha1            | 356a192b7913b04c54574d18c28d46e6395428ab                                                                                         |
Zu der Identifikation des Hashes kann man dann z.B. Crack Station nutzen 
https://crackstation.net/
oder hashes.com
https://hashes.com/en/decrypt/hash

