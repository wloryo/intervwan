# How to interconnect multiple Virtual Wan by using cisco virtual router c1000v

Thanks Daniel Mauser's [post](https://github.com/dmauser/Lab/tree/master/VWAN/Multi-vWAN-Dev-Prod) to start the discussion about how to connect multiple virtual wan.


This article is a PoC for that article and use a "all in Azure" approach to connect multiple virtual wan.

## Scenarios
Azure Virtual WAN is a networking service that brings many networking, security, and routing functionalities together to provide a single operational interface. 
There are some scenarios may need to interconnect multiple virtual wan into one routing domain.
1. More branches or vnet need to be added to the virtual wan.

Resource	Limit (202009)
 - Virtual WAN hubs per region:	1
 - VPN (branch) connections per hub:	1,000
 - VNet connections per hub:	500 minus total number of hubs in Virtual WAN

2. There are multiple virtual wan is created by different organizations and need to be connected together.

## The Lab
### Network Diagram

We have 2 virtual wan used in the testing.
- vwan global
- vwan yosemite

<img src="/1.png" width="90%">

### Solution Highlight

highlight for this solution
- Use cisco 1000v as the NVA within the a VNET to interconnect 2 virtual wan
- Use bgp AS override feature to override the virtual wan 65515 AS number as a local AS number (which is 4 in this article)

Cisco c1000v BGP settings

Full configraion is uploaded [here](/c1000v-intervwan.conf.txt)

```
router bgp 4
 bgp log-neighbor-changes
 neighbor 10.166.3.12 remote-as 65515
 neighbor 10.166.3.12 ebgp-multihop 5
 neighbor 10.166.3.12 update-source Loopback4
 neighbor 10.166.3.13 remote-as 65515
 neighbor 10.166.3.13 ebgp-multihop 5
 neighbor 10.166.3.13 update-source Loopback4
 neighbor 10.167.1.12 remote-as 65515
 neighbor 10.167.1.12 ebgp-multihop 5
 neighbor 10.167.1.12 update-source Loopback4
 neighbor 10.167.1.13 remote-as 65515
 neighbor 10.167.1.13 ebgp-multihop 5
 neighbor 10.167.1.13 update-source Loopback4
 !
 address-family ipv4
  network 10.100.101.0 mask 255.255.255.0
  neighbor 10.166.3.12 activate
  neighbor 10.166.3.12 as-override
  neighbor 10.166.3.13 activate
  neighbor 10.166.3.13 as-override
  neighbor 10.167.1.12 activate
  neighbor 10.167.1.12 as-override
  neighbor 10.167.1.13 activate
  neighbor 10.167.1.13 as-override
  maximum-paths 4
 exit-address-family
```

### Routing tables
1. vwan yosemite - vhub westus

<img src="/3.png" width="100%">

2. vwan global - vhub eastus

<img src="/4.png" width="100%">

3. c1000v

Filtered some routes which are not related to the testing.

```
c1000v-intervwan#show ip route
S*    0.0.0.0/0 [254/0] via 10.170.0.1
B        10.2.0.0/16 [20/0] via 10.166.3.13, 21:02:56        <- vwan global eastus vnet
                     [20/0] via 10.166.3.12, 21:02:56
B        10.50.0.0/16 [20/0] via 10.167.1.13, 21:02:56        <- vwan yosemite westus vnet
                      [20/0] via 10.167.1.12, 21:02:56
B        10.51.0.0/16 [20/0] via 10.167.1.13, 07:11:52        <- vwan yosemite eastus vnet
                      [20/0] via 10.167.1.12, 07:11:52
C        10.100.101.0/24 is directly connected, Loopback101
L        10.100.101.1/32 is directly connected, Loopback101
B        10.103.1.4/32 [20/0] via 10.166.3.13, 21:02:56        <- vwan global eastasia VPN site
                       [20/0] via 10.166.3.12, 21:02:56
S        10.166.3.12/32 is directly connected, Tunnel3        <- vwan yosemite bgp peer 1
S        10.166.3.13/32 is directly connected, Tunnel4        <- vwan yosemite bgp peer 2
B        10.167.1.0/24 [20/0] via 10.167.1.13, 21:02:56
                       [20/0] via 10.167.1.12, 21:02:56
S        10.167.1.12/32 is directly connected, Tunnel2        <- vwan global bgp peer 2
S        10.167.1.13/32 is directly connected, Tunnel1        <- vwan global bgp peer 1
C        10.170.0.0/24 is directly connected, GigabitEthernet1
L        10.170.0.5/32 is directly connected, GigabitEthernet1
C        10.171.1.1/32 is directly connected, Loopback4
```

### Connectbility test
VNET to VNET

1. vwan yosemite westus -> vwan global east us

<img src="/6.1.png" width="40%">

2. vwan yosemite eastus -> vwan global east us

<img src="/6.2.png" width="40%">

VNET to Branch

3. vwan yosemite westus -> vwan global VPN East Asia

<img src="/6.3.png" width="40%">

Connectability for both vwan is confirmed depend on the those ping test.

## Challenges
1. The increasing latency (up to 350ms from eastus to eastus) for test #2. 
Due to packet flow through a hairpin likely path which increase the latency.

<img src="/5.png" width="60%">

2. The c1000v is connected to both virtual wan as a "VPN site" which support up to 1 gbps throughput.
This is a bandwidth over subscription design in the normal case.

## Conclusion
This PoC had provided an inter connect multiple virtual wan solution by using a cisco c1000v router.
This is still a PoC level idea and need to be tested when go for a production environment.

