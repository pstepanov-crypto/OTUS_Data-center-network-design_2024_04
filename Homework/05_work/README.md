# Домашнее задание №5

## Overlay. VxLAN EVPN L2

### Задача:

- Настроите BGP peering между Leaf и Spine в AF l2vpn evpn
- Настроите связанность между клиентами в первой зоне и убедитесь в её наличии
- Зафиксируете в документации - план работы, адресное пространство, схему сети, конфигурацию устройств
  
## Выполнение:

### Схема сети

![](images/IBGP_EVPN.PNG)

### Конфигурация оборудования

- #### [leaf-1](config/leaf-1.conf)

```
cfs eth distribute
nv overlay evpn
feature ospf
feature bgp
feature fabric forwarding
feature interface-vlan
feature vn-segment-vlan-based
feature bfd
clock timezone MSK 3 0
feature nv overlay

fabric forwarding anycast-gateway-mac 0000.dead.beef
vlan 1,100,200,2000
vlan 100
  name Hosts
  vn-segment 100
vlan 200
  name Servers
  vn-segment 200
vlan 2000
  name VRF_MAIN_VXLAN_FORWARD
  vn-segment 2000

vrf context main
  vni 2000
  rd auto
  address-family ipv4 unicast
    route-target both auto
    route-target both auto evpn
vrf context management

interface Vlan1

interface Vlan100
  no shutdown
  vrf member main
  no ip redirects
  ip address 172.16.100.1/24
  no ipv6 redirects
  fabric forwarding mode anycast-gateway

interface Vlan200
  no shutdown
  vrf member main
  no ip redirects
  ip address 172.16.200.1/24
  no ipv6 redirects
  fabric forwarding mode anycast-gateway

interface Vlan2000
  no shutdown
  mtu 9216
  vrf member main
  no ip redirects
  ip forward
  no ipv6 redirects

interface nve1
  no shutdown
  host-reachability protocol bgp
  advertise virtual-rmac
  source-interface loopback2
  member vni 100
    ingress-replication protocol bgp
  member vni 200
    ingress-replication protocol bgp
  member vni 2000 associate-vrf

interface Ethernet1/1
  description to-spine-1
  no switchport
  no ip redirects
  ip address 10.6.1.1/31
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf UNDERLAY area 0.0.0.30
  no shutdown

interface Ethernet1/2
  description to-spine-2
  no switchport
  no ip redirects
  ip address 10.6.2.1/31
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf UNDERLAY area 0.0.0.30
  ip ospf bfd
  no shutdown

interface Ethernet1/3
  description VPC1
  switchport access vlan 100

interface mgmt0
  vrf member management

interface loopback2
  ip address 10.1.0.1/32
  ip router ospf UNDERLAY area 0.0.0.30
icam monitor scale

line console
line vty
router ospf UNDERLAY
  bfd
  router-id 10.1.0.1
router bgp 65200
  router-id 10.1.0.1
  address-family ipv4 unicast
    maximum-paths 2
  address-family l2vpn evpn
    advertise-pip
  template peer RR
    bfd
    remote-as 65200
    log-neighbor-changes
    update-source loopback2
    address-family l2vpn evpn
      send-community
      send-community extended
  neighbor 10.1.1.0
    inherit peer RR
    address-family l2vpn evpn
  neighbor 10.2.1.0
    inherit peer RR
    address-family l2vpn evpn
  vrf main
    address-family ipv4 unicast
      advertise l2vpn evpn
      maximum-paths 2
evpn
  vni 100 l2
    rd auto
    route-target import auto
    route-target export auto
  vni 200 l2
    rd auto
    route-target import auto
    route-target export auto

```

- #### [leaf-2](config/leaf-2.conf)

