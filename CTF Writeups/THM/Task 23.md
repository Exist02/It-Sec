I didnt have a clue abt Modbus to begin with.

I watched a couple YT Vids and then went to the Obvious (ChatGPT) to get more help. It Suggested me to use "mbpoll". I checked if it was available in the attackbox but no.

So after that i went around the THM Site looking for Rooms abt Modbus since it should be logical that THM has a room abt it when they use it in a CTF. 

Then I Found the room "# Attacking ICS Plant #1" where they Used "pymodbus" i went and checked if it was available in the Attack box and it was. 

After that i Went and wrote a Python Script with help of Chat GPT to Connect to the Modbus Device and Pull Registers 0-200. 

This is the Script Since Im German the Dialoug is in German aswell: 

```
from pymodbus.client.sync import ModbusTcpClient

def is_probably_ascii(word):
    return all(32 <= b <= 126 for b in word)

def scan_holding_registers_for_ascii(ip, port=502, unit_id=1, start=0, end=200, block_size=20):
    client = ModbusTcpClient(ip, port=port)

    if not client.connect():
        print(f"Verbindung zu {ip}:{port} fehlgeschlagen.")
        return

    print(f"Suche nach ASCII-Daten in Holding-Register {start} bis {end}...\n")

    try:
        for address in range(start, end, block_size):
            result = client.read_holding_registers(address, block_size, unit=unit_id)
            if result.isError():
                print(f"[{address:03d}] Fehler beim Lesen.")
                continue

            registers = result.registers
            raw_bytes = []

            for reg in registers:
                high = (reg >> 8) & 0xFF
                low = reg & 0xFF
                raw_bytes.extend([high, low])

            # Versuche in ASCII umzuwandeln, ersetze unlesbare Zeichen mit '.'
            ascii_str = ''.join(chr(b) if 32 <= b <= 126 else '.' for b in raw_bytes)

            # Zeige nur, wenn mindestens 4 lesbare Zeichen enthalten sind
            if ascii_str.count('.') < len(ascii_str) - 4:
                print(f"[{address:03d}] -> '{ascii_str}'")

    finally:
        client.close()

# Starte den Scan
scan_holding_registers_for_ascii(ip="10.10.51.226", port=502, start=0, end=200, block_size=20)

```

After Running it it literally only was a look into the output that was 

```
[000] -> '..&.!.;H..X*y=..=._...<`1..4(..T.Q..L/."'
[020] -> '..td..#.........X..`.\... ..eQL...y.l.V\'
[040] -> '=".Y..u.._.e..6.l.....N...K.3....#..l.8.'
[060] -> '..Y.....vh]...............{...Q.>.....`.'
[080] -> '..0w...f.:.1x..%...g...d.E.|......?..tg.'
[100] -> './.h...1..M:3....o...*v..!..=...9....o..'
[120] -> 'a.oNBN.K...vWpB.......K...2p9.V..z.S.I.r'
[140] -> '..@.*.V#..QR...Z........nFa......?(h....'
[160] -> '.^.[..J7..../...}.THM{m4nu4l_p0ll1ng_r3g'
[180] -> '1st3rs}..y......U..5.y....iZ....A.+v....'

```