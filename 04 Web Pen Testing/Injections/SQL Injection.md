### Was ist das? 
Eine SQL Injection ist wenn wine Webanwendung die SQL benutzt die vom Nutzer gelieferte Daten nicht überprüft und einfach in den Query String aufnimmt.

Beispiel: 
Nehmen wir das folgende Szenario an: Du bist auf einen Online-Blog gestoßen, und jeder Blogeintrag hat eine eindeutige ID-Nummer. Die Blogeinträge können entweder als öffentlich oder privat eingestuft sein, je nachdem, ob sie zur Veröffentlichung bereit sind. Die URL für jeden Blogeintrag könnte etwa so aussehen:

https://website.thm/blog?id=1

Aus der obigen URL ist ersichtlich, dass der ausgewählte Blogeintrag aus dem id-Parameter in der Abfragezeichenfolge stammt. Die Webanwendung muss den Artikel aus der Datenbank abrufen und kann eine SQL-Anweisung verwenden, die etwa wie folgt aussieht:
```
SELECT * from blog where id=1 and private=0 LIMIT 1;
```

Anhand des Gelernten solltest du in der Lage sein, herauszufinden, dass die obige SQL-Anweisung in der Blog-Tabelle nach einem Artikel mit der ID-Nummer 1 sucht und die private Spalte auf 0 gesetzt ist, was bedeutet, dass er von der Öffentlichkeit eingesehen werden kann und die Ergebnisse auf einen einzigen Treffer beschränkt.

### Welche Arten der SQL Injection gibts? 

In-Band SQL Injection
In-Band SQL Injection ist die am einfachsten zu entdeckende und auszunutzende Art; In-Band bezieht sich lediglich darauf, dass dieselbe Kommunikationsmethode verwendet wird, um die Schwachstelle auszunutzen und auch die Ergebnisse zu erhalten, z. B. die Entdeckung einer SQL Injection-Schwachstelle auf einer Website und die anschließende Extraktion von Daten aus der Datenbank auf derselben Seite.
Praktisches Beispiel: https://imgur.com/AhGnoQD
  

Fehlerbasierte SQL-Injektion
Diese Art der SQL-Injektion ist am nützlichsten, um auf einfache Weise Informationen über die Datenbankstruktur zu erhalten, da Fehlermeldungen der Datenbank direkt auf dem Browserbildschirm ausgegeben werden. Dies kann oft dazu verwendet werden, eine ganze Datenbank zu durchforsten.

  
Union-Based SQL Injection
Bei dieser Art der Injektion wird der SQL-Operator UNION zusammen mit einer SELECT-Anweisung verwendet, um zusätzliche Ergebnisse auf der Seite auszugeben. Diese Methode ist die häufigste Art, große Datenmengen über eine SQL-Injection-Schwachstelle zu extrahieren.****

Blind SQL Injection
Im Gegensatz zur In-Band-SQL-Injektion, bei der wir die Ergebnisse unseres Angriffs direkt auf dem Bildschirm sehen können, erhalten wir bei der blinden SQL-Injektion nur wenig bis gar keine Rückmeldung darüber, ob unsere injizierten Abfragen tatsächlich erfolgreich waren oder nicht, weil die Fehlermeldungen deaktiviert wurden, die Injektion aber trotzdem funktioniert. Es mag überraschend sein, dass wir nur diese kleine Rückmeldung brauchen, um eine ganze Datenbank erfolgreich aufzählen zu können.
Praktisches Beispiel mit Auth bypass
http://imgur.com/WSIbg8k

