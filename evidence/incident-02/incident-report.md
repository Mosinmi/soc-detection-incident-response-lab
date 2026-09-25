# Incident Report — Incident #2

## 1. Incident Summary

**Incident ID:** INC-002  
**Incident Title:** Privileged Sudo Activity Investigation  
**Date:** September 25, 2026  
**Host:** Ubuntu Server  
**Host IP:** 192.168.0.162  
**Analyst:** Mosinmi  
**Severity:** Informational / Low  
**Classification:** Benign / Authorized Administrative Activity  
**Status:** Closed — No malicious activity identified

This incident documents the detection and investigation of privileged `sudo` activity on the Ubuntu Server.

The activity was intentionally generated as part of a controlled Security Operations Center (SOC) detection and incident response laboratory. The purpose was to demonstrate the collection and analysis of Linux privileged-access logs and to determine whether observed root-level activity represented legitimate administration or suspicious privilege escalation.

The investigation identified multiple `sudo` commands executed by the local user `mosinmi`. The commands successfully executed with root privileges and generated corresponding session-open and session-close events in `/var/log/auth.log`.

No evidence was identified indicating unauthorized privilege escalation, exploitation, account compromise, or malicious execution.

---

## 2. Incident Classification

**Category:** Privileged Access / Sudo Activity

**Classification:** Benign Administrative Activity

**Severity:** Informational / Low

**Detection Source:** Linux `/var/log/auth.log`

**Affected Account:** `mosinmi`

**Privileged Account:** `root`

**Source Context:** Local terminal session (`TTY=pts/0`)

**Disposition:** Closed as authorized activity

---

## 3. Environment

The investigation was performed on an Ubuntu Server used as the monitored endpoint in the SOC laboratory.

### System Information

- Operating System: Ubuntu Server 22.04.5 LTS
- Host IP: `192.168.0.162`
- Local User: `mosinmi`
- User ID: `1000`
- Primary Group: `mosinmi`
- Sudo Group: `sudo`
- Additional Group: `vboxsf`
- Privileged User: `root`

### Network Context

The Ubuntu server is part of a controlled laboratory environment used for defensive cybersecurity testing.

The Windows host used for security testing has the address:

`192.168.0.136`

---

## 4. Detection

Privileged activity was detected through the Linux authentication log:

`/var/log/auth.log`

The following command was used to identify sudo activity:

