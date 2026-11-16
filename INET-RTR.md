# Internet-Router — WAN Transit Hop (labeled "internet" in the diagram)
Gi0/0 = to vIOS8 outside (Site 1), Gi0/1 = to vIOS9 outside (Site 2). This is
a plain IP relay — it is **not** an OSPF peer and **not** a BGP peer. It has
no visibility into the GRE/IPsec tunnel either; it just forwards IP packets
between its two directly-connected /30s, the same way a generic ISP hop
would. In a real deployment this device is outside your administrative
control, so hardening here is best-effort/for-lab-completeness only.

```
hostname INET-RTR
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
interface GigabitEthernet0/0
 description Transit to vIOS8 (Site 1 outside)
 ip address 100.64.0.2 255.255.255.252
 no cdp enable
!
interface GigabitEthernet0/1
 description Transit to vIOS9 (Site 2 outside)
 ip address 100.64.0.5 255.255.255.252
 no cdp enable
!
! No OSPF, no BGP. Static routes only - this device does not participate
! in either site's routing domain, it only needs to know how to reach each
! site's internal /16 so the IP SLA-triggered unencrypted fallback path
! (see vIOS8/vIOS9 floating static routes) actually has somewhere to go.
! The GRE/IPsec tunnel itself doesn't need these routes - it's addressed
! to the vIOS outside IPs directly, which are already connected routes here.
ip route 10.10.0.0 255.255.0.0 100.64.0.1
ip route 10.20.0.0 255.255.0.0 100.64.0.6
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

Notes:
- The two static routes here are what actually make the **unencrypted IP
  SLA fallback path** end-to-end functional. Without them, if the tunnel
  ever goes down, vIOS8/vIOS9 would have a floating static route pointing
  at this router, but this router wouldn't know where to send the traffic
  next and would black-hole it.
- The primary encrypted path (BGP-over-tunnel) never depends on these
  routes — GRE-encapsulated packets are addressed to the vIOS outside IPs,
  which this router already reaches via its own connected interfaces.
- If your lab's "internet" node is meant to be fully out-of-scope /
  unmanaged (representing a real ISP you don't control), you can strip the
  hardening block and just keep the two interfaces + two static routes.
