# Lab 01 — Building and Troubleshooting a Small LAN in Packet Tracer

**Log Type:** `[Simulation]`
**Date:** October 30, 2026
**System / Hardware Profile:** Cisco Packet Tracer 9.0.1. One router (R1), one switch, two hosts (Laptop0 and PC0). Network 192.168.10.0/24.
**Test Equipment:** Router console (Cisco IOS CLI), host command prompt (`ping`), `show ip interface brief`, `show interfaces status`
**Category:** Networking: Layer 1, 2 and 3 troubleshooting (simulated)

---

> **Simulation: practice lab, not hands-on experience.** I did this in Packet Tracer. The `.pkt` file is in this repo.

## 1. Objective & Purpose
Build a small network with a router, a switch and two computers, and make sure they can all ping each other. I wanted to practice cabling, IP addressing, and finding out why a ping fails by checking one layer at a time.

## 2. Safety
Nothing physical can go wrong in a simulation. On real equipment I would power down before moving cables, use ESD protection, and label both ends of every cable first.

## 3. Procedure
1. Placed the router, switch, Laptop0 and PC0 and connected them with straight-through cables.
2. Made an IP plan: router 192.168.10.1, Laptop0 192.168.10.10, PC0 192.168.10.11. All /24, and both computers use 192.168.10.1 as the gateway.
3. Set up the router from the console:
```
enable
configure terminal
hostname R1
interface gigabitEthernet 0/0/0
 ip address 192.168.10.1 255.255.255.0
 no shutdown
end
```
4. Typed the IP, mask and gateway into both computers.
5. Tested with `ping` from the command prompts.

**Final cabling**
- Switch Fa0/1 to Router Gi0/0/0
- Switch Fa1/1 to Laptop0 Fa0
- Switch Fa2/1 to PC0 Fa0

## 4. Diagnostics
Every ping failed with 100% loss and the router interface said `down/down`. I checked from the bottom layer up:
1. **Layer 1, router cable.** The cable was in the router's console port, which is only for management. I moved it to the data port. Later it ended up in the switch's console port, so I moved it to Fa0/1.
2. **Layer 1, computer cables.** Both were plugged into the RS 232 port instead of FastEthernet0, and they were crossover cables, which are the wrong type for switch-to-computer. I swapped them for straight-through cables in FastEthernet0.
3. **Layer 2.** I ran `show interfaces status` on the switch and compared the connected ports to my devices. One link wasn't connected.
4. **Layer 3.** The router IP had a typo, `192.169.10.1` instead of `192.168.10.1`, so it didn't match the computers' gateway.
5. **Narrowing it down.** Once the computer cables were fixed, the two computers could ping each other but not the router. That told me the computer side was fine and the problem was the router's cable.

## 5. Observations
- A router interface can be `down/down` just because the cable is in the wrong port. Check the ports first.
- Pinging your own IP always works, so it doesn't prove anything.
- The computers could ping each other before the router worked, so the switch side was good and only the router link was wrong.
- The router's line protocol went up and down while I moved cables, which is normal.
- Reply TTL was 128 from the PC and 255 from the router.

## 6. Verification & Final Results
- Laptop0 to PC0 (`ping 192.168.10.11`): 4 of 4 replies, 0% loss.
- Laptop0 to router (`ping 192.168.10.1`): 4 of 4 replies, 0% loss.
- `show ip interface brief` on R1: Gi0/0/0 has 192.168.10.1, status `up`, protocol `up`.
- Saved with `copy running-config startup-config`.

## 7. Prevention & Best Practices
- Check the port on both ends of every cable before configuring anything.
- Use straight-through cables from router to switch and switch to computer.
- Look for link lights on both ends before testing.
- After typing an IP, check it with `show ip interface brief`.
- Ping computer to computer first, then computer to router, to tell a Layer 2 problem from a Layer 3 problem.
- Write the IP plan down first and save the config once it works.

**Skills Demonstrated:** `Networking` `OSI Model` `Layer 1 Troubleshooting` `Layer 2 Switching` `IP Addressing` `Cisco IOS CLI` `Cabling` `Packet Tracer` `Documentation`
