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

## Mitigation Meassures 

- Ein Autorisierungsmechanismus, der sich auf Benutzerrichtlinien und Hierarchien stützt, sollte angemessen implementiert sein.
- Strenge Zugangskontrollmethoden, um zu prüfen, ob der angemeldete Benutzer berechtigt ist, bestimmte Aktionen durchzuführen.
- Förderung der Verwendung völlig zufälliger Werte (starke Ver- und Entschlüsselungsmechanismen) für nahezu unvorhersehbare Token.

## Praktisches Beispiel
