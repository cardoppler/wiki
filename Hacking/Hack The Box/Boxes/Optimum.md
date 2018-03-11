# Optimum

### Tools
- [nishang `Invoke-PowerShellTcp.ps1`](https://github.com/samratashok/nishang/blob/master/Shells/Invoke-PowerShellTcp.ps1)
- [rasta-mouse `Sherlock.ps1`](https://github.com/rasta-mouse/Sherlock/blob/master/Sherlock.ps1)
- [Empire `Invoke-MS16032.ps1`](https://github.com/EmpireProject/Empire/blob/master/data/module_source/privesc/Invoke-MS16032.ps1)

### Procedure
- `nmap -sC -sV -oA devel.nmap 10.10.10.5`
    - Discovered port 80
- Browse to http://10.10.10.8
    - Found <SOFTWARE> <VERSION>
        - Vulnerable to <XYZ>
- Set Burp in Intercept mode on 127.0.0.1:8080
- Set FoxyProxy to redirect to Burp
- Search for `%00` in the searchbox
- In Burp, send the GET to Repeater (`CTRL + R + R`) and modify it as: `QUERY`

- Start `python -m SimpleHTTPServer`
- Start netcat to capture the reverse shell `nc -lnvp 1337`
- Execute the GET query in Burp
    - Look for a GET request in SimpleHTTPServer and 
    - You should get a Powershell running as normal user

###
- User Sherlock to identify unpatched vulnerabilities
    - Discovered MS16032   

### Escalate
- Edit `Invoke-MS16032.ps1`:
`SHOWFILE`
- Start netcat to capture the Empire reverse shell `nc -lnvp 1338`
- From the open Powershell session, get the Empire script
    - Obtained privileged shell

