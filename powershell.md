### Active-Directory
```
Get-ADUser SOMEUSERNAME -Properties *
Get-ADComputer SOMECOMPUTERNAME -Properties *
```

### Convert time to human readable
```
[DateTime]::FromFileTime(131628203805456193)
```

### Calculate MD5 hash
With cmd prompt: `CertUtil -hashfile C:\path\to\file.exe MD5`
With Powershell: `Get-FileHash C:\path\to\file.exe -Algorithm MD5`

Start with a different HOME location: `Right click on PW icon > Property > "Start in"`

### Profile location:

```
> $profile | Format-List -Force
\\path\to\<USERNAME>\Documents\WindowsPowerShell\profile.ps1
\\SOMEHOSTNAME\userdata$\pietriri\Documents\WindowsPowerShell\Microsoft.PowerShell_profile.ps1
```

### Change prompt
In "profile.ps1":
``` 
function prompt { "[$(Get-Date -Format s)] > " }
```

- ESC ESC: cancel line

```
> $PSVersionTable
```
```
> help-update
> Get-Command -Name "*child*"
> help Get-ChildItem
> Get-ChildItem C:\Users\cardo\.vscode\extensions\ -Name
```
```
> Get-Process | Format-List
> Get-Service | ConvertTo-Json
> Get-Process | ConvertTo-HTML | Out-File ps.html
```
```
> ls *.md
> ls | Get-Member
> (ls).FullName
> ls | Select-Object -Property Name,CreationTime
```
```
> $value = "hello world"
> $value
hello world
```
```
> gc .\file.txt | select -First 10
> gc .\file.txt | Measure-Object -Line
> gc file.txt | %{ $_.Split(':')[1]; }
> Select-String somestring .\file.txt
> gc test.txt | Select-String “example" | %{ $_ -replace ‘example', 'elpmaxe' }
> gc .\microservices.log | Select-String -NotMatch "No Token" | Select-String -Pattern "GET /html/health" -NotMatch | Add-Content .\microservices-04.log

Select-String -Pattern "SOMETHING|SOMWTHINGELSE" -NotMatch 

> gc .\microservices-04.log | %{$_.split(':')[3]}
> $var = gc .\microservices-04.log | %{$_.split(':')[3]}
> $var.substring(50) | Add-Content .\microservices-05.log
> gc .\entries.log |  %{"$($_.Split(' ')[9..200])"} | Sort-Object | Get-Unique | Add-Content .\entries-v03.log
> gc '.\2017-09-21 CA API GW logs.csv' |  %{"$($_.Split(',')[61..200])"} | %{"$($_.Split(' ')[10..1000])"}
> gc .\scmpe.log | findstr "ldnsudzl71" | findstr "stalled" | Add-Content ldnsudzl71-stalled.log
> gc .\ldnsudzl71-stalled.log | Measure-Object
```
Reverse string:
```
> $var = gc .\teststring.log
> $var
teststring
> $array = $var.ToCharArray()
> $array
t
e
s
t
s
t
r
i
n
g
> [array]::Reverse($array)
> $array
g
n
i
r
t
s
t
s
e
t

> $var
teststring
> $rav = $var[($var.Length-1)..0]
> $rav
g
n
i
r
t
s
t
s
e
t

> (gc .\teststring.log).ToCharArray().GetType()

IsPublic IsSerial Name                                     BaseType
-------- -------- ----                                     --------
True     True     Char[]                                   System.Array
```

## Regex
```
$inputFile = gc .\test.log
$inputFile -match '(?<change>C\d+ xxx (?<numbers>\d+))'
```
```
> [regex]$regex = "(?<GroupNameHere>C\d+)"
> $inputFile = gc .\someInputFile.txt
> $matches = $regex.Matches($inputFile)
> $matches
> $matches.Value
> $matches.Value | Out-File results.txt
> gc .\results.txt
```
```
> $input_path = ".\input-file.csv"
> $output = ".\output-file.csv"
> $regex = '^"\w+","(\w+)"'
> Select-String -Path $input_path -Pattern $regex -AllMatches | % { $_.Matches } | % { $_.Value } > $output
> Select-String -Path $input_path -Pattern $regex -AllMatches | Format-Table | select -First 10
> Select-String -Path $input_path -Pattern $regex -AllMatches | %{$null = $_.Line -match $regex; $matches['target'] }
```

