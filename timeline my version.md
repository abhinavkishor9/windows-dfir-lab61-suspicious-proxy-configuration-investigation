# Timeline 

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

