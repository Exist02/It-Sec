Wenn man über die Interaktion mit Webanwendungen nachdenkt, sollte man sich bewusst machen, dass man nicht bei jeder Anfrage seinen Benutzernamen und sein Passwort an die Webanwendung übermittelt. Stattdessen erhält man nach der Authentifizierung eine Session. Diese Session wird von der Webanwendung verwendet, um den Status zu speichern, die Aktionen zu verfolgen und zu entscheiden, ob man die gewünschte Aktion ausführen darf oder nicht. Das Session-Management soll sicherstellen, dass diese Schritte korrekt ausgeführt werden. Andernfalls könnte ein Angreifer die Session kompromittieren und effektiv kapern!

# Session Management Lifecycle

https://imgur.com/2Jv45I6

Erklärung der schritte aus dem Screenshot: 
## Session Creation: 
Man könnte meinen, dass dieser erste Schritt im Lebenszyklus einer Session erst erfolgt, nachdem man seine Zugangsdaten wie Benutzername und Passwort eingegeben hat. Bei vielen Webanwendungen wird die initiale Session jedoch bereits beim Aufruf der Anwendung erstellt. Der Grund dafür ist, dass manche Anwendungen die Aktionen des Benutzers bereits vor der Authentifizierung verfolgen möchten. Sobald man seinen Benutzernamen und sein Passwort eingegeben hat, erhält man einen Session-Wert, der dann mit jeder neuen Anfrage gesendet wird. Wie diese Session-Werte generiert, verwendet und gespeichert werden, ist für die Sicherheit der Session-Erstellung von entscheidender Bedeutung.

## Session Tracking

Sobald du deinen Session-Wert erhalten hast, wird dieser mit jeder neuen Anfrage übermittelt. So kann die Webanwendung deine Aktionen verfolgen, obwohl das HTTP-Protokoll von Natur aus zustandslos ist. Bei jeder Anfrage kann die Webanwendung den Session-Wert aus der Anfrage abrufen und eine serverseitige Suche durchführen, um zu verstehen, wem die Session gehört und welche Berechtigungen sie hat. Falls es Probleme beim Verfolgen der Session gibt, kann das dazu führen, dass jemand eine Session kapert oder sich als jemand anderes ausgibt.

## Session Expiry

Da das HTTP-Protokoll zustandslos ist, kann es vorkommen, dass ein Benutzer der Webanwendung plötzlich aufhört, sie zu verwenden. Beispielsweise könnten er den Tab oder seinen gesamten Browser schließen. Da das Protokoll zustandslos ist, hat die Webanwendung keine Möglichkeit zu erkennen, dass diese Aktion stattgefunden hat. Hier kommt das Ablaufen der Sitzung ins Spiel. Der Session-Wert selbst sollte mit einer Lebensdauer versehen sein. Wenn die Lebensdauer abgelaufen ist und man einen alten Session-Wert an die Webanwendung übermittelt, sollte dieser abgelehnt werden, da die Session abgelaufen sein sollte. Stattdessen sollte man zur Anmeldeseite weitergeleitet werden, um sich erneut zu authentifizieren und den Lebenszyklus der Session-Verwaltung von vorne zu beginnen!

## Session Termination

In einigen Fällen kann es jedoch vorkommen, dass der Benutzer eine Abmeldung erzwingt. In diesem Fall sollte die Webanwendung die Sitzung des Benutzers beenden. Dies ähnelt zwar dem Ablauf einer Sitzung, ist jedoch insofern einzigartig, als dass die Sitzung selbst beendet werden sollte, auch wenn ihre Lebensdauer noch gültig ist. Probleme bei diesem Beendigungsprozess könnten es einem Angreifer ermöglichen, dauerhaften Zugriff auf ein Konto zu erlangen.

# Das IAAA Modell

https://imgur.com/WCOHJXR

Das IAAA Modell gliedert sich auf in Identifikation, Authentifizierung, Autorisierung und Accountability.

