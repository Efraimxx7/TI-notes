# 🪟 Windows — Essential Commands

> A practical reference of Windows commands for administration, networking, troubleshooting and IT support.  
> Includes both **CMD** and **PowerShell** approaches.

---

## 📋 Table of Contents

- [CMD — System Information](#-cmd--system-information)
- [CMD — Users & Groups](#-cmd--users--groups)
- [CMD — Network](#-cmd--network)
- [CMD — Processes & Services](#-cmd--processes--services)
- [CMD — Files & Disk](#-cmd--files--disk)
- [PowerShell — Users & Groups](#-powershell--users--groups)
- [PowerShell — Processes & Services](#-powershell--processes--services)
- [PowerShell — Network & Firewall](#-powershell--network--firewall)
- [PowerShell — Event Logs](#-powershell--event-logs)
- [PowerShell — Files & Scheduled Tasks](#-powershell--files--scheduled-tasks)
- [PowerShell — Policy, Startup & Defender](#-powershell--policy-startup--defender)
- [Useful Event IDs](#-useful-event-ids)
- [Quick Troubleshooting](#-quick-troubleshooting)
- [Recommended Tools](#-recommended-tools)

---

## 💻 CMD — System Information

```cmd
:: OS version, hostname, domain, patches
systeminfo

:: Current user, privileges and groups
whoami
whoami /priv
whoami /groups

hostname
ver
set                           :: environment variables
net statistics workstation    :: uptime and stats
net share                     :: shared resources
net file                      :: files open over the network
```

---

## 👤 CMD — Users & Groups

```cmd
net user                                  :: list local users
net user username                         :: user details
net user newuser Password123! /add
net user username /delete
net user username /active:no              :: disable account
net user username /active:yes             :: enable account

net localgroup                            :: list groups
net localgroup Administrators             :: members of a group
net localgroup Administrators username /add
net localgroup Administrators username /delete

net accounts                              :: password policy
net accounts /domain
```

---

## 🌐 CMD — Network

```cmd
ipconfig /all
ipconfig /release
ipconfig /renew
ipconfig /flushdns
ipconfig /displaydns

netstat -ano                              :: connections + PIDs
netstat -ano | findstr LISTENING
netstat -ano | findstr ESTABLISHED

arp -a
route print
ping -n 4 8.8.8.8
tracert google.com
nslookup google.com
```

---

## ⚙️ CMD — Processes & Services

```cmd
tasklist
tasklist | findstr "chrome"
taskkill /PID 1234 /F
taskkill /IM notepad.exe /F

sc query type= all
sc query type= all state= running
sc qc "wuauserv"                          :: service config
net start "Windows Update"
net stop "Windows Update"
sc config "servicename" start= disabled
sc config "servicename" start= auto

:: Programs that start with Windows
reg query HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
reg query HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
```

---

## 📁 CMD — Files & Disk

```cmd
dir /a /q C:\path
dir /ah C:\                               :: hidden files
where /r C:\ filename.txt
findstr /s /i "text" C:\path\*.txt
copy source.txt destination.txt
robocopy source dest /E                   :: robust folder copy
attrib C:\path\file.txt

fsutil volume diskfree C:                 :: free space
chkdsk C: /f /r                           :: check and repair disk
sfc /scannow                              :: check system files
DISM /Online /Cleanup-Image /RestoreHealth
gpupdate /force                           :: refresh group policies
cipher /w:C:\path                         :: wipe free space of a folder
```

---

## 👤 PowerShell — Users & Groups

```powershell
Get-LocalUser
Get-LocalUser -Name "username"
Get-LocalGroup
Get-LocalGroupMember -Group "Administrators"

New-LocalUser -Name "newuser" -Password (ConvertTo-SecureString "Password123!" -AsPlainText -Force)
Add-LocalGroupMember -Group "Administrators" -Member "username"
Remove-LocalGroupMember -Group "Administrators" -Member "username"
Disable-LocalUser -Name "username"
Enable-LocalUser -Name "username"

# Am I running as Administrator?
([Security.Principal.WindowsPrincipal][Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator)
```

---

## ⚙️ PowerShell — Processes & Services

```powershell
Get-Process
Get-Process -Name "notepad"
Get-Process | Sort-Object CPU -Descending | Select-Object -First 20
Get-Process | Select-Object Name, Id, Path | Sort-Object Name
Stop-Process -Name "notepad" -Force
Stop-Process -Id 1234 -Force

Get-Service
Get-Service | Where-Object { $_.Status -eq "Running" }
Get-Service | Where-Object { $_.StartType -eq "Automatic" -and $_.Status -ne "Running" }
Start-Service -Name "wuauserv"
Stop-Service -Name "wuauserv"
Restart-Service -Name "wuauserv"
Set-Service -Name "Spooler" -StartupType Disabled
```

---

## 🌐 PowerShell — Network & Firewall

```powershell
Get-NetIPAddress
Get-NetAdapter
Get-NetRoute
Get-NetTCPConnection
Get-NetTCPConnection -State Listen
Get-NetTCPConnection -State Established
Test-NetConnection google.com -Port 443
Resolve-DnsName google.com
Get-DnsClientCache
Clear-DnsClientCache

# Connections with process names
Get-NetTCPConnection -State Established |
  Select-Object LocalPort, RemoteAddress, RemotePort, OwningProcess,
    @{n="Process";e={(Get-Process -Id $_.OwningProcess -ErrorAction SilentlyContinue).Name}}

# Firewall
Get-NetFirewallProfile
Get-NetFirewallRule | Where-Object { $_.Enabled -eq "True" }
Get-NetFirewallRule -Direction Inbound | Where-Object { $_.Enabled -eq "True" }
New-NetFirewallRule -DisplayName "Block Telnet" -Direction Inbound -Protocol TCP -LocalPort 23 -Action Block
New-NetFirewallRule -DisplayName "Allow SSH" -Direction Inbound -Protocol TCP -LocalPort 22 -Action Allow
New-NetFirewallRule -DisplayName "Block IP" -Direction Outbound -RemoteAddress 203.0.113.5 -Action Block
Remove-NetFirewallRule -DisplayName "Block Telnet"
Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled True
```

---

## 📊 PowerShell — Event Logs

```powershell
Get-EventLog -List                                        # available logs
Get-EventLog -LogName System -Newest 50
Get-EventLog -LogName Application -Newest 50
Get-EventLog -LogName Security -Newest 50

# Filter by Event ID
Get-EventLog -LogName Security | Where-Object { $_.EventID -eq 4625 }   # failed logons
Get-EventLog -LogName System   | Where-Object { $_.EventID -eq 7045 }   # new service installed

# Filter by date
Get-EventLog -LogName Security -After "2025-04-01" -Before "2025-04-30"

# Faster and more flexible: Get-WinEvent
Get-WinEvent -FilterHashtable @{LogName='System'; Level=2} -MaxEvents 20   # errors only

# Export for analysis
Get-EventLog -LogName Security -Newest 1000 | Export-Csv -Path "events.csv" -NoTypeInformation

# Audit policy
auditpol /get /category:*
```

---

## 📁 PowerShell — Files & Scheduled Tasks

```powershell
# Hashes
Get-FileHash -Path "C:\path\file.txt" -Algorithm SHA256
$expected = "abc123..."
$actual = (Get-FileHash "C:\file.txt" -Algorithm SHA256).Hash
if ($expected -eq $actual) { "File is intact" } else { "File has been modified!" }

# Disks
Get-PSDrive -PSProvider FileSystem
Get-Volume
Get-PhysicalDisk

# Search
Get-ChildItem -Path C:\ -Recurse -ErrorAction SilentlyContinue |
  Where-Object { $_.LastWriteTime -gt (Get-Date).AddDays(-1) }          # modified in last 24h
Get-ChildItem -Path C:\ -Recurse -ErrorAction SilentlyContinue |
  Where-Object { $_.Length -gt 100MB }                                  # large files
Select-String -Path "C:\logs\*" -Pattern "error"                        # text inside files

# Scheduled tasks
Get-ScheduledTask
Get-ScheduledTask | Where-Object { $_.State -eq "Ready" }
Get-ScheduledTaskInfo -TaskName "TaskName"
Disable-ScheduledTask -TaskName "TaskName"
Unregister-ScheduledTask -TaskName "TaskName" -Confirm:$false
```

**Script example — disk space check**

```powershell
$disk = Get-PSDrive C
$freePct = [math]::Round(($disk.Free / ($disk.Used + $disk.Free)) * 100, 1)
if ($freePct -lt 15) { Write-Output "WARNING: only $freePct% free on C:" }
```

---

## 🔐 PowerShell — Policy, Startup & Defender

```powershell
# Execution policy
Get-ExecutionPolicy -List
Set-ExecutionPolicy RemoteSigned -Scope LocalMachine

# Programs that run at startup
Get-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Run"
Get-ItemProperty -Path "HKCU:\SOFTWARE\Microsoft\Windows\CurrentVersion\Run"

# Installed updates
Get-HotFix

# Windows Defender
Get-MpComputerStatus
Update-MpSignature
Start-MpScan -ScanType QuickScan
Start-MpScan -ScanType FullScan
Get-MpThreatDetection

# Software with winget
winget search firefox
winget install Mozilla.Firefox
winget upgrade --all

# Remote management
mstsc /v:192.168.1.10
Enter-PSSession -ComputerName PC01
Invoke-Command -ComputerName PC01 -ScriptBlock { Get-Service }
```

---

## 🔑 Useful Event IDs

| Event ID | Log | Description |
|---|---|---|
| **4624** | Security | Successful logon |
| **4625** | Security | Failed logon |
| **4634** | Security | Logoff |
| **4648** | Security | Logon with explicit credentials (RunAs) |
| **4672** | Security | Special privileges assigned to a logon |
| **4688** | Security | New process created |
| **4698** | Security | Scheduled task created |
| **4720** | Security | User account created |
| **4722 / 4725** | Security | User account enabled / disabled |
| **4723 / 4724** | Security | Password change / reset |
| **4726** | Security | User account deleted |
| **4732** | Security | User added to local Administrators |
| **4740** | Security | User account locked out |
| **1102** | Security | Audit log cleared |
| **6005 / 6006** | System | Event Log service started / stopped (boot / shutdown) |
| **7034** | System | Service crashed unexpectedly |
| **7036** | System | Service started or stopped |
| **7045** | System | New service installed |

---

## 🔧 Quick Troubleshooting

| Problem | What to run |
|---|---|
| No internet | `ipconfig /all`, `ping 8.8.8.8`, `nslookup google.com`, `ipconfig /flushdns` |
| Can't reach a port | `Test-NetConnection host -Port 443`, `netstat -ano` |
| Slow PC | `Get-Process \| Sort CPU -Descending`, Task Manager, `Get-Volume` |
| Corrupted system files | `sfc /scannow`, then `DISM /Online /Cleanup-Image /RestoreHealth` |
| Disk errors | `chkdsk C: /f /r`, `Get-PhysicalDisk` |
| Service stopped | `Get-Service`, `Restart-Service`, Event ID 7034 / 7036 |
| User locked out | `net user username`, Event ID 4740, `net user username /active:yes` |
| Policy not applying | `gpupdate /force`, `gpresult /r` |

---

## 🛠️ Recommended Tools

| Tool | Purpose |
|---|---|
| [Sysinternals Suite](https://learn.microsoft.com/en-us/sysinternals/) | Advanced process, network and registry tools |
| [Process Explorer](https://learn.microsoft.com/en-us/sysinternals/downloads/process-explorer) | Detailed process inspection (better Task Manager) |
| [Autoruns](https://learn.microsoft.com/en-us/sysinternals/downloads/autoruns) | View everything that starts with Windows |
| [TCPView](https://learn.microsoft.com/en-us/sysinternals/downloads/tcpview) | Live network connections with process names |
| Event Viewer | GUI for Windows event logs |
| [Wireshark](https://www.wireshark.org/) | Network packet capture and analysis |
| [Zabbix Agent 2](https://www.zabbix.com/) | Monitoring agent for Windows hosts |

---

<div align="center">

*Maintained as a personal reference for IT support and system administration.*  
*Commands tested on Windows 10/11.*

</div>
