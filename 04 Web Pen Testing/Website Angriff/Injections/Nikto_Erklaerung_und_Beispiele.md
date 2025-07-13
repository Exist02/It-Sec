
# 🔍 Nikto Webserver Scanner – Erklärung mit praktischen Beispielen

**Nikto** ist ein Open-Source Webserver-Scanner, der bekannte Schwachstellen, veraltete Softwareversionen, unsichere Dateien oder Konfigurationsfehler auf Webservern aufdeckt.

---

## 🧠 Was ist Nikto?

Nikto scannt Webserver auf über 6700 potenziell gefährliche Dateien/Programme, prüft auf veraltete Server-Softwareversionen und erkennt häufige Konfigurationsprobleme.

### ✅ Eigenschaften
- Erkennt über 6700 bekannte Probleme
- Unterstützt SSL, HTTP-Auth, Proxys, Cookies etc.
- Kann benutzerdefinierte Plugins verwenden
- Unterstützt Logging, Reporting und evasion techniques

---

## 🔧 Grundsyntax

```bash
nikto -h <ZIEL>
```

📌 `-h`: Zielhost oder URL

---

## 📌 Beispiel 1: Einfache Zielanalyse

```bash
nikto -h http://example.com
```

🎯 **Ziel**: Beispiel-Webserver auf Standardprobleme prüfen.

🧪 **Was passiert**: Nikto sendet verschiedene Anfragen an den Webserver und prüft auf bekannte Schwachstellen wie:
- Offen zugängliche `admin/` oder `phpinfo.php`
- Veraltete Apache-Versionen
- Unsichere HTTP-Header

---

## 📌 Beispiel 2: HTTPS-Scan

```bash
nikto -h https://secure-site.local
```

🎯 **Ziel**: SSL-geschützter Webserver

🧪 **Was passiert**: Nikto nutzt HTTPS und überprüft unter anderem:
- SSL-Konfigurationen
- Mögliche veraltete TLS-Versionen
- Sicherheitsprobleme auf HTTPS-Endpunkten

---

## 📌 Beispiel 3: Scan auf bestimmtem Port

```bash
nikto -h 192.168.1.10 -p 8080
```

🎯 **Ziel**: Interner Webserver auf Port 8080

🧪 **Was passiert**: Nützlich z. B. bei Tomcat/Web-App-Servern. Nikto prüft:
- Default-Tomcat Seiten
- Zugriff auf `/manager/` oder `/host-manager/`
- Bekannte Exploits für Services auf ungewöhnlichen Ports

---

## 📌 Beispiel 4: Scan durch Proxy (z. B. Burp Suite)

```bash
nikto -h http://target.com -useproxy http://127.0.0.1:8080
```

🎯 **Ziel**: Analyse des Datenverkehrs mit Burp oder ZAP

🧪 **Was passiert**: Alle HTTP-Requests laufen über den Proxy. Du kannst:
- Traffic in Echtzeit analysieren
- Manuell eingreifen
- Modifikationen testen

---

## 📌 Beispiel 5: Scan-Ergebnisse in Datei speichern

```bash
nikto -h http://example.com -o scan_output.txt -Format txt
```

🎯 **Ziel**: Report speichern für spätere Analyse

🧪 **Was passiert**:
- Die Ausgabe wird in eine Textdatei geschrieben
- Auch andere Formate möglich: `html`, `csv`, `nbe`, `xml`

---

## 📌 Beispiel 6: Benutzerdefinierter User-Agent

```bash
nikto -h http://victim.local -useragent "Mozilla/5.0 (Nikto Scanner)"
```

🎯 **Ziel**: Umgehen einfacher Bot-Blocker oder Analyse der Reaktion auf verschiedene User-Agents

🧪 **Was passiert**: Der Scanner sendet Requests mit dem angegebenen User-Agent. Nützlich zum Testen von Sicherheitsregeln wie ModSecurity, WAFs oder Rate-Limiting.

---

## 📌 Beispiel 7: Evasion-Techniken verwenden

```bash
nikto -h http://target.com -evasion 1
```

🎯 **Ziel**: Umgehung einfacher IDS/WAF-Systeme

🧪 **Was passiert**:
- Nikto verändert URL-Encoding, Pfadangaben etc.
- Beispiel: `/admin/` → `/%61dmin/` (URL-Encoding von "a")

⚠️ Hinweis: Evasion-Techniken können zu Fehlalarmen oder False Positives führen.

---

## 📌 Beispiel 8: Scan auf bestimmte Plugins begrenzen

```bash
nikto -h http://target.com -Tuning 1,2
```

🎯 **Ziel**: Nur gezielt nach bestimmten Schwachstellen suchen

🧪 **Tuning-Codes**:
- 1: Datei-Check
- 2: CGI-Check
- 3: Datenleck-Checks
- ...

👁️‍🗨️ Beispiel: Nur auf anfällige CGI-Skripte und Dateien scannen (schneller & gezielter)

---

## ⚙️ Weitere nützliche Optionen

| Option | Beschreibung |
|--------|--------------|
| `-ssl` | HTTPS erzwingen |
| `-Display V` | Zeigt alle geprüften URLs |
| `-nointeractive` | Unterdrückt interaktive Fragen |
| `-Plugins` | Listet verfügbare Plugins |
| `-list-plugins` | Zeigt alle installierten Plugins an |

---

## 🔐 Typische Funde mit Nikto

| Fund | Bedeutung |
|------|-----------|
| `/phpinfo.php` gefunden | Könnte sensible Serverinfos preisgeben |
| Apache 2.4.6 | Veraltete Version → potenzielle Exploits |
| Directory Listing aktiviert | Angreifer können Datei- und Ordnerstruktur sehen |
| TRACE-Method erlaubt | Kann XST-Angriffe ermöglichen |

---

## ⚠️ Wichtige Hinweise

- Nikto ist **kein leiser Scanner** – er wird definitiv im Log auffallen!
- Verwende es nur auf Systemen, für die du **eine Genehmigung** hast!
- Es ersetzt **keine vollständige Sicherheitsanalyse**, sondern ergänzt sie.

---

## 📚 Weitere Ressourcen

- Offizielle Seite: [https://cirt.net/Nikto2](https://cirt.net/Nikto2)
- GitHub Repo: [https://github.com/sullo/nikto](https://github.com/sullo/nikto)
- Cheat Sheet: [https://highon.coffee/blog/nikto-cheat-sheet/](https://highon.coffee/blog/nikto-cheat-sheet/)

---

## ✅ Fazit

Nikto ist ein effektives und einfaches Werkzeug für schnelle Webserver-Checks. Mit gezieltem Einsatz und klarer Zieldefinition lässt sich damit zuverlässig eine erste Einschätzung der Angriffsfläche eines Webservers gewinnen.
