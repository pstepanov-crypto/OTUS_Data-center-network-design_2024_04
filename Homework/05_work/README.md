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
interface Ethernet3
   description to-client-1
   switchport access vlan 10
!
interface Ethernet4
   description to-client-2
   switchport access vlan 20
!
interface Loopback1
   ip address 10.1.0.1/32
!
interface Loopback100
   description NVE Loopback
   ip address 10.100.0.1/32
!
interface Vxlan1
   vxlan source-interface Loopback100
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
   vxlan vlan 20 vni 10020
   vxlan learn-restrict any
!
ip routing
!
ip prefix-list PL_LOOP
   seq 10 permit 10.1.0.1/32
   seq 20 permit 10.100.0.1/32
!
route-map RM_CONN permit 10
   match ip address prefix-list PL_LOOP
!
router bgp 65001
   router-id 10.1.0.1
   timers bgp 3 9
   maximum-paths 2 ecmp 2
   neighbor EVPN peer group
   neighbor EVPN remote-as 65000
   neighbor EVPN update-source Loopback1
   neighbor EVPN ebgp-multihop 3
   neighbor EVPN send-community extended
   neighbor SPINE peer group
   neighbor SPINE remote-as 65000
   neighbor SPINE bfd
   neighbor SPINE allowas-in 1
   neighbor SPINE rib-in pre-policy retain all
   neighbor SPINE password 7 9cdhNWhDdTM=
   neighbor SPINE send-community extended
   neighbor SPINE maximum-routes 1000
   neighbor 10.1.1.0 peer group EVPN
   neighbor 10.1.2.0 peer group EVPN
   neighbor 10.4.1.0 peer group SPINE
   neighbor 10.4.2.0 peer group SPINE
   redistribute connected route-map RM_CONN
   !
   vlan 10
      rd 65001:10010
      route-target both 10:10010
      redistribute learned
   !
   vlan 20
      rd 65001:10020
      route-target both 20:10020
      redistribute learned
   !
   address-family evpn
      neighbor EVPN activate
   !
   address-family ipv4
      neighbor SPINE activate
```

- #### [leaf-2](config/leaf-2.conf)

```
interface Ethernet3
   description to-clien-3
   switchport access vlan 10
!
interface Ethernet4
   description to-clien-4
   switchport access vlan 20
!
interface Loopback1
   ip address 10.1.0.2/32
!
interface Loopback100
   description NVE Loopback
   ip address 10.100.0.2/32
!
interface Management1
!
interface Vxlan1
   vxlan source-interface Loopback100
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
   vxlan vlan 20 vni 10020
   vxlan learn-restrict any
!
ip routing
!
ip prefix-list PL_LOOP
   seq 10 permit 10.1.0.2/32
   seq 20 permit 10.100.0.2/32
!
route-map RM_CONN permit 10
   match ip address prefix-list PL_LOOP
!
router bgp 65002
   router-id 10.1.0.2
   timers bgp 3 9
   maximum-paths 2 ecmp 2
   neighbor EVPN peer group
   neighbor EVPN remote-as 65000
   neighbor EVPN update-source Loopback1
   neighbor EVPN ebgp-multihop 3
   neighbor EVPN send-community extended
   neighbor SPINE peer group
   neighbor SPINE remote-as 65000
   neighbor SPINE bfd
   neighbor SPINE allowas-in 1
   neighbor SPINE rib-in pre-policy retain all
   neighbor SPINE password 7 9cdhNWhDdTM=
   neighbor SPINE send-community extended
   neighbor SPINE maximum-routes 1000
   neighbor 10.1.1.0 peer group EVPN
   neighbor 10.1.2.0 peer group EVPN
   neighbor 10.4.1.2 peer group SPINE
   neighbor 10.4.2.2 peer group SPINE
   redistribute connected route-map RM_CONN
   !
   vlan 10
      rd 65002:10010
      route-target both 10:10010
      redistribute learned
   !
   vlan 20
      rd 65002:10020
      route-target both 20:10020
      redistribute learned
   !
   address-family evpn
      neighbor EVPN activate
   !
   address-family ipv4
      neighbor SPINE activate
```

- #### [leaf-3](config/leaf-3.conf)

```
interface Ethernet3
   description to-clien-5
   switchport access vlan 100
!
interface Ethernet4
   description to-clien-6
   switchport access vlan 200
!
interface Loopback1
   ip address 10.1.0.3/32
!
interface Loopback100
   description NVE Loopback
   ip address 10.100.0.3/32
!
interface Management1
!
interface Vxlan1
   vxlan source-interface Loopback100
   vxlan udp-port 4789
   vxlan vlan 100 vni 10010
   vxlan vlan 200 vni 10020
   vxlan learn-restrict any
!
ip routing
!
ip prefix-list PL_LOOP
   seq 10 permit 10.1.0.3/32
   seq 20 permit 10.100.0.3/32
!
route-map RM_CONN permit 10
   match ip address prefix-list PL_LOOP
!
router bgp 65003
   router-id 10.1.0.3
   timers bgp 3 9
   maximum-paths 2 ecmp 2
   neighbor EVPN peer group
   neighbor EVPN remote-as 65000
   neighbor EVPN update-source Loopback1
   neighbor EVPN ebgp-multihop 3
   neighbor EVPN send-community extended
   neighbor SPINE peer group
   neighbor SPINE remote-as 65000
   neighbor SPINE bfd
   neighbor SPINE allowas-in 1
   neighbor SPINE rib-in pre-policy retain all
   neighbor SPINE password 7 9cdhNWhDdTM=
   neighbor SPINE send-community
   neighbor SPINE maximum-routes 1000
   neighbor 10.1.1.0 peer group EVPN
   neighbor 10.1.2.0 peer group EVPN
   neighbor 10.4.1.4 peer group SPINE
   neighbor 10.4.2.4 peer group SPINE
   redistribute connected route-map RM_CONN
   !
   vlan 100
      rd 65003:10010
      route-target both 10:10010
      redistribute learned
   !
   vlan 200
      rd 65003:10020
      route-target both 20:10020
      redistribute learned
   !
   address-family evpn
      neighbor EVPN activate
   !
   address-family ipv4
      neighbor SPINE activate
