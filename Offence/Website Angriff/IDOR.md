### Was ist IDOR? 
IDOR steht für "Insecure Direct Object Refrence"

Diese Art an Schwachstelle tritt auf wen ein Webserver einem User Input mit dem Objekt (Dateien/Informationen) abzurufen zu viel Vertrauen/Trust gibt und sie deshalb nicht serverseititg Bestätigt ob die abgerufenen Daten zu dem Benutzer gehören.

THM Definition:
This type of vulnerability can occur when a web server receives user-supplied input to retrieve objects (files, data, documents), too much trust has been placed on the input data, and it is not validated on the server-side to confirm the requested object belongs to the user requesting it.

Beispiel: 
Stellen dir vor, du hast sich gerade bei einem Online-Dienst angemeldet und möchtest deine Profilinformationen ändern. Der Link, auf den Sie klicken, führt zu http://online-service.thm/profile?user_id=1305, und du kannst deine Daten sehen.

Du wirst neugierig und versuchst, den Wert user_id auf 1000 zu ändern (http://online-service.thm/profile?user_id=1000). Zu deiner Überraschung kannst du nun die Daten eines anderen Benutzers sehen. Damit hast du nun eine IDOR-Schwachstelle entdeckt! Da Idealerweise auf der Website überprüft werden sollte, ob die Benutzerdaten zu dem angemeldeten Benutzer gehören, der sie abfragt.


### IDORS in Verschlüsselten (Encoded) URLs

Wenn Websites Daten von Seite zu Seite weitergeben (via Post, querystring, oder cookie) werden die zu Übermittelnden Daten erstmal encoded. Das sorgt dafür das die Daten meistens in das ASCII Format übertragen werden (mit Fillern/Padding: a-z, A-Z, 0-9 and = ). Die Bekannteste Encoding Variante ist Base64 welche man aber einfach via Online Tool de/encoden kann. 

De/Encoding links
https://www.base64decode.org/
https://www.base64encode.org/

Link unterhalb Verständnis Bild 
https://imgur.com/PrqtlZ8


### Finden von IDORs in Hashed IDs

Gehashte IDs sind etwas komplizierter zu handhaben als kodierte, aber sie können einem vorhersehbaren Muster folgen, z. B. die gehashte Version des Integer-Wertes sein. Zum Beispiel würde die Kennnummer 123 zu 202cb962ac59075b964b07152d234b70 werden, wenn md5-Hashing verwendet wird

Es lohnt sich, alle entdeckten Hashes durch einen Webdienst wie https://crackstation.net/ (der eine Datenbank mit Milliarden von Hash-to-Value-Ergebnissen hat) zu schicken, um zu sehen, ob wir irgendwelche Übereinstimmungen finden.


### Unvorhersehbare IDs

Wenn die Kennung mit den oben genannten Methoden nicht ermittelt werden kann, besteht eine ausgezeichnete Methode zur IDOR-Erkennung darin, zwei Konten zu erstellen und die Kennungsnummern zwischen ihnen auszutauschen. 
Wenn du den Inhalt des anderen Benutzers mit dessen Id-Nummer sehen kannst, während du mit einem anderen Konto (oder gar nicht) eingeloggt bist, hast du eine gültige IDOR-Schwachstelle gefunden.


