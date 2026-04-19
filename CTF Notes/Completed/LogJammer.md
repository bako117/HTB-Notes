First I took the windows event logs and parsed them with Kape on my Flare VM at home. then I launched timeline explorer. 

Question 1: I utilized searching for 4624 and found the soonest logon, then I used the timestamp. 

Question 2, 3: I sort the rules by the provider and began looking through added firewall rules. I noticed a lot of rules were added prior to first logon. That didn't make sense so I looked newer until I found one that was obviously suspicious. The log file event shows the rule's direction.

Question 4: Found the event with event code 4719 for audit policy changes. Then you have to run a command on your machine and match the subcategory GUID
`auditpol /list /subcategory:* /v`

Question 5, 6, 7: I found the "Scheduled Task Created" event in timeline explorer, then I utilized Chat GPT to make it look much nicer and in XML. The answers were a part of the XML dump.

Question 8, 9, 10: I found the event within the win defender logs which shows as a "Tool" being discovered. I used the tool's name and within that event are the answers to 9 & 10.

Question 11: I looked at PowerShell script control events to find this one. It will have prompt event's on either side. I needed to sot by time to look at recent items as to not get confused. 

Question 12: Event code 104s, one of these events stand out and its that. I thought it would be 1102, I guess I was wrong, idrk.