```
nv overlay evpn
feature ospf
feature bgp
feature fabric forwarding
feature interface-vlan
feature vn-segment-vlan-based
feature bfd
clock timezone MSK 3 0
feature nv overlay

fabric forwarding anycast-gateway-mac 0000.dead.beef
vlan 1,100,200,2000
vlan 100
  name Hosts
  vn-segment 100
vlan 200
  name Servers
  vn-segment 200
vlan 2000
  name VRF_MAIN_VXLAN_FORWARD
  vn-segment 2000

vrf context main
  vni 2000
  rd auto
  address-family ipv4 unicast
    route-target both auto
    route-target both auto evpn
vrf context management

interface Vlan1

interface Vlan100
  no shutdown
  vrf member main
  no ip redirects
  ip address 172.16.100.1/24
  no ipv6 redirects
  fabric forwarding mode anycast-gateway

interface Vlan200
  no shutdown
  vrf member main
  no ip redirects
  ip address 172.16.200.1/24
  no ipv6 redirects
  fabric forwarding mode anycast-gateway

interface Vlan2000
  no shutdown
  mtu 9216
  vrf member main
  no ip redirects
  ip forward
  no ipv6 redirects

interface nve1
  no shutdown
  host-reachability protocol bgp
  advertise virtual-rmac
  source-interface loopback2
  member vni 100
    ingress-replication protocol bgp
  member vni 200
    ingress-replication protocol bgp
  member vni 2000 associate-vrf

interface Ethernet1/1
  description to-spine-1
  no switchport
  no ip redirects
  ip address 10.6.1.3/31
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf UNDERLAY area 0.0.0.30
  no shutdown

interface Ethernet1/2
  description to-spine-2
  no switchport
  no ip redirects
  ip address 10.6.2.3/31
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf UNDERLAY area 0.0.0.30
  no shutdown

interface Ethernet1/3
  description VPC2
  switchport access vlan 200

interface mgmt0
  vrf member management

interface loopback2
  ip address 10.1.0.2/32
  ip router ospf UNDERLAY area 0.0.0.30
icam monitor scale

line console
line vty
router ospf UNDERLAY
  bfd
  router-id 10.1.0.2
  passive-interface default
router bgp 65200
  router-id 10.1.0.2
  address-family ipv4 unicast
    maximum-paths 2
  address-family l2vpn evpn
    advertise-pip
  template peer RR
    bfd
    remote-as 65200
    log-neighbor-changes
    update-source loopback2
    address-family l2vpn evpn
      send-community
      send-community extended
  neighbor 10.1.1.0
    inherit peer RR
    address-family l2vpn evpn
  neighbor 10.2.1.0
    inherit peer RR
    address-family l2vpn evpn
  vrf main
    address-family ipv4 unicast
      advertise l2vpn evpn
      maximum-paths 2
evpn
  vni 100 l2
    rd auto
    route-target import auto
    route-target export auto
  vni 200 l2
    rd auto
    route-target import auto
    route-target export auto

```

- #### [leaf-3](config/leaf-3.conf)

