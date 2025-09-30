Samba ist die Standard-Suite von Programmen für die Interoperabilität zwischen Windows und Linux bzw. Unix. Sie ermöglicht Endbenutzern den Zugriff auf und die Nutzung von Dateien, Druckern und anderen gemeinsam genutzten Ressourcen im Intranet oder Internet eines Unternehmens. Sie wird oft als Netzwerkdateisystem bezeichnet.
Samba basiert auf dem gängigen Client/Server-Protokoll Server Message Block (SMB). SMB wurde ausschließlich für Windows entwickelt. Ohne Samba wären andere Computerplattformen von Windows-Rechnern isoliert, selbst wenn sie Teil desselben Netzwerks wären.

# Enumeration


Mit nmap können wir einen Rechner nach SMB-Freigaben durchsuchen.
Nmap kann eine Vielzahl von Netzwerkaufgaben automatisieren. Es gibt ein Skript zum Durchsuchen von Freigaben!

```
nmap -p 445 --script=smb-enum-shares.nse,smb-enum-users.nse 10.10.86.148
```

Noch besser klappt das aber mit Enum4Linux via

```
enum4linux -a 10.10.31.60
```

Sollte man einen Offenen Share finden lohnt es sich mit diesem zu Verbinden

```
smbclient //10.10.31.60/anonymous
```