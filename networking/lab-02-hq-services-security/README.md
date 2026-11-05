# Lab 02: HQ Services and Security

Second phase of the Solace Financial Group HQ network in Cisco Packet Tracer.
This part covers HQ services (DHCP and DNS on HQ-SRV-01), faster HSRP failover,
and the failover tests. Security hardening and forensics follow in later sections.

## A3. HSRP timers and failover

| Test | Packets lost |
|---|---|
| Week 1 baseline (default timers) | 4 |
| Fast timers (hello 1 / hold 3), run 1 | 1 |
| Fast timers, later runs | 0 |

Hello 1 / hold 3 cut failover loss from 4 packets to 0-1. Packet Tracer does not
keep these timers in the running configuration. As a result, they could not be
saved in the startup configuration and had to be re-applied after every reload
(see configs/hsrp-timers.txt). On real devices, the timers are accepted in the
running configuration and can be saved in the startup configuration.

### Whole-switch failure test

All CD01 links were shut down to simulate a switch failure and test the failover.
CD02 took over routing for groups 10, 20, 30 and 100 and was Active for all seven
groups (failover-cd01-down-cd02-standby-brief.png).

All pings from Finance PC-1 (VLAN 10) were successful, with 0% loss:
- the VLAN 10 gateway, 10.0.0.1 (TTL 255)
- IT PC-1, 10.0.0.101, inter-VLAN, routed through CD02 (TTL 127)
- the simulated internet address, 8.8.8.8, on HQ-RTR-01 (TTL 254)

These pings were run after failover completed, so they show that service was
restored, not how long failover took.

After restoring all the links with the "no shutdown" command, preempt returned
groups 10, 20, 30 and 100 to CD01.

Screenshots are in the screenshots folder (failover-cd01-down-* and
failover-cd01-restored-standby-brief.png).

## A4. VLAN 10 shutdown on HQ-CD01 (open issue)

Test: VLAN 10 was shut down on CD01 while pinging from Finance PC-1.

Observation: CD02 took over as Active for group 10 after the hold time. Pings to
10.0.0.1 and to hosts in other VLANs failed, while ping to CD02's SVI (10.0.0.3)
succeeded. Finance PC-1's ARP table resolved 10.0.0.1 to the HSRP virtual MAC
(0000.0c07.ac0a), so Layer 2 reachability to the gateway was intact.

Explanation: A shut SVI is a down interface and sends no HSRP hellos. CD02, hearing
nothing, became Active after the hold time. CD01 still reported itself Active
because it received no hellos from CD02, leaving two Active routers for group 10.
CD01 listed its Standby as unknown. Restoring VLAN 10 on CD01 (no shutdown)
returned CD01 to Active and CD02 to Standby for group 10 (log: Standby -> Active
on CD01, Speak -> Standby on CD02).

Layer 2 path: Acc01 forwards VLANs 10, 20, 30 and 100 on Fa0/20 (Root FWD, toward
CD01) and blocks them on Fa0/19 (Altn BLK), by design. Fa0/19 forwards VLANs 40, 50
and 60. Shutting an SVI does not change spanning tree, so VLAN 10 traffic keeps
using Fa0/20.

Unresolved: Why ICMP to 10.0.0.1 failed while ARP resolved. Asymmetric return
traffic (CD01 still Active for other groups with its VLAN 10 down) can explain the
failed pings to other VLANs but not the failed ping to the virtual IP itself,
which may be a Packet Tracer behaviour. The realistic failure test is the
whole-switch failure in A3, and every ping passed in it.
