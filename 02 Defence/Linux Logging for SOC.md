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

# Common Linux Logs

## Generic System Logs

Linux keeps track of many other events scattered across files in `/var/log`: kernel logs, network changes, service or cron runs, package installation, and many more. Their content and format can differ depending on the OS, and the most common log files are:

- `/var/log/kern.log`: Kernel messages and errors, useful for more advanced investigations
- `/var/log/syslog (or /var/log/messages)`: A consolidated stream of various Linux events
- `/var/log/dpkg.log (or /var/log/apt)`: Package manager logs on Debian-based systems
- `/var/log/dnf.log (or /var/log/yum.log)`: Package manager logs on RHEL-based systems

## App-Specific Logs

In SOC, you might also monitor a specific program, and to do this effectively, you need to use its logs. For example, analyze database logs to see which queries were run, mail logs to investigate phishing, container logs to catch anomalies, and web server logs to know which pages were opened, when, and by whom. You will explore these logs in the upcoming modules, but to give an overview, here is an example from the typical Nginx web server logs:

Nginx Web Access Logs

```shell-session
root@thm-vm:~$ cat /var/log/nginx/access.log
# Every log line corresponds to a web request to the web server
10.0.1.12 - - [11/08/2025:14:32:10 +0000] "GET / HTTP/1.1" 200 3022
10.0.1.12 - - [11/08/2025:14:32:14 +0000] "GET /login HTTP/1.1" 200 1056
10.0.1.12 - - [11/08/2025:14:33:09 +0000] "POST /login HTTP/1.1" 302 112
10.0.4.99 - - [11/08/2025:17:11:20 +0000] "GET /images/logo.png HTTP/1.1" 200 5432
10.0.5.21 - - [11/08/2025:17:56:23 +0000] "GET /admin HTTP/1.1" 403 104
```

## Bash History

Another valuable log source is Bash history - a feature that records each command you run after pressing Enter. By default, commands are first stored in memory during your session, and then written to the per-user `~/.bash_history` file when you log out. You can open the `~/.bash_history` file to review commands from previous sessions or use the `history` command to view commands from both your current and past sessions:

Although the Bash history file looks like a vital log source, it is rarely used by SOC teams in their daily routine. This is because it does not track non-interactive commands (like those initiated by your OS, cron jobs, or web servers) and has some other limitations. While you can [configure it](https://datawookie.dev/blog/2023/04/configuring-bash-history/) to be more useful, there are still a few issues you should know about:

Bash History Limitations

```bash
# Attackers can simply add a leading space to the command to avoid being logged
ubuntu@thm-vm:~$  echo "You will never see me in logs!"

# Attackers can paste their commands in a script to hide them from Bash history
ubuntu@thm-vm:~$ nano legit.sh && ./legit.sh
 
# Attackers can use other shells like /bin/sh that don't save the history like Bash
ubuntu@thm-vm:~$ sh
$ echo "I am no longer tracked by Bash!"
```

# Runtime Monitoring

Up to this point, you have explored various Linux log sources, but none can reliably answer questions like "Which programs did Bob launch today?" or "Who deleted my home folder, and when?". That's because, by default, Linux doesn't log process creation, file changes, or network-related events, collectively known as **runtime** events. Interestingly, Windows faces the same limitation, which is why in the [Windows Logging for SOC](https://tryhackme.com/room/windowsloggingforsoc) room we had to use an additional tool: Sysmon. In Linux, we'll take a similar approach.

## System Calls

Before moving on, let's explore a core OS concept that might help you understand many other topics: system calls. In short, whenever you need to open a file, create a process, access the camera, or request any other OS service, you make a specific system call. There are [over 300](https://man7.org/linux/man-pages/man2/syscalls.2.html) system calls in Linux, like `execve` to execute a program. Below is a high-level flowchart of how it works:

![A flowchart of a Linux system call: you start a program, the "execve" system call is passed to the kernel, the kernel uses hardware resources, and returns the results.](https://tryhackme-images.s3.amazonaws.com/user-uploads/678ecc92c80aa206339f0f23/room-content/678ecc92c80aa206339f0f23-1757002115631.svg)

Why do you need to know about system calls? Well, all modern EDRs and logging tools rely on them - they monitor the main system calls and log the details in a human-readable format. Since there is nearly no way for attackers to bypass system calls, all you have to do is choose the system calls you'd like to log and monitor. In the next task, you will try it in practice using auditd.