# Understanding APIs eine Auffrischung

### Was ist eine API und warum ist das Wichtig?

API steht für Anwendungsprogrammierschnittstelle. Es handelt sich um eine Middleware, die die Kommunikation zwischen zwei Softwarekomponenten unter Verwendung einer Reihe von Protokollen und Definitionen erleichtert. Im API-Kontext bezieht sich der Begriff „Anwendung“ auf jede Software mit einer bestimmten Funktionalität, und „Schnittstelle“ bezieht sich auf den Dienstleistungsvertrag zwischen zwei Anwendungen, der die Kommunikation über Anfragen und Antworten ermöglicht. Die API-Dokumentation enthält alle Informationen darüber, wie die Entwickler diese Antworten und Anfragen strukturiert haben. Die Bedeutung von APIs für die Anwendungsentwicklung lässt sich in einem einzigen Satz zusammenfassen: API ist ein Baustein für die Entwicklung komplexer und unternehmensweiter Anwendungen.

###  Kürzliche Data Breaches durch APIs
- LinkedIn-Datenpanne: Im Juni 2021 wurden die Daten von über 700 Millionen LinkedIn-Nutzern in einem Dark-Web-Forum zum Verkauf angeboten, die durch Ausnutzung der LinkedIn-API abgegriffen wurden. Der Hacker veröffentlichte eine Stichprobe von 1 Million Datensätzen, um die Legitimität der LinkedIn-Verletzung zu bestätigen. Diese enthielt die vollständigen Namen der Nutzer, E-Mail-Adressen, Telefonnummern, Geolocation-Datensätze, LinkedIn-Profil-Links, Informationen zur Berufserfahrung und andere Kontodaten für soziale Medien.

- Twitter- Datenleck: Im Juni 2022 wurden die Daten von mehr als 5,4 Millionen Twitter-Nutzern zum Verkauf im Dark Web freigegeben. Die Hacker nutzten dazu eine Sicherheitslücke in der Twitter-API, die den Twitter-Handle mit einer Handynummer oder einer E-Mail verglich.

 - PIXLR- Datenleck: Im Januar 2021 kam es bei PIXLR, einer Online-Fotoeditor-App, zu einem Datenleck, von dem rund 1,9 Millionen Nutzer betroffen waren. Alle von den Hackern gesammelten Daten wurden in einem Dark-Web-Forum veröffentlicht, darunter Nutzernamen, E-Mail-Adressen, Länder und gehashte Passwörter.

# Schwachstelle 1 -Broken Object Level Autorisation (BOLA)

## Wie kann sowas Passieren? 
Im Allgemeinen werden API-Endpunkte für die gängige Praxis des Abrufs und der Bearbeitung von Daten über Objektkennungen verwendet. BOLA bezieht sich auf Insecure Direct Object Reference (IDOR) - ein Szenario, bei dem der Benutzer die Eingabefunktionalität verwendet und Zugang zu Ressourcen erhält, für die er keine Zugriffsberechtigung hat. In einer API werden solche Kontrollen in der Regel durch die Programmierung in Modellen (Model-View-Controller Architecture) auf der Code-Ebene implementiert.

## Was ist der erwartete Impact davon?

Das Fehlen von Kontrollen zur Verhinderung des unbefugten Zugriffs auf Objekte kann zu Datenlecks und in einigen Fällen zur vollständigen Übernahme von Konten führen. Die in der Datenbank gespeicherten Nutzer- oder Abonnentendaten spielen eine entscheidende Rolle für den Ruf der Marke eines Unternehmens; wenn solche Daten über das Internet durchsickern, kann dies zu erheblichen finanziellen Verlusten führen.

## Mitigation Measures 

- Ein Autorisierungsmechanismus, der sich auf Benutzerrichtlinien und Hierarchien stützt, sollte angemessen implementiert sein.
- Strenge Zugangskontrollmethoden, um zu prüfen, ob der angemeldete Benutzer berechtigt ist, bestimmte Aktionen durchzuführen.
- Förderung der Verwendung völlig zufälliger Werte (starke Ver- und Entschlüsselungsmechanismen) für nahezu unvorhersehbare Token.

## Praktisches Beispiel

Rahmen des Beispiels 
Provided ist eine Testumgebung mit einem Online Tool welches zum Debuggen von API Endpoints genutzt wird. 