```
nv overlay evpn
feature ospf
feature bgp
feature fabric forwarding
feature interface-vlan
feature vn-segment-vlan-based
feature bfd
clock timezone MSK 3 0
feature nv overlay

fabric forwarding anycast-gateway-mac 0000.dead.beef
vlan 1,100,200,2000
vlan 100
  name Hosts
  vn-segment 100
vlan 200
  name Servers
  vn-segment 200
vlan 2000
  name VRF_MAIN_VXLAN_FORWARD
  vn-segment 2000

vrf context main
  vni 2000
  rd auto
  address-family ipv4 unicast
    route-target both auto
    route-target both auto evpn
vrf context management

interface Vlan1

interface Vlan100
  no shutdown
  vrf member main
  no ip redirects
  ip address 172.16.100.1/24
  no ipv6 redirects
  fabric forwarding mode anycast-gateway

interface Vlan200
  no shutdown
  vrf member main
  no ip redirects
  ip address 172.16.200.1/24
  no ipv6 redirects
  fabric forwarding mode anycast-gateway

interface Vlan2000
  no shutdown
  mtu 9216
  vrf member main
  no ip redirects
  ip forward
  no ipv6 redirects

interface nve1
  no shutdown
  host-reachability protocol bgp
  advertise virtual-rmac
  source-interface loopback2
  member vni 100
    ingress-replication protocol bgp
  member vni 200
    ingress-replication protocol bgp
  member vni 2000 associate-vrf

interface Ethernet1/1
  description to-spine-1
  no switchport
  no ip redirects
  ip address 10.6.1.5/31
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf UNDERLAY area 0.0.0.30
  no shutdown

interface Ethernet1/2
  description to-spine-2
  no switchport
  no ip redirects
  ip address 10.6.2.5/31
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf UNDERLAY area 0.0.0.30
  no shutdown

interface Ethernet1/3
  description VPC3
  switchport access vlan 100

interface Ethernet1/4
  description VPC4
  switchport access vlan 200

interface mgmt0
  vrf member management

interface loopback2
  ip address 10.1.0.3/32
  ip router ospf UNDERLAY area 0.0.0.30
icam monitor scale

line console
line vty
router ospf UNDERLAY
  bfd
  router-id 10.1.0.3
  passive-interface default
router bgp 65200
  router-id 10.1.0.3
  address-family ipv4 unicast
    maximum-paths 2
  address-family l2vpn evpn
    advertise-pip
  template peer RR
    bfd
    remote-as 65200
    log-neighbor-changes
    update-source loopback2
    address-family l2vpn evpn
      send-community
      send-community extended
  neighbor 10.1.1.0
    inherit peer RR
    address-family l2vpn evpn
  neighbor 10.2.1.0
    inherit peer RR
    address-family l2vpn evpn
  vrf main
    address-family ipv4 unicast
      advertise l2vpn evpn
      maximum-paths 2
evpn
  vni 100 l2
    rd auto
    route-target import auto
    route-target export auto
  vni 200 l2
    rd auto
    route-target import auto
    route-target export auto


```

- #### [spine-1](config/spine-1.conf)

```
cfs eth distribute
nv overlay evpn
feature ospf
feature bgp
feature fabric forwarding
feature interface-vlan
feature vn-segment-vlan-based
feature bfd
clock timezone MSK 3 0
feature nv overlay

fabric forwarding anycast-gateway-mac 0000.dead.beef
vlan 1

vrf context management

interface Vlan1

interface Ethernet1/1
  description to-leaf-1
  no switchport
  no ip redirects
  ip address 10.6.1.0/31
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf UNDERLAY area 0.0.0.30
  ip ospf bfd
  no shutdown

interface Ethernet1/2
  description to-leaf-2
  no switchport
  no ip redirects
  ip address 10.6.1.2/31
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf UNDERLAY area 0.0.0.30
  no shutdown

interface Ethernet1/3
  description to-leaf-3
  no switchport
  no ip redirects
  ip address 10.6.1.4/31
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf UNDERLAY area 0.0.0.30
  no shutdown

interface mgmt0
  vrf member management

interface loopback1
  ip address 10.1.1.0/32
  ip router ospf UNDERLAY area 0.0.0.30
icam monitor scale

line console
line vty
router ospf UNDERLAY
  bfd
  router-id 10.1.1.0
router bgp 65200
  router-id 10.1.1.0
  address-family l2vpn evpn
    maximum-paths 2
    retain route-target all
  template peer RR
    remote-as 65200
    update-source loopback1
    address-family l2vpn evpn
      send-community extended
      route-reflector-client
  template peer RRC
    remote-as 65200
    log-neighbor-changes
    update-source loopback1
    address-family l2vpn evpn
      send-community extended
      route-reflector-client
  neighbor 10.1.0.1
    inherit peer RRC
  neighbor 10.1.0.2
    inherit peer RRC
  neighbor 10.1.0.3
    inherit peer RRC
  neighbor 10.2.1.0
    inherit peer RR

```

- #### [spine-2](config/spine-2.conf)

