# Windows DFIR Lab 61 — Suspicious Proxy Configuration Investigation

## Overview

This lab investigates suspicious proxy configuration changes on a Windows endpoint. Proxy settings can be legitimate in enterprise environments, but unexpected changes may redirect application traffic, interfere with normal network controls, or indicate unauthorized system modification.

The investigation began by establishing the endpoint's existing proxy configuration. Windows Internet Settings, WinHTTP, environment variables, and machine-level Internet Settings were checked before any controlled modification was attempted.

A loopback proxy value using `127.0.0.1:8080` was then introduced as a harmless simulation. During the exercise, a configuration mistake occurred in which the proxy string was written into the `ProxyEnable` value instead of the `ProxyServer` value. This was documented as part of the lab rather than treated as malicious behavior.

PowerShell Event ID 4104 and Security Event ID 4688 searches did not return relevant results. Sysmon Event ID 1 and Event ID 3 were available, but the available evidence did not establish a specific malicious process or network connection responsible for the proxy configuration change.

## Investigation Objectives

- Establish the endpoint's proxy configuration before any controlled change.
- Compare user-level, machine-level, WinHTTP, and environment-based proxy settings.
- Determine which Windows configuration area was modified during the simulation.
- Document the exact values written to the proxy configuration.
- Correlate configuration changes with process and PowerShell telemetry.
- Examine surrounding network telemetry without assuming causation.
- Identify configuration mistakes and distinguish them from suspicious activity.
- Determine whether the proxy change created evidence of actual traffic redirection.
- Document telemetry sources that did not provide relevant evidence.
- Restore and verify the configuration where supported by the recorded baseline.
- Produce an evidence-based assessment of the proxy change.

## Investigation Scenario

A Windows workstation is suspected of having its proxy configuration modified without an obvious administrative reason. The analyst needs to determine the original proxy state, identify the setting that changed, establish when the change occurred, and investigate whether any process or network activity supports malicious intent.

The investigation focuses on the relationship between:

- User Internet Settings
- WinHTTP configuration
- Environment variables
- Registry state
- Process creation
- PowerShell activity
- Network connections
- Configuration restoration

A controlled loopback proxy value is used so that no external proxy infrastructure is required.

## Lab Environment

| Component | Details |
|---|---|
| Operating System | Windows |
| Investigation Type | Windows DFIR / Network Configuration |
| User Proxy Location | `HKCU:\Software\Microsoft\Windows\CurrentVersion\Internet Settings` |
| Machine Proxy Location | `HKLM:\Software\Microsoft\Windows\CurrentVersion\Internet Settings` |
| WinHTTP | `netsh winhttp show proxy` |
| Controlled Proxy | `127.0.0.1:8080` |
| Sysmon Event ID 1 | Available |
| Sysmon Event ID 3 | Available |
| PowerShell Event ID 4104 | No relevant result |
| Security Event ID 4688 | No relevant result |

## Initial Proxy Baseline

The initial user-level Internet Settings query showed:

```text
ProxyEnable : 0
```

with no proxy server value shown.

The machine-level Internet Settings query did not return a configured proxy value.

The WinHTTP configuration showed:

```text
Direct access (no proxy server).
```

No proxy-related environment variables were returned by the environment-variable search.

These observations established the pre-change baseline.

## Controlled Configuration Change

The lab simulated a suspicious user proxy configuration using:

```text
127.0.0.1:8080
```

The intended configuration model was:

```text
ProxyEnable = 1
ProxyServer = 127.0.0.1:8080
```

However, during the lab, the proxy string was accidentally written into `ProxyEnable` instead of `ProxyServer`.

This produced an invalid or unintended configuration state and became an important troubleshooting finding.

## Registry Evidence

The relevant user configuration location was:

```text
HKCU:\Software\Microsoft\Windows\CurrentVersion\Internet Settings
```

The investigation examined:

```text
ProxyEnable
ProxyServer
ProxyOverride
AutoConfigURL
```

The AutoConfigURL check returned no value.

## WinHTTP Evidence

The command:

```powershell
netsh winhttp show proxy
```

continued to show:

```text
Direct access (no proxy server).
```

This demonstrated an important distinction between user Internet Settings and WinHTTP proxy configuration.

## Process Telemetry

Sysmon Event ID 1 was available.

A PowerShell-related process creation event was observed at:

