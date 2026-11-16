# BR2-SW15 — Access Switch (VLAN 20)

```
hostname BR2-SW15
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
!
vtp mode transparent
spanning-tree mode rapid-pvst
spanning-tree portfast bpduguard default
!
vlan 10
 name USERS_VLAN10
vlan 20
 name USERS_VLAN20
vlan 30
 name USERS_VLAN30
vlan 40
 name USERS_VLAN40
!
interface GigabitEthernet0/0
 description Host port
 switchport mode access
 switchport access vlan 20
 switchport port-security
 switchport port-security maximum 2
 switchport port-security violation shutdown
 switchport port-security mac-address sticky
 spanning-tree portfast
 spanning-tree bpduguard enable
 no cdp enable
!
interface range GigabitEthernet0/1 - 2
 description Unused (host count reduced to 1 per access switch)
 shutdown
!
interface GigabitEthernet0/3
 description Uplink to BR2-L3-SW1 Gi0/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40
 no cdp enable
!
interface GigabitEthernet1/0
 description Uplink to BR2-L3-SW2 Gi0/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40
 no cdp enable
!
interface range GigabitEthernet0/4 - 23
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