For loop:
```
for ($i=0; $i -lt 10; $i++) { echo $i }

> $array = @("hostname01", "hostname02", "hostname03") 
> foreach ($item in $array) { nslookup $item }
```
### Sort IPs
```
> $ips = @('10.212.10.88','10.132.232.96','10.9.216.135','10.210.32.20')
> $ips
10.212.10.88
10.132.232.96
10.9.216.135
10.210.32.20
> $ips | Sort { [version]$_ }
10.9.216.135
10.132.232.96
10.210.32.20
10.212.10.88
```

## PingAll
```
[CmdletBinding()]
param (
     [Parameter(Mandatory=$true)]
     [string] $HostnamesInputFile = $null,
     [Parameter(Mandatory=$true)]
     [string] $PingProbesCount = $null
     )

$OutputFileName = "pingAll_output.csv"
$FileExists = Test-Path $OutputFileName 
If ($FileExists -eq $True) {
    Write-Host "[!] pingAll_output.csv already exists. Please delete the file and run the script again. Exiting now."
    Exit
}

Write-Output "[+] Sorting & Unique..."
$hostnames = Get-content "$HostnamesInputFile" | Sort-Object | Get-Unique

Write-Output "[+] Pinging..."

foreach ($name in $hostnames) {
  if (Test-Connection -ComputerName $name -Count $PingProbesCount -ErrorAction SilentlyContinue){
   Write-Output "[>] $name is up"
   Add-Content $OutputFileName "$name,up"
  }
  else{
    Write-Output "[<] $name is unreachable"
    Add-Content $OutputFileName "$name,unreachable"
  }
}

Write-Output "[+] Add done. File: <pingAll_output.csv> created."
```

```
> $Env:ASPNETCORE_ENVIRONMENT = "Development"
> Get-ChildItem Env:
> $Env:ASPNETCORE_ENVIRONMENT
```


Get all Domain Controllers from all Domains:
```
> nltest /dclist:somedomain.com
```

```
> query session
SESSIONNAME       USERNAME                 ID  STATE   TYPE        DEVICE
services                                    0  Disc
console                                     1  Conn
rdp-tcp#1         username1                 4  Active  rdpwd
>rdp-tcp#0         username2                8  Active  rdpwd
rdp-tcp                                 65536  Listen
```

```
> (Get-Location | Get-Item).Name
> Get-Date
> Get-CimInstance -ClassName win32_operatingsystem | select csname, lastbootuptime
``` 
## telnet equivalent
```
New-Object System.Net.Sockets.TcpClient("HOSTNAME", 445)
```

## rsync equivalent
```
$ Copy-Item .\*.log -Destination \\hostname\path\to\file.log
$ net user /do username
$ Get-Module -ListAvailable
```

Make upper case:
```
Get-Content dictionary.txt | foreach {$_.toupper()}
```

```
C:\Windows>set SYSTEMDRIVE
SystemDrive=C:
```

## Recursive tree like info
```
PS C:\Temp_2> Get-ChildItem -recurse -force . | Select-Object FullName,Attributes,Length,LastWriteTimeUtc | Sort-Object

FullName                              Attributes Length LastWriteTimeUtc
--------                              ---------- ------ ----------------
C:\Temp_2\folder1\folder1_1            Directory        12/10/2017 10:43:07
C:\Temp_2\folder1\file2.txt              Archive 20     12/10/2017 10:42:58
C:\Temp_2\folder1\folder1_1\file3.txt    Archive 34     12/10/2017 10:57:03
C:\Temp_2\file1.txt                      Archive 20     12/10/2017 10:42:20
C:\Temp_2\folder1                      Directory        12/10/2017 10:42:58
C:\Temp_2\Export.csv                     Archive 503    12/10/2017 11:11:28
C:\Temp_2\Export.log                     Archive 595    12/10/2017 11:01:37
```
## and export into csv
```
PS C:\Temp_2> Get-ChildItem -recurse -force . | Select-Object FullName,Attributes,Length,LastWriteTimeUtc | Sort-Object | Export-Csv -Encoding ascii -NoTypeInformation -Path "Export for NAME $(((get-date).ToUniversalTime()).ToString("yyyy-MM-dd HH-mm-ss")).csv"
```
Recursively list all directories up to depth 2:
```
> Get-ChildItem -recurse -force . -Depth 2 -Directory | Select-Object FullName,Attributes,Length,LastWriteTimeUtc | Sort-Object
```

