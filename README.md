# SOC Detection & Incident Response Lab

## Overview

This project is a hands-on Security Operations Center (SOC) detection and incident response laboratory built on an Ubuntu Server environment.

The lab demonstrates a complete defensive security workflow:

**Generate Controlled Security Event → Collect Logs → Detect → Investigate → Build Timeline → Contain → Verify → Document**

The activities were deliberately performed in a controlled lab environment to simulate security events that a SOC analyst or security engineer may encounter during routine monitoring and incident investigation.

---

## Objectives

The objectives of this laboratory were to:

- Generate controlled security events in a Linux environment.
- Collect and analyze authentication and system logs.
- Detect suspicious authentication activity.
- Investigate privileged administrative activity.
- Investigate persistence mechanisms.
- Analyze local network listeners and running processes.
- Build incident timelines from available evidence.
- Perform containment and remediation actions.
- Verify that containment actions were successful.
- Produce structured incident reports.
- Preserve technical evidence for security investigation and portfolio documentation.

---

## Lab Environment

| Component | Configuration |
|---|---|
| Operating System | Ubuntu 22.04.5 LTS |
| Linux Kernel | 6.8.0-138-generic |
| Ubuntu Server IP | 192.168.0.162 |
| Windows Host IP | 192.168.0.136 |
| Network | Local LAN / VirtualBox bridged networking |
| SSH | OpenSSH |
| Firewall | UFW |
| Intrusion Prevention | Fail2Ban |
| Logging | `/var/log/auth.log`, systemd journal |
| Monitoring Tools | `ss`, `ps`, `lsof`, `journalctl`, `grep` |
| Service Management | systemd |
| Controlled Network Service | Python HTTP server |

---

# Incident 01 — Controlled SSH Authentication Attack

## Scenario

A controlled authentication attack was generated against the Ubuntu SSH service from the Windows host.

The purpose was to simulate repeated failed authentication attempts and demonstrate how authentication logs and Fail2Ban can be used for detection and automated response.

## Attack Simulation

Five connection attempts were generated using an invalid username:

