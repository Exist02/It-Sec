# Understanding APIs eine Auffrischung

### Was ist eine API und warum ist das Wichtig?

API steht für Anwendungsprogrammierschnittstelle. Es handelt sich um eine Middleware, die die Kommunikation zwischen zwei Softwarekomponenten unter Verwendung einer Reihe von Protokollen und Definitionen erleichtert. Im API-Kontext bezieht sich der Begriff „Anwendung“ auf jede Software mit einer bestimmten Funktionalität, und „Schnittstelle“ bezieht sich auf den Dienstleistungsvertrag zwischen zwei Anwendungen, der die Kommunikation über Anfragen und Antworten ermöglicht. Die API-Dokumentation enthält alle Informationen darüber, wie die Entwickler diese Antworten und Anfragen strukturiert haben. Die Bedeutung von APIs für die Anwendungsentwicklung lässt sich in einem einzigen Satz zusammenfassen: API ist ein Baustein für die Entwicklung komplexer und unternehmensweiter Anwendungen.

###  Kürzliche Data Breaches durch APIs
- LinkedIn-Datenpanne: Im Juni 2021 wurden die Daten von über 700 Millionen LinkedIn-Nutzern in einem Dark-Web-Forum zum Verkauf angeboten, die durch Ausnutzung der LinkedIn-API abgegriffen wurden. Der Hacker veröffentlichte eine Stichprobe von 1 Million Datensätzen, um die Legitimität der LinkedIn-Verletzung zu bestätigen. Diese enthielt die vollständigen Namen der Nutzer, E-Mail-Adressen, Telefonnummern, Geolocation-Datensätze, LinkedIn-Profil-Links, Informationen zur Berufserfahrung und andere Kontodaten für soziale Medien.

- Twitter- Datenleck: Im Juni 2022 wurden die Daten von mehr als 5,4 Millionen Twitter-Nutzern zum Verkauf im Dark Web freigegeben. Die Hacker nutzten dazu eine Sicherheitslücke in der Twitter-API, die den Twitter-Handle mit einer Handynummer oder einer E-Mail verglich.

 - PIXLR- Datenleck: Im Januar 2021 kam es bei PIXLR, einer Online-Fotoeditor-App, zu einem Datenleck, von dem rund 1,9 Millionen Nutzer betroffen waren. Alle von den Hackern gesammelten Daten wurden in einem Dark-Web-Forum veröffentlicht, darunter Nutzernamen, E-Mail-Adressen, Länder und gehashte Passwörter.

# Schwachstelle 1 -Broken Object Level Authorisation (BOLA)

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


# Schwachstelle 2 -Broken User Authentification (BUA)