# Was ist das? 

Die Möglichkeit, Dateien auf einen Server hochzuladen, ist zu einem festen Bestandteil unserer Interaktion mit Webanwendungen geworden. Sei es ein Profilbild für eine Social-Media-Website, ein Bericht, der in einen Cloud-Speicher hochgeladen wird, oder das Speichern eines Projekts auf Github; die Anwendungsmöglichkeiten für Datei-Upload-Funktionen sind grenzenlos.

Leider können Datei-Uploads, wenn sie schlecht gehandhabt werden, auch schwerwiegende Sicherheitslücken auf dem Server öffnen. Dies kann von relativ kleinen, lästigen Problemen bis hin zur vollständigen Remote Code Execution (RCE) führen, wenn es einem Angreifer gelingt, eine Shell hochzuladen und auszuführen. Mit uneingeschränktem Upload-Zugriff auf einen Server (und der Möglichkeit, Daten nach Belieben abzurufen) kann ein Angreifer vorhandene Inhalte verunstalten oder anderweitig verändern - bis hin zur Einschleusung bösartiger Webseiten, die zu weiteren Schwachstellen wie XSS oder CSRF führen. Durch das Hochladen beliebiger Dateien könnte ein Angreifer den Server möglicherweise auch dazu nutzen, illegale Inhalte zu hosten und/oder zu verbreiten oder vertrauliche Informationen weiterzugeben. Realistisch betrachtet ist ein Angreifer, der eine beliebige Datei auf Ihren Server hochladen kann - ohne jegliche Einschränkungen - in der Tat sehr gefährlich.

# Generelles Vorgehen
Wir haben also einen Datei-Upload-Punkt auf einer Website. Wie können wir ihn ausnutzen?

Wie bei jeder Art von Hacking ist die Aufzählung der Schlüssel. Je mehr wir über unsere Umgebung wissen, desto mehr können wir mit ihr anfangen. Ein Blick auf den Quellcode der Seite ist gut, um zu sehen, ob irgendeine Art von clientseitiger Filterung angewendet wird. Das Scannen mit einem Verzeichnisbruteforcer wie Gobuster ist normalerweise bei Webangriffen hilfreich und kann aufdecken, wohin Dateien hochgeladen werden; Gobuster ist unter Kali nicht mehr standardmäßig installiert, kann aber mit `sudo apt install gobuster` installiert werden. Das Abfangen von Upload-Anfragen mit **Burpsuite** kann ebenfalls nützlich sein. Browser-Erweiterungen wie **Wappalyser** können auf einen Blick wertvolle Informationen über die Website liefern, die Sie ins Visier nehmen.

Mit einem grundlegenden Verständnis dafür, wie die Website unsere Eingaben behandelt, können wir dann versuchen, herauszufinden, was wir hochladen können und was nicht. Wenn die Website client-seitige Filter einsetzt, können wir uns den Code des Filters ansehen und versuchen, ihn zu umgehen (mehr dazu später!). Wenn die Website über serverseitige Filter verfügt, müssen wir möglicherweise erraten, wonach der Filter sucht, eine Datei hochladen und dann anhand der Fehlermeldung etwas anderes versuchen, wenn der Upload fehlschlägt. Das Hochladen von Dateien, die Fehler provozieren sollen, kann dabei helfen. Tools wie Burpsuite oder **OWASP Zap** können in diesem Stadium sehr hilfreich sein.


# Überschreiben von Vorhandenen Dateien 

