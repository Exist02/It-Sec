
# Authentifikation

| **Event ID**                     | **Purpose**                                                                 | **Logging**                                                    | **Limitations**                                                                                              |
| -------------------------------- | --------------------------------------------------------------------------- | -------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| **4624  <br>**(Successful Logon) | Detect suspicious RDP/network logins and identify the attack starting point | Logged on the target machine, the one you are trying to access | **Noisy**. You will see hundreds of logon events per minute on loaded servers                                |
| **4625  <br>**(Failed Logon)     | Detect brute force, password spraying, or vulnerability scanning            | Logged on the target machine, the one you are trying to access | **Inconsistent**. The logs have lots of caveats that may trick you into the wrong understanding of the event |
|                                  |                                                                             |                                                                |                                                                                                              |


# User Management

|**Event ID**|**Description**|**Malicious Usage**|
|---|---|---|
|**4720** / **4722** / **4738**|A user account was  <br>created / enabled / changed|Attackers might create a backdoor account or even enable an old one to avoid detection|
|**4725** / **4726**|A user account was  <br>disabled / deleted|In some advanced cases, threat actors may disable privileged SOC accounts to slow down their actions|
|**4723** / **4724**|A user changed their password /  <br>User's password was reset|Given enough permissions, threat actors might reset the password and then access the required user|
|**4732** / **4733**|A user was added to /  <br>removed from a security group|Attackers often add their backdoor accounts to privileged groups like "[Administrators](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/understand-security-groups#administrators)"|

# Process Monitoring (Sysmon)


Sysmon is a free tool from the Microsoft Sysinternals suite that became a de facto standard for advanced monitoring in addition to the default system logs. For this task, we'll jump right into analyzing Sysmon logs but you can learn more about this great tool in another TryHackMe [room](https://tryhackme.com/room/sysmon).

So, if I were to choose between enabling the basic, noisy 4688 event ID or spending some time installing Sysmon to receive more powerful and flexible logs, I would proceed with Sysmon, and you are encouraged to do the same! Once installed, Sysmon logs are found in Event Viewer under `Applications & Services -> Microsoft -> Windows -> Sysmon -> Operational`.

|**Event Code**|**Purpose**|**Limitations**|
|---|---|---|
|**4688  <br>**(Security Log: Process Creation)|Log an event every time a new process is launched, including its command line and parent process details|Disabled by default, you need to enable it by following the [official documentation](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/component-updates/command-line-process-auditing)|
|**1  <br>**(Sysmon: Process Creation)|Replace 4688 event code and provide more advanced fields like process hash and its signature|Sysmon is an external tool not installed by default. Check out the [Sysmon official page](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)|
## Sysmon Event ID 1 in Action

As you can see on the screenshot above, event ID 1 has a lot of different fields, the most important of which can be grouped as:

- **Process Info**: Context of the launched process, including its PID, path (image), and command line
- **Parent Info**: Context of the parent process, very useful to build a process tree or an attack chain
- **Binary Info**: Process hash, signature, and PE metadata. You will need it for more advanced rooms
- **User Context**: A user running the process and, most importantly, Logon ID - same as in the Security logs

# Sysmon Files and Network

|**Event ID**|**Security Log Alternative**|**Event Purpose**|
|---|---|---|
|**11 / 13  <br>**(File Create / Registry Value Set)|**4656** for file changes and **4657** for registry changes, both disabled by default|Detect files dropped by malware or its changes to the registry (e.g. for persistence)|
|**3 / 22  <br>**(Network Connection / DNS Query)|No direct alternative, requires additional firewall and DNS configuration|Detect traffic from untrusted processes or to known malicious destinations|
# PowerShell: Logging Commands

## How It Works

Every program has a specific purpose: **firefox.exe** is a web browser, **notepad.exe** is a text editor, and **whoami.exe** simply outputs your username. If you're just browsing the web, you might only create a single Firefox process. However, with every out-of-scope task like RDP access or photo editing, you will have to open new programs and create additional logs.

PowerShell, on the other hand, is a powerful all-in-one tool for managing the system. Once you launch `powershell.exe`, you can run hundreds of different commands within the same terminal session without creating new processes for each action. This is why Sysmon is not very helpful here, and you'll need to find an alternative logging approach.

## PowerShell History File

There are at least five methods to monitor PowerShell, each with its own pros and cons. While you can check out the [Logless Hunt](https://tryhackme.com/room/loglesshunt) room and research AMSI and Transcript Logging topics, in this room, we will focus on a simple but effective way to track PowerShell commands - the PowerShell history file:

```plaintext
C:\Users\<USER>\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
```
