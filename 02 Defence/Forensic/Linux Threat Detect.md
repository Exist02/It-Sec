# Detecting SSH Attacks

On Linux, you don't need to learn a dozen fields like logon type to figure out what's going on, making log analysis more straightforward. Your starting point in detecting SSH attacks can be as simple as listing all successful SSH logins and analyzing a few fields. Let's imagine you queried the logs and found three successful SSH logins, each of which could indicate an attack. How would you distinguish bad from good? Logs regardung SSH Login/Login attempts usually are storred in `/var/log/auth.log`

SuccessfulSSHLogins

```shell-session
ubuntu@thm-vm:~$ cat /var/log/auth.log | grep -E 'Accepted'
2025-08-19T14:00:02 thm-vm sshd[1013]: Accepted publickey for ansible from 10.14.105.255 port 18442 ssh2: [...]
2025-08-20T12:56:49 thm-vm sshd[2830]: Accepted password for jsmith from 54.155.224.201 port 51058 ssh2
2025-08-22T03:14:06 thm-vm sshd[2830]: Accepted password for jsmith from 196.251.118.184 port 51058 ssh2
```

**Login of Ansible**

The first login appears legitimate: It used public-key authentication from an internal IP, likely an Ansible automation account. Moreover, the login at exactly 14:00 matches periodic task behavior. But to be sure, you'd still need to verify that `10.14.105.255` is an Ansible server and review the following user's activity for signs of a breach.

**Logins of Jsmith**

The two logins of jsmith are more interesting, as there are three red flags: Password-based authentication, logins from external IPs, and time difference between the logins (one of the logins must be at night for the user, right?). Still, to make a final verdict, you might need to investigate more details:

- **Username**: Who owns the user? Is it expected for them to log in at this time and from this IP?
- **Source IP**: What do [TI tools](https://tryhackme.com/room/ipanddomainthreatintel) and [asset lookups](https://tryhackme.com/room/socworkbookslookups) say about the IP? Is it trusted or malicious?
- **Login history**: Was the login preceded by brute force or other suspicious system events?
- **Next steps**: Is the login suspicious? Should I analyze user actions following the login?

## Using Application Logs

If you want to know whether your email server was breached, you naturally reach for the email logs. On the other hand, can you expect an application to log "I am being exploited with a zero-day right now"? Of course not. That’s the nature of application logs - they rarely tell the full story, but they can still provide unique artifacts for analysis. For example, you can:

- Use **web logs** to detect a variety of web attacks
- Use **database logs** to detect suspicious SQL queries
- Use **VPN logs** to detect abnormal VPN login events
- Refer to **other logs** for specific events like bank transactions

## Web as Initial Access

Any publicly exposed application can lead to a Linux breach, especially vulnerable web servers. Let's see an example: The IT team creates a simple web application called TryPingMe, where you can ping the specified IP online. Internally, the app runs a system command `ping -c 2 **[YOUR-INPUT]**` to test the connection, without any input filtering. The attackers would easily spot a [command injection](https://tryhackme.com/room/oscommandinjection) there, but can you spot the exploitation in the TryPingMe web logs?

TryPingMe Web Logs

```shell-session
ubuntu@thm-vm:~$ cat /var/log/nginx/access.log
10.2.33.10 - - [19/Aug/2025:12:26:07] "GET /ping?host=3.109.33.76 HTTP/1.1" 200 [...]
10.12.88.67 - - [23/Aug/2025:09:32:22] "GET /ping?host=54.36.19.83 HTTP/1.1" 200 [...]
10.14.105.255 - - [26/Aug/2025:20:09:43] "GET /ping?host=hello HTTP/1.1" 500 [...]
10.14.105.255 - - [26/Aug/2025:20:09:46] "GET /ping?host=whoami HTTP/1.1" 500 [...]
10.14.105.255 - - [26/Aug/2025:20:09:49] "GET /ping?host=;whoami HTTP/1.1" 200 [...]
10.14.105.255 - - [26/Aug/2025:20:10:41] "GET /ping?host=;ls HTTP/1.1" 200 [...]
```

**Web Logs Analysis**

The requests coming from `10.14.105.255` seem odd. Instead of IPs, the client puts Linux commands inside the query parameters - a clear sign of command injection! Although now you will need to start a deep investigation to unravel the whole story, from web logs alone you can assume that:

- `10.14.105.255` is likely the attacker's IP
- The `/ping` page is vulnerable and allows code execution
- The attacker executed OS commands like `whoami` and `ls`
- The entire system is now at risk because of the TryPingMe vulnerability