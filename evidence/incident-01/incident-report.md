# Incident 01 — SSH Authentication Attack

## 1. Incident Summary

A series of failed SSH authentication attempts was detected against the Ubuntu Server from the Windows host on the local laboratory network.

The activity involved repeated attempts to authenticate using the nonexistent username `wronguser`.

Five authentication attempts were recorded within approximately 17 seconds.

Fail2Ban detected the repeated authentication failures and placed the source IP address `192.168.0.136` on its SSH ban list.

A subsequent connectivity test confirmed that the Windows host could still reach the Ubuntu server at the network layer, but TCP port 22 was no longer reachable from the banned source.

No evidence of successful authentication or account compromise was identified during the investigation.

---

# 2. Incident Classification

| Field | Value |
|---|---|
| Incident ID | IR-001 |
| Incident Type | SSH Authentication Attack |
| Category | Authentication / Access Attempt |
| Severity | Medium |
| Status | Contained |
| Detection Mechanism | Fail2Ban |
| Affected Service | SSH |
| Source IP | `192.168.0.136` |
| Destination IP | `192.168.0.162` |
| Target Username | `wronguser` |
| Successful Authentication | Not observed |
| Account Compromise | No evidence observed |

---

# 3. Environment

The investigation was performed in a controlled cybersecurity laboratory.

### Source

```text
Windows Host
IP: 192.168.0.136

Target
Ubuntu Server
IP: 192.168.0.162
Target Service
SSH
TCP Port: 22
Security Controls

The Ubuntu server was configured with:

PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes

Fail2Ban was also enabled with an SSH jail.

4. Detection

The initial security events were identified in:

/var/log/auth.log

The SSH service recorded multiple invalid-user events.

Observed events included:

Sep 25 11:48:35 Ubuntu sshd[3563]: Invalid user wronguser from 192.168.0.136 port 49304

Sep 25 11:48:44 Ubuntu sshd[3565]: Invalid user wronguser from 192.168.0.136 port 49308

Sep 25 11:48:47 Ubuntu sshd[3567]: Invalid user wronguser from 192.168.0.136 port 49309

Sep 25 11:48:50 Ubuntu sshd[3569]: Invalid user wronguser from 192.168.0.136 port 49310

Sep 25 11:48:52 Ubuntu sshd[3571]: Invalid user wronguser from 192.168.0.136 port 49311

A total of five failed authentication events were identified.

5. Incident Timeline
Time	Event
11:48:35	Invalid SSH user wronguser from 192.168.0.136
11:48:44	Second invalid-user SSH attempt
11:48:47	Third invalid-user SSH attempt
11:48:50	Fourth invalid-user SSH attempt
11:48:52	Fifth invalid-user SSH attempt
After repeated failures	Fail2Ban detected the SSH activity
Investigation time	Fail2Ban reported 192.168.0.136 as banned
Verification	TCP/22 connectivity from Windows failed

The five observed authentication attempts occurred over approximately 17 seconds.

6. Detection Evidence

Fail2Ban reported:

Currently failed: 0
Total failed: 5
Currently banned: 1
Total banned: 1
Banned IP list: 192.168.0.136

This confirms that Fail2Ban had registered the five failed authentication events and that the source IP was currently listed as banned.

7. Containment

The primary containment mechanism was Fail2Ban.

After detecting the repeated SSH authentication failures, Fail2Ban placed:

192.168.0.136

on its SSH ban list.

A connectivity test from the Windows host produced:

PingSucceeded     : True
TcpTestSucceeded  : False

This demonstrated that:

The Ubuntu server remained reachable at the IP/network layer.
TCP port 22 was not reachable from the banned source.
The source IP was therefore being prevented from establishing an SSH connection.
8. Investigation

The investigation examined the following evidence sources:

/var/log/auth.log

and:

Fail2Ban SSH jail status

The authentication log showed repeated attempts using:

wronguser

The source of every observed attempt was:

192.168.0.136

The SSH service was confirmed to be running.

The server's SSH configuration was also reviewed.

The relevant configuration was:

permitrootlogin no
pubkeyauthentication yes
passwordauthentication no

These settings indicate that root login through SSH was disabled and password authentication was disabled.

9. Impact Assessment

The investigation found no evidence of successful authentication.

The username used in the observed events was:

wronguser

which was an invalid/nonexistent account.

No successful login event was identified in the evidence collected for this incident.

Therefore, the observed activity represents an authentication attempt rather than a confirmed system compromise.

10. Indicators of Activity

The primary indicators associated with this incident were:

Source IP
192.168.0.136
Target IP
192.168.0.162
Target Service
SSH / TCP 22
Target Username
wronguser
Event Pattern
Five invalid SSH authentication attempts
within approximately 17 seconds

11. Detection Logic

A basic detection rule for this type of activity can be expressed as:

IF
multiple SSH authentication failures
occur from the same source IP
within a short time period

THEN
generate an authentication attack alert
and investigate the source.

For this laboratory, Fail2Ban provided the automated detection and containment mechanism.

The relevant evidence was visible through:

sudo fail2ban-client status sshd

and:

sudo grep -E "Invalid user wronguser|Failed password" /var/log/auth.log
12. Response

The response performed by the security control was:

SSH authentication failures were logged.
Fail2Ban detected the repeated failures.
The source IP was added to the SSH ban list.
A connectivity test was performed.
TCP/22 connectivity from the banned source was unsuccessful.
No successful authentication was identified.
13. Root Cause

The immediate cause of the incident was repeated SSH authentication attempts against the Ubuntu server using an invalid username.

The activity was intentionally generated as part of a controlled cybersecurity laboratory exercise.

Therefore, this incident should not be interpreted as evidence of a real external attacker.

14. Lessons Learned

The investigation demonstrated several important SOC concepts.

Centralized Log Visibility

Authentication logs provide the evidence required to investigate SSH activity.

Detection

Repeated authentication failures can be detected through log analysis and automated security controls.

Automated Response

Fail2Ban can automatically respond to repeated SSH authentication failures by banning the source address.

Evidence-Based Investigation

Security conclusions should be based on observable evidence.

Authentication Does Not Equal Compromise

An authentication attempt does not mean that an attacker successfully gained access.

The investigation must establish whether authentication actually succeeded.

15. Recommended Monitoring

For a production environment, SOC monitoring should alert on:

Repeated SSH authentication failures
Invalid usernames
Multiple authentication attempts from a single source
Authentication attempts against privileged accounts
Successful login following multiple failures
New or unexpected source IP addresses
Changes to SSH configuration
Unexpected SSH service exposure
16. Evidence Files

The raw evidence associated with this incident is stored in:

~/soc-evidence/incident-01/

Expected files:

ssh-authentication-events.txt
fail2ban-status.txt
ssh-security-config.txt
incident-report.md
17. Incident Conclusion

The investigation identified five invalid SSH authentication attempts originating from 192.168.0.136 against the Ubuntu Server at 192.168.0.162.

Fail2Ban detected the repeated failures and placed the source IP on its SSH ban list.

A subsequent connectivity test confirmed that the source could reach the server at the network layer but could not establish a TCP connection to SSH.

No evidence of successful authentication or account compromise was identified.

The incident was therefore classified as a controlled SSH authentication attack that was successfully detected and contained by the configured security controls.


Analyst

Cybersecurity Portfolio Lab

Incident ID: IR-001

Date: 25 September 2026
