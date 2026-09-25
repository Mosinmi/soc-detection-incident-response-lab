# Incident Report — Incident #3

## 1. Incident Summary

**Incident ID:** INC-003 
**Incident Title:** Suspicious Systemd Persistence Mechanism 
**Date:** September 25, 2026 
**Host:** Ubuntu Server 
**Host IP:** 192.168.0.162 
**Analyst:** Mosinmi 
**Severity:** Medium 
**Classification:** Controlled / Benign Persistence Activity 
**Status:** Closed — Persistence Successfully Removed

This incident documents the detection, investigation, containment, and removal of a controlled systemd persistence mechanism on the Ubuntu Server.

The persistence mechanism was intentionally created as part of a controlled SOC detection and incident response laboratory. The purpose of the exercise was to simulate a suspicious persistence technique, provide the SOC analyst with realistic host-based evidence, investigate the mechanism, and demonstrate containment and remediation.

A custom systemd service named `soc-lab-test.service` was created under:

```text
/etc/systemd/system/soc-lab-test.service

The service was configured to execute a command with root privileges and was enabled through the multi-user.target systemd target.

When executed, the service created a marker file:

/tmp/soc-lab-test-marker

The investigation confirmed that the service was enabled, had a persistence symlink, executed successfully, and generated a corresponding systemd journal entry.

The persistence mechanism was subsequently disabled and removed. Verification confirmed that the service registration, persistence symlink, service file, and test artifact were all successfully removed.

2. Incident Classification

Category: Persistence / Systemd Service

Classification: Controlled / Benign Activity

Severity: Medium

Detection Source: systemd service configuration and journal logs

Affected Host: Ubuntu Server

Persistence Mechanism: Custom systemd service

Privileged Context: Root

Status: Closed

3. Environment

The investigation was conducted on the Ubuntu Server used as the monitored endpoint in the SOC laboratory.

System Information
Operating System: Ubuntu Server 22.04.5 LTS
Host IP: 192.168.0.162
Local Administrative User: mosinmi
Root User: root
System Initialization Framework: systemd
Laboratory Context

The system is part of a controlled cybersecurity laboratory used to practice:

Security monitoring
Host-based detection
Persistence investigation
Incident response
Evidence preservation
Containment
Remediation

The persistence mechanism used in this exercise was intentionally created and contained by the analyst.

4. Detection

The incident began with the creation of a controlled systemd service designed to simulate a suspicious persistence mechanism.

The service was created at:

/etc/systemd/system/soc-lab-test.service

The service configuration was:

[Unit]
Description=SOC Lab Controlled Test Service
After=network.target

[Service]
Type=oneshot
ExecStart=/usr/bin/touch /tmp/soc-lab-test-marker
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target

The service was enabled and started.

The system reported:

Created symlink /etc/systemd/system/multi-user.target.wants/soc-lab-test.service

This demonstrated that the service had been configured for persistence through the multi-user.target startup mechanism.

5. Initial Detection Evidence

The service status showed:

soc-lab-test.service - SOC Lab Controlled Test Service
Loaded: loaded (/etc/systemd/system/soc-lab-test.service; enabled; vendor preset: enabled)
Active: active (exited)

The service executed:

ExecStart=/usr/bin/touch /tmp/soc-lab-test-marker

with:

code=exited, status=0/SUCCESS

This confirmed that the service successfully executed its configured command.

The service was also confirmed to be enabled:

enabled
6. Persistence Mechanism Analysis

The systemd persistence mechanism was identified through the following symbolic link:

/etc/systemd/system/multi-user.target.wants/soc-lab-test.service

The link pointed to:

/etc/systemd/system/soc-lab-test.service

The observed link was:

lrwxrwxrwx 1 root root 40 Sep 25 12:40 /etc/systemd/system/multi-user.target.wants/soc-lab-test.service -> /etc/systemd/system/soc-lab-test.service

This demonstrated that the service had been registered with the multi-user.target startup target.

From a SOC perspective, unexpected services or startup links under systemd directories can represent persistence and should be investigated.

7. Service File Analysis

The service file was located at:

/etc/systemd/system/soc-lab-test.service

The file contained:

[Unit]
Description=SOC Lab Controlled Test Service
After=network.target

[Service]
Type=oneshot
ExecStart=/usr/bin/touch /tmp/soc-lab-test-marker
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target

The service was owned by:

root:root

with permissions:

0644

The file metadata showed that it was created at approximately:

12:40:29

and modified at approximately:

12:40:29

The close timing between creation and execution provided a clear laboratory timeline for the persistence test.

8. Execution Analysis

The service executed successfully at approximately:

12:40:32

The systemd status reported:

Process: 5237 ExecStart=/usr/bin/touch /tmp/soc-lab-test-marker
(code=exited, status=0/SUCCESS)

The service used:

Type=oneshot

Therefore, the process executed the configured command and exited.

The process was not present when the later process check was performed.

This behavior was expected because the service was designed as a one-shot service rather than a continuously running daemon.

9. Artifact Analysis

The service created the following artifact:

/tmp/soc-lab-test-marker

The artifact metadata showed:

Size: 0
Uid: root
Gid: root

The file was created at approximately:

12:40:32

The creation time corresponded closely with the systemd service execution.

This provided evidence linking the persistence mechanism to the generated artifact.

10. Systemd Journal Analysis

The systemd journal contained:

Sep 25 12:40:32 Ubuntu systemd[1]: Starting SOC Lab Controlled Test Service...
Sep 25 12:40:32 Ubuntu systemd[1]: Finished SOC Lab Controlled Test Service.

These entries confirmed that systemd started and completed execution of the service.

The journal did not indicate an execution failure.

11. Incident Timeline
12:40:29

The controlled systemd service was created.

/etc/systemd/system/soc-lab-test.service
12:40:30

The service file metadata indicated its creation and modification around this time.

12:40:32

The service executed:

/usr/bin/touch /tmp/soc-lab-test-marker

The command completed successfully.

12:40:32

The test marker was created:

/tmp/soc-lab-test-marker
12:40:32

Systemd recorded:

Starting SOC Lab Controlled Test Service...
Finished SOC Lab Controlled Test Service.
Investigation Phase

The analyst examined:

Service configuration
Service status
Enablement state
Persistence symlink
Service metadata
Artifact metadata
Systemd journal
Running processes
Containment Phase

The service was:

Stopped
Disabled
Removed
Persistence symlink removed
Systemd configuration reloaded
Test artifact removed
Final Verification

All persistence indicators were confirmed removed.

12. Investigation Findings

The investigation established the following:

Finding 1 — Custom Systemd Service

A custom service existed at:

/etc/systemd/system/soc-lab-test.service
Finding 2 — Service Was Enabled

The service returned:

enabled
Finding 3 — Persistence Symlink Existed

A symbolic link existed under:

/etc/systemd/system/multi-user.target.wants/
Finding 4 — Root Execution

The service executed its command under the system service context and created a root-owned artifact.

Finding 5 — Successful Execution

The service completed with:

status=0/SUCCESS
Finding 6 — Artifact Creation

The service created:

/tmp/soc-lab-test-marker
Finding 7 — Controlled Activity

The service was intentionally created as part of the SOC laboratory.

Therefore, the observed persistence mechanism was not evidence of an actual compromise.

13. Indicators of Activity
Service Name
soc-lab-test.service
Service File
/etc/systemd/system/soc-lab-test.service
Persistence Location
/etc/systemd/system/multi-user.target.wants/soc-lab-test.service
Executed Command
/usr/bin/touch /tmp/soc-lab-test-marker
Created Artifact
/tmp/soc-lab-test-marker
Execution Result
status=0/SUCCESS
Journal Source
systemd[1]
14. Detection Logic

A SOC detection rule can monitor for newly created or modified systemd services.

Potential monitoring locations include:

/etc/systemd/system/
/usr/lib/systemd/system/
/etc/systemd/system/*.wants/

Useful detection concepts include:

New systemd service created
New service enabled
New symbolic link created inside *.wants directories
Unexpected service executing as root
New service created shortly before execution

A stronger detection strategy would correlate multiple indicators:

New systemd service
        +
Service enabled
        +
Root execution
        +
Unexpected executable/command
        +
New filesystem artifact

This correlation can increase confidence that a persistence mechanism requires investigation.

15. SOC Investigation Workflow

The investigation followed the following workflow:

Detection
   ↓
Identify suspicious service
   ↓
Inspect service configuration
   ↓
Check enablement
   ↓
Inspect persistence symlink
   ↓
Review execution status
   ↓
Review systemd journal
   ↓
Identify generated artifacts
   ↓
Determine legitimacy
   ↓
Contain
   ↓
Remove persistence
   ↓
Verify removal

This workflow demonstrates a practical host-based SOC investigation process.

16. Containment

Containment was performed by stopping and disabling the service.

The service was disabled using:

sudo systemctl disable soc-lab-test.service

The system reported:

Removed /etc/systemd/system/multi-user.target.wants/soc-lab-test.service.

The service file was then removed:

/etc/systemd/system/soc-lab-test.service

The persistence symlink was also removed.

Systemd was reloaded to ensure that the removed service configuration was no longer registered.

17. Remediation

The controlled persistence mechanism was completely removed.

The following components were deleted:

/etc/systemd/system/soc-lab-test.service
/etc/systemd/system/multi-user.target.wants/soc-lab-test.service
/tmp/soc-lab-test-marker

Systemd was reloaded after the removal.

18. Containment Verification

The following checks were performed after remediation.

Service Registration

Result:

PASS: Service no longer registered
Persistence Symlink

Result:

PASS: Persistence symlink removed
Service File

Result:

PASS: Service file removed
Test Artifact

Result:

PASS: Test artifact removed

All four containment checks passed.

19. Impact Assessment

No production impact occurred.

The persistence mechanism was intentionally created within the controlled laboratory environment.

The service performed only the following action:

/usr/bin/touch /tmp/soc-lab-test-marker

No destructive operation, credential collection, network exploitation, or unauthorized system modification was performed.

The exercise therefore resulted in:

Confidentiality Impact: None

Integrity Impact: None beyond controlled laboratory changes

Availability Impact: None

Data Loss: None

Credential Exposure: None

20. Root Cause

The root cause of the observed persistence mechanism was the intentional creation of a custom systemd service for SOC laboratory testing.

The service was configured to execute automatically through:

multi-user.target

The resulting symbolic link under the target's .wants directory provided the persistence mechanism.

Because the activity was deliberately generated by the analyst, there was no actual attacker or compromise involved.

21. Lessons Learned
21.1 Persistence Can Be Detected Through System Configuration

Systemd configuration provides valuable host-based evidence for identifying persistence mechanisms.

21.2 Enabled Services Require Investigation

An unfamiliar service marked as enabled should be investigated because it may execute automatically during system startup.

21.3 Configuration and Execution Should Be Correlated

The service configuration alone is not enough.

The analyst should also examine:

Enablement status
Service execution
Journal logs
Executed commands
File artifacts
Ownership
Timestamps
21.4 One-Shot Services May Not Appear in Process Listings

The process check returned no running process because the service used:

Type=oneshot

The command executed and terminated successfully.

This demonstrates why process listings should not be the only source of evidence during persistence investigations.

21.5 Containment Requires Verification

Simply deleting a service file is not sufficient.

The investigation verified:

Service registration
Persistence symlink
Service file
Generated artifact

All were successfully removed.

22. Recommended SOC Monitoring

A production SOC should consider monitoring the following:

Systemd Service Creation

Alert on new files under:

/etc/systemd/system/
Service Enablement

Monitor changes involving:

*.wants/

directories.

Suspicious Root Services

Investigate services that:

Run as root
Execute unusual commands
Launch shell interpreters
Execute from temporary directories
Execute newly created binaries
Make unexpected network connections
File Creation Correlation

Correlate new systemd services with filesystem changes.

Example:

New systemd service
        ↓
Service enabled
        ↓
Service executed
        ↓
New file created
Administrative Attribution

Record which account created or modified the service whenever possible.

23. MITRE ATT&CK Context

Systemd-based persistence is relevant to the broader MITRE ATT&CK persistence category because attackers can abuse legitimate service-management mechanisms to maintain execution on Linux systems.

However, this laboratory event was intentionally generated and therefore is not classified as confirmed malicious ATT&CK activity.

The exercise demonstrates the defensive capability to identify and investigate potential persistence through system services.

24. Evidence Files

Evidence for this incident should be preserved under:

~/soc-evidence/incident-03/

Recommended evidence files include:

incident-03/
├── service-details.txt
├── service-status.txt
├── persistence-symlink.txt
├── service-metadata.txt
├── artifact-metadata.txt
├── systemd-journal.txt
└── incident-report.md

The evidence should document the state of the persistence mechanism before containment and the verification results after remediation.

25. Final Containment Status

The final containment checks produced:

PASS: Service no longer registered
PASS: Persistence symlink removed
PASS: Service file removed
PASS: Test artifact removed

This confirms that the controlled persistence mechanism was successfully eliminated.

26. Incident Conclusion

The investigation identified a custom systemd service named:

soc-lab-test.service

The service was enabled through the systemd multi-user.target and executed a root-level command that created a test artifact.

The analyst investigated the service configuration, enablement state, persistence symlink, service metadata, systemd journal, and generated artifact.

The evidence confirmed successful execution of the persistence mechanism.

Because the service was intentionally created as part of the controlled SOC laboratory, the event was classified as:

CONTROLLED / BENIGN PERSISTENCE ACTIVITY

The persistence mechanism was subsequently contained and removed.

Final verification confirmed:

Service registration: REMOVED
Persistence symlink: REMOVED
Service file: REMOVED
Test artifact: REMOVED

No unauthorized compromise or malicious activity occurred.

The incident is therefore classified as:

CLOSED — PERSISTENCE SUCCESSFULLY REMOVED

This exercise demonstrates the ability to detect, investigate, contain, remediate, and verify the removal of a Linux persistence mechanism using systemd and host-based evidence.
