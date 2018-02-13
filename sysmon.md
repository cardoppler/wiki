
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
