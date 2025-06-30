# Basics

Was das und was kann das?

Modbus gehört zur OP das steht für Operational Technology. Unter OT versteht man technik die die zur Überwachung und Steuerung industrieller Prozesse eingesetzt werden. Industrielle Kontrollsysteme (ICS) umfassen Systeme zur Überwachung und Steuerung industrieller Prozesse.

Während Normale Netzte von IT Atmins abgesichert werden ist es bei den OT Operatoren etwas anders. Da hier der Fokus weniger auf Sicherheit liegt sondern mehr auf Verfügbarkeit und Geschwindigkeit gesetzt ist. In anderen Worten OT/ICS Software ist Designed dafür schnell und zuverlässig zu sein und ist dafür im Tausch meistens sehr unsicher und Angreifbar.

###### **SCADA Systeme:**
SCADA-Systeme (Supervisory Control and Data Acquisition) werden zur Steuerung und Automatisierung von Industrieprozessen eingesetzt. SCADA-Systeme umfassen:
- **Überwachungscomputer**: die Server, die zur Verwaltung des Prozesses verwendet werden, indem sie Daten über den Prozess sammeln und mit den hinterlegten Geräten (PLC/RTU) kommunizieren. Bei kleineren Installationen ist die Mensch-Maschine-Schnittstelle in einen einzelnen Computer eingebettet, bei größeren Installationen ist die Mensch-Maschine-Schnittstelle in einem eigenen Computer untergebracht.
- Speicherprogrammierbare Steuerungen (SPS): digitale Computer, die hauptsächlich zur Automatisierung von Industrieprozessen eingesetzt werden. Sie werden zur kontinuierlichen Überwachung von Sensoren (Input) und zur Entscheidungsfindung bei der Steuerung von Geräten (Output) eingesetzt.
- Remote Terminal Units (RTU): Heutzutage überschneiden sich die Funktionen von RTUs und PLCs. RTUs werden in der Regel für die geografische Telemetrie bevorzugt, während PLCs besser für die lokale Steuerung geeignet sind.
- Kommunikationsnetz: das Netz, das alle SCADA-Komponenten miteinander verbindet (Ethernet, seriell, Telefon, Funk, Mobilfunk...). Netzwerkausfälle wirken sich nicht unbedingt negativ auf den Anlagenprozess aus.  Sowohl RTUs als auch PLCs sollten so ausgelegt sein, dass sie autonom arbeiten und die letzte Anweisung des Überwachungssystems verwenden.
- Mensch-Maschine-Schnittstelle (HMI): zeigt eine digitalisierte Darstellung der Anlage an. Die Bediener können mit der Anlage interagieren und Befehle über Maus, Tastatur oder Touchscreen erteilen. Die Bediener können Überwachungsentscheidungen treffen und das normale Verhalten der Anlage anpassen oder außer Kraft setzen.

## TLDR:
In kurzen und einfachen Worten:
Die Industrie wird von hochentwickelten, unternehmenskritischen Computern (SCADA-Systemen) gesteuert;
Sicherheit steht bei OT/ICS nicht an erster Stelle;
Bediener können das Verhalten der Anlage manuell über Maus/Tastatur/Touchscreen, lokal oder aus der Ferne, beeinflussen;
eine bösartige Software kann das Verhalten der Anlage wie bei der Mensch-Maschine-Schnittstelle (HMI) beeinflussen.
https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-82r2.pdf

# Modbus

Modbus ist ein von Modicon entwickeltes Industrieprotokoll, das von Schneider Electric übernommen wurde. Modbus ist weit verbreitet, um industrielle Geräte zu verbinden; die Protokollspezifikation ist verfügbar und lizenzfrei.
Das Modbus-Protokoll ist ein Master/Slave-Protokoll: Der Master liest und schreibt die Register der Slaves. Modbus RTU wird in der Regel über RS-485 (serielles Netzwerk) verwendet: ein Master ist mit einem oder mehreren Slaves vorhanden. Jeder Slave hat eine eindeutige 8-Bit-Adresse.

Modbus-Daten werden zum Lesen und Schreiben von „Registern“ verwendet, die 16 Bit lang sind. Das gebräuchlichste Register ist das „“Holding-Register„“, das gelesen und geschrieben werden kann; das Register „Input-Register“ ist nur lesbar. Die Register „Spule“ und „diskreter Eingang“ sind 1 Bit lang: Spulen sind lesbar und beschreibbar, diskrete Eingänge sind nur lesbar.

Modbus-Registertypen:
- Diskreter Eingang (Statuseingang): 1bit, RO
- Spule (Diskreter Ausgang): 1Bit, R/W
- Eingangsregister: 16bit, RO
- Halteregister: 16bit R/W

Most commonly Modbus function:

| Function Code | Register Type                    |
| ------------- | -------------------------------- |
| 1             | Read Coil                        |
| 2             | Read Discrete Input              |
| 3             | Read Holding Registers           |
| 4             | Read Input Registers             |
| 5             | Write Single Coil                |
| 6             | Write Single Holding Register    |
| 15            | Write Multiple Coils             |
| 16            | Write Multiple Holding Registers |
Modbus TCP kapselt Modbus RTU-Anfrage- und Antwortdatenpakete in ein TCP-Paket, das über Standard-Ethernet-Netzwerke übertragen wird. Der TCP-Modbus-Standardanschluss ist 502.
Installation von pymodbus um mit Modbus richtig interagieren zu können:

```
pip3 install pymodbus==1.5.2
```

Generell weitere Infos zu modbus gibt es hier: 
https://www.csimn.com/CSI_pages/Modbus101.html

Gutes Script um zu schauen was aktuell passiert in der Regestry: 
```
#!/usr/bin/env python3

import sys
import time
from pymodbus.client.sync import ModbusTcpClient as ModbusClient
from pymodbus.exceptions import ConnectionException

client = ModbusClient("IP Adresse Einfügen", port=502)
client.connect()
while True:
    rr = client.read_holding_registers(1, 16)
    print(rr.registers)
    time.sleep(1)

```