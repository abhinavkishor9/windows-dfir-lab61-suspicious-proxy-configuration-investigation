# Timeline — Lab 61 Suspicious Proxy Configuration Investigation

## Timeline Purpose

This timeline documents the baseline proxy configuration, controlled Registry change, telemetry review, troubleshooting, and configuration cleanup performed during Lab 61.

Only timestamps present in the supplied evidence are included explicitly.

## Investigation Timeline

| Time | Source | Activity | Result |
|---|---|---|---|
| Initial baseline | Registry | User Internet Settings inspected | Proxy initially disabled |
| Initial baseline | WinHTTP | `netsh winhttp show proxy` | Direct access |
| Initial baseline | Environment | Proxy variables searched | None returned |
| Initial baseline | Registry | Machine Internet Settings inspected | No meaningful proxy values |
| 06:50:24 | Sysmon Event ID 1 | PowerShell process observed | Process telemetry available |
| 07:18:44 | Registry / PowerShell | Proxy state checked after simulation | `ProxyEnable`/proxy values inspected |
| After change | Registry | Controlled proxy value written | Loopback proxy simulation attempted |
| After change | PowerShell | Proxy configuration rechecked | `127.0.0.1:8080` appeared in the queried output |
| After change | PowerShell | Event ID 4104 searched | No relevant result |
| After change | Security | Event ID 4688 searched | No relevant result |
| After change | Sysmon Event ID 3 | Network events reviewed | Network telemetry available |
| Cleanup | Registry | `ProxyServer` cleared | Loopback proxy string removed |
| Final review | Registry | Final state requires verification | No final ProxyEnable=0 write shown in supplied evidence |

## Initial Baseline

### User Internet Settings

The initial query showed:

```text
ProxyEnable = 0
```

No proxy server value was shown.

This established the starting configuration.

### WinHTTP

The initial WinHTTP query showed:

```text
Direct access (no proxy server).
```

### Environment Variables

The environment-variable search returned no proxy-related variables.

### AutoConfigURL

No AutoConfigURL value was identified.

### Machine Internet Settings

The checked machine-level Internet Settings properties did not show a configured proxy.

---

## Controlled Proxy Simulation

The investigation used:

```text
127.0.0.1:8080
```

as a harmless loopback proxy value.

The intended model was:

```text
ProxyEnable = 1
ProxyServer = 127.0.0.1:8080
```

This was performed only as a configuration simulation.

No real proxy server was created.

---

## Configuration Error

During the simulation, the proxy string was incorrectly written into:

```text
ProxyEnable
```

instead of:

```text
ProxyServer
```

The resulting query showed:

```text
ProxyEnable = 127.0.0.1:8080
```

This was recorded as a laboratory troubleshooting issue.

---

## 07:18:44 — Proxy State Check

A timestamp was recorded:

```text
25 August 2026 07:18:44
```

The proxy configuration was examined at this point.

The extracted evidence showed:

```text
ProxyEnable = 1
```

in one configuration check and later showed:

```text
127.0.0.1:8080
```

appearing in the ProxyEnable field after the configuration mistake.

---

## Process Telemetry

Sysmon Event ID 1 showed a PowerShell process event at:

```text
25-08-2026 06:50:24
```

This provided supporting process context.

The supplied evidence does not establish that this event directly caused the proxy configuration change.

---

## PowerShell Telemetry

The investigation searched PowerShell Event ID 4104 for:

```text
ProxyEnable
ProxyServer
127.0.0.1:8080
Internet Settings
```

No relevant event was returned.

Therefore, no direct Script Block Logging evidence of the proxy change was established.

---

## Security Event ID 4688

Security Event ID 4688 was queried for:

```text
powershell.exe
```

No relevant result was returned.

Therefore, no Security process-creation evidence directly attributable to the proxy change was established.

---

## Network Telemetry

Sysmon Event ID 3 was available and returned numerous network events.

The supplied evidence included events throughout the investigation timeframe, including:

```text
06:39
06:40
06:41
06:42
06:43
06:46
06:47
06:49
06:54
06:55
06:58
07:00
07:03
07:06
07:09
07:13
07:17
07:19
07:20
07:24
07:30
07:31
07:34
07:35
07:38
07:39
07:40
07:41
07:42
```

The evidence did not establish that these network events resulted from traffic being redirected through the controlled proxy.

---

## Cleanup

The following operation was performed:

```powershell
Set-ItemProperty `
-Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Internet Settings" `
-Name ProxyServer `
-Value ""
```

This removed the loopback proxy string from `ProxyServer`.

However, the supplied evidence does not show a final command setting:

```text
ProxyEnable = 0
```

Therefore, final restoration should be verified separately.

---

## Final Evidence Chain

```text
Initial Proxy Baseline
        |
        v
WinHTTP Baseline
        |
        v
Environment Baseline
        |
        v
Controlled Registry Modification
        |
        v
PowerShell / Process Telemetry Review
        |
        v
Network Telemetry Review
        |
        v
Configuration Cleanup
        |
        v
Final State Verification
```

## Final Assessment

The investigation established a controlled proxy configuration change attempt and documented an important Registry-property error.

The available process and network telemetry did not establish malicious attribution or actual proxy-based traffic redirection.

The investigation therefore supports:

```text
Proxy Configuration Change Attempt
```

but does not support:

```text
Confirmed Malicious Proxy Use
```

## Investigation Conclusion

The lab demonstrated why proxy investigations require both **configuration evidence** and **behavioral evidence**.

The configuration state can show that a proxy value changed, but a stronger security conclusion requires additional evidence linking:

```text
Configuration Change
    +
Process
    +
User
    +
Timestamp
    +
Actual Proxy Use
    +
Network Behavior
```

The final configuration should be verified against the original baseline before the investigation is considered fully closed.
