# Windows Event Logs

Windows event logs are a key part of the windows operating system. They store system logs and logs from applications.



There are 3 key event logs:

* Application
* System
* Security



.evtx files are windows log files and can be opened with event viewer

## Useful Event Logs

Here is a no comprehensive list of windows event logs. I bolded the ones that seems most important to me.

### System

1074 - System Shutdown

6005/6006 - Event Log service started/stopped

6013 - Windows uptime

7040 - Service Status change



### Security

1102 - The audit log was cleared, this could be an attempt to remove evidence.

1116 - AV Defender detection

1118/1119/1120 - AV remediation Start/Finish/Failure

**4624 - Successful Logon**

**4625 - Failed logon**

**4648 - Explicit credential logon - When a user logs in with explicit creds to run a program. Could  be indicative of lateral movement.**

4656 - Object Handle Requested - Access to a sensitive object was requested

**4672 - Special privilege logon (SUDO Logon)**

**4700/4701 - Scheduled Task enabled/disabled**

**4702 - Scheduled Task edited**

4719 - System Audit Policy Change

4738 - A user account changed

**4771 - Kerberos preauthentication failed**

**4776 - A domain controller attempted to validate the credentials for an account - logins type data at a AD level**

5001 - AV protection has changed

**5140 - A network share was accessed**

5142 - A network share has been newly created

5145 - The permissions of a network share were checked

5157 - The Windows Filtering Platform has blocked a connection

**7045 - A service was installed**



## ETW - Event Tracing for Windows



Windows Event Logs are persistent, disk-based logs used for auditing and investigations, while ETW is a real-time, high-performance tracing framework that provides deeper and more granular telemetry. Event Logs are easier to use and widely ingested into SIEMs, whereas ETW is typically leveraged by EDR tools for advanced behavioral detection and low-level system visibility.



You can use built in logman.exe to assist in viewing ETW information.



## Sysmon



Sysmon provides detailed information about process creation, network connections, changes to file creation time, and more.



Sysmon's primary components include:

* A Windows service for monitoring system activity.
* A device driver that assists in capturing the system activity data.
* An event log to display captured activity data.



Sysmon can log additional information that normally is not found within standard logging data. The list of all Sysmon events are located at:

https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon

You can config Sysmon to display different types of activity using it's xml config file. The most popular one is:
https://github.com/SwiftOnSecurity/sysmon-config





#### Important Sysmon Events

1 - Process Creation

3 - Network Connection

7 - Module load



## Get-WinEvent

In IR and threat hunting scenarios, using powershell to analyze win event logs en masse is a great resource.

To view log sources run:
`Get-WinEvent -ListLog \\\* | Select-Object LogName, RecordCount, IsClassicLog, IsEnabled, LogMode, LogType | Format-Table -AutoSize`



To retrieve the top 50 system logs run:

`Get-WinEvent -LogName 'System' -MaxEvents 50 | Select-Object TimeCreated, ID, ProviderName, LevelDisplayName, Message | Format-Table -AutoSize`

To use logs files from a different device .evt or .evtx files you can specifiy a path:

`Get-WinEvent -Path 'C:\\\\Tools\\\\chainsaw\\\\EVTX-ATTACK-SAMPLES\\\\Execution\\\\exec\\\_sysmon\\\_1\\\_lolbin\\\_pcalua.evtx' -MaxEvents 5 | Select-Object TimeCreated, ID, ProviderName, LevelDisplayName, Message | Format-Table -AutoSize`





