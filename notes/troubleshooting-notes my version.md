# Troubleshooting Notes

## 1. ProxyEnable and ProxyServer Were Confused

### Problem

The proxy address:

```text
127.0.0.1:8080
```

was accidentally written into:

```text
ProxyEnable
```

instead of:

```text
ProxyServer
```

### Correct Semantics

```text
ProxyEnable
```

is intended to represent whether the user proxy is enabled.

```text
ProxyServer
```

is where the proxy address belongs.

### Correct Configuration Model

```text
ProxyEnable = 1
ProxyServer = 127.0.0.1:8080
```

### DFIR Lesson

Registry values must be interpreted according to their intended semantics.

An unexpected value in a Registry property does not automatically represent attacker behavior.

## 2. ProxyEnable Originally Showed 0

The initial user-level proxy query showed:

```text
ProxyEnable = 0
```

This established the pre-test baseline.

That baseline should always be recorded before making a controlled configuration change.

## 3. ProxyServer Was Later Cleared

The following operation was performed:

```powershell
Set-ItemProperty `
-Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Internet Settings" `
-Name ProxyServer `
-Value ""
```

This removed the loopback proxy string from the ProxyServer property.

### Remaining Verification

The supplied evidence does not show a final:

```powershell
ProxyEnable = 0
```

write after the configuration mistake.

Therefore, the final state should be verified before closing the lab.

## 4. WinHTTP Still Showed Direct Access

The command:

```powershell
netsh winhttp show proxy
```

returned:

```text
Direct access (no proxy server).
```

### Explanation

WinHTTP and user Internet Settings are separate configuration areas.

A user-level proxy change does not automatically change WinHTTP.

### DFIR Lesson

Do not assume that one Windows proxy mechanism represents all proxy configuration.

## 5. No Proxy Environment Variables

The environment-variable search returned no proxy entries.

This established that the lab endpoint did not expose proxy configuration through the checked environment variables.

## 6. No AutoConfigURL

The AutoConfigURL query returned no value.

Therefore, no PAC URL was identified through the checked property.

## 7. PowerShell Event ID 4104 Had No Relevant Result

The search for:

```text
ProxyEnable
ProxyServer
127.0.0.1:8080
Internet Settings
```

returned no matching Event ID 4104 records.

### Interpretation

The commands were not directly visible through the available Script Block Logging search.

### DFIR Lesson

A missing 4104 event does not prove that the command was not executed.

## 8. Security Event ID 4688 Had No Relevant Result

The PowerShell-related 4688 search returned no matching results.

### Interpretation

Process Creation telemetry from the Security log was not available for directly attributing the configuration change.

### Alternative Evidence

Sysmon Event ID 1 was available.

## 9. Sysmon Event ID 1 Was Available

The PowerShell process search returned an event at:

```text
25-08-2026 06:50:24
```

### Limitation

The available extraction does not establish that the event directly corresponds to the proxy Registry modification.

A timestamp match alone is insufficient attribution.

## 10. Sysmon Event ID 3 Was Available

The system generated numerous network connection events.

### Important

The existence of network events after a proxy configuration change does not prove:

```text
Proxy change
    ->
That network event
```

Additional process, destination, and proxy-use evidence would be required.

## 11. Do Not Use a Real External Proxy for This Lab

The controlled test used:

```text
127.0.0.1:8080
```

This avoided routing normal traffic through an external or unknown proxy.

No proxy server was actually started.

This kept the lab isolated.

## 12. Configuration Restoration

The safest restoration procedure is to use the recorded baseline values.

If the original state was:

```text
ProxyEnable = 0
ProxyServer = empty
ProxyOverride = original value
AutoConfigURL = original value
```

restore those values exactly.

Do not assume that empty strings are appropriate if the baseline contained something else.