```text
25-08-2026 06:50:24
```

The event was treated as supporting process telemetry rather than definitive proof that the proxy configuration was changed by that process.

## PowerShell Event ID 4104

A search was performed for:

```text
ProxyEnable
ProxyServer
127.0.0.1:8080
Internet Settings
```

No relevant PowerShell Event ID 4104 results were returned.

Therefore, Script Block Logging did not provide direct evidence connecting the proxy configuration commands to the modification.

## Security Event ID 4688

Security Event ID 4688 was searched for PowerShell activity.

No relevant result was returned.

This was documented as a telemetry limitation rather than evidence that PowerShell did not execute.

## Network Telemetry

Sysmon Event ID 3 was available and produced numerous network connection events.

The available evidence included events throughout the investigation period, but no specific network connection was established as being caused by the proxy configuration change.

The investigation therefore did not claim that the controlled proxy setting successfully redirected traffic.

## Key Findings

- The initial user proxy configuration was disabled.
- WinHTTP was configured for direct access.
- No proxy-related environment variables were identified.
- No relevant machine-level proxy configuration was identified in the checked Registry location.
- A loopback proxy value of `127.0.0.1:8080` was used for the controlled simulation.
- A configuration mistake placed the proxy string into `ProxyEnable`.
- Sysmon Event ID 1 was available.
- Sysmon Event ID 3 was available.
- No relevant PowerShell Event ID 4104 result was identified.
- No relevant Security Event ID 4688 result was identified.
- No evidence established that the controlled setting caused malicious traffic redirection.
- The configuration mistake was treated as a lab troubleshooting issue, not as malicious behavior.

## DFIR Interpretation

A proxy configuration change should be investigated as a configuration artifact first and a security finding second.

The important questions are:

```text
What was the original value?
        |
        v
What changed?
        |
        v
Who changed it?
        |
        v
When did it change?
        |
        v
What proxy was specified?
        |
        v
Did traffic actually use the proxy?
```

A suspicious proxy value alone does not prove compromise.

Likewise, a process or network event occurring near the same time does not prove that the process caused the configuration change without stronger correlation.

## Important Configuration Distinction

The lab demonstrated that:

```text
WinINet / Internet Settings
```

and:

```text
WinHTTP
```

are separate configuration areas.

Changing a user-level Internet Settings proxy does not automatically mean that:

```text
netsh winhttp show proxy
```

will report the same proxy.

## MITRE ATT&CK Relevance

Potentially relevant techniques in a real incident may include:

**T1112 — Modify Registry**

Relevant when registry values are modified to establish or change system behavior.

**T1059.001 — PowerShell**

Relevant when PowerShell is used to modify configuration.

The controlled lab does not establish malicious use of either technique.

## Evidence Limitations

- PowerShell Event ID 4104 did not provide a relevant configuration-change event.
- Security Event ID 4688 did not provide a relevant PowerShell event.
- Sysmon Event ID 1 provided process telemetry but did not, from the available extraction, directly prove the registry modification.
- Sysmon Event ID 3 showed network activity but did not establish malicious proxy use.
- A final `ProxyEnable = 0` restoration write is not visible in the supplied evidence; therefore the final restored state should be verified separately.

## Conclusion

This investigation demonstrated how a suspicious proxy configuration should be examined through baseline comparison, Registry state, WinHTTP configuration, environment variables, process telemetry, PowerShell logging, and network evidence.

The controlled activity also exposed an important troubleshooting lesson: configuration values must be written to the correct Registry properties. Writing `127.0.0.1:8080` into `ProxyEnable` created an unintended state because that property is intended to represent whether the proxy is enabled, while the proxy address belongs in `ProxyServer`.

The available evidence did not establish malicious proxy use.

## Skills Demonstrated

- Windows DFIR
- Proxy Configuration Investigation
- Registry Analysis
- WinHTTP Investigation
- Internet Settings Analysis
- PowerShell Investigation
- Sysmon Event ID 1
- Sysmon Event ID 3
- Network Configuration Analysis
- Process Correlation
- Configuration Baseline Analysis
- Troubleshooting
- Evidence-Based Assessment

## Disclaimer

This lab used a harmless loopback proxy value for controlled testing. No external proxy server was used and the investigation did not intentionally redirect normal internet traffic through an unknown service.
