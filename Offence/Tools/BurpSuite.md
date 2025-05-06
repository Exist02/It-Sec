### Repeater
Der Burp Suite Repeater ermöglicht es uns, abgefangene Anfragen zu modifizieren und erneut an ein Ziel unserer Wahl zu senden. Er ermöglicht es uns, die vom Burp Proxy aufgefangenen Anfragen zu manipulieren und sie bei Bedarf wiederholt zu senden. Alternativ können wir Anfragen auch manuell von Grund auf neu erstellen, ähnlich wie mit einem Befehlszeilen-Tool wie cURL.

Die Möglichkeit, Anfragen mehrfach zu bearbeiten und erneut zu senden, macht Repeater zu einem unschätzbaren Werkzeug für die manuelle Erkundung und das Testen von Endpunkten. Repeater bietet eine benutzerfreundliche grafische Oberfläche für die Erstellung von Anfragen und bietet verschiedene Ansichten der Antwort, einschließlich einer Rendering-Engine für eine grafische Darstellung.


### Intruder
Intruder ist ein Tool, das in Burp Suite integriert ist und es ermöglicht, automatisch Anfragen zu ändern und wiederholt mit verschiedenen Eingabewerten zu testen. Es sendet mehrere Anfragen mit geringfügig unterschiedlichen Werten basierend auf benutzerdefinierten Konfigurationen. Intruder hat verschiedene Anwendungsmöglichkeiten, wie das Brute-Forcing von Anmeldeformularen durch Ersetzen von Benutzername- und Passwortfeldern mit Werten aus einer Wortliste oder das Durchführen von Fuzzing-Angriffen mit Wortlisten zur Testung von Unterverzeichnissen, Endpunkten oder virtuellen Hosts. Die Funktionalität von Intruder ist vergleichbar mit Tools wie Wfuzz oder ffuf. Allerdings ist es wichtig zu beachten, dass die Geschwindigkeit von Intruder in der Burp Community Edition begrenzt ist, was im Vergleich zur Burp Professional-Version zu einer deutlich reduzierten Geschwindigkeit führt. Daher greifen Sicherheitsexperten oft auf andere Fuzzing- und Brute-Forcing-Tools zurück. Trotzdem ist Intruder ein wertvolles Werkzeug und es lohnt sich, zu lernen, wie man es effektiv einsetzt.

Intruder wird in 4 Tabs Aufgeteilt diese sind: 
**Positions:** Dieser Tab lässt uns eine Angriffsart aus wählen und einstellen wo wir unser Payload in der Request Vorlage einfügen wollen
**Payloads:** Hier können wir Werte auswählen, die in die auf der Registerkarte „Position“ definierten Positionen eingefügt werden sollen. Es gibt verschiedene Payload-Optionen, z. B. das Laden von Elementen aus einer Wortliste. Die Art und Weise, wie diese Payloads in die Vorlage eingefügt werden, hängt von dem auf der Registerkarte Positionen gewählten Angriffstyp ab. Die Registerkarte Payloads ermöglicht es uns auch, das Verhalten von Intruder in Bezug auf Payloads zu ändern, z. B. die Definition von Vorverarbeitungsregeln für jede Payload (z. B. Hinzufügen eines Präfixes oder Suffixes, Durchführen von Vergleichen und Ersetzen oder Überspringen von Payloads auf der Grundlage einer definierten Regex)
**Ressource Pool**: Diese Registerkarte ist in der Burp Community Edition nicht besonders nützlich. Sie ermöglicht die Ressourcenzuweisung unter verschiedenen automatisierten Aufgaben in Burp Professional. Ohne Zugriff auf diese automatisierten Aufgaben ist diese Registerkarte von begrenzter Bedeutung.
**Settings**: Dieser Tab erlaubt es uns Angriffs verhalten zu Konfigurieren. Es geht vor allem darum, wie Burp Ergebnisse und den Angriff selbst behandelt. Zum Beispiel können wir Anfragen, die einen bestimmten Text enthalten, kennzeichnen oder die Reaktion von Burp auf Weiterleitungsantworten (3xx) definieren.

Funktionsweisen dieser Felder genauer Erklärt:

**Positions**: 
Bevor man einen Angriff Starten kann muss man festlegen an welchen stellen die Payloads eingefügt werden sollen. Burp versucht hier auch schon selbständig an den Passenden Stellen entsprechende Payloads einzusetzen. Auf der Rechten seite sind auch die Operatoren um zu markieren wo Payloads hin sollen siehe unten erklärt