```
nv overlay evpn
feature ospf
feature bgp
feature fabric forwarding
feature interface-vlan
feature vn-segment-vlan-based
feature bfd
clock timezone MSK 3 0
feature nv overlay

fabric forwarding anycast-gateway-mac 0000.dead.beef
vlan 1

vrf context management

interface Vlan1

interface Ethernet1/1
  description to-leaf-1
  no switchport
  no ip redirects
  ip address 10.6.2.0/31
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf UNDERLAY area 0.0.0.30
  no shutdown

interface Ethernet1/2
  description to-leaf-2
  no switchport
  no ip redirects
  ip address 10.6.2.2/31
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf UNDERLAY area 0.0.0.30
  no shutdown

interface Ethernet1/3
  description to-leaf-3
  no switchport
  no ip redirects
  ip address 10.6.2.4/31
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf UNDERLAY area 0.0.0.30
  no shutdown

interface mgmt0
  vrf member management

interface loopback1
  ip address 10.2.1.0/32
  ip router ospf UNDERLAY area 0.0.0.30
icam monitor scale

line console
line vty
router ospf UNDERLAY
  bfd
  router-id 10.2.1.0
router bgp 65200
  router-id 10.2.1.0
  address-family l2vpn evpn
    maximum-paths 2
    retain route-target all
  template peer RR
    remote-as 65200
    update-source loopback1
    address-family l2vpn evpn
      send-community extended
      route-reflector-client
  template peer RRC
    remote-as 65200
    log-neighbor-changes
    update-source loopback1
    address-family l2vpn evpn
      send-community extended
      route-reflector-client
  neighbor 10.1.0.1
    inherit peer RRC
  neighbor 10.1.0.2
    inherit peer RRC
  neighbor 10.1.0.3
    inherit peer RRC
  neighbor 10.1.1.0
    inherit peer RR



---
### Проверка связанности клиентов по L2
- #### spine-1
```
sh bgp l2vpn evpn summary
Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.1.0.1        4 65200      22      21       14    0    0 00:15:47 1
10.1.0.2        4 65200      23      21       14    0    0 00:16:25 1
10.1.0.3        4 65200       7       6       14    0    0 00:00:30 1
10.1.2.0        4 65200      22      22       14    0    0 00:16:02 0
```

- #### spine-2
```
Spine-2# sh bgp l2vpn evpn summary
Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.1.0.1        4 65200      18      34       47    0    0 00:04:55 2
10.1.0.2        4 65200      18      30       47    0    0 00:06:07 1
10.1.0.3        4 65200      18      20       47    0    0 00:06:07 1
10.1.1.0        4 65200      41      35       47    0    0 00:05:01 4
```

- #### leaf-1

```
Leaf-1# sh ip route
10.1.0.1/32, ubest/mbest: 2/0, attached
    *via 10.1.0.1, Lo2, [0/0], 04:36:21, local
    *via 10.1.0.1, Lo2, [0/0], 04:36:20, direct
10.1.0.2/32, ubest/mbest: 2/0
    *via 10.6.1.0, Eth1/1, [110/81], 00:02:04, ospf-UNDERLAY, intra
    *via 10.6.2.0, Eth1/2, [110/81], 00:01:56, ospf-UNDERLAY, intra
10.1.0.3/32, ubest/mbest: 2/0
    *via 10.6.1.0, Eth1/1, [110/81], 01:23:35, ospf-UNDERLAY, intra
    *via 10.6.2.0, Eth1/2, [110/81], 01:18:38, ospf-UNDERLAY, intra
10.1.1.0/32, ubest/mbest: 1/0
    *via 10.6.1.0, Eth1/1, [110/41], 03:28:55, ospf-UNDERLAY, intra
10.2.1.0/32, ubest/mbest: 1/0
    *via 10.6.2.0, Eth1/2, [110/41], 01:18:39, ospf-UNDERLAY, intra
10.6.1.0/31, ubest/mbest: 1/0, attached
    *via 10.6.1.1, Eth1/1, [0/0], 03:30:04, direct
10.6.1.1/32, ubest/mbest: 1/0, attached
    *via 10.6.1.1, Eth1/1, [0/0], 03:30:04, local
10.6.1.2/31, ubest/mbest: 1/0
    *via 10.6.1.0, Eth1/1, [110/80], 03:28:55, ospf-UNDERLAY, intra
10.6.1.4/31, ubest/mbest: 1/0
    *via 10.6.1.0, Eth1/1, [110/80], 03:28:55, ospf-UNDERLAY, intra
10.6.2.0/31, ubest/mbest: 1/0, attached
    *via 10.6.2.1, Eth1/2, [0/0], 03:29:32, direct
