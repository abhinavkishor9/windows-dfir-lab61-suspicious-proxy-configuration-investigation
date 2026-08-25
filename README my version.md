# Windows DFIR Lab 61 — Suspicious Proxy Configuration Investigation

## Overview

Concept

A proxy sits between an application and its destination:

Normal:

Application
    ↓
Internet


With Proxy:

Application
    ↓
Proxy
    ↓
Internet

A legitimate organization might intentionally use a proxy for:

Web filtering
Malware inspection
Authentication
Logging
Content controls
Corporate internet access

But an unexpected proxy can be suspicious.

For example:

User
  ↓
Browser
  ↓
Unexpected Proxy
  ↓
Internet

An attacker could potentially use proxy configuration to:

Redirect traffic.
Intercept communications.
Hide the real destination.
Force applications through attacker-controlled infrastructure.
Maintain access to traffic after compromise.
Bypass certain network controls.

However:

A proxy configured on a machine is not automatically malicious.

We need to establish what changed, who changed it, when it changed, and where the proxy points.

This lab investigates suspicious proxy configuration changes on a Windows endpoint. Proxy settings can be legitimate in enterprise environments, but unexpected changes may redirect application traffic, interfere with normal network controls, or indicate unauthorized system modification.

The investigation began by establishing the endpoint's existing proxy configuration. Windows Internet Settings, WinHTTP, environment variables, and machine-level Internet Settings were checked before any controlled modification was attempted.

A loopback proxy value using `127.0.0.1:8080` was then introduced as a harmless simulation. During the exercise, a configuration mistake occurred in which the proxy string was written into the `ProxyEnable` value instead of the `ProxyServer` value. This was documented as part of the lab rather than treated as malicious behavior.

PowerShell Event ID 4104 and Security Event ID 4688 searches did not return relevant results. Sysmon Event ID 1 and Event ID 3 were available, but the available evidence did not establish a specific malicious process or network connection responsible for the proxy configuration change.

## Investigation Objectives

- Establish the workstation's proxy state before any configuration change.
- Identify the different locations where proxy configuration may exist.
- Determine which proxy-related setting was modified during the controlled activity.
- Compare the configuration before and after the change.
- Examine whether the change can be associated with a PowerShell process.
- Determine whether available Windows telemetry preserves evidence of the configuration change.
- Examine surrounding network activity without assuming that it was caused by the proxy setting.
- Understand the difference between user-level Internet Settings and WinHTTP configuration.
- Identify configuration mistakes that could produce misleading forensic evidence.
- Determine whether the evidence supports only a configuration change or also supports actual proxy use.
- Document telemetry gaps and avoid treating a proxy setting by itself as proof of compromise.

## Investigation Scenario

A Windows workstation is suspected of having its internet proxy configuration changed unexpectedly. The analyst needs to determine whether the system's proxy state differs from its normal baseline and whether the change can be linked to a specific process or user activity.

The investigation examines the user-level Internet Settings, WinHTTP configuration, environment variables, and Registry state. The analyst then reviews process, PowerShell, and network telemetry around the configuration-change timeframe.

The investigation must determine:

- What the original proxy configuration was.
- What value was introduced during the change.
- Whether the change was correctly applied or resulted from a configuration error.
- Whether any process or PowerShell activity can be associated with it.
- Whether the available network evidence shows actual traffic being redirected through the proxy.

The final assessment should distinguish a proxy configuration change from confirmed malicious proxy usage.

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


## MITRE ATT&CK Relevance

Potentially relevant techniques in a real incident may include:

**T1112 — Modify Registry**

Relevant when registry values are modified to establish or change system behavior.

**T1059.001 — PowerShell**

Relevant when PowerShell is used to modify configuration.

The controlled lab does not establish malicious use of either technique.


## Disclaimer

This lab used a harmless loopback proxy value for controlled testing. No external proxy server was used and the investigation did not intentionally redirect normal internet traffic through an unknown service.
