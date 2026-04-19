
Splunk is a very popular enterprise SIEM solution. I use it everyday at work.
## Splunk Architecture

Splunk consists of several layers that work together. The main components are: 
* **Universal forwarder**: A lightweight agent who forwards data to indexers without processing
* **Heavy forwarder:**  According to Splexicon, heavy forwarders stand out from other types of forwarders as they parse data before forwarding, allowing them to route data based on specific criteria such as event source or type.
* **Indexers:** Indexers receive data from forwarders, put it into indexes and organize it. Indexers also process user search queries and return data.
* **Search Heads:** Provide the UI for splunk

![[Pasted image 20260417135500.png]]

Splunk applications are packages you can add to your deployment to extend capabilities or even add out-of-the-box performance. In the labs we utilize the Sysmon App for Splunk. However, to make the out of the box app work, sometimes we have to adjust the sources the visualizations are built off of. 

While windows event logs may be good for a single host, we need to use Splunk or a SIEM solution to take this up a notch. It will let us make more powerful queries for our data, and alet us visualize data across multiple hosts at once. 

# Investigating with Splunk

First we should look at what data we have available to us to better understand how these things work. Please look at [Sysmon Code notes. ](obsidian://open?vault=HTB-Notes&file=Module%20Notes%2FSysmon%20Codes) We can identify all the sysmon codes in our data by starting with: 
````SPL
index="main" sourcetype="WinEventLog:Sysmon" | stats count by EventCode
````

If we were trying to look for suspicious parent childe relationships, we could look at sysmon code 1s, process creation. Then write a search displaying the parent and child processes. If that data is too much, we can look instead for different "problem" process like cmd or ps. 

To filter more noise: 
```
index="main" sourcetype="WinEventLog:Sysmon" EventCode=1 (Image="*cmd.exe" OR Image="*powershell.exe") | stats count by ParentImage, Image
```

Then as we notice something stand out as a unusual we might investigate deeper. this shows us all the events. I then added a table command to help identify more of what actually is occurring. 

```
index="main" sourcetype="WinEventLog:Sysmon" EventCode=1 (Image="*cmd.exe" OR Image="*powershell.exe") ParentImage="C:\\Windows\\System32\\notepad.exe"
```

``` 
index="main" sourcetype="WinEventLog:Sysmon" EventCode=1 (Image="*cmd.exe" OR Image="*powershell.exe") ParentImage="C:\\Windows\\System32\\notepad.exe"
|  table _time ComputerName CommandLine
```

From this search in the lab environment, we identify an odd web request to download a file from an internal IP. A search for this IP shows it belongs to a Linux system. This could be strong indication a compromise has occurred on a Linux system. 

# Using Statistics to Identify Anomalies

By profiling `normal` behavior and identifying deviations from this baseline, we can uncover suspicious activities that may signify an intrusion.

A good example of this approach in Splunk is the use of the `streamstats` command. This command allows us to perform real-time analytics on the data, which can be useful for identifying unusual patterns or trends.

If we consider looking at Net Conns (Event Code 3) we might want to use this search. 

```SPL
index="main" sourcetype="WinEventLog:Sysmon" EventCode=3 
| bin _time span=1h 
| stats count as NetworkConnections by _time, Image 
| streamstats time_window=24h avg(NetworkConnections) as avg stdev(NetworkConnections) as stdev by Image 
| eval isOutlier=if(NetworkConnections > (avg + (0.5*stdev)), 1, 0) | search isOutlier=1
```

Sometimes when we launch stats searches, we can then filter out noise quickly. For example, when looking at the number of dll's loaded (Event Code 7) in rapid succession. We may start with this search: 

```SPL
index="main" EventCode=7 
| bucket _time span=1h 
| stats dc(ImageLoaded) as unique_dlls_loaded by _time, Image 
| where unique_dlls_loaded > 3 
| stats count by Image, unique_dlls_loaded
```

Then after noticing some benign activity and patterns, we can filter our data to: 

```SPL
index="main" EventCode=7 NOT (Image="C:\\Windows\\System32*") NOT (Image="C:\\Program Files (x86)*") NOT (Image="C:\\Program Files*") NOT (Image="C:\\ProgramData*") NOT (Image="C:\\Users\\waldo\\AppData*")
| bucket _time span=1h 
| stats dc(ImageLoaded) as unique_dlls_loaded by _time, Image 
| where unique_dlls_loaded > 3 
| stats count by Image, unique_dlls_loaded 
| sort - unique_dlls_loaded
```

By establishing a profile of "normal" behavior and utilizing a statistical model to identify deviations from a baseline, we could have detected the compromise of our environment more rapidly, especially with a thorough understanding of attacker tactics, techniques, and procedures (TTPs). ***However, it is important to acknowledge that relying solely on this approach when crafting queries is inadequate.***

# Lab Section 

Navigate to http://[Target IP]:8000, open the "Search & Reporting" application, and find through an analytics-driven SPL search against all data the source process images that are creating an unusually high number of threads in other processes. Enter the outlier process name as your answer where the number of injected threads is greater than two standard deviations above the average. Answer format: _.exe

