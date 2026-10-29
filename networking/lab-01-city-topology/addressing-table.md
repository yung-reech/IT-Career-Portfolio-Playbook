# Solace Financial Group — HQ Addressing Table

Base block: `10.0.0.0/8`. HQ currently uses `10.0.0.0/24`. VLSM with ~2x growth headroom; WAN links use exact-fit /30.

## VLANs and subnets

| VLAN | Name | Subnet | Mask | Usable range | Broadcast | HSRP virtual IP (gateway) |
|---|---|---|---|---|---|---|
| 10 | Finance | 10.0.0.0/26 | 255.255.255.192 | .1 – .62 | .63 | 10.0.0.1 |
| 20 | HR | 10.0.0.64/27 | 255.255.255.224 | .65 – .94 | .95 | 10.0.0.65 |
| 30 | IT_Server | 10.0.0.96/27 | 255.255.255.224 | .97 – .126 | .127 | 10.0.0.97 |
| 60 | Voice | 10.0.0.128/27 | 255.255.255.224 | .129 – .158 | .159 | 10.0.0.129 |
| 50 | Guest | 10.0.0.160/27 | 255.255.255.224 | .161 – .190 | .191 | 10.0.0.161 |
| 40 | Corporate_WiFi | 10.0.0.192/27 | 255.255.255.224 | .193 – .222 | .223 | 10.0.0.193 |
| 100 | Management | 10.0.0.248/29 | 255.255.255.248 | .249 – .254 | .255 | 10.0.0.254 |
| 199 | Native | not routed | | | | |
| 299 | Black_Hole | not routed | | | | |

## Point-to-point links

| Link | Subnet | Side A | Side B | Status |
|---|---|---|---|---|
| HQ-RTR-01 G0/1 ↔ HQ-CD01 G0/1 | 10.0.0.240/30 | RTR .241 | CD01 .242 | Built |
| HQ-RTR-01 G0/2 ↔ HQ-CD02 G0/2 | 10.0.0.244/30 | RTR .245 | CD02 .246 | Built |
| HQ ↔ Branch A | 10.0.0.224/30 | HQ S0/2/0 .225 | Branch A .226 | Planned |
| HQ ↔ Branch B | 10.0.0.228/30 | HQ S0/2/1 .229 | Branch B .230 | Planned |
| HQ ↔ Branch C | 10.0.0.232/30 | HQ S0/3/0 .233 | Branch C .234 | Planned |
| HQ ↔ Data Center | 10.0.0.236/30 | HQ S0/3/1 .237 | Data Center .238 | Planned |

## Device addresses (SVIs)

| VLAN | HQ-CD01 | HQ-CD02 |
|---|---|---|
| 10 | 10.0.0.2 | 10.0.0.3 |
| 20 | 10.0.0.66 | 10.0.0.67 |
| 30 | 10.0.0.98 | 10.0.0.99 |
| 40 | 10.0.0.194 | 10.0.0.195 |
| 50 | 10.0.0.162 | 10.0.0.163 |
| 60 | 10.0.0.130 | 10.0.0.131 |
| 100 (mgmt) | 10.0.0.249 | 10.0.0.250 |

Access switch management (VLAN 100, default gateway 10.0.0.254): HQ-Acc01 `10.0.0.251`, HQ-Acc02 `10.0.0.252`, HQ-Acc03 `10.0.0.253`.

## HSRP and STP

| Group / VLAN | CD01 priority | CD02 priority | Active | STP root |
|---|---|---|---|---|
| 10 Finance | 105 | 100 (default) | CD01 | CD01 |
| 20 HR | 105 | 100 (default) | CD01 | CD01 |
| 30 IT_Server | 105 | 100 (default) | CD01 | CD01 |
| 40 Corporate_WiFi | 100 (default) | 105 | CD02 | CD02 |
| 50 Guest | 100 (default) | 105 | CD02 | CD02 |
| 60 Voice | 100 (default) | 105 | CD02 | CD02 |
| 100 Management | 105 | 100 (default) | CD01 | CD01 |

Preempt is enabled on every group. Each switch tracks its router-facing link (default decrement 10), so the 5-point gap hands the group to its peer when that link fails. Bridge priorities: root 24576, secondary 28672. Mode: rapid-PVST on all switches.

## Cabling

| From | To | Type |
|---|---|---|
| HQ-RTR-01 G0/1 | HQ-CD01 G0/1 | Routed /30 |
| HQ-RTR-01 G0/2 | HQ-CD02 G0/2 | Routed /30 |
| HQ-CD01 G0/2 | HQ-CD02 G0/1 | Trunk |
| HQ-CD01 Fa0/20 | HQ-Acc01 Fa0/20 | Trunk |
| HQ-CD01 Fa0/22 | HQ-Acc02 Fa0/22 | Trunk |
| HQ-CD01 Fa0/24 | HQ-Acc03 Fa0/24 | Trunk |
| HQ-CD02 Fa0/19 | HQ-Acc01 Fa0/19 | Trunk |
| HQ-CD02 Fa0/21 | HQ-Acc02 Fa0/21 | Trunk |
| HQ-CD02 Fa0/23 | HQ-Acc03 Fa0/23 | Trunk |

## End-device ports

| Switch | Port | VLAN | Device |
|---|---|---|---|
| HQ-Acc01 | Fa0/1 | 10 | Finance PC-1 |
| HQ-Acc01 | Fa0/2 | 20 | HR PC-2 |
| HQ-Acc02 | Fa0/1 | 20 | HR PC-1 |
| HQ-Acc02 | Fa0/2 | 30 | IT PC-2 |
| HQ-Acc03 | Fa0/1 | 30 | IT PC-1 |
| HQ-Acc03 | Fa0/2 | 10 | Finance PC-2 |

## Static routing on HQ-RTR-01

| Destination | Next hop |
|---|---|
| 10.0.0.0/25 (VLANs 10, 20, 30) | 10.0.0.242 (CD01), backup 10.0.0.246 (AD 5) |
| 10.0.0.128/25 (VLANs 60, 50, 40, management) | 10.0.0.246 (CD02), backup 10.0.0.242 (AD 5) |

CD01 default route: `10.0.0.241`. CD02 default route: `10.0.0.245`.
