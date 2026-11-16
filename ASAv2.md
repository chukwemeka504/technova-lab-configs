# ASAv2 — Site 2 Firewall
Now dual-homed for HA: inside1 = Gi0/1 (to BR2-L3-SW1 Gi1/0), inside2 = Gi0/0
(to BR2-L3-SW2 Gi1/0), outside = Gi0/2 (to vIOS9 Gi0/0). Pure OSPF transit
device between core and edge router — no BGP here.

```
hostname ASAv2
domain-name lab.local
enable password [REDACTED] encrypted
username netadmin password [REDACTED] encrypted privilege 15
!
banner motd AUTHORIZED ACCESS ONLY. All activity is logged and monitored.
!
interface GigabitEthernet0/0
 description Inside leg 2 - to BR2-L3-SW2 Gi1/0
 nameif inside2
 security-level 100
 ip address 10.20.100.10 255.255.255.252
 no shutdown
!
interface GigabitEthernet0/1
 description Inside leg 1 - to BR2-L3-SW1 Gi1/0
 nameif inside1
 security-level 100
 ip address 10.20.100.2 255.255.255.252
 no shutdown
!
interface GigabitEthernet0/2
 description Outside - to vIOS9 Gi0/0
 nameif outside
 security-level 0
 ip address 10.20.100.5 255.255.255.252
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
access-list OUTSIDE-IN extended permit ip object SITE1-INTERNAL object SITE2-INTERNAL
access-list OUTSIDE-IN extended deny ip any any log
access-group OUTSIDE-IN in interface outside
!
! No NAT between the two private sites - routed end to end via OSPF/BGP/tunnel
nat (inside1,outside) source static SITE2-INTERNAL SITE2-INTERNAL destination static SITE1-INTERNAL SITE1-INTERNAL no-proxy-arp route-lookup
nat (inside2,outside) source static SITE2-INTERNAL SITE2-INTERNAL destination static SITE1-INTERNAL SITE1-INTERNAL no-proxy-arp route-lookup
!
router ospf 1
 router-id 10.20.100.2
 network 10.20.100.0 255.255.255.252 area 0
 network 10.20.100.8 255.255.255.252 area 0
 network 10.20.100.4 255.255.255.252 area 0
 area 0
!
route outside 0.0.0.0 0.0.0.0 10.20.100.6 254
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

Notes: same design notes as ASAv1 (dual inside interfaces need
`same-security-traffic permit inter-interface`, two interface-specific NAT
statements, three OSPF /30 networks for automatic HA reconvergence).
