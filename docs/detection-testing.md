# Detection Testing

These tests were performed in a controlled home-lab environment to verify that endpoint actions generate useful telemetry in Wazuh.

## Test 1: Account creation and deletion

### Goal
Confirm that Windows account-management activity reaches Wazuh.

### Action

```powershell
net user WazuhTestUser <temporary-lab-password> /add
net user WazuhTestUser /delete
```

### Observed Wazuh activity

- User account enabled or created
- User account changed
- Users Group Changed
- Domain Users Group Changed
- User account disabled or deleted

A reviewed account deletion event used Windows Security Event ID `4726`.

### Result
**PASS** — Windows account lifecycle telemetry was visible in Wazuh.

---

## Test 2: Failed authentication

### Goal
Confirm that Wazuh detects an invalid network logon attempt.

### Action

```powershell
net use \\localhost\IPC$ /user:FakeWazuhUser WrongPassword123!
```

### Endpoint result

```text
System error 1326 has occurred.
The user name or password is incorrect.
```

### Observed Wazuh alert

```text
Rule description: Logon Failure - Unknown user or bad password
Rule ID: 60122
Rule level: 5
Windows Event ID: 4625
Target user: FakeWazuhUser
Logon type: 3
Authentication package: NTLM
Logon process: NtLmSsp
IP address: ::1
```

`::1` is IPv6 localhost, which was expected because the test targeted the same computer.

### Result
**PASS** — the controlled authentication failure was visible and searchable in Wazuh.

---

## Test 3: Microsoft Defender detection

### Goal
Verify that Microsoft Defender real-time protection is enabled and reacts to a standard antivirus test artifact.

### Defender check

```powershell
Get-MpComputerStatus | Select-Object AntivirusEnabled,RealTimeProtectionEnabled
```

Observed result:

```text
AntivirusEnabled             True
RealTimeProtectionEnabled    True
```

### Action
The industry-standard harmless EICAR antivirus test string was written to a test file.

### Endpoint result
Windows Security displayed a `Threats found` notification from Microsoft Defender Antivirus.

### Result
**PASS** — Defender's endpoint detection worked as expected.

---

## Future detection tests

- Linux failed SSH authentication
- Linux sudo activity
- File Integrity Monitoring changes on Windows
- File Integrity Monitoring changes on Linux
- Multiple failed logons for correlation practice
- Custom Wazuh rule testing
- Cross-agent event comparison
- MITRE ATT&CK mapping for selected detections