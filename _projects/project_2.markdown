---
layout: page
title: Web Attack Forensics - Drone Alone |TryHackMe
description: "Investigated a web attack scenario on TryHackMe, analysing Apache logs and Sysmon data to reconstruct a full attack chain using Splunk."
tech: [Splunk, Sysmon, Log Analysis, Blue Team, Forensics]
importance: 1

---
## Objective

- Detect and analyze malicious web activity through Apache access and error logs
- Investigate OS- level attacker actions using Sysmon data
- Identify and decode suspicious or obfuscated attacker payloads
- Reconstruct the full attack chain using Splunk for Blue Team investigation

## Key Concepts Learned

- Command  Injection Attack  ( Shell Injection) : It allows an attacker to execute OS commands on the server that is running an application
- Invoke Expression :   A method of executing code in powershell that allows for the evaluation of expressions and the execution of code that is stored in a variable. It preferred by attackers cause it can be used to launch both local and remote payloads.
    
    
- Recon vs Enumeration
Reconnaissance is the first step in any hacking engagement. It's all about gathering information on your target systems or networks to build a comprehensive understanding of the environment you plan to assess

Enumeration is the process of extracting more detailed information  about the assets we discovered during our initial recon.  These information includes usernames and passwords, accessible network folders, hostnames, and machine  names .

## Tools  Used / Involved

Splunk -  A data platform that collects indexes, searches and analyzes large amounts of machine generated data from various data sources and transforms it into dashboards and other visuals that can be used to gain actionable insights.  — It’s basically a SIEM tool

Sysmon -  A windows system service and device driver that monitors and creates detailed logs of events such as file creation and modification, network activity and other processes.

### Walkthrough Summary

TBFC’s drone scheduler web UI is getting strange, long HTTP requests containing Base64 chunks. Splunk raises an alert: “Apache spawned an unusual process.” On some endpoints, these requests cause the web server to execute shell code, which is obfuscated and hidden within the Base64 payloads. For this room, your job as the Blue Teamer is to triage the incident, identify compromised hosts, extract and decode the payloads and determine the scope."