## Identification

Identifikation ist der Prozess der Überprüfung der Identität eines Benutzers. Dieser beginnt damit, dass der Benutzer angibt, eine bestimmte Identität zu haben. In den meisten Webanwendungen erfolgt dies durch die Eingabe Ihres Benutzernamens. Dadurch geben der Benutzer an, dass er die Person ist, die mit diesem bestimmten Benutzernamen verbunden ist. Einige Anwendungen verwenden eindeutig erstellte Benutzernamen, während andere Ihre E-Mail-Adresse als Benutzernamen verwenden.

## Authentifikation

Die Authentifizierung ist der Prozess, mit dem sichergestellt wird, dass der Benutzer tatsächlich die Person ist, für die er sich ausgibt. Bei der Identifizierung gibt man einen Benutzernamen an, bei der Authentifizierung hingegen einen Nachweis dafür, dass man tatsächlich die Person ist, für die man sich ausgibt. Beispielsweise kann man das Passwort angeben, das mit dem angegebenen Benutzernamen verknüpft ist. Die Webanwendung kann diese Informationen auf ihre Gültigkeit überprüfen. An diesem Punkt würde die Erstellung einer Session beginnen.

## Authorisierung

Die Autorisierung ist der Prozess, mit dem sichergestellt wird, dass der jeweilige Benutzer über die erforderlichen Rechte verfügt, um die angeforderte Aktion auszuführen. Beispielsweise können zwar alle Benutzer Daten anzeigen, aber nur einige wenige dürfen sie ändern. Im Lebenszyklus der Sitzungsverwaltung spielt die Sitzungsverfolgung eine entscheidende Rolle bei der Autorisierung.

## Accountability

Verantwortlichkeit ist der Prozess der Erstellung einer Aufzeichnung der von Benutzern durchgeführten Aktionen. Wir sollten die Sitzung des Benutzers verfolgen und alle Aktionen protokollieren, die unter Verwendung dieser bestimmten Sitzung durchgeführt wurden. Diese Informationen spielen im Falle eines Sicherheitsvorfalls eine entscheidende Rolle, um die Geschehnisse zusammenzufügen.

## IAAA und Sitzungsmanagement

Nachdem wirnun die Unterschiede zwischen Authentifizierung und Autorisierung verstanden haben, kommen wir zurück zum Session-Management. Die Authentifizierung spielt eine Rolle dabei, wie Sessions erstellt werden. Die Autorisierung ist wichtig, um zu überprüfen, ob der mit einer bestimmten Session verbundene Benutzer die Berechtigung hat, die von ihm angeforderte Aktion auszuführen. Die Nachvollziehbarkeit ist für uns entscheidend, um zusammenzusetzen, was bei einem Vorfall tatsächlich passiert ist. Das bedeutet, dass es wichtig ist, dass Anfragen protokolliert werden und dass die mit jeder Anfrage verbundene Session ebenfalls protokolliert wird.

# Cookies vs Tokens
Das letzte Thema, das vor dem Einstieg in die Sicherheit des Session-Managements behandelt werden soll, ist die Art der verwendeten Sessions. Die beiden wichtigsten Ansätze sind Cookies und Tokens, die jeweils ihre eigenen Vor- und Nachteile haben.

## Cookie Based 
Die Cookie-basierte Sitzungsverwaltung wird oft als die altmodische Art der Sitzungsverwaltung bezeichnet. Sobald die Webanwendung mit der Nachverfolgung beginnen möchte, wird in einer Antwort der Set-Cookie-Header-Wert gesendet. Der Browser interpretiert diesen Header und speichert einen neuen Cookie-Wert. Sehen wir uns einen solchen Set-Cookie-Header einmal genauer an:

`Set-Cookie: session=12345;`

