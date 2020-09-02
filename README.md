# How to interconnect multiple Virtual Wan

Thank you Daniel Mauser's post to start the disccustion about how to connect mutiple virtual wan.
https://github.com/dmauser/Lab/tree/master/VWAN/Multi-vWAN-Dev-Prod

This article is a PoC for that article and use a "all in Azure" approch to connect mutiple virtual wan.

## Scenarios
Azure Virtual WAN is a networking service that brings many networking, security, and routing functionalities together to provide a single operational interface. 
There are some scenarios may need to interconnect multiple virtual wan into one routing domain.
1. More branchs or vnet need to be added to the virutal wan.
Resource	Limit
- Virtual WAN hubs per region	1
- VPN (branch) connections per hub	1,000
- VNet connections per hub	500 minus total number of hubs in Virtual WAN
2. There are multiple virtual wan is created by different organazations and need to be connected together.

## Network Diagram
We have 2 virtual wan used in the testing.
- vwan global
- vwan yosemite

Key points here for this solution
- Use cisco 1000v on the azure side to interconnet 2 virtual wan
- Use bgp as override feature to override the virtual wan 65515 AS as a local AS number (which is 4 in this artical)



