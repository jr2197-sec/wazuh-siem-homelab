# Adding Wazuh Agents

This document tracks the next phase of the home lab: expanding from one Windows endpoint into a multi-agent SIEM environment.

## Current agent

| Agent ID | Name | Platform | Status |
|---|---|---|---|
| 001 | Jeremy-Windows-PC | Windows | Connected/tested |

## Planned agents

Suggested next systems:

1. Ubuntu/Linux VM — practice Linux authentication, sudo, SSH, package and file events.
2. Additional Windows VM/server — compare Windows telemetry between multiple hosts.
3. Other home-lab servers — add systems that make sense as the environment grows.

## Standard onboarding process

For each endpoint:

1. Assign a descriptive hostname.
2. Verify the endpoint can reach the Wazuh manager at `10.0.0.105`.
3. Install the correct Wazuh agent package.
4. Configure the manager address.
5. Start/enable the agent service.
6. Confirm the endpoint appears as a unique Wazuh agent.
7. Inspect local Wazuh logs for connection errors.
8. Generate a harmless test event.
9. Find the event in Threat Hunting.
10. Record the test and results in the agent inventory below.

## Agent inventory

Update this table whenever another system is added.

| Agent ID | Agent name | OS | Purpose | Test performed | Result |
|---|---|---|---|---|---|
| 001 | Jeremy-Windows-PC | Windows | Primary workstation monitoring | Account lifecycle + failed logon + Defender test | Successful |
| TBD | TBD-Linux | Linux | Linux monitoring | Failed SSH/sudo + file change | Planned |
| TBD | TBD-Windows | Windows | Multi-Windows comparison | Failed logon + file change | Planned |

## Tests for a Linux agent

After the Linux endpoint is enrolled, perform controlled tests such as:

```bash
# Confirm agent service
sudo systemctl status wazuh-agent

# Generate a harmless authentication-related event
sudo -k
sudo whoami

# Generate a file change for FIM testing
mkdir -p ~/wazuh-lab
printf 'Wazuh FIM test\n' > ~/wazuh-lab/fim-test.txt
```

Only monitor directories intentionally configured for the lab.

## Tests for another Windows agent

Useful controlled tests include:

```powershell
# Check agent
Get-Service -Name WazuhSvc

# Generate a temporary test account
net user WazuhTestUser2 <temporary-lab-password> /add
net user WazuhTestUser2 /delete

# Generate an expected failed authentication event
net use \\localhost\IPC$ /user:FakeWazuhUser2 WrongPassword123!
```

Remove temporary test accounts after testing.

## What to compare across agents

Once multiple agents are reporting, investigate:

- Alert counts by `agent.name`
- Authentication failures by endpoint
- Windows vs Linux event fields
- Rule IDs and severity levels
- File-integrity alerts by host
- SCA findings by operating system
- Timeline correlation between systems

## Success criteria

The multi-agent phase is complete when at least two different endpoints are independently reporting to Wazuh, controlled test events can be located for each host, and the dashboard can distinguish activity by agent.