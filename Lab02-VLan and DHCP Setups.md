# Lab 
02 — VLANs, Trunking, Inter-VLAN Routing and DHCP

**Log Type:** `[Simulation]`
**Date:** November 4, 2026
**System / Hardware Profile:** Cisco Packet Tracer. One router (PT8200, hostname RT1), one 2960-24TT switch (hostname SW1), four PCs
**Test Equipment:** Packet Tracer CLI (`show vlan`, `show interfaces trunk`, `show ip interface brief`, `show ip dhcp binding`), PC command prompt (`ping`)
**Category:** Layer 2 / Layer 3 Networking

---

## 1. Objective & Purpose

I wanted to split four PCs into two VLANs, let them talk to each other across VLANs through a router, and have the router hand out their IP addresses with DHCP.

- VLAN 10 (Outbound): 10.0.0.0/24, PC0 and PC1 on Fa0/1 and Fa0/2
- VLAN 20 (Inbound): 20.0.0.0/24, PC2 and PC3 on Fa0/3 and Fa0/4
- Switch port Gi0/1 is a trunk to the router's Gi0/0/0

This is a router-on-a-stick setup: one physical router port, one sub-interface per VLAN.

## 2. Safety

This lab is a simulation, so there were no physical hazards. In a real closet I would check power and labeling before touching any cables or ports.

## 3. Procedure

1. Gave both devices their own hostnames (SW1 and RT1).
2. Created VLAN 10 and VLAN 20 on the switch and named them from inside each VLAN's own config mode.
3. Set Fa0/1 to Fa0/4 as access ports in the right VLANs, setting both the mode and the VLAN on each port.
4. Set Gi0/1 as a trunk.
5. On the router, created sub-interfaces Gi0/0/0.10 and Gi0/0/0.20 with `encapsulation dot1Q` first, then each VLAN's gateway address (10.0.0.1 and 20.0.0.1, /24).
6. Excluded both gateway addresses from DHCP, then built one DHCP pool per VLAN, each with its own network and default router.
7. Set all four PCs to DHCP.
8. Saved both devices.

## 4. Diagnostics

Problems I hit and how I found them:

- **Wrong mask on the PCs.** I first used 255.0.0.0 (/8) because I thought that was the "default" for 10.x.x.x. With /8 the PCs think the whole 10.x.x.x range is one network. I changed it to /24 to match the router and the DHCP pools.
- **Confused naming a VLAN with naming a device.** `hostname` names the device. `name` inside `vlan 10` names the VLAN.
- **Sub-interfaces with a VLAN tag but no IP address.** I read the saved config and saw `encapsulation dot1Q 10` with `no ip address`. That meant no gateway existed, so nothing could route and DHCP could not match a pool to that interface.
- **Missing DHCP exclusion for 20.0.0.1.** The router logged `DHCPD-4-PING_CONFLICT: DHCP address conflict: server pinged 20.0.0.1`. The server tried to hand out the router's own address.
- **Typed the exclusion on the wrong device.** My first attempt went into SW1. The switch accepted it, but it does nothing there because the router is the DHCP server. I confirmed this because `show ip dhcp binding` on the switch was empty and the router's saved config still had only one exclusion.

## 5. Observations

- A link and port can look fine and still not carry traffic. Each layer needs its own check.
- A trunk only matters if the VLANs exist on the switch, and `show interfaces trunk` showed 1, 10 and 20 active.
- The router picks a DHCP pool by matching the subnet of the interface the request arrives on, so the sub-interface address has to be right first.
- Pool and hostname details are easy to put on the wrong device. The prompt shows which one you're on.

## 6. Verification & Final Results

- `show vlan` showed VLAN 10 (Outbound) on Fa0/1 and Fa0/2, VLAN 20 (Inbound) on Fa0/3 and Fa0/4.
- `show interfaces trunk` showed Gi0/1 trunking, 802.1q, native VLAN 1.
- `show ip dhcp binding` on RT1 showed four leases: 10.0.0.2, 10.0.0.3, 20.0.0.2 and 20.0.0.3.
- PC0 pinged both gateways (10.0.0.1 and 20.0.0.1), 4/4 replies, TTL 255.
- PC0 pinged 20.0.0.2 and 20.0.0.3 across VLANs, 4/4 replies on the repeat run, TTL 127. The first packet of the first run was lost, which is the normal first-packet ARP drop.
- Both devices saved to startup-config.

## 7. Prevention & Best Practices

- Look at the prompt before pressing Enter. Hostnames are there to show which device I'm on.
- Exclude gateway addresses before creating DHCP pools.
- Set both the mode and the VLAN on every access port.
- Always `show` the result after a change, and save last.
- Check TTL in ping replies. TTL 255 means the router answered, and 127 means the packet crossed one router hop.
- Next time, add `description` lines to the router port and the trunk so they are labeled for whoever works on them later.

**Skills Demonstrated:** `VLANs` `802.1Q Trunking` `Router-on-a-Stick` `Inter-VLAN Routing` `DHCP` `Subnetting` `IOS CLI` `Troubleshooting` `Documentation`
