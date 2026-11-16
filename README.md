# TechNova Ltd. — Dual-Branch Network Lab Configurations

This repository contains the full running configurations for every device in the
EVE-NG laboratory topology built for the thesis *"Design and Implementation of a
Resilient, Segmented Dual-Branch Network"* (University of Dunaújváros).

All passwords, enable secrets, and pre-shared keys have been redacted and replaced
with `[REDACTED]`. No other content has been altered from the live device configuration.

## Devices

| File | Device | Role |
|---|---|---|
| vIOS8.md | vIOS8 | Site 1 (HQ) Edge Router, AS 65001 |
| vIOS9.md | vIOS9 | Site 2 (Branch) Edge Router, AS 65002 |
| ASAv1.md | ASAv1 | HQ Outer Firewall |
| ASAv2.md | ASAv2 | Branch Firewall |
| BR1-L3-SW1.md | BR1-L3-SW1 | HQ Core Switch 1 |
| BR1-L3-SW2.md | BR1-L3-SW2 | HQ Core Switch 2 |
| BR2-L3-SW1.md | BR2-L3-SW1 | Branch Core Switch 1 |
| BR2-L3-SW2.md | BR2-L3-SW2 | Branch Core Switch 2 |
| BR1-SW1.md | BR1-SW1 | HQ Access Switch (VLAN 10) |
| BR1-SW2.md | BR1-SW2 | HQ Access Switch (VLAN 20) |
| BR1-SW3.md | BR1-SW3 | HQ Access Switch (VLAN 30) |
| BR1-SW4.md | BR1-SW4 | HQ Access Switch (VLAN 40) |
| BR2-SW14.md | BR2-SW14 | Branch Access Switch (VLAN 10) |
| BR2-SW15.md | BR2-SW15 | Branch Access Switch (VLAN 20) |
| BR2-SW16.md | BR2-SW16 | Branch Access Switch (VLAN 30) |
| BR2-SW17.md | BR2-SW17 | Branch Access Switch (VLAN 40) |
| INET-RTR.md | INET-RTR | WAN Transit Router |

## Context

These configurations support the thesis's Chapter 5 (Implementation) and the
validation results reported in Chapter 6 (Discussion). See the thesis document
itself for the full topology diagram, addressing plan, and design rationale.
