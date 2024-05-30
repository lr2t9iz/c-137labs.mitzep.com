---
title: Hunting Malicious Activity in Scheduled Task
date: 2024-05-30 06:00:00 -0600
categories: [blueteam, hunting]
tags: [difficulty:low, endpoint, os:windows]
image:
  path: https://github.com/lr2t9iz/lr2t9iz.github.io/assets/46981088/01a0f58c-c82f-4675-9486-cef48ff1fa0f
---

When we talk about hunting in cybersecurity, it's likely that the first thing that comes to mind is querying a SIEM, our security event database. But what happens when the SIEM was implemented after the adversary had already configured a malicious scheduled task? This scenario presents a significant challenge, as the threat may be latent and operational without being detected by our latest monitoring tools.

In this post, we will explore how to hunt for these types of threats in our infrastructure, learn how to identify and analyze malicious scheduled tasks using techniques and tools that go beyond traditional SIEM capabilities. In addition, we will configure detection rules in [S1EM](https://c-137lab.com/posts/wazuh-s1em/) to improve our ability to detect and respond to threats more effectively.

## Scheduled Task
Scheduled tasks are very useful for IT tasks, developers use them to automate tasks, testing, and more. Adversaries also exploit this functionality as technique ([T1053.005: Scheduled Task/Job: Scheduled Task](https://attack.mitre.org/techniques/T1053/005/)) for execution, persistence, or privilege escalation.
 
## Simulation
For our test, we will use **Test Number 1** of **T1053.005** technique from the [Atomic Red Team project](https://atomicredteam.io/privilege-escalation/T1053.005/#atomic-test-1---scheduled-task-startup-script)
. We will create a scheduled task that simulates a malicious activity, where the task executes a process each time the system starts.
```batch
schtasks /create /tn "T1053_005_OnStartup" /sc onstart /ru system /tr "cmd.exe /c calc.exe"
REM > SUCCESS: The scheduled task "T1053_005_OnStartup" has successfully been created.
```

## Hunting
To retrieve scheduled tasks, we can use either CMD or PowerShell. Let's look at the commands for each:
### With Windows CommandLine (cmd)
Open cmd as adminsitrator and run the following command to view all scheduled tasks
```batch
schtasks /query
```
- Task list
![image](https://github.com/lr2t9iz/lr2t9iz.github.io/assets/46981088/711624ee-437b-4f2a-ad82-9b7323a476a7)
- We can also filter the tasks that are active, by task name, and to get more detail of the task with the following commands.

```batch
REM ready and running tasks
schtasks /query | findstr "Ready"
schtasks /query | findstr "Running"

REM filter by taskname
schtasks /query /tn "T1053_005_OnStartup"

REM more details
schtasks /query /tn "T1053_005_OnStartup" /xml
```
- The fields to pay attention to are trigger and action
![image](https://github.com/lr2t9iz/lr2t9iz.github.io/assets/46981088/9382eddc-7a64-417c-9274-8afae78b9e58)
- In the **Triggers** section we see **BootTrigger** which defines the trigger of the task, in this case is configured to be activated at system startup.
- In the **Actions** section we see that it executes **cmd** with argument **/c calc.exe**

### With PowerShell
Open PowerShell as adminsitrator and run the following command to view all scheduled tasks
```powershell
Get-ScheduledTask
```
- Task list
![image](https://github.com/lr2t9iz/lr2t9iz.github.io/assets/46981088/47a78e66-4821-49f8-bbed-3066cd6e4fef)
- We can also make the same filters as with cmd

```powershell
# active tasks
Get-ScheduledTask | ? State -ne Disabled

# by taskname
Get-ScheduledTask | ? TaskName -eq T1053_005_OnStartup

# more details
(Get-ScheduledTask | ? TaskName -eq "T1053_005_OnStartup").triggers | fl Enabled, CimClass
(Get-ScheduledTask | ? TaskName -eq "T1053_005_OnStartup").actions | fl Execute, Arguments
```
- Task details
![image](https://github.com/lr2t9iz/lr2t9iz.github.io/assets/46981088/c5d0e7c2-ce9d-485f-a9c9-cff7c3feaade)
- We have the same result as in cmd with the difference of the name of the trigger object.

### With Osquery
Osquery is a SQL-based operating system instrumentation, monitoring and analysis framework developed by facebook. We can download from the [following page](https://www.osquery.io/downloads/official/), and install on the windows to hunt.  

Osquery has several tables to do searches, for this post we will only use the following table, [scheduled_tasks](https://www.osquery.io/schema/#scheduled_tasks)
- We start osquery 
```powershell
Start-Service osqueryd
& 'C:\Program Files\osquery\osqueryi.exe'
```
- Execute the following queries

```sql
-- show all scheduled_tasks
SELECT * FROM scheduled_tasks;

-- show all active scheduled_tasks
SELECT path, name, state FROM scheduled_tasks WHERE state != "disabled";

-- show specific task, by taskname
SELECT path, enabled, state, action, last_run_time, last_run_message FROM scheduled_tasks WHERE name == "T1053_005_OnStartup";
```
- Task list
![image](https://github.com/lr2t9iz/lr2t9iz.github.io/assets/46981088/77a04a5f-dcaa-4a72-a0e2-3bcbf275e368)
- The disadvantage we have with osquery it does not get the bootrigger object, but we can see when it was last executed in unix timestamps format and the last message result.

## Collection & Detection
By default windows does not have task scheduler logs enabled, to enable them, run the following command in cmd as adminsitrator.
```batch
wevtutil sl Microsoft-Windows-TaskScheduler/Operational /enabled:true
```
- With the configuration of groups and rules of the [S1EM](https://c-137lab.com/posts/wazuh-s1em/) we obtain the following result
![image](https://github.com/lr2t9iz/lr2t9iz.github.io/assets/46981088/06d563d5-2a79-4383-8e07-e05f440d2917)

Although with this method we can only see the name of the task, to view the trigger and action of the task, we will apply the detailed techniques mentioned earlier. By combining these approaches, we can effectively uncover and analyze scheduled tasks, ensuring our infrastructure remains secure against hidden threats.

Happy hunting!