10.6.2.1/32, ubest/mbest: 1/0, attached
    *via 10.6.2.1, Eth1/2, [0/0], 03:29:32, local
10.6.2.2/31, ubest/mbest: 1/0
    *via 10.6.2.0, Eth1/2, [110/80], 01:18:39, ospf-UNDERLAY, intra
10.6.2.4/31, ubest/mbest: 1/0
    *via 10.6.2.0, Eth1/2, [110/80], 01:18:39, ospf-UNDERLAY, intra
    
Leaf-1# sh ip route vrf main
172.16.100.0/24, ubest/mbest: 1/0, attached
    *via 172.16.100.1, Vlan100, [0/0], 00:48:27, direct
172.16.100.1/32, ubest/mbest: 1/0, attached
    *via 172.16.100.1, Vlan100, [0/0], 00:48:27, local
172.16.100.10/32, ubest/mbest: 1/0, attached
    *via 172.16.100.10, Vlan100, [190/0], 00:43:37, hmm
172.16.100.20/32, ubest/mbest: 1/0
    *via 10.1.0.3%default, [200/0], 00:40:34, bgp-65200, internal, tag 65200, se
gid: 2000 tunnelid: 0xa010003 encap: VXLAN

172.16.200.0/24, ubest/mbest: 1/0, attached
    *via 172.16.200.1, Vlan200, [0/0], 00:48:15, direct
172.16.200.1/32, ubest/mbest: 1/0, attached
    *via 172.16.200.1, Vlan200, [0/0], 00:48:15, local
172.16.200.20/32, ubest/mbest: 1/0
    *via 10.1.0.3%default, [200/0], 00:32:13, bgp-65200, internal, tag 65200, se
gid: 2000 tunnelid: 0xa010003 encap: VXLAN


Leaf-1# sh bgp l2vpn evpn summary
Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.1.1.0        4 65200     163     109      270    0    0 01:41:16 10
10.2.1.0        4 65200     150      94      270    0    0 01:20:51 10


leaf-1#show vxlan address-table
          Vxlan Mac Address Table
----------------------------------------------------------------------

VLAN  Mac Address     Type      Prt  VTEP             Moves   Last Move
----  -----------     ----      ---  ----             -----   ---------
  10  0050.7966.6807  EVPN      Vx1  10.100.0.3       1       0:01:15 ago
  10  0050.7966.6809  EVPN      Vx1  10.100.0.2       1       0:01:30 ago
  20  0050.7966.680a  EVPN      Vx1  10.100.0.2       1       0:00:58 ago
  20  0050.7966.680b  EVPN      Vx1  10.100.0.3       1       0:00:44 ago
Total Remote Mac Addresses for this criterion: 4

leaf-1#show vxlan vtep
Remote VTEPS for Vxlan1:

VTEP             Tunnel Type(s)
---------------- --------------
10.100.0.2       unicast, flood
10.100.0.3       unicast, flood

Total number of remote VTEPS:  2

leaf-1#show bgp evpn route-type mac-ip
BGP routing table information for VRF default
Router identifier 10.1.0.1, local AS number 65001
Route status codes: s - suppressed, * - valid, > - active, E - ECMP head, e - ECMP
                    S - Stale, c - Contributing to ECMP, b - backup
                    % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >     RD: 65001:10010 mac-ip 0050.7966.6806
                                 -                     -       -       0       i
 * >Ec   RD: 65003:10010 mac-ip 0050.7966.6807
                                 10.100.0.3            -       100     0       65000 65003 i
 *  ec   RD: 65003:10010 mac-ip 0050.7966.6807
                                 10.100.0.3            -       100     0       65000 65003 i
 * >     RD: 65001:10020 mac-ip 0050.7966.6808
                                 -                     -       -       0       i
 * >Ec   RD: 65002:10010 mac-ip 0050.7966.6809
                                 10.100.0.2            -       100     0       65000 65002 i
 *  ec   RD: 65002:10010 mac-ip 0050.7966.6809
                                 10.100.0.2            -       100     0       65000 65002 i
 * >Ec   RD: 65002:10020 mac-ip 0050.7966.680a
                                 10.100.0.2            -       100     0       65000 65002 i
 *  ec   RD: 65002:10020 mac-ip 0050.7966.680a
                                 10.100.0.2            -       100     0       65000 65002 i
 * >Ec   RD: 65003:10020 mac-ip 0050.7966.680b
                                 10.100.0.3            -       100     0       65000 65003 i
 *  ec   RD: 65003:10020 mac-ip 0050.7966.680b
                                 10.100.0.3            -       100     0       65000 65003 i

