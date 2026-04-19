Here are Sysmon codes and their respective meanings. I've also included any additional info about them.

# Event ID 1 - Process Creation
The event that gets generated when a process is created. Similar to a PR2. 
- Useful for finding abnormal parent-child processes
# Event ID 2 - Process Changed a File Creation Time
The event that is generated when a process modifies the creation time of a file.
- Useful at finding "Time-Stomp" attacks.
- Can contain FPs, not every change is malicious
# Event ID 3 - Network Connection
A very noisy source. Generates an event with each net connection. 
- Should rarely start here
# Event ID 4 - Sysmon Service State Change
A change in the actual sysmon service
- Can be useful if the attacker disables sysmon
- Lots of legit activity here as well though
# Event ID 5 - Process Termination
This might aid us in detecting when attackers kill key processes or use sacrificial ones. For instance, Cobalt Strike often spawns temporary processes like werfault, the termination of which would be logged here, as well as the creation in ID 1.
# Event ID 6 - Driver Loaded
Occurs whenever a driver is loaded in. 
- Can be useful for BYOD attacks, however the attack is uncommon
# Event ID 7 - Image Loaded
DLL loaded in events. 
- Can be very helpful for detecting DLL hijacking events
- Can track malicious DLL loads
# Event ID 8 - 
# Event ID 9
# Event ID 10 - Process Access
Useful for generating when a memory dump or a remote code injection occurs. This event shows when a handle into another process has been created.
# Event ID 11
# Event ID 12
# Event ID 15
# Event ID 
# Event ID 15 - 
# Event ID 16 - Sysmon Config Change
Logs alterations in Sysmon configuration, useful for spotting tampering.
# Event ID 17
# Event ID 22
# Event ID 23
# Event ID 25
