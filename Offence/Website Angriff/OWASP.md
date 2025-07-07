OWASP steht für ***Open Web Application Security Project*** und ist eine gemeinnützige Organisation, die sich für die Verbesserung der Sicherheit von Webanwendungen einsetzt. Das Open Worldwide Application Security Project (OWASP) ist eine Non-Profit-Organisation mit dem Ziel, die Sicherheit von Anwendungen, Diensten und Software im Allgemeinen zu verbessern. Ziel von OWASP ist es unter anderem, Entwickler, Designer, Softwarearchitekten und Unternehmer über die Risiken zu informieren, die mit den häufigsten Sicherheitslücken von Webanwendungen verbunden sind.

# 1. Broken Access Control

## Wie ist das Definiert?

Websites haben Seiten, die vor normalen Besuchern geschützt sind. So sollte beispielsweise nur der Administrator der Website auf eine Seite zugreifen können, um andere Benutzer zu verwalten. Wenn ein Website-Besucher auf geschützte Seiten zugreifen kann, die er nicht sehen soll, ist die Zugriffssteuerung defekt.
  
Wenn ein regulärer Besucher auf geschützte Seiten zugreifen kann, kann dies zu Folgendem führen:
- Einsicht in vertrauliche Informationen anderer Benutzer
- Zugriff auf nicht autorisierte Funktionen

Einfach ausgedrückt: Eine lückenhafte Zugriffssteuerung ermöglicht es Angreifern, die Autorisierung zu umgehen und so vertrauliche Daten einzusehen oder Aufgaben auszuführen, für die sie nicht vorgesehen sind.

### Beispiel 
So wurde beispielsweise 2019 eine Sicherheitslücke gefunden, durch die ein Angreifer jedes einzelne Bild eines als privat gekennzeichneten Youtube-Videos abrufen konnte. Der Forscher, der die Schwachstelle fand, zeigte, dass er mehrere Bilder anfordern und das Video irgendwie rekonstruieren konnte. Da ein Nutzer, der ein Video als privat markiert, davon ausgeht, dass niemand Zugriff darauf hat, wurde dies tatsächlich als Sicherheitslücke in der Zugangskontrolle akzeptiert.

## Broken Access Control IDOR 

IDOR oder Insecure Direct Object Reference bezeichnet eine Schwachstelle in der Zugriffskontrolle, durch die man auf Ressourcen zugreifen kann, die man normalerweise nicht sehen könnte. Dies geschieht, wenn der Programmierer eine direkte Objektreferenz preisgibt, bei der es sich lediglich um einen Identifikator handelt, der auf spezifische Objekte innerhalb des Servers verweist. Mit Objekt können wir eine Datei, einen Benutzer, ein Bankkonto in einer Bankanwendung oder etwas anderes meinen.

### Beispiel zur genaueren Erläuterung
https://imgur.com/ov47sgS

# 2. Cryptographic Failures

## Definition

Ein kryptografisches Versagen bezieht sich auf jede Schwachstelle, die durch den Missbrauch (oder die mangelnde Verwendung) kryptografischer Algorithmen zum Schutz sensibler Informationen entsteht. Webanwendungen erfordern Kryptographie, um die Vertraulichkeit für ihre Benutzer auf vielen Ebenen zu gewährleisten.