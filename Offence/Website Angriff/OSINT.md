-source### Interessanter Content

###### **Seiten Quelle** 
via: view-source:_URL_
kann man sich den Seiten Quell Code anzeigen lassen
Kann zum Beispiel Rückschluss u.A. auf Frameworks geben anhand welcher man sich z.B. Login Pages, Default Credentials etc. herleiten kann

###### **Robots.txt**
via: * URL */robots.txt
ist ein Dokument, das Suchmaschinen mitteilt, welche Seiten sie in ihren Suchergebnissenanzeigen dürfen und welche nicht, oder das bestimmten Suchmaschinen verbietet, die Website überhaupt zu crawlen

###### **Sitemap.xml**
via: * URL */sitemap.xml
ist das Gegenstück zu der Robots.txt und spezifiziert was ein Browser finden kann. Kann wenn nicht ausreichend gepflegt veraltete/ungepflegte Webpages zeigen

###### **Favicons**
Manchmal werden nach einer Installation einer Website mit einem Framework vergessen Custom Favicons einzurichten Dies kann dann dazu führen das man das Favicon des Frameworks angezeigt bekommt Link zu Favicon bekommt man aus der Source der Website Zum feststellen welches Framework es ist die URL des Favicons via Curl laden und dann einen md5 Hash daraus erstellen und hier abgleichen

![[image 1.png]]
https://wiki.owasp.org/index.php/OWASP_favicon_database


###### **HTTP Header**
via: curl * url * -v
Zeigt die Version des Servers und der Plugins an


###### Weitere Optionen

**Wappalyzer** 
Ist ein Online Tool welches hilft herauszufinden welche Frameworks, Content Management Systeme(CMS), Payment-processor etc. genutzt werden
[Find out what websites are built with - Wappalyzer](https://www.wappalyzer.com/)

**GitHub**
Manchmal wird es Versäumt ein Firmen Repo auf Privat zu stellen was dann externen zugriff möglich macht

**S3 Buckets**
Falsche Einstellungen in der Konfiguration können dazu führen das die Buckets von Dritten einsehbar und Bearbeitbar sind
Buckets lassen sich recht einfach über Links auffinden, da das Schema einfach gehalten ist
Eine gängige Automatisierungsmethode ist die Verwendung des Firmennamens gefolgt von üblichen Begriffen wie {name}-assets, {name}-www, {name}-public, {name}-private, usw.
Schema:
http(s)://{name}.s3.amazonaws.com
Bsp.: [tryhackme-assets.s3.amazonaws.com](http://tryhackme-assets.s3.amazonaws.com/)