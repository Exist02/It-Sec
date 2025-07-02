# Understanding APIs eine Auffrischung

## Was ist eine API und warum ist das Wichtig?

API steht für Anwendungsprogrammierschnittstelle. Es handelt sich um eine Middleware, die die Kommunikation zwischen zwei Softwarekomponenten unter Verwendung einer Reihe von Protokollen und Definitionen erleichtert. Im API-Kontext bezieht sich der Begriff „Anwendung“ auf jede Software mit einer bestimmten Funktionalität, und „Schnittstelle“ bezieht sich auf den Dienstleistungsvertrag zwischen zwei Anwendungen, der die Kommunikation über Anfragen und Antworten ermöglicht. Die API-Dokumentation enthält alle Informationen darüber, wie die Entwickler diese Antworten und Anfragen strukturiert haben. Die Bedeutung von APIs für die Anwendungsentwicklung lässt sich in einem einzigen Satz zusammenfassen: API ist ein Baustein für die Entwicklung komplexer und unternehmensweiter Anwendungen.

##  Kürzliche Data Breaches durch APIs

- LinkedIn-Datenpanne: Im Juni 2021 wurden die Daten von über 700 Millionen LinkedIn-Nutzern in einem Dark-Web-Forum zum Verkauf angeboten, die durch Ausnutzung der LinkedIn-API abgegriffen wurden. Der Hacker veröffentlichte eine Stichprobe von 1 Million Datensätzen, um die Legitimität der LinkedIn-Verletzung zu bestätigen. Diese enthielt die vollständigen Namen der Nutzer, E-Mail-Adressen, Telefonnummern, Geolocation-Datensätze, LinkedIn-Profil-Links, Informationen zur Berufserfahrung und andere Kontodaten für soziale Medien.

- Twitter- Datenleck: Im Juni 2022 wurden die Daten von mehr als 5,4 Millionen Twitter-Nutzern zum Verkauf im Dark Web freigegeben. Die Hacker nutzten dazu eine Sicherheitslücke in der Twitter-API, die den Twitter-Handle mit einer Handynummer oder einer E-Mail verglich.

 - PIXLR- Datenleck: Im Januar 2021 kam es bei PIXLR, einer Online-Fotoeditor-App, zu einem Datenleck, von dem rund 1,9 Millionen Nutzer betroffen waren. Alle von den Hackern gesammelten Daten wurden in einem Dark-Web-Forum veröffentlicht, darunter Nutzernamen, E-Mail-Adressen, Länder und gehashte Passwörter.



Man kann mit Fug und Recht behaupten, dass mehr als die Hälfte der Top-10-Liste der OWASP-API-Sicherheit für die Autorisierung und Authentifizierung relevant ist. In den meisten Fällen werden API-Systeme aufgrund von Fehlern in den Autorisierungs- und Authentifizierungsmechanismen und Fehlkonfigurationen der Sicherheit gehackt.

Kurz gesagt, API-Entwickler müssen APIs gemäß den besten Cybersicherheitspraktiken absichern. Den Modulen wie Anmeldung, rollenbasierter Zugriff, Einstellung von Benutzerprofilen usw. muss mehr Bedeutung beigemessen werden, da böswillige Akteure dazu neigen, bekannte Endpunkte anzusteuern, um Zugang zum System zu erhalten.


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
Fehlende Ressourcen und  rate limiting  bedeuten, dass APIs keine Beschränkung der Häufigkeit der von Kunden angeforderten Ressourcen oder der Dateigröße durchsetzen, was die Leistung des API-Servers stark beeinträchtigt und zu DoS (Denial of Service) oder Nichtverfügbarkeit des Dienstes führt. Stellen Sie sich ein Szenario vor, in dem eine API-Beschränkung nicht durchgesetzt wird, so dass ein Benutzer (in der Regel ein Eindringling) mehrere GB-Dateien gleichzeitig hochladen oder eine beliebige Anzahl von Anfragen pro Sekunde stellen kann. Solche API-Endpunkte führen zu einer übermäßigen Nutzung von Netzwerk-, Speicher-, Rechenressourcen usw.

