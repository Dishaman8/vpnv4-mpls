# IPv6 VPN Configuration

This folder contains dual-stack startup configurations for the seven routers in the VPNv4/MPLS lab. They keep the original IPv4 services and add IPv6 branch addressing and a VPNv6 service for QNB and CIB.

## Configuration files

| Router | File | Role |
| --- | --- | --- |
| R1 | [`R1_i1_startup-config.cfg`](R1_i1_startup-config.cfg) | Left provider edge; CIB1 and QNB1 |
| R2 | [`R2_i2_startup-config.cfg`](R2_i2_startup-config.cfg) | Provider core router |
| R3 | [`R3_i4_startup-config.cfg`](R3_i4_startup-config.cfg) | Right provider edge; CIB2 and QNB2 |
| R4 | [`R4_i3_startup-config.cfg`](R4_i3_startup-config.cfg) | CIB branch on the left, AS 200 |
| R5 | [`R5_i5_startup-config.cfg`](R5_i5_startup-config.cfg) | CIB branch on the right, AS 200 |
| R6 | [`R6_i6_startup-config.cfg`](R6_i6_startup-config.cfg) | QNB branch on the right, AS 300 |
| R7 | [`R7_i7_startup-config.cfg`](R7_i7_startup-config.cfg) | QNB branch on the left, AS 300 |

The matching IPv4-only source configurations remain in the repository root. The PE VRFs in these variants use the multiprotocol VRF syntax so IPv4 and IPv6 can share each customer VRF. The existing IPv4 addresses, OSPF, MPLS, BGP, route distinguishers, and route targets are retained.

## IPv6 design

- R1, R2, and R3 get IPv6 addresses on their core links and run a separate OSPFv3 process.
- The customer-facing links are dual stack. CIB and QNB deliberately reuse the same IPv6 link prefixes in separate VRFs, matching the lab's overlapping IPv4 design.
- R4–R7 advertise IPv6 branch prefixes to their PE routers using eBGP.
- R1 and R3 exchange customer routes with the VPNv6 BGP address family. The existing IPv4 MPLS core transports the labeled IPv6 VPN traffic.
- CIB keeps route target `100:1`; QNB keeps route target `300:1`. The VRF boundary and route targets keep the customers isolated. The RD makes the VPN route key unique.

Addresses use the documentation-only `2001:db8::/32` prefix. Replace them with an assigned prefix or a properly planned ULA before using the design outside a lab.

## Important transport limitation

This is 6VPE: it carries IPv6 customer traffic over the existing **IPv4-signaled MPLS** provider core. It adds a parallel IPv6 VPN, but does not configure applications or hosts to switch automatically from IPv4 to IPv6. IPv6-capable branch endpoints can use the IPv6 VPN when their IPv4 service is unavailable, provided their physical links and the provider's IPv4 MPLS transport still work. It does **not** provide an independent backup if the IPv4 provider core, its OSPF/LDP reachability, or the PE-to-PE IPv4 BGP session fails. The IPv6 OSPFv3 core adjacencies added here do not carry the VPNv6 MPLS labels.

Cisco documents 6VPE as using an existing IPv4 MPLS core and states that an IPv6-signaled MPLS core is unsupported for this feature in the IOS 15M&T guide. Confirm exact commands and feature availability against the lab's `c7200-advipservicesk9-mz.152-4.S5` image before applying: [Cisco IPv6 VPN over MPLS guide](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/mp_l3_vpns/configuration/15-mt/mp-l3-vpns-15-mt-book/ip6-mpls-6vpn.html).

## Verification commands

On R1 and R3:

```text
show ipv6 ospf neighbor
show bgp vpnv6 unicast all summary
show bgp vpnv6 unicast all
show bgp ipv6 unicast vrf CIB1
show bgp ipv6 unicast vrf QNB1
show ipv6 route vrf CIB1
show ipv6 route vrf QNB1
```

On R3, use `CIB2` and `QNB2` for the VRF-specific commands. From the branch routers, test IPv6 reachability to the opposite branch loopback in the matching customer VPN.
