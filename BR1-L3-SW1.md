# BR1-L3-SW1 — Core/Distribution Switch (Site 1)
VRRP master: VLAN10, VLAN20 | STP root: VLAN10, VLAN20 | ASAv is now dual-homed — this is inside-leg 1 (BR1-L3-SW2 carries inside-leg 2).

```
hostname BR1-L3-SW1
no ip domain-lookup
ip domain-name lab.local
ip routing
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
!
vtp mode transparent
spanning-tree mode rapid-pvst
spanning-tree vlan 10,20 priority 4096
spanning-tree vlan 30,40 priority 8192
!
vlan 10
 name USERS_VLAN10
vlan 20
 name USERS_VLAN20
vlan 30
 name USERS_VLAN30
vlan 40
 name USERS_VLAN40
vlan 99
 name CORE_TRANSIT
!
interface Loopback0
 ip address 10.10.255.1 255.255.255.255
!
interface GigabitEthernet0/0
 description Downlink to BR1-SW1 (all VLANs trunked)
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40
 spanning-tree guard root
 no cdp enable
!
interface GigabitEthernet0/1
 description Downlink to BR1-SW2 (all VLANs trunked)
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40
 spanning-tree guard root
 no cdp enable
!
interface GigabitEthernet0/2
 description Downlink to BR1-SW3 (all VLANs trunked)
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40
 spanning-tree guard root
 no cdp enable
!
interface GigabitEthernet0/3
 description Downlink to BR1-SW4 (all VLANs trunked)
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40
 spanning-tree guard root
 no cdp enable
!
interface GigabitEthernet1/0
 description Uplink to ASAv1 inside1 (Gi0/1) - HA pair with BR1-L3-SW2
 no switchport
 ip address 10.10.100.1 255.255.255.252
 no cdp enable
!
interface range GigabitEthernet1/1 - 3
 description EtherChannel to BR1-L3-SW2 (3-member LACP)
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40,99
 channel-group 1 mode active
 no cdp enable
!
interface Port-channel1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40,99
!
interface Vlan10
 ip address 10.10.10.2 255.255.255.0
 ip ospf priority 0
 vrrp 10 ip 10.10.10.1
 vrrp 10 priority 150
 vrrp 10 preempt
!
interface Vlan20
 ip address 10.10.20.2 255.255.255.0
 ip ospf priority 0
 vrrp 20 ip 10.10.20.1
 vrrp 20 priority 150
 vrrp 20 preempt
!
interface Vlan30
 ip address 10.10.30.2 255.255.255.0
 ip ospf priority 0
 vrrp 30 ip 10.10.30.1
 vrrp 30 priority 100
 vrrp 30 preempt
!
interface Vlan40
 ip address 10.10.40.2 255.255.255.0
 ip ospf priority 0
 vrrp 40 ip 10.10.40.1
 vrrp 40 priority 100
 vrrp 40 preempt
!
interface Vlan99
 description Core-to-core OSPF transit
 ip address 10.10.99.1 255.255.255.252
!
router ospf 1
 router-id 10.10.255.1
 passive-interface Vlan10
 passive-interface Vlan20
 passive-interface Vlan30
 passive-interface Vlan40
 network 10.10.10.0 0.0.0.255 area 0
 network 10.10.20.0 0.0.0.255 area 0
 network 10.10.30.0 0.0.0.255 area 0
 network 10.10.40.0 0.0.0.255 area 0
 network 10.10.99.0 0.0.0.3 area 0
 network 10.10.100.0 0.0.0.3 area 0
 network 10.10.255.1 0.0.0.0 area 0
!
interface range GigabitEthernet0/5, GigabitEthernet0/6 - 23
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
