# Investigation Notes 

## Initial User Proxy Configuration

The user-level Internet Settings location was:

`HKCU:\Software\Microsoft\Windows\CurrentVersion\Internet Settings`

The initial query showed:

```text
ProxyEnable : 0
```

No proxy server value was shown.

This established that the user-level proxy was initially disabled.

## Initial Machine-Level Configuration

The machine-level location was checked:

`HKLM:\Software\Microsoft\Windows\CurrentVersion\Internet Settings`

No meaningful proxy values were returned from the queried properties.

## Initial WinHTTP Configuration

The command:

```powershell
netsh winhttp show proxy
```

returned:

```text
Direct access (no proxy server).
```

This established that WinHTTP was not configured to use a proxy.

## Environment Proxy Variables

The environment-variable search used:

```powershell
Get-ChildItem Env: |
Where-Object {
    $_.Name -match "PROXY"
} |
Select-Object Name, Value
```

No proxy-related environment variables were returned.

## AutoConfigURL

The query:

```powershell
Get-ItemProperty "HKCU:\Software\Microsoft\Windows\CurrentVersion\Internet Settings" |
Select-Object AutoConfigURL
```

did not return a configured value.

This indicated that no PAC URL was identified through that property.

## Controlled Proxy Simulation

The investigation used:

```text
127.0.0.1:8080
```

as a harmless loopback proxy value.

The intended configuration was conceptually:

```text
ProxyEnable = 1
ProxyServer = 127.0.0.1:8080
```

The loopback address was used so that no external proxy infrastructure was involved.

## Configuration Error

During the simulation, the proxy string was accidentally written into:

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

This was an incorrect configuration state.

The mistake was documented as a lab troubleshooting issue.

## ProxyServer Cleanup

A later command explicitly set:

```text
ProxyServer = ""
```

This removed the loopback proxy string from the `ProxyServer` property.

However, the supplied evidence does not show a final write setting:

```text
ProxyEnable = 0
```

after the mistaken assignment.

Therefore, the final user proxy state should be independently verified before considering the system fully restored.

## Registry LastWriteTime

The Registry key:

`HKCU:\Software\Microsoft\Windows\CurrentVersion\Internet Settings`

was inspected for its `LastWriteTime`.

The timestamp was recorded as part of the investigation.

The value is supporting evidence only because the key timestamp does not identify which property changed or who changed it.

## Sysmon Event ID 1

Sysmon Event ID 1 was queried for:

```text
powershell.exe
```

A PowerShell process creation event was observed at:

```text
25-08-2026 06:50:24
```

This provides process context.

The available extraction does not directly prove that this particular process performed the proxy Registry modification.

## PowerShell Event ID 4104

The investigation searched for:

```text
ProxyEnable
ProxyServer
127.0.0.1:8080
Internet Settings
```

No relevant Event ID 4104 result was returned.

This means Script Block Logging did not provide direct evidence of the configuration commands.

## Security Event ID 4688

The investigation searched for PowerShell-related 4688 events.

No relevant event was returned.

Therefore, Security Event ID 4688 was not useful for directly attributing the proxy modification.

## Sysmon Event ID 3

Sysmon Event ID 3 produced numerous network events.

The available results included network events from:

```text
06:39
through
07:42
```

The telemetry demonstrated that network monitoring was active.

The available evidence did not establish that the controlled proxy configuration caused any particular network connection.

## Direct vs WinHTTP Proxy Evidence

The lab demonstrated an important configuration distinction.

User-level Internet Settings can contain proxy information while:

```text
netsh winhttp show proxy
```

continues to report:

```text
Direct access (no proxy server).
```

These are separate configuration mechanisms.

## Evidence Correlation

The investigation sequence was:

```text
Initial Proxy Baseline
        |
        v
WinHTTP Baseline
        |
        v
Environment Variables
        |
        v
Controlled Registry Change
        |
        v
PowerShell / Process Telemetry
        |
        v
Network Telemetry
        |
        v
Configuration Cleanup
        |
        v
Final State Verification
```

