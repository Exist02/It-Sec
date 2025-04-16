Was ist das? 
Es handelt sich um eine Schwachstelle, die es einem Angreifer ermöglicht, den Webserver dazu zu bringen, eine zusätzliche oder geänderte HTTP-Anfrage an die vom Angreifer gewählte Ressource zu stellen.
Generell werden SSRF in Zwei Kategorien aufgeteilt 
1. Die Reguläre SSRF hier werden die Daten an den Bildschirm des Angreifers zurückgegeben werden
2. Die Blinde SSRF hier passiert zwar eine SSRF allerdings werden keine Daten zurück zu dem Bildschirm des Angreifers gespielt

Ein erfolgreicher SSRF-Angriff kann eine der folgenden Folgen haben: 
- Zugang zu nicht autorisierten Bereichen.
- Zugriff auf Kunden-/Organisationsdaten.
- Fähigkeit zur Skalierung auf interne Netzwerke.
- Offenlegung von Authentifizierungs-Tokens/Zugangsdaten.


#### Finden von SSRF
Bildlich dargestellte Beispiele 
https://imgur.com/09sIUOI

Wichtig bei der Blinden SSRF muss eine Software Zusätzlich genutzt werden welche HTTP Logging/Monitoring Betreibt, da bei Blind ja kein Visueller output da ist-
Software die Genutzt werden kann:
- requestbin.com
- Burp Suite Collaborator

#### Möglichkeiten wie man Sicherheitsmaßnahmen übergehen kann

#### Deny Liste
Eine Webanwendung kann eine Verweigerungsliste verwenden, um sensible Endpunkte, IP-Adressen oder Domänen vor dem Zugriff der Öffentlichkeit zu schützen, während der Zugriff auf andere Standorte weiterhin möglich ist. 

Ein spezieller Endpunkt, auf den der Zugriff beschränkt werden soll, ist der localhost, der Daten zur Serverleistung oder weitere sensible Informationen enthalten kann. 
Daher würden Domänennamen wie localhost und 127.0.0.1 in einer Deny-Liste erscheinen. 
Angreifer können eine Deny-Liste umgehen, indem sie alternative localhost-Referenzen wie 0, 0.0.0.0, 0000, 127.1, 127.*.*.*, 2130706433, 017700000001 oder Subdomänen verwenden, die einen DNS-Eintrag haben, der zur IP-Adresse 127.0.0.1 auflöst, wie 127.0.0.1.nip.io.

#### Allow List
Bei einer Allow List werden alle Anfragen abgelehnt, die nicht in einer Liste aufgeführt sind oder einem bestimmten Muster entsprechen, z. B. einer Regel, dass eine in einem Parameter verwendete URL mit https://website.thm beginnen muss. 

Ein Angreifer könnte diese Regel schnell umgehen, indem er eine Subdomain auf dem Domänennamen des Angreifers erstellt, z. B. https://website.thm.attackers-domain.thm. 
Die Anwendungslogik würde nun diese Eingabe zulassen und einem Angreifer die Kontrolle über die interne HTTP-Anfrage ermöglichen.


#### Open Redirect
Wenn die oben genannten Umgehungen nicht funktionieren, haben die Angreifer noch einen weiteren Trick in petto: die offene Weiterleitung (Open Redirect). 

Eine offene Weiterleitung ist ein Endpunkt auf dem Server, an dem der Website-Besucher automatisch zu einer anderen Website-Adresse weitergeleitet wird. 
Nehmen wir zum Beispiel den Link https://website.thm/link?url=https://tryhackme.com. Dieser Endpunkt wurde eingerichtet, um die Anzahl der Besucher aufzuzeichnen, die zu Werbe-/Marketingzwecken auf diesen Link geklickt haben. 
Aber stellen Sie sich vor, es gäbe eine potenzielle SSRF-Schwachstelle mit strengen Regeln, die nur URLs zulassen, die mit https://website.thm/ beginnen. Ein Angreifer könnte die obige Funktion nutzen, um die interne HTTP-Anfrage an eine Domäne seiner Wahl umzuleiten.
Bei einer Erlaubnisliste werden alle Anfragen abgelehnt, sofern sie nicht auf einer Liste stehen oder einem bestimmten Muster entsprechen, z. B. einer Regel, dass eine in einem Parameter verwendete URL mit https://website.thm beginnen muss. 
Ein Angreifer könnte diese Regel schnell umgehen, indem er eine Subdomain auf dem Domänennamen des Angreifers erstellt, z. B. https://website.thm.attackers-domain.thm. Die Anwendungslogik würde nun diese Eingabe zulassen und einem Angreifer die Kontrolle über die interne HTTP-Anfrage ermöglichen.