```

- #### [spine-1](config/spine-1.conf)

```
peer-filter EVPN
   10 match as-range 65001-65003 result accept
!
peer-filter LEAF
   10 match as-range 65001-65003 result accept
!
router bgp 65000
   router-id 10.1.1.0
   timers bgp 3 9
   maximum-paths 2 ecmp 2
   bgp listen range 10.1.0.0/24 peer-group EVPN peer-filter EVPN
   bgp listen range 10.4.1.0/24 peer-group LEAF peer-filter LEAF
   neighbor EVPN peer group
   neighbor EVPN next-hop-unchanged
   neighbor EVPN update-source Loopback1
   neighbor EVPN ebgp-multihop 3
   neighbor EVPN send-community extended
   neighbor LEAF peer group
   neighbor LEAF bfd
   neighbor LEAF rib-in pre-policy retain all
   neighbor LEAF password 7 7zCpOlU0aME=
   neighbor LEAF send-community
   neighbor LEAF maximum-routes 1000
   redistribute connected route-map RM_CONN
   !
   address-family evpn
      neighbor EVPN activate
   !
   address-family ipv4
      neighbor LEAF activate
```

- #### [spine-2](config/spine-2.conf)

```
peer-filter EVPN
   10 match as-range 65001-65003 result accept
!
peer-filter LEAF
   10 match as-range 65001-65003 result accept
!
router bgp 65000
   router-id 10.1.2.0
   timers bgp 3 9
   maximum-paths 2 ecmp 2
   bgp listen range 10.1.0.0/24 peer-group EVPN peer-filter EVPN
   bgp listen range 10.4.2.0/24 peer-group LEAF peer-filter LEAF
   neighbor EVPN peer group
   neighbor EVPN next-hop-unchanged
   neighbor EVPN update-source Loopback1
   neighbor EVPN ebgp-multihop 3
   neighbor EVPN send-community extended
   neighbor LEAF peer group
   neighbor LEAF bfd
   neighbor LEAF rib-in pre-policy retain all
   neighbor LEAF password 7 7zCpOlU0aME=
   neighbor LEAF send-community
   neighbor LEAF maximum-routes 1000
   redistribute connected route-map RM_CONN
   !
   address-family evpn
      neighbor EVPN activate
   !
   address-family ipv4
      neighbor LEAF activate
```
---

### Проверка связанности клиентов по L2

- #### spine-1

```
spine-1#sh bgp evpn summary
BGP summary information for VRF default
Router identifier 10.1.1.0, local AS number 65000
Neighbor Status Codes: m - Under maintenance
  Neighbor         V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.1.0.1         4 65001           2373      2366    0    0 01:40:03 Estab   2      2
  10.1.0.2         4 65002           2362      2357    0    0 01:39:27 Estab   2      2
  10.1.0.3         4 65003           2357      2355    0    0 01:39:15 Estab   2      2

```

- #### spine-2

```
spine-2#sh bgp evpn summary
BGP summary information for VRF default
Router identifier 10.1.2.0, local AS number 65000
Neighbor Status Codes: m - Under maintenance
  Neighbor         V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.1.0.1         4 65001             21        17    0    0 00:00:16 Estab   2      2
  10.1.0.2         4 65002             20        17    0    0 00:00:13 Estab   2      2
  10.1.0.3         4 65003             20        17    0    0 00:00:14 Estab   2      2
```

- #### leaf-1

```
leaf-1#sh ip route

 C        10.1.0.1/32 is directly connected, Loopback1
 B E      10.1.0.2/32 [200/0] via 10.4.1.0, Ethernet1
                              via 10.4.2.0, Ethernet2
 B E      10.1.0.3/32 [200/0] via 10.4.1.0, Ethernet1
                              via 10.4.2.0, Ethernet2
 B E      10.1.1.0/32 [200/0] via 10.4.1.0, Ethernet1
 B E      10.1.2.0/32 [200/0] via 10.4.2.0, Ethernet2
 C        10.4.1.0/31 is directly connected, Ethernet1
 C        10.4.2.0/31 is directly connected, Ethernet2
 C        10.100.0.1/32 is directly connected, Loopback100
 B E      10.100.0.2/32 [200/0] via 10.4.1.0, Ethernet1
                                via 10.4.2.0, Ethernet2
 B E      10.100.0.3/32 [200/0] via 10.4.1.0, Ethernet1
                                via 10.4.2.0, Ethernet2

leaf-1#sh ip bgp summary
BGP summary information for VRF default
Router identifier 10.1.0.1, local AS number 65001
Neighbor Status Codes: m - Under maintenance
  Neighbor         V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.1.1.0         4 65000           5249      5254    0    0 01:43:58 Estab   5      5
  10.1.2.0         4 65000           4612      4603    0    0 01:40:03 Estab   5      5
  10.4.1.0         4 65000           5208      5217    0    0 03:41:50 Estab   5      5
  10.4.2.0         4 65000           5215      5204    0    0 03:16:13 Estab   5      5

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