### Proxy
```
> netsh winhttp show proxy
> netsh winhttp import proxy source=ie
> $webclient=New-Object System.Net.WebClient
> $creds=Get-Credential
> $webclient.Proxy.Credentials=$creds
```
```
> Invoke-WebRequest https://google.com/
> Invoke-WebRequest https://google.com/ | Select-Object -ExpandProperty Content
> Invoke-WebRequest https://google.com/ -Proxy http://someproxy.com:8080 -ProxyUseDefaultCredentials
> Invoke-WebRequest http://google.com -Proxy http://someproxy.com:8080:8080 -ProxyUseDefaultCredentials | Select-Object -ExpandProperty Headers
```

### curl like
```
> Invoke-RestMethod "https://jsonplaceholder.typicode.com/users" -Proxy http://someproxy.com:8080 -ProxyUseDefaultCredentials | ConvertTo-Json -Depth 10
```

## Wifi
```
> netsh wlan show profile
> netsh wlan show profile SOMENETWORKHERE key=clear
```

## Simple PS script to export a list of files into CSV and then archive everything into a ZIP:
```
# Getting user inputs:
[CmdletBinding()]
param (
     [Parameter(Mandatory=$true)]
     [string] $ReceiverName = $null,
     [Parameter(Mandatory=$true)]
     [string] $SourceFolder = $null
)

# Test if provided firectory exists:
$FileExists = Test-Path $SourceFolder
If ($FileExists -eq $False) {
    Write-Host "[-] Directory does NOT exist. Exiting now."
    Exit
}

# Creating CSV with export info:
$csvFilename = "$ReceiverName $(((get-date).ToUniversalTime()).ToString("yyyy-MM-dd HH-mm-ss")) UTC.csv"
$csvAbsPath = Join-Path -Path $SourceFolder -ChildPath $csvFilename
Write-Output "[+] Creating CSV with export info..."
Get-ChildItem -recurse -force $SourceFolder | Select-Object FullName,Attributes,Length,LastWriteTimeUtc | Sort-Object -Property FullName | Export-Csv -Encoding ascii -NoTypeInformation -Path $csvAbsPath
Write-Output "[+] Created archive: $csvAbsPath"

# Creating ZIP archive:
Add-Type -assembly "system.io.compression.filesystem"
$zipFilename = "$ReceiverName $(((get-date).ToUniversalTime()).ToString("yyyy-MM-dd HH-mm-ss")) UTC.zip"
$zipAbsPath = Join-Path -Path (Split-Path -Path $SourceFolder ) -ChildPath $zipFilename
Write-Output "[+] Creating ZIP archive..."
[io.compression.zipfile]::CreateFromDirectory($SourceFolder, $zipAbsPath) 

Write-Output "[+] Created archive: $zipAbsPath"
Write-Output "[+] All done"
```
then add an alias in profile.ps1 to call it from anywhere:
```
New-Alias someocmmandnamehere \\path\to\user\Documents\PowerShell\script.ps1
```

# Windows 10 - Burn ISO to USB
```
Microsoft Windows [Version 10.0.16288.1]
(c) 2017 Microsoft Corporation. All rights reserved.
C:\WINDOWS\system32>diskpart.exe
Microsoft DiskPart version 10.0.16288.1
Copyright (C) Microsoft Corporation.
On computer: DESKTOP
DISKPART> list disk
  Disk ###  Status         Size     Free     Dyn  Gpt
  --------  -------------  -------  -------  ---  ---
  Disk 0    Online          238 GB  1024 KB        *
  Disk 1    Online         3700 MB      0 B
DISKPART> select disk 1
Disk 1 is now the selected disk.
DISKPART> clean
DiskPart succeeded in cleaning the disk.
DISKPART> create partition primary
DiskPart succeeded in creating the specified partition.
DISKPART> list partition
  Partition ###  Type              Size     Offset
  -------------  ----------------  -------  -------
* Partition 1    Primary           3700 MB    64 KB
DISKPART> select partition 1
Partition 1 is now the selected partition.
DISKPART> active
DiskPart marked the current partition as active.
DISKPART> format fs=fat32 quick
  100 percent completed
DiskPart successfully formatted the volume.
DISKPART> exit
Leaving DiskPart...
C:\WINDOWS\system32>
```

Right click the ISO and "Mount". Now copy paste the files.