leaf-1#sh interfaces vxlan 1
Vxlan1 is up, line protocol is up (connected)
  Hardware is Vxlan
  Source interface is Loopback100 and is active with 10.100.0.1
  Replication/Flood Mode is headend with Flood List Source: EVPN
  Remote MAC learning via EVPN
  VNI mapping to VLANs
  Static VLAN to VNI mapping is
    [10, 10010]       [20, 10020]
  Note: All Dynamic VLANs used by VCS are internal VLANs.
        Use 'show vxlan vni' for details.
  Static VRF to VNI mapping is not configured
  Headend replication flood vtep list is:
    10 10.100.0.2      10.100.0.3
    20 10.100.0.2      10.100.0.3
  Shared Router MAC is 0000.0000.0000
  
 leaf-1#show vxlan vni
VNI to VLAN Mapping for Vxlan1
VNI         VLAN       Source       Interface       802.1Q Tag
----------- ---------- ------------ --------------- ----------
10010       10         static       Ethernet3       untagged
                                    Vxlan1          10
10020       20         static       Ethernet4       untagged
                                    Vxlan1          20

VNI to dynamic VLAN Mapping for Vxlan1
VNI       VLAN       VRF       Source
--------- ---------- --------- ------------

```

- #### leaf-2

```
leaf-2#sh ip bgp summary
BGP summary information for VRF default
Router identifier 10.1.0.2, local AS number 65002
Neighbor Status Codes: m - Under maintenance
  Neighbor         V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.1.1.0         4 65000           5255      5268    0    0 01:43:36 Estab   5      5
  10.1.2.0         4 65000           4612      4626    0    0 01:40:13 Estab   5      5
  10.4.1.2         4 65000           5229      5221    0    0 03:41:57 Estab   5      5
  10.4.2.2         4 65000           5228      5223    0    0 03:16:27 Estab   5      5

leaf-2#show vxlan address-table
          Vxlan Mac Address Table
----------------------------------------------------------------------

VLAN  Mac Address     Type      Prt  VTEP             Moves   Last Move
----  -----------     ----      ---  ----             -----   ---------
  10  0050.7966.6806  EVPN      Vx1  10.100.0.1       1       0:01:36 ago
  10  0050.7966.6807  EVPN      Vx1  10.100.0.3       1       0:01:21 ago
  20  0050.7966.6808  EVPN      Vx1  10.100.0.1       1       0:01:04 ago
  20  0050.7966.680b  EVPN      Vx1  10.100.0.3       1       0:00:50 ago
Total Remote Mac Addresses for this criterion: 4

leaf-2#show bgp evpn route-type mac-ip
BGP routing table information for VRF default
Router identifier 10.1.0.2, local AS number 65002
Route status codes: s - suppressed, * - valid, > - active, E - ECMP head, e - ECMP
                    S - Stale, c - Contributing to ECMP, b - backup
                    % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec   RD: 65001:10010 mac-ip 0050.7966.6806
                                 10.100.0.1            -       100     0       65000 65001 i
 *  ec   RD: 65001:10010 mac-ip 0050.7966.6806
                                 10.100.0.1            -       100     0       65000 65001 i
 * >Ec   RD: 65003:10010 mac-ip 0050.7966.6807
                                 10.100.0.3            -       100     0       65000 65003 i
 *  ec   RD: 65003:10010 mac-ip 0050.7966.6807
                                 10.100.0.3            -       100     0       65000 65003 i
 * >Ec   RD: 65001:10020 mac-ip 0050.7966.6808
                                 10.100.0.1            -       100     0       65000 65001 i
 *  ec   RD: 65001:10020 mac-ip 0050.7966.6808
                                 10.100.0.1            -       100     0       65000 65001 i
 * >     RD: 65002:10010 mac-ip 0050.7966.6809
                                 -                     -       -       0       i
 * >     RD: 65002:10020 mac-ip 0050.7966.680a
                                 -                     -       -       0       i
 * >Ec   RD: 65003:10020 mac-ip 0050.7966.680b
                                 10.100.0.3            -       100     0       65000 65003 i
 *  ec   RD: 65003:10020 mac-ip 0050.7966.680b
                                 10.100.0.3            -       100     0       65000 65003 i

