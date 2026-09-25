# Incident Report — Incident #4

## 1. Incident Summary

**Incident ID:** INC-004  
**Incident Title:** Suspicious Network Service / Local Network Listener Investigation  
**Date:** September 25, 2026  
**Host:** Ubuntu Server  
**Host IP:** 192.168.0.162  
**Analyst:** Mosinmi  
**Severity:** Informational / Low  
**Classification:** Controlled / Benign Network Activity  
**Status:** Closed — Network Service Successfully Contained

This incident documents the detection, investigation, evidence preservation, containment, and verification of a locally hosted network service on the Ubuntu Server.

The activity was intentionally generated as part of a controlled SOC detection and incident response laboratory.

A Python HTTP server was launched using:

python3 -m http.server 8080 --bind 127.0.0.1

The service created a TCP listener on:

127.0.0.1:8080

The investigation mapped the network listener to process ID 5481, identified the executing user as mosinmi, examined the process command line, inspected the network file descriptor, reviewed the HTTP request log, and established the process timeline.

The service was bound only to the loopback interface and therefore was not directly exposed through the server's LAN interface.

The service was subsequently terminated and verification confirmed that the process had stopped and port 8080 was no longer listening.

No evidence of malicious command-and-control communication or external network compromise was identified.

---

## 2. Incident Classification

**Category:** Network Service / Suspicious Listening Port

**Classification:** Controlled / Benign Activity

**Severity:** Informational / Low

**Detection Source:** `ss`, `lsof`, process inspection, and HTTP server logs

**Affected Host:** Ubuntu Server

**Process:** `python3`

**PID:** 5481

**User:** mosinmi

**Listening Port:** TCP 8080

**Listening Address:** 127.0.0.1

**Status:** Closed

---

## 3. Environment

The investigation was conducted on an Ubuntu Server used as the monitored endpoint in the SOC laboratory.

### System Information

- Operating System: Ubuntu Server 22.04.5 LTS
- Host IP: 192.168.0.162
- Local User: mosinmi
- Network Interface: enp0s3
- SSH Service: TCP port 22
- Controlled Test Service: TCP port 8080

The Ubuntu server is part of a controlled cybersecurity laboratory environment used to practice:

- Network monitoring
- Process identification
- Socket-to-process attribution
- Host-based investigation
- Incident containment
- Evidence preservation
- Security incident documentation

---

## 4. Initial Network Baseline

Before generating the controlled event, active network connections and listening services were examined using:

