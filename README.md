# Wazuh SIEM Home Lab

Hands-on blue-team lab for endpoint monitoring, Windows security-event analysis, threat hunting, security configuration assessment, file-integrity monitoring, alert investigation, and multi-agent monitoring.

## Current environment

- Wazuh manager/dashboard: `10.0.0.105`
- First endpoint: `Jeremy-Windows-PC`
- Wazuh Agent ID: `001`
- Observed endpoint IP: `192.168.1.140`
- Agent-to-manager connection confirmed over TCP/1514

```text
                    HOME LAB

+--------------------------+       +--------------------------+
| Wazuh Manager / SIEM     |<------| Jeremy-Windows-PC       |
| 10.0.0.105               | 1514  | Wazuh Agent 001         |
| Dashboard / Indexer      |       | Windows endpoint        |
+--------------------------+       +--------------------------+
             ^
             |
             | planned agents
      +------+------+----------------+
      |             |                |
  Linux VM     Server VM       Other endpoints
```

## Completed work

- [x] Deployed Wazuh SIEM environment
- [x] Installed Wazuh Windows agent
- [x] Registered Windows endpoint with the manager
- [x] Confirmed the endpoint connects successfully
- [x] Verified Windows Security events in Threat Hunting
- [x] Reviewed CIS Windows 11 SCA findings
- [x] Generated Windows account lifecycle events
- [x] Generated a failed authentication alert
- [x] Tested Microsoft Defender using the harmless EICAR test string
- [x] Troubleshot a broken `ossec.conf` XML configuration
- [x] Restored the Wazuh service
- [x] Confirmed real-time File Integrity Monitoring started

## Detection testing

### Windows account activity

A temporary local account was created and deleted to generate real Windows Security audit events.

```powershell
net user WazuhTestUser <temporary-lab-password> /add
net user WazuhTestUser /delete
```

Wazuh detected events including account creation/change, group changes, and account deletion. A reviewed deletion event corresponded to Windows Security Event ID `4726`.

### Failed authentication

A controlled failed SMB authentication attempt was generated against localhost:

```powershell
net use \\localhost\IPC$ /user:FakeWazuhUser WrongPassword123!
```

Windows returned expected error `1326`. Wazuh recorded:

```text
Logon Failure - Unknown user or bad password
Rule ID: 60122
Rule level: 5
Windows Event ID: 4625
Target user: FakeWazuhUser
Logon type: 3
Authentication package: NTLM
Logon process: NtLmSsp
Source: ::1 (IPv6 localhost)
```

### Microsoft Defender / EICAR

Defender was verified first:

```powershell
Get-MpComputerStatus | Select-Object AntivirusEnabled,RealTimeProtectionEnabled
```

Both antivirus and real-time protection were enabled. The harmless EICAR antivirus test string was then used to verify Defender detection; Windows Security immediately reported a threat. EICAR is a standard antivirus test string, not real malware.

## Security Configuration Assessment

The Windows agent ran Wazuh Security Configuration Assessment against the CIS Microsoft Windows 11 Enterprise benchmark. This generated hundreds of initial events, explaining the large spike visible after agent startup. Those events were primarily configuration checks rather than hundreds of attacks.

Example agent log activity:

```text
Starting evaluation of policy: cis_win11_enterprise.yml
Security Configuration Assessment scan finished
```

## Troubleshooting performed

The Wazuh service stopped after an invalid XML configuration was introduced. The endpoint log reported:

```text
ERROR: (1226): Error reading XML file 'ossec.conf': (line 0).
```

Configuration file:

```text
C:\Program Files (x86)\ossec-agent\ossec.conf
```

It was opened, corrected, saved, and the Wazuh service was restarted:

```powershell
notepad "C:\Program Files (x86)\ossec-agent\ossec.conf"
NET START Wazuh
```

The service then started successfully. Logs were verified with:

```powershell
Get-Content "C:\Program Files (x86)\ossec-agent\ossec.log" -Tail 30
```

Healthy messages included:

```text
Connected to the server ([10.0.0.105]:1514/tcp)
Agent is now online. Process unlocked, continuing...
File integrity monitoring scan ended.
Real-time file integrity monitoring started.
```

## Useful endpoint commands

```powershell
Get-Service -Name WazuhSvc
NET START Wazuh
Restart-Service -Name WazuhSvc
Get-Content "C:\Program Files (x86)\ossec-agent\ossec.log" -Tail 30
Get-MpComputerStatus | Select-Object AntivirusEnabled,RealTimeProtectionEnabled
```

## Skills practiced

Windows Security Event Logs, SIEM architecture, endpoint-agent deployment, authentication monitoring, account-management auditing, Wazuh rules and severity levels, Threat Hunting, CIS/SCA findings, Microsoft Defender validation, File Integrity Monitoring, Windows services, XML troubleshooting, endpoint log analysis, and distinguishing normal security telemetry from suspicious events.

## Next phase: add agents

The next goal is to turn this into a multi-endpoint SIEM lab.

- [ ] Add a Linux/Ubuntu Wazuh agent
- [ ] Add another Windows/server endpoint
- [ ] Give each endpoint a descriptive hostname/agent name
- [ ] Confirm each agent independently reports to the manager
- [ ] Generate authentication events on multiple hosts
- [ ] Compare alerts by `agent.name`
- [ ] Test file-integrity changes on multiple endpoints
- [ ] Create dashboard views for authentication and endpoint activity
- [ ] Add custom Wazuh detection rules
- [ ] Practice alert triage and document investigation notes
- [ ] Map selected detections to MITRE ATT&CK
- [ ] Add screenshots and a final network diagram

For every new endpoint: prepare the VM/device, assign a hostname, install the Wazuh agent, point it to `10.0.0.105`, start the service, confirm it appears in Wazuh, inspect its logs, generate a harmless test event, locate the event in Threat Hunting, and document the result here.

## Security note

This is a controlled home-lab project. Do not commit passwords, API keys, enrollment secrets, tokens, public IP addresses, or other credentials. Test accounts should be temporary and removed after testing.

## Purpose

The goal is to develop practical SOC/blue-team skills by building, testing, troubleshooting, monitoring, and expanding a working SIEM environment.