```bash
sudo grep "sudo:" /var/log/auth.log | tail -n 20

The resulting events contained records showing the user mosinmi executing commands with:

USER=root

The logs also contained PAM session events showing when root sessions were opened and closed.

Examples of observed administrative commands included:

COMMAND=/usr/bin/id
COMMAND=/usr/bin/whoami
COMMAND=/usr/bin/fail2ban-client status sshd
COMMAND=/usr/sbin/sshd -T

These events provided sufficient information to begin an investigation into the nature of the privileged activity.

5. Incident Timeline
12:09:35

A sudo session-close event was recorded:

sudo: pam_unix(sudo:session): session closed for user root
12:12:49

The user mosinmi executed:

/usr/bin/fail2ban-client status sshd

The command was executed with:

USER=root

The corresponding root session was opened and subsequently closed.

12:19:15

The user executed privileged commands associated with the investigation of Incident #1.

Observed commands included:

/usr/bin/tee /home/mosinmi/soc-evidence/incident-01/ssh-authentication-events.txt
/usr/bin/grep -E 'Invalid user wronguser|Failed password' /var/log/auth.log

Root sessions were opened and closed for these commands.

12:19:16

The following privileged commands were executed:

/usr/bin/fail2ban-client status sshd
/usr/sbin/sshd -T

These commands were used to investigate SSH security configuration and Fail2Ban status.

12:28:18

The controlled Incident #2 test generated the following sudo command:

/usr/bin/id

The command executed successfully as root.

A corresponding root session was opened and closed.

12:28:27

The following command was executed:

/usr/bin/whoami

The command executed successfully as root.

A corresponding root session was opened and closed.

12:28:43–12:31:53

Additional grep commands were executed against /var/log/auth.log to review sudo activity during the investigation.

These commands were executed by mosinmi through sudo.

6. Detection Evidence

The primary evidence was obtained from:

/var/log/auth.log

Representative events included:

Sep 25 12:28:18 Ubuntu sudo:  mosinmi : TTY=pts/0 ; PWD=/home/mosinmi ; USER=root ; COMMAND=/usr/bin/id
Sep 25 12:28:18 Ubuntu sudo: pam_unix(sudo:session): session opened for user root(uid=0) by (uid=1000)
Sep 25 12:28:18 Ubuntu sudo: pam_unix(sudo:session): session closed for user root

A second controlled privileged command was:

Sep 25 12:28:27 Ubuntu sudo:  mosinmi : TTY=pts/0 ; PWD=/home/mosinmi ; USER=root ; COMMAND=/usr/bin/whoami

The associated PAM records confirmed the root session was opened and closed.

These events demonstrate that privileged execution occurred through the system's configured sudo mechanism.

7. User Identity Analysis

The current user was identified as:

mosinmi

The user identity information was:

uid=1000(mosinmi) gid=1000(mosinmi) groups=1000(mosinmi),27(sudo),999(vboxsf)

This confirms that mosinmi is a member of the Linux sudo group.

The account therefore has administrative privileges on the system.

8. Sudo Privilege Analysis

The following command was used to inspect the user's sudo permissions:

sudo -l

The system returned:

Matching Defaults entries for mosinmi on Ubuntu:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User mosinmi may run the following commands on Ubuntu:
    (ALL : ALL) ALL

This configuration means that the mosinmi account is authorized to execute commands as any user, including root, through sudo.

The presence of unrestricted sudo privileges represents a significant administrative capability, but it does not by itself demonstrate malicious activity.

9. Investigation

The investigation focused on determining:

Which account generated the privileged activity.
Which commands were executed.
Whether root privileges were obtained through the expected sudo mechanism.
Whether there was evidence of unauthorized privilege escalation.
Whether the activity corresponded with known laboratory operations.

The logs consistently identified:

mosinmi

as the invoking user.

The commands were executed from:

TTY=pts/0

and the working directory was:

PWD=/home/mosinmi

The privileged target was:

USER=root

The commands observed were consistent with system administration and SOC investigation tasks.

In particular, commands such as:

fail2ban-client status sshd
sshd -T
grep /var/log/auth.log

were associated with the investigation of the previous SSH security incident.

The controlled commands:

id
whoami

were intentionally executed to generate and validate privileged activity for this SOC laboratory.

10. Privilege Escalation Assessment

The investigation did not identify evidence of an exploit-based privilege escalation.

The account mosinmi already possessed authorized sudo privileges.

The observed transition was:

mosinmi
    |
    | sudo
    v
root

This is expected behavior for an account belonging to the sudo group with the following authorization:

(ALL : ALL) ALL

There was no evidence in the collected logs of:

Exploitation of a vulnerable service
Unauthorized modification of sudo configuration
Creation of an unexpected privileged account
Successful unauthorized authentication
Abuse of a SUID binary
Kernel-level privilege escalation
Malicious payload execution
Persistence through root privileges

Therefore, the observed activity was classified as authorized administrative activity rather than a confirmed privilege-escalation incident.

11. Indicators of Activity
User
mosinmi
User ID
uid=1000
Privileged User
root
Source Terminal
pts/0
Commands Observed
/usr/bin/id
/usr/bin/whoami
/usr/bin/fail2ban-client status sshd
/usr/sbin/sshd -T
/usr/bin/grep
/usr/bin/tee
Log Source
/var/log/auth.log


12. Detection Logic

A SOC monitoring rule for privileged Linux activity could identify sudo events using:

sudo:

within:

/var/log/auth.log

A more targeted detection concept would identify commands executed as root:

USER=root

Combined monitoring logic could identify:

sudo + USER=root + COMMAND

This allows a SOC analyst to identify privileged commands and investigate whether the activity is expected or suspicious.

Additional detection opportunities include:

Multiple privileged commands within a short period
Sudo activity outside expected administrative hours
Sudo activity from an unexpected account
Repeated failed sudo authentication
Creation or modification of sudoers configuration
Privileged commands executed immediately after suspicious authentication
Privilege escalation followed by persistence activity
Sudo commands executed from unexpected sessions

13. Containment

No containment action was required.

The activity was determined to be authorized administrative behavior generated within the controlled SOC laboratory.

No malicious process, unauthorized account, or compromised credential was identified.

Terminating the user's sudo privileges would have disrupted the laboratory environment and was therefore not appropriate based on the available evidence.

14. Response

The response consisted of:

Reviewing /var/log/auth.log.
Identifying the account responsible for privileged activity.
Reviewing the commands executed through sudo.
Confirming the current user's identity.
Reviewing group membership.
Reviewing sudo authorization using sudo -l.
Comparing the observed commands against the known laboratory activities.
Determining that the activity was authorized.
Preserving the evidence for the SOC investigation.


15. Root Cause

There was no security compromise associated with this event.

The root-level activity occurred because the mosinmi account is intentionally configured as an administrative account and belongs to the Linux sudo group.

The authorization configuration permits:

(ALL : ALL) ALL

which allows the user to execute commands with elevated privileges through sudo.

The activity observed during this incident was therefore the expected result of authorized administrative access.

16. Security Considerations

Although the activity was benign, unrestricted sudo privileges increase the potential impact if the account is compromised.

An attacker who successfully compromises the mosinmi account could potentially use the account's sudo authorization to execute commands as root.

For production environments, privileged access should therefore be controlled using appropriate administrative security measures such as:

Strong authentication
SSH key-based authentication
MFA where supported
Least-privilege sudo policies
Centralized logging
Privileged command monitoring
Alerting on unusual sudo behavior
Regular review of administrative group membership
Removal of unnecessary administrative privileges

These controls reduce the potential impact of compromised privileged accounts.

17. Lessons Learned

This incident demonstrated several important SOC investigation techniques.

17.1 Log-Based Detection

Linux authentication logs provide valuable visibility into privileged activity.

17.2 User Attribution

The sudo logs provided the invoking username, terminal, working directory, target user, and executed command.

17.3 Context Matters

A root-level command should not automatically be classified as malicious.

The analyst must establish:

Who executed it
What command was executed
Why it was executed
Whether the account was authorized
Whether the activity corresponds to expected operations
17.4 Privilege Escalation Requires Investigation

The presence of USER=root indicates privileged execution, but it does not independently prove exploitation or unauthorized escalation.

17.5 SOC Analysts Must Distinguish Signal From Noise

Legitimate administrative activity can generate alerts that resemble suspicious behavior.

Proper investigation prevents legitimate system administration from being incorrectly classified as a security incident.

18. Recommended Monitoring

For a production SOC environment, the following monitoring rules would improve detection of suspicious privileged activity:

Rule 1 — Sudo Command Monitoring

Alert when privileged commands are executed by monitored accounts.

sudo + USER=root + COMMAND
Rule 2 — Unusual Administrative Account

Alert when a non-standard or unexpected account executes sudo commands.

Rule 3 — Privileged Activity After Authentication Anomaly

Correlate suspicious SSH authentication events with subsequent sudo activity.

Example sequence:

Failed SSH authentication
        ↓
Successful authentication
        ↓
sudo command
        ↓
root activity
Rule 4 — Sudo Configuration Modification

Monitor changes to:

/etc/sudoers
/etc/sudoers.d/
Rule 5 — Privileged Account Creation

Monitor creation or modification of users belonging to:

sudo

or other administrative groups.

Rule 6 — Suspicious Root Commands

Monitor root execution of commands associated with:

Credential access
Persistence
Firewall modification
Account creation
Security control disabling
Log deletion
Remote access configuration


19. Evidence Files

The following evidence was preserved during the investigation:

~/soc-evidence/incident-02/
├── sudo-events.txt
└── user-context.txt
sudo-events.txt

Contains relevant sudo and PAM session events extracted from:

/var/log/auth.log
user-context.txt

Contains:

Current username
User ID and group membership
Sudo privilege configuration
20. MITRE ATT&CK Mapping

The activity can be considered in the context of MITRE ATT&CK privilege-related behavior.

The observed event involved execution of commands with elevated privileges through the legitimate Linux sudo mechanism.

However, because the activity was authorized and intentionally generated for the laboratory, this report does not classify the event as confirmed malicious ATT&CK activity.

The relevant defensive concept is monitoring for abuse of legitimate privilege mechanisms and distinguishing authorized administrative use from unauthorized privilege escalation.

21. Incident Conclusion

The investigation determined that the observed privileged activity was authorized administrative activity performed by the local user mosinmi.

The account is a member of the Linux sudo group and is authorized to execute commands with elevated privileges. The observed commands, including id, whoami, fail2ban-client, sshd -T, grep, and tee, were consistent with the controlled SOC laboratory exercises and system security investigation.

The authentication logs confirmed that the commands were executed through the expected sudo mechanism and that the corresponding root sessions were opened and closed normally.

No evidence was identified indicating unauthorized privilege escalation, exploitation, account compromise, malicious persistence, or unauthorized root access.

The incident is therefore classified as:

BENIGN / AUTHORIZED ADMINISTRATIVE ACTIVITY

and is considered:

CLOSED — NO MALICIOUS ACTIVITY IDENTIFIED

This incident demonstrates the importance of investigating privileged-access alerts in context rather than treating every root-level command as malicious. It also establishes a foundation for future SOC correlation rules involving authentication anomalies, privileged commands, persistence, and suspicious system modifications.