```bash
sudo ss -tunap
sudo ss -tulnp
sudo ss -tnp state established
sudo lsof -i -n -P


The baseline showed legitimate services including:

sshd
systemd-resolve
NetworkManager
avahi-daemon
cupsd

An established SSH connection was observed between:

192.168.0.162:22

and:

192.168.0.136:49247

This represented the analyst's active SSH session.

No suspicious external TCP connection was identified during the baseline investigation.

5. Controlled Network Event

A controlled local HTTP service was launched using:

python3 -m http.server 8080 --bind 127.0.0.1

The process was assigned:

PID: 5481

The service began listening on:

127.0.0.1:8080

The listener was verified using:

sudo ss -lntp | grep ':8080'

The result was:

LISTEN 0 5 127.0.0.1:8080 0.0.0.0:* users:(("python3",pid=5481,fd=3))

This confirmed that the Python process was responsible for the listening socket.

6. Detection

The suspicious-looking network service was detected through host-based network inspection.

The following command identified the listener:

sudo ss -lntp

The output identified:

127.0.0.1:8080

as a listening TCP endpoint.

The process associated with the endpoint was:

python3

with PID:

5481

This demonstrates a fundamental SOC investigation technique:

Network Port
     ↓
Socket
     ↓
Process
     ↓
User
     ↓
Command
7. Process Attribution

The process details were obtained using:

ps -fp 5481

The result identified:

UID          PID    PPID  C STIME TTY          TIME CMD
mosinmi     5481    3318  0 13:17 pts/0    00:00:00 python3 -m http.server 8080 --bind 127.0.0.1

The process was therefore directly attributable to the local user:

mosinmi

The process command line confirmed that it was running the Python built-in HTTP server.

8. Command-Line Analysis

The process command line was independently verified using:

sudo tr '\0' ' ' < /proc/5481/cmdline

The result was:

python3 -m http.server 8080 --bind 127.0.0.1

The command indicates:

Python 3 was used.
The built-in HTTP server module was executed.
TCP port 8080 was selected.
The service was explicitly bound to 127.0.0.1.

The loopback binding is significant because it restricts the service to the local host rather than exposing it directly on the server's LAN address.

9. Network Listener Analysis

The listening socket was identified as:

127.0.0.1:8080

The associated process was:

python3

with:

PID=5481

The process-owned network descriptor was:

3u IPv4 TCP 127.0.0.1:8080 (LISTEN)

This provided direct attribution between the network socket and the Python process.

10. HTTP Activity

A controlled request was generated using:

curl -I http://127.0.0.1:8080

The server responded:

HTTP/1.0 200 OK
Server: SimpleHTTP/0.6 Python/3.10.12
Date: Fri, 25 Sep 2026 12:17:31 GMT
Content-type: text/html; charset=utf-8
Content-Length: 3267

The HTTP server log recorded:

127.0.0.1 - - [25/Sep/2026 13:17:31] "HEAD / HTTP/1.1" 200 -

This confirmed that the service was reachable locally and successfully processed the HTTP request.

11. Process Timeline

The process start time was recorded as:

13:17:28

The HTTP request occurred at approximately:

13:17:31

The process was later terminated during containment.

The timeline was therefore:

13:17:28
Python HTTP service started
        ↓
13:17:31
Local HTTP HEAD request received
        ↓
13:20+
SOC investigation performed
        ↓
Containment
        ↓
Process terminated
        ↓
Port 8080 verified closed
12. Process and File Descriptor Analysis

The lsof investigation showed that PID 5481 was associated with:

/usr/bin/python3.10

The process working directory was:

/home/mosinmi

The process output was redirected to:

/home/mosinmi/soc-evidence/incident-04/http-server.log

The network descriptor was:

3u IPv4 TCP 127.0.0.1:8080 (LISTEN)

This provided additional evidence connecting the Python process to the network listener.

13. Environment Analysis

The process environment showed:

USER=mosinmi
HOME=/home/mosinmi
PWD=/home/mosinmi
SHELL=/bin/bash
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:/snap/bin

This further confirmed that the network service was launched from the mosinmi user context.

The environment did not indicate an unexpected privileged execution context.

14. Command-and-Control Assessment

A key objective of this investigation was to determine whether the network activity could represent command-and-control behavior.

The investigation did not identify evidence of C2 communication.

The reasons include:

The listener was bound to 127.0.0.1.
The process was owned by the known local user mosinmi.
The executable was the standard system Python interpreter.
The command line explicitly used Python's built-in HTTP server.
The only observed HTTP request was a controlled local request from 127.0.0.1.
No external destination was associated with the test service.
No suspicious outbound connection was identified during the investigation.

Therefore, the activity was classified as controlled and benign.

15. Indicators of Activity
Process
python3
PID
5481
User
mosinmi
Executable
/usr/bin/python3.10
Command
python3 -m http.server 8080 --bind 127.0.0.1
Listening Address
127.0.0.1
Listening Port
8080/TCP
HTTP Request
HEAD /
HTTP Response
200 OK
Evidence Log
/home/mosinmi/soc-evidence/incident-04/http-server.log
16. Detection Logic

A SOC detection strategy for suspicious network services could monitor for:

New listening TCP port
        +
Unexpected process
        +
Unexpected user
        +
Unexpected executable

Additional detection opportunities include:

New listeners on uncommon ports
User processes opening network sockets
Network listeners created shortly after login
Services bound to all interfaces
Processes executing from temporary directories
Python, Perl, Ruby, or shell interpreters opening network sockets
Unexpected outbound connections
Connections to unknown external destinations
Long-lived connections from unusual processes

A high-value correlation could be:

New process
     +
Network socket
     +
Unusual port
     +
External connection

Such a combination should receive additional investigation.

17. Investigation Workflow

The investigation followed this workflow:

Network Baseline
       ↓
Identify Listening Port
       ↓
Identify Process
       ↓
Identify User
       ↓
Inspect Command Line
       ↓
Inspect Network Descriptors
       ↓
Review Application Logs
       ↓
Assess Network Exposure
       ↓
Determine Legitimacy
       ↓
Preserve Evidence
       ↓
Contain
       ↓
Verify

This workflow demonstrates practical SOC host and network investigation techniques.

18. Containment

The controlled network service was terminated using:

kill 5481

The operating system reported that the process was terminated.

A process verification check was then performed.

Result:

PASS: Process terminated
19. Network Containment Verification

After termination, the analyst checked whether port 8080 remained active.

The verification produced:

PASS: Port 8080 no longer listening

A final network listener check produced:

PASS: No listener detected on port 8080

This confirmed that the controlled network service had been successfully contained.

20. Impact Assessment

No production impact occurred.

The network service:

Was intentionally generated.
Was bound to the loopback interface.
Was not configured as a persistent service.
Did not communicate with an external C2 server.
Served only the local test request.
Was terminated after investigation.
Impact

Confidentiality: None

Integrity: None

Availability: None

Network Exposure: Localhost only

External Communication: None identified

Credential Exposure: None

21. Root Cause

The root cause was the intentional execution of Python's built-in HTTP server as part of the SOC laboratory.

The command:

python3 -m http.server 8080 --bind 127.0.0.1

created a temporary local network listener for the purpose of demonstrating network-service detection and process attribution.

There was no attacker or actual compromise associated with the event.

22. Lessons Learned
22.1 Network Ports Must Be Attributed to Processes

A listening port alone does not provide enough information to determine whether activity is malicious.

The analyst should identify:

Process
PID
User
Executable
Command line
Network destination
22.2 Loopback and External Exposure Are Different

The service was bound to:

127.0.0.1

rather than:

0.0.0.0

This significantly changed the exposure of the service.

22.3 Process Ownership Provides Valuable Context

The process was owned by:

mosinmi

and executed from:

/home/mosinmi

This helped establish that the activity originated from the analyst's controlled session.

22.4 Application Logs Support Network Investigations

The HTTP log provided direct evidence that the listener processed a request.

22.5 Network Alerts Require Context

A new listening port can be suspicious, but the analyst must establish what created it and why before classifying it as malicious.

23. Recommended SOC Monitoring

A production SOC should monitor:

Listening Ports

Monitor for new TCP and UDP listeners.

Process-to-Socket Relationships

Correlate network connections with:

PID
Process name
Executable path
User
Parent process
Unusual Interpreters

Investigate interpreters such as:

python
perl
ruby
bash
sh

when they create network listeners or unexpected outbound connections.

External Connections

Monitor for connections to:

Unknown IP addresses
Rare destinations
Newly observed domains
Suspicious geographic locations
Known malicious infrastructure
Long-Lived Connections

Investigate unexpected persistent connections that remain established for extended periods.

24. Evidence Files

The following evidence was preserved:

~/soc-evidence/incident-04/

Files include:

incident-04/
├── http-server.log
├── http-server.log-copy
├── http-server.pid
├── network-listener.txt
├── network-process.txt
├── process-command.txt
├── process-details.txt
└── process-timeline.txt
Evidence Description

http-server.log

Contains the HTTP server request generated during the controlled test.

http-server.log-copy

Preserved copy of the HTTP activity used during the investigation.

http-server.pid

Contains the PID associated with the controlled service.

network-listener.txt

Contains evidence identifying TCP port 8080 and the associated Python process.

network-process.txt

Contains process-to-network socket attribution.

process-command.txt

Contains the command line used to launch the service.

process-details.txt

Contains process identity and ownership information.

process-timeline.txt

Contains the process start time and execution details.

25. Final Containment Status

The final containment verification produced:

PASS: Process terminated
PASS: Port 8080 no longer listening
PASS: No listener detected on port 8080

This confirms that the network service was successfully terminated and that the listening socket was removed.

26. Incident Conclusion

The investigation identified a temporary Python HTTP service listening on:

127.0.0.1:8080

The service was attributed to:

PID 5481
User mosinmi
Executable /usr/bin/python3.10

The process command line was:

python3 -m http.server 8080 --bind 127.0.0.1

The investigation demonstrated the complete process-to-network attribution chain:

TCP 8080
   ↓
Python process
   ↓
PID 5481
   ↓
User mosinmi
   ↓
Known command

The service was bound exclusively to the loopback interface and only processed a controlled local HTTP request.

No evidence of external command-and-control communication, malicious network infrastructure, unauthorized access, or compromise was identified.

The service was terminated during containment and verification confirmed that the process was no longer running and TCP port 8080 was no longer listening.

The incident is therefore classified as:

CONTROLLED / BENIGN NETWORK ACTIVITY

and is considered:

CLOSED — NETWORK SERVICE SUCCESSFULLY CONTAINED

This exercise demonstrates the ability to identify suspicious network listeners, attribute sockets to processes and users, investigate application-level activity, preserve evidence, perform containment, and verify that the network service has been removed.
