# Fragen: 

How many ports are open?  

What is the version of nginx?  

What is running on the highest port?

Using GoBuster, find flag 1.  

Further enumerate the machine, what is flag 2?  

Crack the hash with easypeasy.txt, What is the flag 3?  

What is the hidden directory?  

Using the wordlist that provided to you in this task crack the hash  
what is the password?

What is the password to login to the machine via SSH?  


What is the user flag?  

What is the root flag?

# Scanning und Enumeration 

Um Fragen 1 - 3 zu Beantorten 

```
nmap -sV *ip* -p 0-65535
```

Antwort Frage 1
3
Antwort Frage 2
1.16.1
Antwort Frage 3 
65524 Apache

Kontrolle der Verbindung unter port 65524 (da Apache)

Gefunden Apache 2 Base Page auf der auch Flag 3 Versteckt ist 
Fl4g 3 : flag{9fdafbd64c47471a8f54cd3fc64cd312}

Rest Vergessen zu Notieren Schade :/