Ihr Browser erstellt einen Cookie-Eintrag für ein Cookie namens „session“ mit dem Wert 12345, das für die Domain gültig ist, von der das Cookie empfangen wurde. Diesem Header können auch mehrere Attribute hinzugefügt werden. Wenn Sie mehr über alle Attribute erfahren möchten, die kann man hier nachlesen (https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Set-Cookie). Einige der wichtigsten Attribute sind:

- Sicher – Weist den Browser darauf hin, dass das Cookie nur über verifizierte HTTPS-Kanäle übertragen werden darf. Bei Zertifikatsfehlern oder Verwendung von HTTP wird der Cookie-Wert nicht übertragen.
- HTTPOnly – Weist den Browser an, dass der Cookie-Wert nicht von clientseitigem JavaScript gelesen werden darf. Expire – Weist den Browser an, wann ein Cookie-Wert nicht mehr gültig ist und entfernt werden sollte.
- Expire – Gibt dem Browser an, wann ein Cookie-Wert nicht mehr gültig ist und entfernt werden sollte.
- SameSite – Gibt dem Browser an, ob das Cookie in Cross-Site-Anfragen übertragen werden darf, um vor CSRF-Angriffen zu schützen.

Bei der Cookie-basierten Authentifizierung ist zu beachten, dass der Browser selbst entscheidet, wann ein bestimmter Cookie-Wert mit einer Anfrage gesendet wird. Nach Überprüfung der Domain und der Attribute des Cookies trifft der Browser diese Entscheidung, und das Cookie wird automatisch ohne zusätzlichen JavaScript-Code auf der Client-Seite angehängt.

## Token Based 

Die tokenbasierte Sitzungsverwaltung ist ein relativ neues Konzept. Anstelle der automatischen Cookie-Verwaltungsfunktionen des Browsers wird hierfür clientseitiger Code verwendet. Nach der Authentifizierung stellt die Webanwendung ein Token im Request-Body bereit. Mithilfe von clientseitigem JavaScript-Code wird dieser Token dann im LocalStorage des Browsers gespeichert.

Wenn eine neue Anfrage gestellt wird, muss der JavaScript-Code den Token aus dem Speicher laden und als Header anhängen. Eine der gängigsten Arten von Tokens sind JSON Web Tokens (JWT), die über den Header „`Authorization: Bearer`“ übergeben werden. Da wir jedoch nicht die integrierten Cookie-Verwaltungsfunktionen des Browsers verwenden, herrscht hier ein wenig Wildwest-Stimmung, in der alles möglich ist. Es gibt zwar Standards, aber nichts zwingt wirklich dazu, sich an diese Standards zu halten.

## Vor und Nachteile der beiden 

| **Cookie-Session Management**                                                                                                                         | **Token-Based Session Management**                                                                                                                                        |
| ----------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Cookie is automatically sent by the browser with each request                                                                                         | Token has to be submitted as a header with each request using client-side JavaScript                                                                                      |
| Cookie attributes can be used to enhance the browser's protection of the cookie                                                                       | Tokens do not have automatic security protections enforced and should, therefore, be safeguarded against disclosures                                                      |
| Cookies can be vulnerable to conventional client-side attacks such as CSRF, where the browser is tricked into making a request on behalf of the user. | As the token is not automatically added to any request and cannot be read from LocalStorage by other domains, conventional client-side attacks such as CSRF are blocked.  |
| As cookies are locked to a specific domain, it can be difficult to use them securely in decentralised web applications.                               | Tokens work well in decentralised web applications, as they are managed through JavaScript and can often contain all the information required to verify the token itself. |

# Sichern des Session Lifecycles

## Session Creation
Bei der Erstellung von Sitzungen können die meisten Schwachstellen auftreten. Sehen wir uns einige der häufigsten an.

