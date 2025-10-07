# Ziel: 10.10.207.184

# Scanning 

## Nmap 

```
PORT     STATE SERVICE            VERSION
80/tcp   open  http               Microsoft IIS httpd 8.5
| http-methods: 
|_  Potentially risky methods: TRACE
| http-robots.txt: 6 disallowed entries 
| /Account/*.* /search /search.aspx /error404.aspx 
|_/archive /archive.aspx
|_http-server-header: Microsoft-IIS/8.5
|_http-title: hackpark | hackpark amusements

3389/tcp open  ssl/ms-wbt-server?
|_ssl-date: 2025-10-07T12:28:48+00:00; -1s from scanner time.


MAC Address: 02:C8:10:B2:62:29 (Unknown)

Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

```
## Nikto 

```
- Nikto v2.1.5
---------------------------------------------------------------------------
+ Target IP:          10.10.207.184
+ Target Hostname:    10.10.207.184
+ Target Port:        80
+ Start Time:         2025-10-07 13:29:17 (GMT1)
---------------------------------------------------------------------------
+ Server: Microsoft-IIS/8.5
+ Retrieved x-powered-by header: ASP.NET
+ The anti-clickjacking X-Frame-Options header is not present.
+ Uncommon header 'content-style-type' found, with contents: text/css
+ Uncommon header 'content-script-type' found, with contents: text/javascript
+ Server leaks inodes via ETags, header found with file /robots.txt, fields: 0x3d1dc2fc254ad51:0 
+ File/dir '/search/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ File/dir '/search.aspx' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ File/dir '/archive/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ File/dir '/archive.aspx' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ "robots.txt" contains 6 entries which should be manually viewed.
+ Server banner has changed from 'Microsoft-IIS/8.5' to 'Microsoft-HTTPAPI/2.0' which may suggest a WAF, load balancer or proxy is in place
+ Allowed HTTP Methods: GET, HEAD, OPTIONS, TRACE 
+ Uncommon header 'x-aspnetwebpages-version' found, with contents: 3.0
+ OSVDB-3092: /archive/: This might be interesting...
+ OSVDB-3092: /archives/: This might be interesting...
+ 6544 items checked: 0 error(s) and 14 item(s) reported on remote host
+ End Time:           2025-10-07 13:29:40 (GMT1) (23 seconds)
---------------------------------------------------------------------------

```

## Walking the site: 
Login Page 
`http://10.10.207.184/Account/login.aspx?ReturnURL=/admin`

-> Vermutlich login name Admin

Brute forcing via Burp Intruder und Sniper attack mode

->  Passwort found 
1qaz2wsx


### FINDINGS
Admin Portal unter 
`http://10.10.207.184/Account/login.aspx?ReturnURL=/admin`

Username: admin
Passwort: 1qaz2wsx

# Initial Access 

Gegeben von zuvor 

Admin Portal unter 
`http://10.10.207.184/Account/login.aspx?ReturnURL=/admin`

Username: admin
Passwort: 1qaz2wsx

Feststellung BlogEngine.NET Version: 3.3.6.0 

## Kontrolle auf Exploits mit searchsploit

root@ip-10-10-226-42:~# searchsploit BlogEngine.NET 3.3.6
--------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                                       |  Path
--------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
BlogEngine.NET 3.3.6 - Directory Traversal / Remote Code Execution                                                                                                   | aspx/webapps/46353.cs
BlogEngine.NET 3.3.6/3.3.7 - 'dirPath' Directory Traversal / Remote Code Execution                                                                                   | aspx/webapps/47010.py
BlogEngine.NET 3.3.6/3.3.7 - 'path' Directory Traversal                                                                                                              | aspx/webapps/47035.py
BlogEngine.NET 3.3.6/3.3.7 - 'theme Cookie' Directory Traversal / Remote Code Execution                                                                              | aspx/webapps/47011.py
BlogEngine.NET 3.3.6/3.3.7 - XML External Entity Injection                                                                                                           | aspx/webapps/47014.py


## Low Hanging Fruit -> Exploiting w Metasploit

-> Keine Resultate 


## Umschauen im Netz um mehr Infos zu den Exploits zu bekommen 

https://www.exploit-db.com/exploits/46353

CVE-2019-6714

Zuerst laden wir uns den Code runter und lesen die Kommentare hier wird die Funktionalität auch schon erklärt
```
```cs
/*
 * CVE-2019-6714
 *
 * Path traversal vulnerability leading to remote code execution.  This 
 * vulnerability affects BlogEngine.NET versions 3.3.6 and below.  This 
 * is caused by an unchecked "theme" parameter that is used to override
 * the default theme for rendering blog pages.  The vulnerable code can 
 * be seen in this file:
 * 
 * /Custom/Controls/PostList.ascx.cs
 *
 * Attack:
 *
 * First, we set the TcpClient address and port within the method below to 
 * our attack host, who has a reverse tcp listener waiting for a connection.
 * Next, we upload this file through the file manager.  In the current (3.3.6)
 * version of BlogEngine, this is done by editing a post and clicking on the 
 * icon that looks like an open file in the toolbar.  Note that this file must
 * be uploaded as PostView.ascx. Once uploaded, the file will be in the
 * /App_Data/files directory off of the document root. The admin page that
 * allows upload is:
 *
 * http://10.10.10.10/admin/app/editor/editpost.cshtml
 *
 *
 * Finally, the vulnerability is triggered by accessing the base URL for the 
 * blog with a theme override specified like so:
 *
 * http://10.10.10.10/?theme=../../App_Data/files
 *
 */
```