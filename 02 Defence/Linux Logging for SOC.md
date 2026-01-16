## Working With Logs

Unlike in Windows, Linux logs most events into plain text files. This means you can read the logs via any text editor without the need for specialized tools like Event Viewer. On the other hand, default Linux logs are less structured as there are no event codes and strict log formatting rules. Most Linux logs are located in the `/var/log` folder, so let's start the journey by checking the `/var/log/syslog` file - an aggregated stream of various system events.

**Filtering Logs**

You will see thousands of events when reading the syslog file on the attached VM, but only a few are useful for SOC. That's why you must filter logs and narrow down your search as much as possible. For example, you can use the "grep" command to filter for the "CRON" keyword and see only the cronjob logs:

Bsp:
```
# Or "grep -v CRON" to exclude "CRON" from results 

root@thm-vm:~$ cat /var/log/syslog | grep CRON 
2025-08-13T14:17:01.025846+00:00 thm-vm CRON[1042]: (root) CMD (cd / && run-parts --report /etc/cron.hourly) 
2025-08-13T14:25:01.043238+00:00 thm-vm CRON[1046]: (root) CMD (command -v debian-sa1 > /dev/null && debian-sa1 1 1) 
2025-08-13T14:30:01.014532+00:00 thm-vm CRON[1048]: (root) CMD (date > mycrondebug.log)
```

**Discovering Logs**

Lastly, let's say you hunt for all user logins, but don't know where to look for them. Linux system logs are stored in the `/var/log/` folder in plain text, so you can simply grep for related keywords like "login", "auth", or "session" in all log files there and narrow down your next searches:


## Authentication Logs

The first and often the most useful log file you want to monitor is `/var/log/auth.log` (or `/var/log/secure` on RHEL-based systems). Although its name suggests it contains authentication events, it can also store user management events, launched sudo commands, and much more! 

**Login and Logout Events**

Local and Remote Logins
```bash
root@thm-vm:~$ cat /var/log/auth.log | grep -E 'session opened|session closed'
```

Cron and Sudo Logins
```bash
root@thm-vm:~$ cat /var/log/auth.log | grep -E 'session opened|session closed'
```

SSH-Specific Events
```bash
root@thm-vm:~$ cat /var/log/auth.log | grep "sshd" | grep -E 'Accepted|Failed'
```

**Miscellaneous Events

You can also use the same log file to detect user management events. This is easy if you know basic Linux commands: If [useradd](https://www.man7.org/linux/man-pages/man8/useradd.8.html) is a command to add new users, just look for a "useradd" keyword to see user creation events! Below is an example of what you can see in the logs: password change, user deletion, and then privileged user creation.

User Management Events

```bash
root@thm-vm:~$ cat /var/log/auth.log | grep -E '(passwd|useradd|usermod|userdel)\['
```

Lastly, depending on system configuration and installed packages, you may encounter interesting or unexpected events. For example, you may find commands launched with sudo, which can help track malicious actions. In the example below, the "ubuntu" user used sudo to stop EDR, read firewall state, and finally access root via "sudo su":

Commands Run With Sudo

```bash
root@thm-vm:~$ cat /var/log/auth.log | grep -E 'COMMAND='
```