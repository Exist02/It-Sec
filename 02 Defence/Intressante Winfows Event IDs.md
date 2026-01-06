| **Event ID**                     | **Purpose**                                                                 | **Logging**                                                    | **Limitations**                                                                                              |
| -------------------------------- | --------------------------------------------------------------------------- | -------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| **4624  <br>**(Successful Logon) | Detect suspicious RDP/network logins and identify the attack starting point | Logged on the target machine, the one you are trying to access | **Noisy**. You will see hundreds of logon events per minute on loaded servers                                |
| **4625  <br>**(Failed Logon)     | Detect brute force, password spraying, or vulnerability scanning            | Logged on the target machine, the one you are trying to access | **Inconsistent**. The logs have lots of caveats that may trick you into the wrong understanding of the event |
|                                  |                                                                             |                                                                |                                                                                                              |


|**Event ID**|**Description**|**Malicious Usage**|
|---|---|---|
|**4720** / **4722** / **4738**|A user account was  <br>created / enabled / changed|Attackers might create a backdoor account or even enable an old one to avoid detection|
|**4725** / **4726**|A user account was  <br>disabled / deleted|In some advanced cases, threat actors may disable privileged SOC accounts to slow down their actions|
|**4723** / **4724**|A user changed their password /  <br>User's password was reset|Given enough permissions, threat actors might reset the password and then access the required user|
|**4732** / **4733**|A user was added to /  <br>removed from a security group|Attackers often add their backdoor accounts to privileged groups like "[Administrators](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/understand-security-groups#administrators)"|