Heutzutage nutzen Angreifer solche Angriffe, um die Nichtverfügbarkeit von Diensten für ein Unternehmen sicherzustellen und so den Ruf der Marke durch erhöhte Ausfallzeiten zu schädigen. Ein einfaches Beispiel ist die Nichteinhaltung des Captcha-Systems auf dem Anmeldeformular, das es jedem ermöglicht, über ein kleines, in Python geschriebenes Skript zahlreiche Abfragen an die Datenbank zu stellen.

## Impact 

Der Angriff zielt in erster Linie auf die Verfügbarkeitsgrundsätze der Sicherheit ab (CIA Triade); er kann jedoch auch den Ruf der Marke schädigen und finanzielle Verluste verursachen.

## Mitigation Measures

- Sicherstellen, dass ein Captcha verwendet wird, um Anfragen von automatisierten Skripten und Bots zu vermeiden.
- Implementierung eines Limits, d.h. wie oft ein Client eine API innerhalb einer bestimmten Zeit aufrufen kann und sofortige Benachrichtigung, wenn das Limit überschritten wird.
- Legen Sie die maximale Datengröße für alle Parameter und Payloads fest, d. h. die maximale Länge von Strings und die maximale Anzahl von Array-Elementen.


## Praktisches Beispiel

Rahmen des Beispiels 
Provided ist eine Testumgebung mit einem Online Tool welches zum Debuggen von API Endpoints genutzt wird. 

- Verwenden Sie weiterhin den Chrome-Browser und Talend API Tester für das Debugging in der VM
- Das Unternehmen MHT kaufte einen E-Mail-Marketingplan (20.000 E-Mails pro Monat) für den Versand von Marketing- und Passwort-Wiederherstellungs-E-Mails usw. Bob erkannte, dass er erfolgreich eine Login-API entwickelt hatte, aber es muss eine "Passwort vergessen"-Option geben, die zur Wiederherstellung eines Kontos verwendet werden kann.
- Er begann mit der Erstellung eines Endpunkts `/apirule4/sendOTP_v,` der einen 4-stelligen Zahlencode an die E-Mail-Adresse des Benutzers sendet. Ein authentifizierter Benutzer verwendet dieses One Time Password (OTP), um das Konto wiederherzustellen.
- Was ist hier das Problem? Bob hat keine Ratenbegrenzung für den Endpunkt aktiviert. Ein böswilliger Akteur kann ein kleines Skript schreiben und den Endpunkt mit brachialer Gewalt aushebeln, um viele E-Mails in wenigen Sekunden zu versenden und den kürzlich erworbenen E-Mail-Marketingplan des Unternehmens zu nutzen (finanzieller Schaden).

Schließlich fand Bob eine intelligente Lösung (`/apirule4/sendOTP_s`) und aktivierte die Ratenbegrenzung, so dass der Benutzer 2 Minuten warten muss, um erneut ein OTP-Token anzufordern.



# Schwachstelle 5 -Broken Function Level Authorisation
## Wie Kann es zu der Schwachstelle kommen? 

Broken Function Level Authorisation reflects a scenario where a low privileged user (e.g., sales) bypasses system checks and gets access to **confidential data by impersonating a high privileged user (Admin)**. Consider a scenario of complex access control policies with various hierarchies, roles, and groups and a vague separation between regular and administrative functions leading to severe authorisation flaws. By taking advantage of these issues, the intruders can easily access the unauthorised resources of another user or, most dangerously – the administrative functions.   

Broken Function Level Authorisation reflects IDOR permission, where a user, most probably an intruder, can perform administrative-level tasks. APIs with complex user roles and permissions that can span the hierarchy are more prone to this attack.

## Impact 

Der Angriff zielt in erster Linie auf die Sicherheitsgrundsätze der Autorisierung und der Nichtabstreitbarkeit ab. Eine gebrochene Autorisierung auf funktionaler Ebene kann dazu führen, dass sich ein Eindringling als autorisierter Benutzer ausgibt und der böswillige Akteur administrative Rechte erhält, um sensible Aufgaben auszuführen.

## Mitigation Measures

