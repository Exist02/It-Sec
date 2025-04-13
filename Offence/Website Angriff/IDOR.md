### Was ist IDOR? 
IDOR steht für "Insecure Direct Object Refrence"

Diese Art an Schwachstelle tritt auf wen ein Webserver einem User Input mit dem Objekt (Dateien/Informationen) abzurufen zu viel Vertrauen/Trust gibt und sie deshalb nicht serverseititg Bestätigt ob die abgerufenen Daten zu dem Benutzer gehören.

THM Definition:
This type of vulnerability can occur when a web server receives user-supplied input to retrieve objects (files, data, documents), too much trust has been placed on the input data, and it is not validated on the server-side to confirm the requested object belongs to the user requesting it.

Beispiel: 
Stellen dir vor, du hast sich gerade bei einem Online-Dienst angemeldet und möchtest deine Profilinformationen ändern. Der Link, auf den Sie klicken, führt zu http://online-service.thm/profile?user_id=1305, und du kannst deine Daten sehen.

Du wirst neugierig und versuchst, den Wert user_id auf 1000 zu ändern (http://online-service.thm/profile?user_id=1000). Zu deiner Überraschung kannst du nun die Daten eines anderen Benutzers sehen. Damit hast du nun eine IDOR-Schwachstelle entdeckt! Da Idealerweise auf der Website überprüft werden sollte, ob die Benutzerdaten zu dem angemeldeten Benutzer gehören, der sie abfragt.


### IDORS in Verschlüsselten (Encoded) URLs


[https://imgur.com/PrqtlZ8]()