---
### Weak Session Values
In der heutigen Zeit, in der Frameworks konsequent eingesetzt werden, sind schwache Sitzungswerte seltener anzutreffen. Mit dem Aufkommen von LLMs und anderen KI-Code-Assistenten-Lösungen würde es Sie jedoch überraschen, wie oft diese altbekannten Schwachstellen wieder auftauchen.
Wenn ein benutzerdefinierter Mechanismus zur Session-Erstellung implementiert wurde, besteht eine hohe Wahrscheinlichkeit, dass die Session-Werte erraten werden können. Ein gutes Beispiel hierfür ist ein Mechanismus, der den Benutzernamen einfach als Session-Wert in Base64 codiert. Wenn ein Angreifer den Prozess der Session-Erstellung rückentwickeln kann, kann er Session-Werte generieren oder erraten, um die Konten legitimer Benutzer zu kapern.

### Kontrollierbare Session Values
In bestimmten Tokens, wie beispielsweise JWTs, sind alle relevanten Informationen sowohl zur Erstellung als auch zur Überprüfung der Gültigkeit des JWT enthalten. Wenn keine Sicherheitsmaßnahmen ergriffen werden, wie beispielsweise die Überprüfung der Signatur des Tokens oder die Sicherstellung, dass die Signatur selbst sicher erstellt wurde, könnte ein Angreifer sein eigenes Token generieren. Diese Art von Angriffen wird in einem späteren Kapitel näher erläutert.

### Session Fixation

Erinnern wir uns an die Webanwendung, die bereits vor der Authentifizierung eine Session vergeben hat? Diese Webanwendungen können für etwas anfällig sein, das als Session Fixation bezeichnet wird. Wenn Ihr Session-Wert nach der Authentifizierung nicht ausreichend rotiert wird, könnte ein entsprechend positionierter Angreifer ihn aufzeichnen, während der Benutzer noch nicht authentifiziert ist, und auf die Authentifizierung warten, um Zugriff auf die Session zu erhalten.

