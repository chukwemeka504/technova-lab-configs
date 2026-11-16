# ASAv1 — Site 1 Firewall
Now dual-homed for HA: inside1 = Gi0/1 (to BR1-L3-SW1 Gi1/0), inside2 = Gi0/0
(to BR1-L3-SW2 Gi1/0), outside = Gi0/2 (to vIOS8 Gi0/0). Pure OSPF transit
device between core and edge router — no BGP here.

```
hostname ASAv1
domain-name lab.local
enable password [REDACTED] encrypted
username netadmin password [REDACTED] encrypted privilege 15
!
banner motd AUTHORIZED ACCESS ONLY. All activity is logged and monitored.
!
interface GigabitEthernet0/0
 description Inside leg 2 - to BR1-L3-SW2 Gi1/0
 nameif inside2
 security-level 100
 ip address 10.10.100.10 255.255.255.252
 no shutdown
!
interface GigabitEthernet0/1
 description Inside leg 1 - to BR1-L3-SW1 Gi1/0
 nameif inside1
 security-level 100
 ip address 10.10.100.2 255.255.255.252
 no shutdown
!
interface GigabitEthernet0/2
 description Outside - to vIOS8 Gi0/0
 nameif outside
 security-level 0
 ip address 10.10.100.5 255.255.255.252
 no shutdown
!
same-security-traffic permit intra-interface
same-security-traffic permit inter-interface
!
object network SITE1-INTERNAL
 subnet 10.10.0.0 255.255.0.0
object network SITE2-INTERNAL
 subnet 10.20.0.0 255.255.0.0
!
access-list OUTSIDE-IN extended permit ip object SITE2-INTERNAL object SITE1-INTERNAL
access-list OUTSIDE-IN extended deny ip any any log
access-group OUTSIDE-IN in interface outside
!
! No NAT between the two private sites - routed end to end via OSPF/BGP/tunnel
nat (inside1,outside) source static SITE1-INTERNAL SITE1-INTERNAL destination static SITE2-INTERNAL SITE2-INTERNAL no-proxy-arp route-lookup
nat (inside2,outside) source static SITE1-INTERNAL SITE1-INTERNAL destination static SITE2-INTERNAL SITE2-INTERNAL no-proxy-arp route-lookup
!
router ospf 1
 router-id 10.10.100.2
 network 10.10.100.0 255.255.255.252 area 0
 network 10.10.100.8 255.255.255.252 area 0
 network 10.10.100.4 255.255.255.252 area 0
 area 0
!
route outside 0.0.0.0 0.0.0.0 10.10.100.6 254
!
ssh 10.0.0.0 255.0.0.0 management
ssh version 2
ssh timeout 5
aaa authentication ssh console LOCAL
!
no http server enable
!
logging enable
logging buffered informational
no logging trap
no snmp-server enable
!
ntp server 10.0.0.100
!
clock timezone UTC 0
!
icmp deny any outside
icmp permit any inside1
icmp permit any inside2
!
end
```

Notes:
- Both inside legs sit at security-level 100 (equal), so `same-security-traffic
  permit inter-interface` is required or traffic between inside1 and inside2
  (i.e. between the two cores, routed via the ASA if the direct core-core
  EtherChannel were ever down) would be dropped by default.
- Two `nat` statements are needed (one per inside interface) since ASA NAT
  rules are interface-pair specific — traffic could ingress from either core.
- OSPF now has three interior-facing /30 networks: 10.10.100.0 (inside1),
  10.10.100.8 (inside2), 10.10.100.4 (outside, toward vIOS8). This gives OSPF
  two equal-cost paths into the ASA, so if one core-to-ASA link or one core
  itself fails, routing reconverges through the other automatically — true
  HA, not just a physically redundant cable.
- `aaa authentication ssh console LOCAL` is local authentication, not an
  external AAA server, consistent with "no AAA server."