- The `Add §` button allows us to define new positions manually by highlighting them within the request editor and then clicking the button.
- The `Clear §` button removes all defined positions, providing a blank canvas where we can define our own positions.
- The `Auto §` button automatically attempts to identify the most likely positions based on the request. This feature is helpful if we previously cleared the default positions and want them back

**Payloads**:
Dieser Tab wird auch nochmalö in 4 Teile Aufgeteilt. 
1. Payload Sets. 
- In diesem Abschnitt können wir die Position auswählen, für die wir ein Payload-Set konfigurieren wollen, und die Art der Payload auswählen, die wir verwenden wollen.
- Bei Angriffstypen, die nur ein einziges Payload-Set erlauben (Sniper oder Rammbock), gibt es im Dropdown-Menü „Payload Set“ nur eine Option, unabhängig von der Anzahl der definierten Positionen.
- Bei Angriffstypen, die mehrere Payload-Sets erfordern (Pitchfork oder Cluster Bomb), gibt es für jede Position einen Eintrag im Dropdown-Menü.
Hinweis: Bei der Zuweisung von Nummern im Dropdown-Menü „Payload Set“ für mehrere Positionen ist eine Reihenfolge von oben nach unten und von links nach rechts einzuhalten. Bei zwei Positionen (username=§pentester§&password=§Expl01ted§) würde sich beispielsweise das erste Element in der Dropdown-Liste „Payload Set“ auf das Feld ‚username‘ und das zweite Element auf das Feld „password“ beziehen.

2. Payload Settings:
- Diese Sektion ermöglicht Einstellungen Spezifisch auf den eingestellten Payload Typ und das aktive Payload Set
- Jede Payload hat seine eigenen Optionen und Funktionen die sich lohnen das man sich diese anschaut

3. Payload Processing
- In diesem Abschnitt können wir Regeln definieren, die auf jede Payload im Satz angewendet werden, bevor sie an das Ziel gesendet wird
- Beispielsweise kann jedes Wort großgeschrieben werden, Payloads, die mit einem Regex-Muster übereinstimmen, können übersprungen werden, oder es können andere Umwandlungen oder Filterungen vorgenommen werden.
- Auch wenn man diesen Abschnitt nicht häufig verwendet, kann er sehr wertvoll sein, wenn für einen Angriff eine spezielle Payload-Verarbeitung erforderlich ist.,
- https://portswigger.net/burp/documentation/desktop/tools/intruder/configure-attack/processing

4. Payload Encoding
- Diese Sektion erlaubt uns wie der Name schon sagt das Encoding für die Payload auszuwählen.
- Standardmäßig nutzt Burp Suite URL Encoding um damit sichergestellt ist das die Payloads übermittelt werden
- Wir können die Standard-URL-Kodierungsoptionen außer Kraft setzen, indem wir die Liste der zu kodierenden Zeichen ändern oder die Markierung des Kontrollkästchens „URL-Kodierung dieser Zeichen“ aufheben.

**Angriffs Typen**

1. Sniper
	- Ist meistens der Standard Angriffs Typ
	- Effektiv für Angriffe an einer einzigen Stelle, wie z. B. Brute-Force-Angriffe auf Passwörter oder Fuzzing für API-Endpunkte
	- Bei einem Sniper Angriff gibt man einen Satz Payloads an. Dieser kann z.B. eine Wordlist oder ein Zahlen Bereich sein. Intruder fügt dann die Payloads an der definierten stelle in dem Request ein
	- Die anzahl der Requests die Sniper dann versendet brechnen sich dann via
		- Anfragen = AnzahlPayloads x AnzahlPositionen
	- 

2. Battering Ram
	-  unterscheidet sich von Sniper dadurch, dass er dieselbe Payload in jeder Position gleichzeitig platziert, anstatt jede Payload nacheinander in jeder Position zu platzieren.
	-  ist nützlich, wenn wir dieselbe Payload gegen mehrere Positionen gleichzeitig testen wollen, ohne dass eine sequenzielle Substitution erforderlich ist.

3. Pitchfork
	- ist vergleichbar mit mehreren Sniper-Attacken, die gleichzeitig ausgeführt werden
	-  Während Sniper einen Payload-Satz verwendet, um alle Positionen gleichzeitig zu testen, verwendet Pitchfork einen Payload-Satz pro Position (bis zu einem Maximum von 20) und durchläuft sie alle gleichzeitig.
	- Wichtig ist wenn die Listen ungleich lang sind dann hört Pitchfork auf wenn die kleinere durch ist
	- Diese Angriffsart ist besonders nützlich bei der Durchführung von Credential-Stuffing-Angriffen oder wenn mehrere Positionen separate Payload-Sets erfordern. Sie ermöglicht das gleichzeitige Testen mehrerer Positionen mit unterschiedlichen Payloads.

