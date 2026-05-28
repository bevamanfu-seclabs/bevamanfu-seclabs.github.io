---
layout: page
title: Web Attack Forensics - Drone Alone |TryHackMe
description: "Investigated a web attack scenario on TryHackMe, analysing Apache logs and Sysmon data to reconstruct a full attack chain using Splunk."
tech: [Splunk, Sysmon, Log Analysis,  Incident investigation]
importance: 1

---
## Introduction
In this TryHackMe Advent of Cyber 2025 room, I investigated suspicious activity targeting TBFC's drone scheduler web application. The Apache web server was receiving unusually long HTTP requests containing Base64-encoded payloads. Security monitoring in Splunk generated alerts indicating that Apache had spawned unexpected system processes.

## Objectives

- Detect and analyze malicious web activity through Apache access and error logs
- Investigate OS- level attacker actions using Sysmon data
- Identify and decode suspicious or obfuscated attacker payloads
- Reconstruct the full attack chain using Splunk for Blue Team investigation

## Key Concepts Learned

**Command  Injection Attack  ( Shell Injection)**

Command Injection is a web application vulnerability that allows attackers to execute operating system commands on the underlying server. This typically occurs when a user input is improperly validated and passed directly into system-level commands.
In this lab, attackers attempted to exploit a vulnerable script through crafted HTTP requests containing malicious commands. 

**PowerShell Invoke Expression**

Invoke-Expression (IEX) is a PowerShell command that evaluates and executes strings as code. Attackers commonly abuse it to execute malicious payloads locally or remotely while obscuring their true intent. This command is often treated as a high-risk indicator during investigations due to its frequent use in malware and post-exploitation activity. 

**Reconnaissance vs Enumeration**

Reconnaissance is the initial phase of an attack where information about the target environment is gathered. The goal is to identify systems, services, and potential attack surfaces. 

Enumeration involves extracting more detailed information about identified assets, such as usernames, hostnames,shares, services, and privilege information. Attackers often perform enumeration immediately after gaining access to validate privileges and identify lateral movement opportunities.
    


## Tools  Used 

**Splunk**  

A data platform that collects, indexes, searches and analyzes large amounts of machine generated data from various data sources and transforms it into dashboards and other visuals that can be used to gain actionable insights. Splunk was used to identify suspicious web activity  and reconstruct the attack timeline. 

**Sysmon**

A Windows system service and device driver that monitors and creates detailed logs of events such as file creation and modification, command -line execution, network activity and other processes.

## Investigation Walkthrough



#### *Detect Suspicious Web Commands*

```splunk
index=windows_apache_access (cmd.exe OR powershell OR "powershell.exe" OR "Invoke-Expression") 
| table _time host clientip uri_path uri_query status
```
  
   The investigation began by reviewing Apache access logs for evidence of command injection attempts.The query instructs Splunk to search the Apache Access logs collected from a windows server for events relating to Windows Command prompt, powershell or a powershell executable( cmd.exe, powershell.exe or invoke expression) and displays the results in table. 

![Splunk results showing suspicious requests]({{ '/assets/projects/drone-alone-1.png' | relative_url }})

   The search revealed multiple suspicious HTTP requests containing references to cmd.exe, powershell.exe and Invoke-Expression. These indicators suggested that the attacker was attempting to execute system-level commands through the vunerable web application. The URI query strings also contained unusually long Base64-encoded data, indicating possible obfuscation attempts designed to evade detection. 


![Splunk results showing suspicious requests]({{ '/assets/projects/drone-alone-decoded.png' | relative_url }})

   After identifying suspicious requests in the Apache access logs, the Base64-encoded payloads embedded within the HTTP requests were extracted and decoded for further analysis to determine the actual commands being executed by the attacker.The Base64 encoded script  was decoded using base64decode.org

<br><br>

#### *Looking for Server-Side Errors or Command Execution in Apache Error Logs*
```splunk
index=windows_apache_error ("cmd.exe" OR "powershell" OR "Internal Server Error”)
```
   To determine whether the malicious requests successfully reached the backend, apache error logs the splunk query above was ran. The query inspected the Apache error logs from the windows server for internal failures or signs of execution attempts that could be as a result of malicious requests. 


![Splunk results showing suspicious requests]({{ '/assets/projects/drone-alone-query2.png' | relative_url }})

   The results suggested that the malicious input had been processed by the server side application but failed during execution. Internal Server error is usually associated with server sides crashes or script failures.This stage of the investigation helped confirm that the attack traffic was interacting directly with the backend system rather than being blocked at the web layer.
<br><br>


#### *Trace Suspicious Process Creation From Apache*
 The next phase focused on identifying  processes launched by the Apache web server. Under normal circumstances, Apache ( httpd.exe) should not spawn system utilities such as cmd.exe or powershell.exe. The following Sysmon query  was used to investigate suspicious parent-child process relationships.
```splunk
index=windows_sysmon ParentImage="*httpd.exe"
```
   The results revealed that Apache had spawned suspicious system processes, strongly indicating successful command execution the vulnerable appilcation. Observing PowerShell and command prompt process originating from the web server provided strong evidence of command injection activity and confirmed that the attacker had gained the ability to execute operating system commands remotely.
![Splunk results showing suspicious requests]({{ '/assets/projects/drone-alone-query3.png' | relative_url }})
<br><br>

#### *Confirm Attacker Enumeration Activity*
```splunk
index=windows_sysmon *cmd.exe*   *whoami *

```
   After establishing evidence of command execution, the investigation shifted toward identifying post-exploitation reconnaissance activity performed by the attacker. The query above was used to search for enumeration commands executed on the system. 

![Splunk results showing suspicious requests]({{ '/assets/projects/drone-alone-query4.png' | relative_url }})

   The logs showed the execution of the *whoami* command, which attackers commonly use after gaining access to determine the current user context and privilege level. The activity confirmed that the attacker had achieved successful command execution on the target host and had begun gathering information about the compromised environment. 
<br><br>

#### *Identify Base64-Encoded PowerShell Payloads*
```splunk
index=windows_sysmon Image="powershell.exe" (CommandLine="enc" OR CommandLine="-EncodedCommand*" OR CommandLine="Base64")

```
   The final stage of the investigation involved searching Sysmon logs for PowerShell executions involving encoded commands. Attackers frequently use Base64-encoded PowerShell payloads to conceal malicious scripts and evade detection. 

![Splunk results showing suspicious requests]({{ '/assets/projects/drone-alone-query5.png' | relative_url }})

   Surprisingly,the search returned no results. Although encoded payloads were clearly identified within the web requests, there was no evidence that encoded PowerShell commands were successfully executed on the host. This suggested that the payloads may have failed during execution or were blocked before completion.  

## Key Takeways

This investigation provided practical experience in detecting and analyzing command injection attacks using Apache and Sysmon logs.It improved my ability to use Splunk for threat hunting, correlate events across multiple log sources and identify suspicious PowerShell activity.  The lab also demonstrated how attackers use Base64 encoding and PowerShell Obfuscation techniques to hide malicious activity and highlighted the importance of process monitoring in identifying successful exploitation attempts. 