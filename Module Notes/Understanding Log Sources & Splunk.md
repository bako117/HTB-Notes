# Understanding Log Sources \& Investigating with Splunk

Splunk is a very popular enterprise SIEM solution. 
## Splunk Architecture

Splunk consists of several layers that work together. The main componenets are: 
* Universal forwarder: A lightweight agent who forwards data to indexers without processing
* Heavy forwarder:  According to Splexicon, heavy forwarders stand out from other types of forwarders as they parse data before forwarding, allowing them to route data based on specific criteria such as event source or type.
* Indexers: Indexers receive data from forwarders, put it into indexes and organize it. Indexers also process user search queries and return data.
* Search Heads: Provide the UI for splunk

![[Pasted image 20260417135500.png]]

