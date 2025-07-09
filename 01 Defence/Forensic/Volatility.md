Volatility ist ein Tool welches Genutzt wird um die Daten aus dem RAM Forensisch zu analysieren Dies ist wichtig da sich aus dem RAM Artefakte und Rückschlüsse zum angriff etc. finden lassen
[GitHub - volatilityfoundation/volatility3: Volatility 3.0 development](https://github.com/volatilityfoundation/volatility3)
Volatility kann leider nicht selbst den RAM dumpen sondern nur die Dumps auswerten

### Generelle Syntax

```bash
python3 vol.py -f <file> windows.info
```

Da es ein Python script im Kern ist “python3 [vol.py](http://vol.py)”
Das “-f <file>” spezifiziert den Dump der Analysiert werden soll
Das “windows.” spezifiziert aus welchen OS der Dump ist
Hingegen das “info” ist hier in dem Moment ein Platzhalter für das Plugin das zur Analyse genutzt werden soll

|Interessante Plugins||
|---|---|
|Windows.cmdline|Lists process command line arguments|
|windows.drivermodule|Determines if any loaded drivers were hidden by a rootkit|
|Windows.filescan|Scans for file objects present in a particular Windows memory image|
|Windows.getsids|Print the SIDs owning each process|
|Windows.handles|Lists process open handles|
|[Windows.info](http://windows.info/)|Show OS & kernel details of the memory sample being analyzed|
|Windows.netscan|Scans for network objects present in a particular Windows memory image|
|Widnows.netstat|Traverses network tracking structures present in a particular Windows memory image.|
|Windows.mftscan|Scans for Alternate Data Stream|
|Windows.pslist|Lists the processes present in a particular Windows memory image|
|Windows.pstree|List processes in a tree based on their parent process ID|
|windows.mftscan.MFTScan|designed to scan for Master File Table (MFT) FILE objects present in a particular Windows memory image|
|||
|Weitere Plugins|[https://github.com/volatilityfoundation/volatility/wiki/Command-Reference](https://github.com/volatilityfoundation/volatility/wiki/Command-Reference)|


### Häufig Genutzte Plugins mit Syntax:

###### Prozesse und Netzwerk Verbindungen

```bash
python3 vol.py -f <file> windows.pslist
```

Zeigt alle prozesse der doubly-linked list an (das ist Quasi der Taskmanager des RAMs) Dabei werden nicht nur die Laufenden sondern auch die Terminierten Prozesse angezeigt

```bash
python3 vol.py -f <file> windows.psscan
```

Wird genutzt da ein Angreifer seinen Prozess aus der doubly linked liste löschen kann dieser Befehl zeigt dann alle Prozesse mit der " _EPROCESS" Struktur an

```bash
python3 vol.py -f <file> windows.pstree
```

Dieser Befehl zeigt alle Befehle anhand der Parent Process ID an

```bash
python3 vol.py -f <file> windows.netstat
```

Zeigt alle Netzwerkverbindungen an Kann etwas instabil sein bei älteren Versionen von Windows hier kann dann aber Bulk Extractor genutzt werden [https://tools.kali.org/forensics/bulk-extractor](https://tools.kali.org/forensics/bulk-extractor) Das sorgt dafür das die Netzverbindungen dann extrahiert werden und fertige pcaps ausgegeben werden

```bash
python3 vol.py -f <file> windows.dlllist
```

Zeigt alle DLLs in Kombination mit den dazugehörenden Prozessen an





###### Hunting/Detection

```bash
python3 vol.py -f <file> windows.malfind
```

Das Plugin versucht alle Prozesse mit PID herauszufinden bei denen Code Injection betrieben wurde dies funktioniert da das Plugin nach RWE oder RX und Memory Mapped Files auf der disk sucht

```bash
python3 vol.py -f <file> windows.yarascan
```

Ermöglicht es den Speicher mit Yara regeln zu scannen





###### Advanced Memory Analysis

```bash
python3 vol.py -f <file> windows.ssdt
```

SSDT = System Service Descriptor Table Der windows Kernel nutzt SSDT um nach System Funktionen zu schauen Angreifer könnten diese Liste Modifizieren damit Zeiger auf andere Orte zeigen z.B. auf ein Rootkit Da in der SSDT Tabelle mehrere hundert Einträge sind lohnt es sich meist erst nach der Investigation des Initialen Befalls sich diese anzuschauen

```bash
python3 vol.py -f <file> windows.driverscan
```

das Driverscan plugin Dumpt alle auf dem System vorhandenen Treiber (Zum Zeitpunlkt der Ausführung) Auch hier wird Empfohlen bereits eine Investigation des Initialen Befalls fertig zu haben




###### Andere Nützliche Plugins zu denen ich noch keinen Eintrag habe

---

modscan

---

driverirp

---

callbacks

---

idt

---

apihooks

---

moddump

---

handles

---