Ablauf:
- Öffnen Sie die VM. Sie werden feststellen, dass der Chrome-Browser und die Anwendung Talend API Tester automatisch ausgeführt werden, die wir zum Debuggen der API-Endpunkte verwenden werden.
- Bob arbeitet als API-Entwickler im Unternehmen MHT und hat einen Endpunkt `/apirule1/users/{ID}` entwickelt, der es anderen Anwendungen oder Entwicklern ermöglicht, Informationen durch Senden einer Mitarbeiter-ID anzufordern. In der VM können Sie Ergebnisse anfordern, indem Sie `GET`-Anforderungen an `http://localhost:80/MHT/apirule1_v/user/1` senden.
- Was ist das Problem mit dem obigen API-Aufruf? Das Problem besteht darin, dass der Endpunkt eingehende API-Aufrufe nicht validiert, um zu bestätigen, dass die Anforderung gültig ist. Es wird nicht geprüft, ob die Person, die den API-Aufruf anfordert, ihn anfordern kann oder nicht.
- Die Lösung für dieses Problem ist recht einfach: Bob wird einen Autorisierungsmechanismus implementieren, mit dem er feststellen kann, wer API-Aufrufe tätigen kann, um auf Mitarbeiter-ID-Informationen zuzugreifen.
- Der Zweck wird durch Zugriffs-Tokens oder Autorisierungs-Tokens in der Kopfzeile erreicht. Im obigen Beispiel fügt Bob ein Autorisierungs-Token hinzu, so dass nur Header mit gültigen Autorisierungs-Tokens einen Aufruf an diesen Endpunkt tätigen können.
- Wenn man in der VM ein gültiges `Autorisierungs-Token` hinzufügt und `http://localhost:80/MHT/apirule1_s/user/1` aufruft, kann man nur dann die richtigen Ergebnisse erhalten. Außerdem wird bei allen API-Aufrufen mit einem ungültigen Token die Fehlermeldung `403 Forbidden` angezeigt.


# Schwachstelle 2 -Broken User Authentifikation (BUA)

## Wie Kann die Schwachstelle Auftreten?
Die Benutzerauthentifizierung ist der wichtigste Aspekt bei der Entwicklung jeder Anwendung, die sensible Daten enthält. Eine fehlerhafte Benutzerauthentifizierung (Broken User Authentication, BUA) ist ein Szenario, in dem ein API-Endpunkt einem Angreifer den Zugriff auf eine Datenbank oder die Erlangung höherer Privilegien als die bestehenden ermöglicht. Der Hauptgrund für BUA ist entweder eine ungültige Implementierung der Authentifizierung, wie z. B. die Verwendung falscher E-Mail-/Passwort-Abfragen usw., oder das Fehlen von Sicherheitsmechanismen wie Autorisierungs-Header, Token usw.

Stellen man sich ein Szenario vor, in dem ein Angreifer die Fähigkeit erlangt, eine Authentifizierungs-API zu missbrauchen; dies führt schließlich zu Datenlecks, Löschung, Änderung oder sogar zur vollständigen Übernahme des Kontos durch den Angreifer. In der Regel haben Hacker spezielle Skripte erstellt, um Profile von Benutzern auf einem System zu erstellen, sie aufzuzählen und Authentifizierungsendpunkte zu identifizieren. Ein schlecht implementiertes Authentifizierungssystem kann dazu führen, dass ein beliebiger Benutzer die Identität eines anderen Benutzers annimmt.

## Impact
Bei einer fehlerhaften Benutzerauthentifizierung können Angreifer die authentifizierte Sitzung oder den Authentifizierungsmechanismus kompromittieren und leicht auf sensible Daten zugreifen. Böswillige Akteure können sich als autorisierte Personen ausgeben und unerwünschte Aktivitäten durchführen, einschließlich einer vollständigen Übernahme des Kontos.

## Mitigation Measures
- Garantieren Sie komplexe Passwörter mit höherer Entropie für Endbenutzer.
- Geben Sie keine sensiblen Anmeldedaten in GET- oder POST-Anfragen preis.
- Aktivieren Sie starke JSON-Web-Tokens (JWT), Autorisierungs-Header usw.
- Sorgen Sie für die Implementierung einer Multifaktor-Authentifizierung (wo möglich), einer Kontosperre oder eines Captcha-Systems, um Brute-Force-Angriffe auf bestimmte Benutzer zu verhindern. 
- Sicherstellen, dass Passwörter nicht im Klartext in der Datenbank gespeichert werden, um eine weitere Übernahme des Kontos durch den Angreifer zu verhindern.

## Praktisches Beispiel: 

Rahmen des Beispiels 
Provided ist eine Testumgebung mit einem Online Tool welches zum Debuggen von API Endpoints genutzt wird. 

- Bob weiß, dass die Authentifizierung von entscheidender Bedeutung ist, und wurde beauftragt, einen API-Endpunkt `apirule2/user/login_v` zu entwickeln, der die Authentifizierung anhand der angegebenen E-Mail und des Passworts vornimmt.
- Der Endpunkt gibt ein Token zurück, das als Authorization-Token-Header (GET-Anfrage) an apirule2/user/details weitergegeben wird, um Details zu dem jeweiligen Mitarbeiter anzuzeigen. Bob hat den Login-Endpunkt erfolgreich entwickelt; allerdings hat er nur die E-Mail zur Validierung des Benutzers aus der Benutzertabelle verwendet und das Passwortfeld in der SQL-Abfrage ignoriert. Ein Angreifer benötigt nur die E-Mail-Adresse des Opfers, um ein gültiges Token zu erhalten oder das Konto zu übernehmen.
- In der VM können Sie dies testen, indem Sie eine `POST`-Anfrage an `http://localhost:80/MHT/apirule2/user/login_v` mit E-Mail und Passwort in den Formularparametern senden.
- Wie wir sehen können, hat der verwundbare Endpunkt ein Token erhalten, welcher an `/apirule2/user/details` weitergeleitet werden kann, um Details über einen Benutzer zu erhalten.

