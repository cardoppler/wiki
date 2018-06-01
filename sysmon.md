
# Show config:
```
PS C:\Program Files (x86)\Sysmon> .\Sysmon64.exe -c


System Monitor v7.01 - System activity monitor
Copyright (C) 2014-2018 Mark Russinovich and Thomas Garnier
Sysinternals - www.sysinternals.com

Current configuration:
 - Service name:                  Sysmon64
 - Driver name:                   SysmonDrv
 - HashingAlgorithms:             SHA1
 - Network connection:            disabled
 - Image loading:                 disabled
 - CRL checking:                  disabled
 - Process Access:                disabled

No rules installed
```
# Enable Netowrk monitoring:
```
PS C:\Program Files (x86)\Sysmon> .\Sysmon64.exe -c -n


System Monitor v7.01 - System activity monitor
Copyright (C) 2014-2018 Mark Russinovich and Thomas Garnier
Sysinternals - www.sysinternals.com

Configuration updated.

PS C:\Program Files (x86)\Sysmon> .\Sysmon64.exe -c


System Monitor v7.01 - System activity monitor
Copyright (C) 2014-2018 Mark Russinovich and Thomas Garnier
Sysinternals - www.sysinternals.com

Current configuration:
 - Service name:                  Sysmon64
 - Driver name:                   SysmonDrv
 - HashingAlgorithms:             SHA1
 - Network connection:            enabled
 - Image loading:                 disabled
 - CRL checking:                  disabled
 - Process Access:                disabled

No rules installed
```

# Sysmon 
Remember to run PS as Administrator

## Install
```
> Sysmon.exe -i -accepteula
```

```
> Get-Service -Name sysmon
Status   Name               DisplayName
------   ----               -----------
Running  Sysmon             sysmon

Check the status:
```
PS C:\Program Files\Sysmon> .\Sysmon64.exe -c

System Monitor v6.01 - System activity monitor
Copyright (C) 2014-2017 Mark Russinovich and Thomas Garnier
Sysinternals - www.sysinternals.com

Current configuration:
 - Service name:                  Sysmon
 - Driver name:                   SysmonDrv
 - HashingAlgorithms:             SHA1
 - Network connection:            disabled
 - Image loading:                 disabled
 - CRL checking:                  disabled
 - Process Access:                disabled

No rules installed
```

Enable networking:
```
> Sysmon.exe -c -n
```

Restore default configuration:
```
> Sysmon.exe -c --
```

- Show events in PS: `Get-WinEvent -FilterHashtable @{ logname = "Microsoft-Windows-Sysmon/Operational"; }`
- Show events in Event Viewer: Go to Applications and Services Logs/Microsoft-Windows-Sysmon/Operational
- Show logs in File Editor: C:\Windows\System32\winevt\Logs\Microsoft-Windows-Sysmon%4Operational