leaf-2#sh interfaces vxlan 1
Vxlan1 is up, line protocol is up (connected)
  Hardware is Vxlan
  Source interface is Loopback100 and is active with 10.100.0.2
  Replication/Flood Mode is headend with Flood List Source: EVPN
  Remote MAC learning via EVPN
  VNI mapping to VLANs
  Static VLAN to VNI mapping is
    [10, 10010]       [20, 10020]
  Note: All Dynamic VLANs used by VCS are internal VLANs.
        Use 'show vxlan vni' for details.
  Static VRF to VNI mapping is not configured
  Headend replication flood vtep list is:
    10 10.100.0.1      10.100.0.3
    20 10.100.0.1      10.100.0.3
  Shared Router MAC is 0000.0000.0000
  
leaf-2#show vxlan vni
VNI to VLAN Mapping for Vxlan1
VNI         VLAN       Source       Interface       802.1Q Tag
----------- ---------- ------------ --------------- ----------
10010       10         static       Ethernet3       untagged
                                    Vxlan1          10
10020       20         static       Ethernet4       untagged
                                    Vxlan1          20

VNI to dynamic VLAN Mapping for Vxlan1
VNI       VLAN       VRF       Source
--------- ---------- --------- ------------
```

- #### leaf-3

```
leaf-3#sh ip bgp summary
BGP summary information for VRF default
Router identifier 10.1.0.3, local AS number 65003
Neighbor Status Codes: m - Under maintenance
  Neighbor         V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.1.1.0         4 65000           4740      4729    0    0 01:43:26 Estab   5      5
  10.1.2.0         4 65000           4620      4632    0    0 01:40:17 Estab   5      5
  10.4.1.4         4 65000           5219      5226    0    0 03:24:57 Estab   5      5
  10.4.2.4         4 65000           5236      5228    0    0 03:16:29 Estab   5      5

leaf-3#show vxlan address-table
          Vxlan Mac Address Table
----------------------------------------------------------------------

VLAN  Mac Address     Type      Prt  VTEP             Moves   Last Move
----  -----------     ----      ---  ----             -----   ---------
 100  0050.7966.6806  EVPN      Vx1  10.100.0.1       1       0:01:40 ago
 100  0050.7966.6809  EVPN      Vx1  10.100.0.2       1       0:01:40 ago
 200  0050.7966.6808  EVPN      Vx1  10.100.0.1       1       0:01:09 ago
 200  0050.7966.680a  EVPN      Vx1  10.100.0.2       1       0:01:09 ago
Total Remote Mac Addresses for this criterion: 4

leaf-3#show bgp evpn route-type mac-ip
BGP routing table information for VRF default
Router identifier 10.1.0.3, local AS number 65003
Route status codes: s - suppressed, * - valid, > - active, E - ECMP head, e - ECMP
                    S - Stale, c - Contributing to ECMP, b - backup
                    % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec   RD: 65001:10010 mac-ip 0050.7966.6806
                                 10.100.0.1            -       100     0       65000 65001 i
 *  ec   RD: 65001:10010 mac-ip 0050.7966.6806
                                 10.100.0.1            -       100     0       65000 65001 i
 * >     RD: 65003:10010 mac-ip 0050.7966.6807
                                 -                     -       -       0       i
 * >Ec   RD: 65001:10020 mac-ip 0050.7966.6808
                                 10.100.0.1            -       100     0       65000 65001 i
 *  ec   RD: 65001:10020 mac-ip 0050.7966.6808
                                 10.100.0.1            -       100     0       65000 65001 i
 * >Ec   RD: 65002:10010 mac-ip 0050.7966.6809
                                 10.100.0.2            -       100     0       65000 65002 i
 *  ec   RD: 65002:10010 mac-ip 0050.7966.6809
                                 10.100.0.2            -       100     0       65000 65002 i
 * >Ec   RD: 65002:10020 mac-ip 0050.7966.680a
                                 10.100.0.2            -       100     0       65000 65002 i
 *  ec   RD: 65002:10020 mac-ip 0050.7966.680a
                                 10.100.0.2            -       100     0       65000 65002 i
 * >     RD: 65003:10020 mac-ip 0050.7966.680b
                                 -                     -       -       0       i