Das sieht dann wie Folgt aus: 
https://imgur.com/bgdEqOb

Um dies zu beheben, werden wir die Logik der Anmeldeabfrage aktualisieren und sowohl E-Mail als auch Passwort für die Validierung verwenden. Der Endpunkt /apirule2/user/login_s ist ein gültiger Endpunkt, der den Benutzer sowohl auf der Grundlage des Passworts als auch der E-Mail autorisiert.

# Schwachstelle 3 -Exzessive Daten Exposure

## Wie Kann es zu der Schwachstelle kommen? 

Eine übermäßige Offenlegung von Daten liegt vor, wenn Anwendungen dazu neigen, dem Benutzer über eine API-Antwort mehr als die gewünschten Informationen preiszugeben. Die Anwendungsentwickler neigen dazu, alle Objekteigenschaften (unter Berücksichtigung der generischen Implementierungen) offenzulegen, ohne deren Empfindlichkeitsgrad zu berücksichtigen. Sie überlassen dem Front-End-Entwickler die Aufgabe des Filterns, bevor sie dem Benutzer angezeigt werden. Folglich kann ein Angreifer die Antwort über die API abfangen und schnell die gewünschten vertraulichen Daten extrahieren. Die Laufzeiterkennungs-Tools oder die allgemeinen Sicherheitsscan-Tools können eine Warnung über diese Art von Schwachstelle ausgeben. Sie können jedoch nicht zwischen legitimen Daten, die zurückgegeben werden sollen, und sensiblen Daten unterscheiden.

## Impact 

Ein böswilliger Akteur kann den Datenverkehr erfolgreich ausspähen und leicht an vertrauliche Daten gelangen, einschließlich persönlicher Daten wie Kontonummern, Telefonnummern, Zugangstoken und vieles mehr. Typischerweise antworten APIs mit sensiblen Token, die später verwendet werden können, um Anrufe an andere kritische Endpunkte zu tätigen.

## Mitigation Measures

- Überlassen Sie die Filterung sensibler Daten niemals dem Front-End-Entwickler.
- Stellen Sie sicher, dass die Antworten der API von Zeit zu Zeit überprüft werden, um zu gewährleisten, dass nur legitime Daten zurückgegeben werden und um zu prüfen, ob sie ein Sicherheitsproblem darstellen.
- Vermeiden Sie die Verwendung generischer Methoden wie to_string() und to_json().
- Verwenden Sie API-Endpunkttests durch verschiedene Testfälle und überprüfen Sie durch automatisierte und manuelle Tests, ob die API zusätzliche Daten auslässt.
## Praktisches Beispiel

Rahmen des Beispiels 
Provided ist eine Testumgebung mit einem Online Tool welches zum Debuggen von API Endpoints genutzt wird. 

- Verwenden Sie weiterhin den Chrome-Browser und Talend API Tester für das Debugging in der VM
- Das Unternehmen MHT führte ein kommentarbasiertes Webportal ein, das die Kommentare der Benutzer aufnimmt und in der Datenbank speichert, sowie andere Informationen wie Standort, Geräteinformationen usw., um die Benutzerfreundlichkeit zu verbessern.
- Bob wurde damit beauftragt, einen Endpunkt zu entwickeln, der die Kommentare der Benutzer auf der Haupt-Website des Unternehmens anzeigt. Er entwickelte einen Endpunkt `apirule3/comment_v/{id}`, der alle für einen Kommentar verfügbaren Informationen aus der Datenbank abruft. Bob nahm an, dass der Front-End-Entwickler die Informationen herausfiltern würde, während er sie auf der Haupt-Website des Unternehmens anzeigt.
- Was ist hier das Problem? Die API sendet mehr Daten als gewünscht. Anstatt sich auf einen Front-End-Ingenieur zu verlassen, um Daten herauszufiltern, müssen nur relevante Daten aus der Datenbank gesendet werden.

Bob erkannte seinen Fehler, aktualisierte den Endpunkt und erstellte einen gültigen Endpunkt `/apirule3/comment_s/{id},` der nur die erforderlichen Informationen an den Entwickler zurückgibt (wie unten gezeigt).

# Schwachstelle 4 -Lack of Resource & Rate Limiting

## Wie Kann es zu der Schwachstelle kommen? 

text

## Impact 

text

## Mitigation Measures

text


## Praktisches Beispiel

Rahmen des Beispiels 
Provided ist eine Testumgebung mit einem Online Tool welches zum Debuggen von API Endpoints genutzt wird. 