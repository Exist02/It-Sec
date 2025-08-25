# Authentification Enumeration -> Sollte ich schon haben

# Enumeration von Usern via Verbose Errors 

Verbose Errors werden  für gewöhnlich von Developern eingesetzt um Heraus zu finden wo es Problem in der Web App gab. Aber ähnlich wie bei einer Belauschten Konversation können auch die se zu viele Informationen Preisgeben. 
Verbose Errors können Informationen wie: 
- **Interne Pfade** Wie eine Karte, die zu einem versteckten Schatz führt, zeigen diese die Dateipfade und Verzeichnisstrukturen des Anwendungsservers, die Konfigurationsdateien oder geheime Schlüssel enthalten können, die für einen normalen Benutzer nicht sichtbar sind.
- **Datenbanken Details** Diese Fehler bieten einen Einblick in die Datenbank und können Geheimnisse wie Tabellennamen und Spaltendetails preisgeben.
- **User Information**  Manchmal können diese Fehler sogar Hinweise auf Benutzernamen oder andere personenbezogene Daten enthalten und damit wichtige Anhaltspunkte für weitere Ermittlungen liefern.

preisgeben.

## Auslösen von Verbose Errors

Angreifer provozieren verbose Fehler, um die Anwendung dazu zu zwingen, ihre Geheimnisse preiszugeben. Im Folgenden sind einige gängige Techniken aufgeführt, mit denen diese Fehler provoziert werden können:

- **Ungültige Anmeldeversuche**: Das ist so, als würde man an jede Tür klopfen, um zu sehen, welche sich öffnet. Durch die absichtliche Eingabe falscher Benutzernamen oder Passwörter können Angreifer Fehlermeldungen auslösen, die dabei helfen, zwischen gültigen und ungültigen Benutzernamen zu unterscheiden. Wenn man beispielsweise einen Benutzernamen eingibt, der nicht existiert, wird möglicherweise eine andere Fehlermeldung angezeigt als bei der Eingabe eines existierenden Benutzernamens, wodurch erkennbar wird, welche Benutzernamen aktiv sind.

- **SQL-Injection**: Bei dieser Technik werden bösartige SQL-Befehle in Eingabefelder eingeschleust, in der Hoffnung, dass das System einen Fehler macht und Informationen über seine Datenbankstruktur preisgibt. Wenn beispielsweise ein einfaches Anführungszeichen ( ') in ein Anmeldefeld eingegeben wird, kann dies dazu führen, dass die Datenbank einen Fehler ausgibt und versehentlich Details über ihr Schema preisgibt.

- **Datei-Einbindung/Pfad-Traversal**: Durch Manipulation von Dateipfaden können Angreifer versuchen, auf eingeschränkte Dateien zuzugreifen, indem sie das System zu Fehlern verleiten, die interne Pfade offenlegen. Beispielsweise kann die Verwendung von Verzeichnis-Traversal-Sequenzen wie `../../` zu Fehlern führen, die eingeschränkte Dateipfade offenlegen.

- **Formularmanipulation:** Durch das Manipulieren von Formularfeldern oder Parametern kann die Anwendung dazu gebracht werden, Fehler anzuzeigen, die Backend-Logik oder sensible Benutzerinformationen offenlegen. Beispielsweise kann das Ändern versteckter Formularfelder, um Validierungsfehler auszulösen, Einblicke in das erwartete Datenformat oder die Struktur geben.

- **Anwendungs-Fuzzing**: Das Senden unerwarteter Eingaben an verschiedene Teile der Anwendung, um deren Reaktion zu beobachten, kann dabei helfen, Schwachstellen zu identifizieren. Tools wie Burp Suite Intruder werden beispielsweise verwendet, um diesen Prozess zu automatisieren. Dabei wird die Anwendung mit verschiedenen Payloads bombardiert, um zu sehen, welche davon informative Fehler provozieren.

## Die Rolle von Enumeration und Brute-Force-Angriffen

Wenn es darum geht, Authentifizierungen zu knacken, gehen Enumeration und Brute-Force-Angriffe oft Hand in Hand:

- **Benutzer-Enumeration:** Das Aufdecken gültiger Benutzernamen schafft die Voraussetzungen dafür, dass bei nachfolgenden Brute-Force-Angriffen weniger geraten werden muss.
- **Ausnutzen von verbosen Fehlern:** Die aus diesen Fehlern gewonnenen Erkenntnisse können Aspekte wie Passwortrichtlinien und Mechanismen zur Kontosperrung aufzeigen und den Weg für effektivere Brute-Force-Strategien ebnen.