leaf-3#sh interfaces vxlan 1
Vxlan1 is up, line protocol is up (connected)
  Hardware is Vxlan
  Source interface is Loopback100 and is active with 10.100.0.3
  Replication/Flood Mode is headend with Flood List Source: EVPN
  Remote MAC learning via EVPN
  VNI mapping to VLANs
  Static VLAN to VNI mapping is
    [100, 10010]      [200, 10020]
  Note: All Dynamic VLANs used by VCS are internal VLANs.
        Use 'show vxlan vni' for details.
  Static VRF to VNI mapping is not configured
  Headend replication flood vtep list is:
   100 10.100.0.1      10.100.0.2
   200 10.100.0.1      10.100.0.2
  Shared Router MAC is 0000.0000.0000

leaf-3#show vxlan vni
VNI to VLAN Mapping for Vxlan1
VNI         VLAN       Source       Interface       802.1Q Tag
----------- ---------- ------------ --------------- ----------
10010       100        static       Ethernet3       untagged
                                    Vxlan1          100
10020       200        static       Ethernet4       untagged
                                    Vxlan1          200

VNI to dynamic VLAN Mapping for Vxlan1
VNI       VLAN       VRF       Source
--------- ---------- --------- ------------
```
- #### client-1

```
VPCS> ping 192.168.10.21
84 bytes from 192.168.10.21 icmp_seq=1 ttl=64 time=11.172 ms
84 bytes from 192.168.10.21 icmp_seq=2 ttl=64 time=10.478 ms
84 bytes from 192.168.10.21 icmp_seq=3 ttl=64 time=11.873 ms
84 bytes from 192.168.10.21 icmp_seq=4 ttl=64 time=10.500 ms
84 bytes from 192.168.10.21 icmp_seq=5 ttl=64 time=10.308 ms

VPCS> ping 192.168.10.31
84 bytes from 192.168.10.31 icmp_seq=1 ttl=64 time=17.274 ms
84 bytes from 192.168.10.31 icmp_seq=2 ttl=64 time=11.062 ms
84 bytes from 192.168.10.31 icmp_seq=3 ttl=64 time=11.495 ms
84 bytes from 192.168.10.31 icmp_seq=4 ttl=64 time=11.849 ms
84 bytes from 192.168.10.31 icmp_seq=5 ttl=64 time=9.009 ms
```

- #### client-2

```
VPCS> ping 192.168.20.21

192.168.20.21 icmp_seq=1 timeout
84 bytes from 192.168.20.21 icmp_seq=2 ttl=64 time=9.820 ms
84 bytes from 192.168.20.21 icmp_seq=3 ttl=64 time=11.197 ms
84 bytes from 192.168.20.21 icmp_seq=4 ttl=64 time=10.320 ms
84 bytes from 192.168.20.21 icmp_seq=5 ttl=64 time=10.782 ms

VPCS> ping 192.168.20.31

84 bytes from 192.168.20.31 icmp_seq=1 ttl=64 time=12.517 ms
84 bytes from 192.168.20.31 icmp_seq=2 ttl=64 time=11.172 ms
84 bytes from 192.168.20.31 icmp_seq=3 ttl=64 time=11.067 ms
84 bytes from 192.168.20.31 icmp_seq=4 ttl=64 time=9.864 ms
84 bytes from 192.168.20.31 icmp_seq=5 ttl=64 time=11.746 ms
```
