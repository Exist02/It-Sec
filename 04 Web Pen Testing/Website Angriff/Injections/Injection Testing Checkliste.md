
## 1. Reconnaissance
- [ ] **Port-Scan & Directory-Fuzzing**: z. B. `nmap`, `gobuster`, `dirsearch`
- [ ] **Source-Analyse**: Suche nach Kommentaren, `.log`, `composer.json`, `vendor/`, `mail.log`, `phpmyadmin`, etc. :contentReference[oaicite:1]{index=1}

## 2. Information Disclosure
- [ ] Überprüfung sichtbarer Dateien (`mail.log` oder `.txt`), um mögliche credentials herauszufinden :contentReference[oaicite:2]{index=2}

## 3. SQL Injection (Login-Formulare)
- [ ] **Front-End-Filter prüfen** (z. B. JavaScript blockiert `select`, `'`, `"`) :contentReference[oaicite:3]{index=3}
- [ ] **Filtern vermeiden**: Groß-/Kleinschreibung ändern, alternative Operatoren (`||`, `&&` statt `OR`, `AND`) :contentReference[oaicite:4]{index=4}
- [ ] **Einfaches Blind-Testing**:

```
  ```sql
  ' || 1=1–  
  ' || 1=1-- 
  ' || 1= sleep(5)--  
```

– auf erfolgreiche Login-Bypass testen [YouTube+2Tech By Lorenzo+2Medium+2](https://techbylorenzo.com/tryhackme-injectics-walkthrough-writeup/?utm_source=chatgpt.com)

- [ ] **Tools einsetzen**: Burp Repeater/Intruder oder `sqlmap`, insbesondere für time-/boolean-based Blind-SQLi [Medium+2aviskase+2Tech By Lorenzo+2](https://www.aviskase.com/articles/2025/02/13/writeup-tryhackme-injectics/?utm_source=chatgpt.com)