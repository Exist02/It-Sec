# OWASP Top 10 Cheat Sheet

Ein tiefergehender Überblick über die OWASP Top 10 Sicherheitsrisiken für Webanwendungen – mit Definitionen und praxisnahen "Hack Along"-Beispielen in einer fiktiven Umgebung.

---

## 1. Broken Access Control

**Definition:** Wenn Benutzer Zugriff auf Daten oder Funktionen erhalten, für die sie keine Berechtigung haben. Diese Schwachstelle tritt häufig auf, wenn Berechtigungen nur auf der Clientseite geprüft werden oder URLs nicht ausreichend geschützt sind.

**Hack Along Beispiel:** Du bist als normaler Nutzer "user123" in einer fiktiven App `PetStorePro` eingeloggt. Du rufst dein Profil unter:

```
GET https://petstorepro.fake/user/profile?id=1001
```

Nun änderst du einfach die URL auf:

```
GET https://petstorepro.fake/user/profile?id=1002
```

und erhältst plötzlich die Profildaten eines anderen Nutzers. Die Zugriffskontrolle prüft nur, ob du eingeloggt bist – nicht **wer** du bist.

---

## 2. Cryptographic Failures

**Definition:** Schwächen im Umgang mit sensiblen Daten wie Passwörtern, Kreditkartennummern oder Gesundheitsdaten – z. B. unverschlüsselte Speicherung oder unsichere Algorithmen.

**Hack Along Beispiel:** Du greifst auf die fiktive App `HealthRecordsX` zu und findest eine ungeschützte Backup-Datei unter:

```
https://healthrecordsx.fake/backups/userdata.sql
```

Du lädst die Datei herunter und siehst Passwörter wie:

```
user1, password123
user2, mysecret
```

Kein Hash, keine Verschlüsselung – ideal für Angreifer.

---

## 3. Injection

**Definition:** Unsichere Benutzereingaben werden direkt in Code (z. B. SQL) eingefügt. Dies kann zu schwerwiegenden Angriffen führen, bei denen Daten verändert oder gelöscht werden.

**Hack Along Beispiel:** In der App `BookReviewZone` gibt es ein Login-Formular:

```sql
SELECT * FROM users WHERE username = 'admin' AND password = 'XYZ';
```

Du gibst Folgendes ein:

```
Benutzername: ' OR '1'='1
Passwort: ' OR '1'='1
```

Der resultierende SQL-Befehl lautet:

```sql
SELECT * FROM users WHERE username = '' OR '1'='1' AND password = '' OR '1'='1';
```

Du wirst als erster Benutzer eingeloggt – meist ein Admin.

---

## 4. Insecure Design

**Definition:** Sicherheitsprobleme, die schon auf Design-Ebene entstehen – z. B. fehlende Schutzmechanismen, keine Threat-Modeling-Prozesse oder clientseitige Logik für sensible Operationen.

**Hack Along Beispiel:** Die App `CouponKing` berechnet den Preis clientseitig in JavaScript:

```js
finalPrice = basePrice - discount;
```

Du öffnest die Dev-Tools und änderst `discount = 95`, obwohl dir eigentlich nur 5 % Rabatt zustehen. Der Server akzeptiert den manipulierten Preis ohne Prüfung.

---

## 5. Security Misconfiguration

**Definition:** Unsichere Standardkonfigurationen, unnötige Dienste, fehlende Headers oder schlecht geschützte Admin-Bereiche.

**Hack Along Beispiel:** Du scanst die fiktive Seite `adminportal.fake` und findest:

```
https://adminportal.fake/phpmyadmin/
```

Du rufst die URL auf – und landest im Datenbankinterface ohne Login. Ein Klassiker: vergessenes, offenes phpMyAdmin in der Produktion.

---

## 6. Vulnerable and Outdated Components

**Definition:** Nutzung veralteter Bibliotheken oder Frameworks mit bekannten Schwachstellen.

**Hack Along Beispiel:** Die App `OldGalleryApp` verwendet jQuery 1.8. In einer Seite findest du:

```html
<div class="gallery" data-desc="<img src=x onerror=alert('XSS')>">
```

Der alte jQuery-Code rendert die `data-desc` direkt in den DOM – das XSS-Popup erscheint. Mit moderner jQuery-Version wäre das entschärft.

---

## 7. Identification and Authentication Failures

**Definition:** Schwächen in Login-Mechanismen, Session-Verwaltung oder Multi-Faktor-Authentifizierung.

**Hack Along Beispiel:** In `SimpleBank.fake` loggst du dich ein und bekommst ein Session-Cookie:

```
Set-Cookie: session=abc123; Path=/
```

Du kopierst das Cookie in ein anderes Gerät – und bist sofort ebenfalls eingeloggt. Kein Device-Binding, kein Token-Timeout. Auch nach Logout bleibt das Cookie gültig.

---

## 8. Software and Data Integrity Failures

**Definition:** Fehlende Validierung der Integrität von Software-Komponenten oder Update-Mechanismen.

**Hack Along Beispiel:** Die App `WidgetDeploy` lädt Plugins zur Laufzeit von:

```
http://updates.widgetdeploy.fake/plugins/liveupdate.js
```

Du führst einen DNS-Spoofing-Angriff durch und leitest die Domain um auf deinen Server. Dein bösartiges `liveupdate.js` wird geladen – du übernimmst die Seite komplett.

---

## 9. Security Logging and Monitoring Failures

**Definition:** Fehlende, unvollständige oder nicht überprüfte Protokollierung sicherheitsrelevanter Ereignisse.

**Hack Along Beispiel:** In der App `LoginLab` startest du ein Skript:

```bash
for i in $(cat common-passwords.txt); do curl -X POST -d "user=admin&pass=$i" https://loginlab.fake/login; done
```

Nach 500 Versuchen landest du den richtigen Treffer – keine Rate-Limiting, keine Warnung im Logfile, kein Alarm.

---

## 10. Server-Side Request Forgery (SSRF)

**Definition:** Der Server wird dazu gebracht, HTTP-Anfragen an interne Systeme zu senden, die vom Angreifer gesteuert werden.

**Hack Along Beispiel:** In `ImgFetcherApp` kannst du Bilder über URLs hochladen. Du gibst ein:

```
http://localhost:8000/internal/adminconfig
```

Der Server lädt diese URL – obwohl sie nur intern verfügbar ist. Du erhältst die Antwort zurück und entdeckst geheime Admin-Informationen.

---

**Hinweis:** Dieses Cheat Sheet basiert auf der OWASP Top 10 – 2021.

Weitere Informationen: [https://owasp.org/Top10](https://owasp.org/Top10)

---

**Autor:** IT Security Cheat Sheet – erstellt mit Hilfe von ChatGPT

**Lizenz:** CC BY-SA 4.0

