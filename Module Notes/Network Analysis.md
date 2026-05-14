
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

Network traffic analysis can be difficult due to how much traffic there is and how foreign it can be. Being able to identify attacks and discern patterns and trends within the data is crucial. 

# Link Layer Attacks