### Insecure Session Transmission
In modernen Umgebungen ist es üblich, dass der Authentifizierungsserver und die Anwendungsserver voneinander getrennt sind. Denken wir beispielsweise an Single-Sign-On-Lösungen (SSO). Eine Anwendung wird für die Authentifizierung bei mehreren anderen Webanwendungen verwendet. Damit dieser Prozess funktioniert, müssen die Sitzungsinformationen über den Browser vom Authentifizierungsserver an den Anwendungsserver übertragen werden. Bei dieser Übertragung können jedoch bestimmte Probleme auftreten, durch die Session einem Angreifer zugänglich wird. Am häufigsten kommt es zu einer unsicheren Weiterleitung, bei der der Angreifer die URL kontrollieren kann, zu der man nach der Authentifizierung weitergeleitet wird. Dadurch könnte der Angreifer die Session kapern. Dies betrifft nicht nur benutzerdefinierte Implementierungen, auch die SSO-Lösung von Oracle wies einen massiven Fehler auf, der dies ermöglichte. (https://krbtgt.pw/oracle-oam-10g-session-hijacking/)

## Session Tracking
Das Session-Tracking ist der zweitgrößte Verursacher von Sicherheitslücken. Schauen wir uns das einmal genauer an.

--- 
### Authorisation Bypass

Ein Autorisierungs-Bypass tritt auf, wenn nicht ausreichend überprüft wird, ob ein Benutzer die von ihm angeforderte Aktion ausführen darf. Im Wesentlichen bedeutet dies, dass die Session des Benutzers und die damit verbundenen Rechte nicht korrekt nachverfolgt werden. Es lohnt sich auch, über die beiden Arten von Autorisierungs-Bypasses zu sprechen:
- Vertical Bypass: Man kann eine Aktion ausführen, die einem Benutzer mit höheren Rechten vorbehalten ist.
- Horizontal Bypass: Man kann eine Aktion ausführen, zu der man berechtigt ist, jedoch auf einem Datensatz, für den man nicht berechtigt sein sollte, diese Aktion auszuführen.
In den meisten Anwendungen lassen sich vertikale Umgehungen leicht abwehren, da Funktions-Dekoratoren und pfadbasierte Zugriffskontrollkonfigurationen verwendet werden. Bei horizontalen Umgehungen führt der Benutzer jedoch eine Aktion aus, zu der er berechtigt sein sollte. Das Problem besteht darin, dass er diese Aktion mit den Daten einer anderen Person durchführt. Um dies zu beheben, muss der tatsächliche Code überprüfen, wer der Benutzer ist (aus seiner Session extrahiert), welche Daten er anfordert und ob er berechtigt ist, den Datensatz anzufordern oder zu ändern.

### Insufficient Logging 

Ein zentrales Problem bei Vorfällen ist, dass nicht genügend Informationen vorhanden sind, um einen Angriff nachzuvollziehen. Während auf Infrastruktur-Ebene viele Protokolle erstellt werden, kann die Protokollierung von Anwendungen entscheidend sein, um zu verstehen, was schiefgelaufen ist. Wenn die von einer bestimmten Session ausgeführten Aktionen und die Möglichkeit, diese Session einem Benutzer zuzuordnen, nicht vorhanden sind, kann dies zu Lücken in der Untersuchung führen, die nicht geschlossen werden können. Es sollte auch sichergestellt werden, dass die Protokolle sowohl akzeptierte als auch abgelehnte Aktionen abdecken. Im Falle eines Session-Hijacking-Angriffs würden die Aktionen als legitim erscheinen. Daher reicht es nicht aus, nur abgelehnte Aktionen zu protokollieren, um sich ein vollständiges Bild zu verschaffen.

## Session Expiry

Das Ablaufen einer Sitzung hat nur eine einzige Schwachstelle, nämlich wenn die Ablaufzeit für Sitzungen zu lang ist. Eine Sitzung sollte wie eine Kinokarte betrachtet werden. Jeden Abend wird derselbe Film gezeigt, aber wir möchten nicht, dass jemand dieselbe Karte verwenden kann, um den Film erneut anzusehen. Das Gleiche gilt für Sitzungen: Wir müssen sicherstellen, dass die Ablaufzeit unserer Sitzung den Anwendungsfall unserer spezifischen Anwendung berücksichtigt. Eine Bankanwendung sollte eine kürzere Session haben als Ihren Webmail-Client.

Darüber hinaus sollte bei langlebigen Sessions, wie beispielsweise bei einem Webmail-Client, die Session selbst den Ort bestätigen, an dem sie genutzt wird. Wenn sich dieser Ort ändert (was ein Hinweis auf eine Session-Übernahme sein könnte), sollte die Session beendet werden.

## Session Termination

Bei der Beendigung von Sitzungen ist das Hauptproblem, dass Sitzungen auf Serverseite nicht ordnungsgemäß beendet werden, wenn die Abmeldeaktion durchgeführt wird. Angenommen, ein Angreifer würde die Sitzung eines Benutzers kapern. Selbst wenn der Benutzer sich des Problems bewusst würde, gäbe es ohne die Möglichkeit, die Sitzung auf Serverseite zu ungültig zu machen, keine Methode für den Benutzer, den Zugriff des Angreifers zu unterbinden. Dies kann jedoch ein großes Problem für Tokens darstellen, deren Lebensdauer in das Token selbst eingebettet ist. In diesen Fällen kann das Token zu einer Sperrliste hinzugefügt werden, um überprüft zu werden. Einige Anwendungen gehen noch einen Schritt weiter und ermöglichen es, alle Sessions des Benutzers anzuzeigen und zu beenden. Darüber hinaus wird empfohlen, nach einer erfolgreichen Passwortzurücksetzung alle Sessions zu beenden, damit der Benutzer wieder die volle Kontrolle über sein Konto erhält.

# Exploiting Insecure Session Management

Gegebenheit: 
- Wir haben eine Schulwebsite mit Login und Register Flächen für Schüler und Lehrer

Zuerst fangen wir an und schauen uns an was für User wir erstellen können. Hier bleibt lediglich der Student User da der Lecturer User einen Verification Code benötigt. Entsprechend Legen wir uns einen Student account an und melden uns auf diesem an. 

Wenn wir nach dem anmelden via F12 in die Dev Tools > Network reinschauen können wir mehrere Sachen in dem Post zum Anmelden erkennen. 
1. Es werden Cookies zur Session verwaltung genutzt. 
2. die HTTP-Only Flag ist gesetzt, d.H. wir können kein Lokales Javascript nutzen um den Cookie zu lesen
3. Der Ablauf zeitraum des Cookies ist Komisch da sowohl die Creation und der Ablauf auf dem Selben Zeitraum gesetzt ist. 

Wenn wir uns den Netzwerk Traffik genauer anschauen erkennen wir auch das dass Session Tracking via Cookie so gemacht wird das er bei jeder Anfrage abgefragt wird. Wenn wir den Cookie Jetzt entfernen und die Seite neu laden stellen wir Fest das die Session allerdings weiterhin besteht. Was uns auf einen persistenten Cookie schließen lässt. 

Da es ein Persistenter Cookie ist lohnt es sich Potenziell sich den Cookie direkt anzuschauen via F12 > Storrage > Cookies. Dies würde weitere Untersuchungen erfordern, um den Lebenszyklus der Session-Verwaltung vollständig abzubilden. Wir werden die Dinge jedoch etwas einfacher halten. Werfen wir einen Blick auf die Abmeldefunktion. Sobald wir auf die Schaltfläche „Abmelden” klicken, sehen wir, dass unsere Session auf der Client-Seite entfernt wird. Jetzt gilt es zu Testen ob die Session auch serverseitig terminiert wurde. Man kann sich erneut bei der Anwendung authentifizieren und das neue Cookie durch den alten Cookie-Wert ersetzen. Es scheint jedoch, als würde die Beendigung der Session bis zu einem gewissen Grad funktionieren, da man beim erneuten Aufruf einen 500 Internal Server Error erhält.

Also gehen wir nochmal einen schritt zurück und schauen uns die Cookies an, diesmal aber unter F12 > Storrage > Local Storrage

https://imgur.com/Aq65QXu

Und schon haben wir was sehr intressantes gefunden. Die user Zeile gibt an wer sich authentifiziert hat und due userRole ob es ein Student oder ein Lehrer ist. Wenn wir hier etwas herumspielen können wir unser userkonto von Student auf lehrer Wechseln und erhalten so weiteren einblick und mehr rechte auf der Seite.


Nun haben wir den Lebenszyklus des Session-Managements dargestellt. Sehen wir uns das einmal an:

| **Lifecycle Phase** | **Observation**                                                                                  |
| ------------------- | ------------------------------------------------------------------------------------------------ |
| Session Creation    | A combination of cookie- and token-based session management is being used                        |
| Session Creation    | A new session value is provided for each login attempt                                           |
| Session Creation    | The session value itself appears to be sufficiently random                                       |
| Session Tracking    | Unauthenticated actions are not tracked via a session                                            |
| Session Tracking    | The cookie is sent in each request for tracking                                                  |
| Session Tracking    | Several requests can be made without a cookie                                                    |
| Session Tracking    | The initial token values that are loaded post-authentication dictate the visible information     |
| Session Expiry      | There is a mismatch between the client-side and server-side expiry times                         |
| Session Expiry      | The expiry time is incredibly long                                                               |
| Session Termination | Users can forcibly terminate their session                                                       |
| Session Termination | Sessions are terminated server-side but reusing an old session leads to an internal server error |
Angesichts all dieser Informationen würden wir bereits die folgenden Schwachstellen in einer Sicherheitsbewertung melden:

- Übermäßige Sitzungsdauer

Außerdem müssten wir Folgendes weiter untersuchen:

- Zugriffskontrollen für alle API-Endpunkte

- Webanwendungsprotokolle zur Nachvollziehbarkeit von Aktionen