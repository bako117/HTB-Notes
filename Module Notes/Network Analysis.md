# Network Analysis

## Wireshark
Wireshark is a critical piece of tooling for analyzing network traffic files, known as pcaps. 

### How To's

**To export objects from Wireshark foudn in the streams go to:**
File > Export Objects > Then Select the protocal you want > then export the object you want

**To discover the top talking addresses:**
Statistics > Ipv4 Statistics > All addresses

**To follow a packet stream:**
Right-click a packet in the stream > Follow

### Filter Repo

**Filter for a specific tcp port:**
`tcp.port == 3389`

**Filter for a specific protocol**
Just type in the protocol, ex include
- http
- rdp
- tcp
- udp 
- dns

### Decrpyting RDP Lab

First get the key neccessary. This is a process not covered in the lab. 