```text
wronguser

The connection originated from:

192.168.0.136

against the Ubuntu server:

192.168.0.162
Evidence

The SSH authentication log recorded five invalid-user events.

The observed events included:

Invalid user wronguser from 192.168.0.136

Five separate attempts were observed during the investigation period.

Detection

Fail2Ban was inspected after the authentication events.

The resulting status showed:

Currently failed: 0
Total failed: 5
Currently banned: 1
Total banned: 1
Banned IP list: 192.168.0.136

This demonstrated successful detection and automated blocking of the simulated authentication activity.

Verification

A subsequent TCP connection test from Windows showed:

TcpTestSucceeded : False

while the host remained reachable.

This confirmed that the SSH connection was no longer succeeding from the blocked source.

Response

The event was investigated through:

/var/log/auth.log
Fail2Ban status
SSH security configuration
Network connectivity testing
Classification

Controlled Security Event

The activity was intentionally generated for the laboratory.

Evidence Location
evidence/incident-01/
├── fail2ban-status.txt
├── incident-report.md
├── ssh-authentication-events.txt
└── ssh-security-config.txt
Incident 02 — Privileged sudo Activity
Scenario

Controlled administrative commands were executed using sudo to demonstrate how privileged activity appears in Linux authentication logs.

The purpose was to practice identifying privileged operations and distinguishing authorized administrative activity from potentially suspicious privilege escalation.

Commands Investigated
sudo id
sudo whoami
Log Evidence

The authentication log recorded privileged commands executed by the mosinmi account, including:

mosinmi : TTY=pts/0 ; PWD=/home/mosinmi ; USER=root ; COMMAND=/usr/bin/id

and:

mosinmi : TTY=pts/0 ; PWD=/home/mosinmi ; USER=root ; COMMAND=/usr/bin/whoami

The corresponding sudo sessions were opened and closed successfully.

User Context

The investigating account was:

mosinmi

with:

uid=1000(mosinmi)

The account belongs to the sudo group.

sudo Configuration

The account was confirmed to have administrative privileges through:

sudo -l

The result indicated:

User mosinmi may run the following commands on Ubuntu:
    (ALL : ALL) ALL
Investigation Result

The observed activity was determined to be authorized administrative activity performed as part of the security laboratory.

There was no evidence in this incident of unauthorized privilege escalation.

Classification

Informational / Low

Disposition: Benign / Authorized

Evidence Location
evidence/incident-02/
├── incident-report.md
├── sudo-events.txt
└── user-context.txt
Incident 03 — Suspicious Systemd Persistence
Scenario

A controlled systemd service was created to simulate a persistence mechanism that a SOC analyst might investigate on a Linux endpoint.

The service was intentionally created as part of the laboratory.

Service

The controlled service was:

soc-lab-test.service

Location:

/etc/systemd/system/soc-lab-test.service

The service executed:

/usr/bin/touch /tmp/soc-lab-test-marker
Persistence Mechanism

The service was enabled through:

/etc/systemd/system/multi-user.target.wants/soc-lab-test.service

This demonstrated how a service can be configured to start as part of the normal system boot process.

Detection

The service status showed:

Loaded: loaded
enabled
Active: active (exited)

The execution created:

/tmp/soc-lab-test-marker

The marker was owned by:

root root
Timeline

The investigation identified:

Creation of the controlled service.
Enabling of the service.
Execution of the service.
Creation of the test marker.
Investigation of service metadata and journal entries.
Removal of the service and persistence mechanism.

Systemd journal evidence recorded the service start and completion.

Containment

The controlled persistence mechanism was removed using:

sudo systemctl stop soc-lab-test.service
sudo systemctl disable soc-lab-test.service
sudo rm -f /etc/systemd/system/soc-lab-test.service
sudo rm -f /etc/systemd/system/multi-user.target.wants/soc-lab-test.service
sudo systemctl daemon-reload
sudo rm -f /tmp/soc-lab-test-marker
Verification

The following checks confirmed successful containment:

PASS: Service no longer registered
PASS: Persistence symlink removed
PASS: Service file removed
PASS: Test artifact removed
Classification

Controlled / Benign Persistence Activity

Severity: Medium

Status: Closed

Evidence Location
evidence/incident-03/
├── artifact-metadata.txt
├── incident-report.md
├── persistence-symlink.txt
├── service-details.txt
├── service-metadata.txt
├── service-status.txt
└── systemd-journal.txt
Incident 04 — Suspicious Network Service / Local Network Listener
Scenario

A controlled Python HTTP server was created to simulate a suspicious network listener on a Linux endpoint.

The purpose was to demonstrate how a SOC analyst can identify an unexpected listening port, map the port to a process, investigate the process, and remove the service.

Baseline

The initial network review identified expected services including:

TCP 0.0.0.0:22
TCP 127.0.0.1:631
TCP 127.0.0.53:53

The SSH service was listening on port 22.

No suspicious external TCP connection was identified during the baseline investigation.

Controlled Listener

A Python HTTP server was started on:

127.0.0.1:8080

The command used was:

python3 -m http.server 8080 --bind 127.0.0.1

The process ID was:

5481
Detection

Network socket inspection identified:

127.0.0.1:8080

with:

python3
PID 5481

The process was mapped to:

/usr/bin/python3.10

running under:

mosinmi
Process Investigation

The investigation identified:

PID: 5481
User: mosinmi
Executable: /usr/bin/python3.10
Working Directory: /home/mosinmi
Listener: 127.0.0.1:8080

The process environment contained normal user-session values such as:

USER=mosinmi
HOME=/home/mosinmi
PWD=/home/mosinmi
SHELL=/bin/bash

The HTTP server generated a successful local request:

HTTP/1.0 200 OK
Network Scope

The listener was bound specifically to:

127.0.0.1

Therefore, it was restricted to the local host and was not exposed as a listener on the external network interface.

No command-and-control activity was identified.

Containment

The controlled process was terminated:

kill 5481
Verification

Post-containment checks confirmed:

PASS: Process terminated
PASS: Port 8080 no longer listening
PASS: No listener detected on port 8080
Classification

Controlled / Benign Local Network Service

The activity was intentionally generated for the laboratory.

Evidence Location
evidence/incident-04/
├── http-server.log
├── http-server.log-copy
├── http-server.pid
├── incident-report.md
├── network-listener.txt
├── network-process.txt
├── process-command.txt
├── process-details.txt
└── process-timeline.txt
Detection and Investigation Techniques

The laboratory used standard Linux investigation techniques.

Authentication Monitoring
sudo grep "sudo:" /var/log/auth.log
sudo tail -n 50 /var/log/auth.log
Network Enumeration
sudo ss -tulnp
Process Investigation
ps
lsof
Systemd Investigation
systemctl status <service>
systemctl is-enabled <service>
journalctl -u <service>
Privilege Investigation
sudo -l
id
Fail2Ban Investigation
sudo fail2ban-client status sshd
SOC Investigation Workflow

The incidents followed a repeatable investigation process:

Security Event
      ↓
Log / System Evidence
      ↓
Detection
      ↓
Initial Triage
      ↓
Evidence Collection
      ↓
Process / Network / Account Investigation
      ↓
Timeline Reconstruction
      ↓
Incident Classification
      ↓
Containment
      ↓
Verification
      ↓
Incident Documentation
Key Skills Demonstrated
SOC operations
Linux security monitoring
Incident detection
Incident investigation
Incident response
Authentication log analysis
Privileged activity monitoring
Fail2Ban
SSH security
systemd persistence investigation
Linux process investigation
Network socket analysis
Service identification
Evidence collection
Incident timeline reconstruction
Containment procedures
Post-containment verification
Security documentation
Tools and Technologies
Ubuntu Linux
OpenSSH
Fail2Ban
UFW
systemd
journalctl
ss
ps
lsof
grep
PowerShell
Windows SSH client
Python HTTP server
Evidence and Documentation

Each simulated incident has its own evidence directory containing relevant logs, investigation output, metadata, and incident reports.

evidence/
├── incident-01/
├── incident-02/
├── incident-03/
└── incident-04/

The evidence demonstrates the investigation process rather than representing a real-world production compromise.

Lab Limitations

This is a controlled cybersecurity laboratory.

All security events were intentionally generated by the lab operator on systems under controlled access.

The project demonstrates:

Detection methodology
Investigation methodology
Evidence handling
Response procedures
Verification techniques
Security documentation

It does not represent an investigation of an actual compromised production environment.
