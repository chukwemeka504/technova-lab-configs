# vIOS9 — Site 2 Edge Router (AS 65002)
inside = Gi0/0 (to ASAv2 outside), outside = Gi0/1 (to Internet-Router Gi0/1 —
transit hop to Site 1), Tunnel0 = GRE-over-IPsec to vIOS8 (built end-to-end,
riding over the Internet-Router as a plain IP transit hop that is NOT an
OSPF or BGP peer).

```
hostname vIOS9
no ip domain-lookup
ip domain-name lab.local
enable secret [REDACTED]
username netadmin privilege 15 secret [REDACTED]
banner motd ^C
AUTHORIZED ACCESS ONLY. All activity is logged and monitored.
^C
!
service password-encryption
service tcp-keepalives-in
service tcp-keepalives-out
no service pad
no ip http server
no ip http secure-server
no ip source-route
no cdp run
!
interface Loopback0
 ip address 10.20.255.254 255.255.255.255
!
interface GigabitEthernet0/0
 description Inside - to ASAv2 outside (Gi0/2)
 ip address 10.20.100.6 255.255.255.252
 no cdp enable
!
interface GigabitEthernet0/1
 description Outside - to Internet-Router Gi0/1 (transit to Site 1)
 ip address 100.64.0.6 255.255.255.252
 no cdp enable
!
! Internet-Router is a plain IP transit hop - not OSPF, not BGP. Default
! route is the only way vIOS9 reaches vIOS8's outside address (100.64.0.1)
! and the far-side Site1 fallback prefix.
ip route 0.0.0.0 0.0.0.0 100.64.0.5
!
! --- IKEv2 / IPsec for GRE tunnel protection (AES-256 / SHA-384) ---
crypto ikev2 proposal IKEV2-PROPOSAL
 encryption aes-cbc-256
 integrity sha384
 group 24
!
crypto ikev2 policy IKEV2-POLICY
 proposal IKEV2-PROPOSAL
!
crypto ikev2 keyring IKEV2-KEYRING
 peer VIOS8
  address 100.64.0.1
  pre-shared-key [REDACTED]
!
crypto ikev2 profile IKEV2-PROFILE
 match identity remote address 100.64.0.1 255.255.255.255
 authentication local pre-share
 authentication remote pre-share
 keyring local IKEV2-KEYRING
!
crypto ipsec transform-set TS-AES256-SHA384 esp-aes 256 esp-sha384-hmac
 mode transport
!
crypto ipsec profile TUNNEL-PROTECTION
 set transform-set TS-AES256-SHA384
 set ikev2-profile IKEV2-PROFILE
!
interface Tunnel0
 description GRE-over-IPsec to vIOS8 (Site 1) - built end-to-end across Internet-Router
 ip address 172.16.0.2 255.255.255.252
 tunnel source GigabitEthernet0/1
 tunnel destination 100.64.0.1
 tunnel protection ipsec profile TUNNEL-PROTECTION
 no cdp enable
!
! --- OSPF: internal side only (inside interface + loopback) ---
router ospf 1
 router-id 10.20.255.254
 network 10.20.100.4 0.0.0.3 area 0
 network 10.20.255.254 0.0.0.0 area 0
 redistribute bgp 65002 subnets route-map BGP-TO-OSPF
!
! --- BGP: eBGP peering with Site 1 over the tunnel ---
router bgp 65002
 bgp router-id 10.20.255.254
 bgp log-neighbor-changes
 neighbor 172.16.0.1 remote-as 65001
 neighbor 172.16.0.1 description eBGP to vIOS8 (Site 1)
 neighbor 172.16.0.1 password [REDACTED]
 network 10.20.0.0 mask 255.255.0.0
 redistribute ospf 1 route-map OSPF-TO-BGP
!
! --- Route-maps: tag-based redistribution loop prevention ---
! Routes coming IN from Site1 via BGP get tagged 65001 when pushed into OSPF.
route-map BGP-TO-OSPF permit 10
 set tag 65001
!
! When pushing OSPF routes OUT to BGP, never re-advertise anything tagged
! 65001 (i.e. don't hand Site1's own routes back to Site1).
route-map OSPF-TO-BGP deny 10
 match tag 65001
route-map OSPF-TO-BGP permit 20
!
! --- IP SLA + object tracking for WAN failover ---
! Tests base reachability to the peer's outside IP directly over the
! outside interface (NOT through the tunnel), independent of BGP/tunnel state.
ip sla 1
 icmp-echo 100.64.0.1 source-interface GigabitEthernet0/1
 frequency 10
ip sla schedule 1 life forever start-time now
track 1 ip sla 1 reachability
!
! Floating route to Site1, AD 200 (worse than eBGP's AD 20) - only used if
! the BGP-learned route disappears AND base WAN connectivity is confirmed up.
! Unencrypted last-resort path, not the primary path. Next-hop is the
! Internet-Router (100.64.0.5), which needs its own static route toward
! 10.10.0.0/16 - see the Internet-Router config.
ip route 10.10.0.0 255.255.0.0 100.64.0.5 200 track 1
!
interface range GigabitEthernet0/2 - 3
 shutdown
!
ip ssh version 2
crypto key generate rsa modulus 2048
line con 0
 login local
 exec-timeout 5 0
line vty 0 4
 login local
 transport input ssh
 access-class MGMT-ACCESS in
 exec-timeout 5 0
!
ip access-list standard MGMT-ACCESS
 permit 10.0.0.0 0.255.255.255
 deny any log
!
ntp server 10.0.0.100
logging buffered 16384
end
```

Notes: same design notes as vIOS8 (mode transport, optional BGP MD5, network
statement) — mirrored addressing throughout.