4. Cluster Bomb
	- ermöglicht es uns, mehrere Payload-Sets zu wählen, eines pro Position (bis zu einem Maximum von 20). Im Gegensatz zu Pitchfork, wo alle Nutzlastsets gleichzeitig getestet werden, durchläuft Cluster Bomb jedes Nutzlastset einzeln, um sicherzustellen, dass jede mögliche Kombination von Nutzlasten getestet wird.
	- Cluster-Bomben-Angriffe können eine beträchtliche Menge an Datenverkehr erzeugen, da sie jede Kombination testen
		- Anfragen = AnzahlZeileninPayload * AnzahlZeilen in payload
	- WICHTIG: Ist sehr mit Vorsicht zu genießen da sehr laut aufgrund der menge an Traffic
	- WICHTIG: in derr Burp Community Edition ist die Intruder und damit auch Cluster bomb Rate limited
	- Der Angriffstyp Cluster-Bombe ist besonders nützlich für Szenarien, bei denen die Zuordnung zwischen Benutzernamen und Kennwörtern unbekannt ist.


### Decoder

Das Decoder-Modul der Burp Suite bietet dem Benutzer Möglichkeiten zur Datenmanipulation. Wie der Name schon sagt, dekodiert es nicht nur Daten, die während eines Angriffs abgefangen wurden, sondern bietet auch die Funktion, unsere eigenen Daten zu kodieren und für die Übertragung an das Ziel vorzubereiten. Mit dem Decoder können wir auch Hashsummen von Daten erstellen und eine Smart Decode-Funktion bereitstellen, die versucht, bereitgestellte Daten rekursiv zu dekodieren, bis sie wieder Klartext sind (wie die „Magic“-Funktion von Cyberchef).

### Comparer

Comparer, as the name implies, lets us compare two pieces of data, either by ASCII words or by bytes.

### Sequenzer

Mit Sequencer können wir die Entropie bzw. Zufälligkeit von „Token“ bewerten. Token sind Zeichenfolgen, die zur Identifizierung einer Sache verwendet werden und idealerweise auf kryptografisch sichere Weise erzeugt werden sollten. Bei diesen Token kann es sich um Sitzungscookies oder CSRF-Tokens (Cross-Site Request Forgery) handeln, die zum Schutz von Formularübermittlungen verwendet werden. Wenn diese Token nicht sicher generiert werden, könnten wir theoretisch kommende Tokenwerte vorhersagen. Dies könnte erhebliche Auswirkungen haben, wenn das betreffende Token beispielsweise zum Zurücksetzen von Passwörtern verwendet wird.

### Organizer
Das Organizer-Modul der Burp Suite hilft dabei, Kopien von HTTP-Anfragen zu speichern und mit Anmerkungen zu versehen, die Sie zu einem späteren Zeitpunkt wieder aufrufen möchten. Dieses Tool kann besonders nützlich sein, um Ihren Arbeitsablauf bei Penetrationstests zu organisieren. Hier sind einige der wichtigsten Funktionen:
- Anfragen speichern, die du später untersuchen möchtest, Anfragen speichern, die du bereits als interessant identifiziert hast, oder Anfragen speichern, die du später zu einem Bericht hinzufügen möchtest.
- Sie können HTTP-Anfragen von anderen Burp-Modulen wie Proxy oder Repeater an den Burp Organizer senden. Klicken Sie dazu mit der rechten Maustaste auf die Anfrage und wählen Sie An Organizer senden oder verwenden Sie den Standard-Hotkey Strg + O. Jede HTTP-Anfrage, die Sie an Organizer senden, ist eine schreibgeschützte Kopie der ursprünglichen Anfrage, die zu dem Zeitpunkt gespeichert wurde, als Sie sie an Organizer gesendet haben.

### Extensions
Zusammenfassend lässt sich sagen, dass die Schnittstelle für Erweiterungen in Burp Suite es den Benutzern ermöglicht, die installierten Erweiterungen zu verwalten und zu überwachen, sie für bestimmte Projekte zu aktivieren oder zu deaktivieren und wichtige Details, Ausgaben und Fehler im Zusammenhang mit jeder Erweiterung anzuzeigen. Durch die Verwendung von Erweiterungen wird Burp Suite zu einer leistungsstarken und anpassbaren Plattform für verschiedene Sicherheitstests und Aufgaben zur Bewertung von Webanwendungen.