- Stellen Sie sicher, dass alle Berechtigungssysteme ordnungsgemäß konzipiert und getestet werden und verweigern Sie standardmäßig jeden Zugang.
- Stellen Sie sicher, dass die Operationen nur den Nutzern erlaubt sind, die der autorisierten Gruppe angehören.
- Stellen Sie sicher, dass die API-Endpunkte auf Schwachstellen in Bezug auf die Autorisierung auf funktionaler Ebene überprüft werden und berücksichtigen Sie die Geschäftslogik der Anwendungen und der Gruppenhierarchie.


## Praktisches Beispiel

Rahmen des Beispiels 
Provided ist eine Testumgebung mit einem Online Tool welches zum Debuggen von API Endpoints genutzt wird. 

- Verwenden Sie weiterhin den Chrome-Browser und Talend API Tester für das Debugging in der VM.
- Bob wurde mit einer weiteren Aufgabe betraut, ein Admin-Dashboard für die Führungskräfte des Unternehmens zu entwickeln, damit diese alle Mitarbeiterdaten einsehen und bestimmte Aufgaben ausführen können.
- Bob entwickelte einen Endpunkt `/apirule5/users_v`, um die Daten aller Mitarbeiter aus der Datenbank abzurufen. Zum Schutz fügte er eine weitere Sicherheitsebene hinzu, indem er einen speziellen Header isAdmin in jede Anfrage einfügte. Die API holt nur dann Mitarbeiterinformationen aus der Datenbank, wenn `isAdmin=1` und das Autorisierungs-Token korrekt sind. Das `Autorisierungs-Token` für HR-Benutzer Alice lautet `YWxpY2U6dGVzdCFAISM6Nzg5Nzg=`.
- Wir sehen, dass Alice ein Nicht-Admin-Benutzer (HR) ist, aber alle Mitarbeiterdaten sehen kann, indem sie benutzerdefinierte Anfragen an den Endpunkt mit `isAdmin-Wert = 1` stellt.

Das Problem kann programmatisch gelöst werden, indem korrekte Autorisierungsregeln implementiert und die funktionalen Rollen der einzelnen Benutzer in der Datenbank während der Abfrage überprüft werden. Bob hat einen weiteren Endpunkt `/apirule5/users_s` implementiert, der die Rolle jedes Benutzers überprüft und die Daten der Mitarbeiter nur anzeigt, wenn die Rolle Admin ist.




# Schwachstelle 6 -Mass Assignment

## Wie Kann es zu der Schwachstelle kommen? 

text

## Impact 

text

## Mitigation Measures

text


## Praktisches Beispiel

Rahmen des Beispiels 
Provided ist eine Testumgebung mit einem Online Tool welches zum Debuggen von API Endpoints genutzt wird. 



# Schwachstelle 7 -Security Misconfiguration

## Wie Kann es zu der Schwachstelle kommen? 

text

## Impact 

text

## Mitigation Measures

text


## Praktisches Beispiel

Rahmen des Beispiels 
Provided ist eine Testumgebung mit einem Online Tool welches zum Debuggen von API Endpoints genutzt wird. **


# Schwachstelle 8 -Injection

## Wie Kann es zu der Schwachstelle kommen? 

text

## Impact 

text

## Mitigation Measures

text


## Praktisches Beispiel

Rahmen des Beispiels 
Provided ist eine Testumgebung mit einem Online Tool welches zum Debuggen von API Endpoints genutzt wird. 

# Schwachstelle 9 -Improper Assets Management
## Wie Kann es zu der Schwachstelle kommen? 

text

## Impact 

text

## Mitigation Measures

text


## Praktisches Beispiel

Rahmen des Beispiels 
Provided ist eine Testumgebung mit einem Online Tool welches zum Debuggen von API Endpoints genutzt wird. 


# Schwachstelle 10 -Insufficient Logging & Monitoring

## Wie Kann es zu der Schwachstelle kommen? 

text

## Impact 

text

## Mitigation Measures

text


## Praktisches Beispiel

Rahmen des Beispiels 
Provided ist eine Testumgebung mit einem Online Tool welches zum Debuggen von API Endpoints genutzt wird. 