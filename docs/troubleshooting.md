# Wazuh Troubleshooting Notes

## Windows agent service stopped after configuration edit

### Symptom

The Windows Wazuh service would not remain running and the endpoint stopped sending new events.

```powershell
Get-Service -Name WazuhSvc
```

showed the service as stopped.

### Investigation

The agent log was inspected:

```powershell
Get-Content "C:\Program Files (x86)\ossec-agent\ossec.log" -Tail 30
```

The important error was:

```text
ERROR: (1226): Error reading XML file 'ossec.conf': (line 0).
```

This indicated a configuration/XML problem rather than a network or dashboard problem.

### Fix

Open the agent configuration:

```powershell
notepad "C:\Program Files (x86)\ossec-agent\ossec.conf"
```

Correct the XML, save it, and start the service:

```powershell
NET START Wazuh
```

### Verification

Check the service and log again:

```powershell
Get-Service -Name WazuhSvc
Get-Content "C:\Program Files (x86)\ossec-agent\ossec.log" -Tail 30
```

Successful recovery produced messages including:

```text
Connected to the server ([10.0.0.105]:1514/tcp)
Agent is now online. Process unlocked, continuing...
File integrity monitoring scan ended.
Real-time file integrity monitoring started.
```

## Lesson learned

When Wazuh stops showing new events, troubleshoot from the endpoint outward:

1. Check whether `WazuhSvc` is running.
2. Read `ossec.log`.
3. Validate recent `ossec.conf` changes.
4. Confirm the manager connection.
5. Only then troubleshoot dashboard filters/time ranges.

This prevents wasting time treating an endpoint configuration failure as a SIEM/dashboard failure.