# Домашнее задание №6

## Overlay. VxLAN EVPN L3

### Задача:

- Настроить клиентов в разных VNI
- Настроить маршрутизацию между клиентами 
- Проверить связанность между клиентами

## Выполнение:

### Схема сети

![](images/Overlay_VXLAN_L3.png)

### Конфигурация оборудования

- #### [leaf-1](config/leaf-1.conf)

```
vlan 11,22

vrf instance OTUS-PROD

interface Ethernet3
   description to-client-1
   switchport access vlan 11

interface Ethernet4
   description to-client-2
   switchport access vlan 11

interface Vlan11
   vrf OTUS-PROD
   ip address virtual 192.168.11.254/24

interface Vlan22
   vrf OTUS-PROD
   ip address virtual 192.168.22.254/24

interface Vxlan1
   vxlan source-interface Loopback100
   vxlan udp-port 4789
   vxlan vlan 11 vni 10011
   vxlan vlan 22 vni 10022

ip virtual-router mac-address 00:00:00:00:00:01

ip routing
ip routing vrf OTUS-PROD

router bgp 65001
   !
   vlan 11
      rd 65001:10011
      route-target both 11:11
      redistribute learned
   
   vlan 22
      rd 65001:10022
      route-target both 22:22
      redistribute learned
```

- #### [leaf-2](config/leaf-2.conf)

```
vlan 11,22

vrf instance OTUS-PROD

interface Ethernet3
   description to-client-3
   switchport access vlan 11

interface Ethernet4
   description to-client-4
   switchport access vlan 22

interface Vlan11
   vrf OTUS-PROD
   ip address virtual 192.168.11.254/24

interface Vlan22
   vrf OTUS-PROD
   ip address virtual 192.168.22.254/24

interface Vxlan1
   vxlan source-interface Loopback100
   vxlan udp-port 4789
   vxlan vlan 11 vni 10011
   vxlan vlan 22 vni 10022

ip virtual-router mac-address 00:00:00:00:00:02

ip routing
ip routing vrf OTUS-PROD

router bgp 65002

   vlan 11
      rd 65002:10011
      route-target both 11:11
      redistribute learned
   
   vlan 22
      rd 65002:10022
      route-target both 22:22
      redistribute learned
```

- #### [leaf-3](config/leaf-3.conf)

```
vlan 11,22

vrf instance OTUS-PROD

interface Ethernet3
   description to-clien-5
   switchport access vlan 22

interface Ethernet4
   description to-clien-6
   switchport access vlan 22

interface Vlan11
   vrf OTUS-PROD
   ip address virtual 192.168.11.254/24

interface Vlan22
   vrf OTUS-PROD
   ip address virtual 192.168.22.254/24

interface Vxlan1
   vxlan source-interface Loopback100
   vxlan udp-port 4789
   vxlan vlan 11 vni 10011
   vxlan vlan 22 vni 10022

ip virtual-router mac-address 00:00:00:00:00:03

ip routing
ip routing vrf OTUS-PROD

router bgp 65003
   
   vlan 11
      rd 65003:10011
      route-target both 11:11
      redistribute learned
   
   vlan 22
      rd 65003:10022
      route-target both 22:22
      redistribute learned
```
---

### Проверка связанности клиентов по L3

- #### leaf-1

```
leaf-1#sh ip route vrf OTUS-PROD

 C        192.168.11.0/24 is directly connected, Vlan11
 C        192.168.22.0/24 is directly connected, Vlan22

```
```
leaf-1#show bgp evpn route-type ip-prefix ipv4
BGP routing table information for VRF default
Router identifier 10.1.0.1, local AS number 65001
Route status codes: s - suppressed, * - valid, > - active, E - ECMP head, e - ECMP
                    S - Stale, c - Contributing to ECMP, b - backup
                    % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path

```
```
leaf-1#sh bgp evpn route-type mac-ip
BGP routing table information for VRF default
Router identifier 10.1.0.1, local AS number 65001
Route status codes: s - suppressed, * - valid, > - active, E - ECMP head, e - ECMP
                    S - Stale, c - Contributing to ECMP, b - backup
                    % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >     RD: 65001:10011 mac-ip 0050.7966.6806
                                 -                     -       -       0       i
 * >     RD: 65001:10011 mac-ip 0050.7966.6806 192.168.11.11
                                 -                     -       -       0       i
 * >Ec   RD: 65003:10022 mac-ip 0050.7966.6807
                                 10.100.0.3            -       100     0       65000 65003 i
 *  ec   RD: 65003:10022 mac-ip 0050.7966.6807
                                 10.100.0.3            -       100     0       65000 65003 i
 * >Ec   RD: 65003:10022 mac-ip 0050.7966.6807 192.168.22.31
                                 10.100.0.3            -       100     0       65000 65003 i
 *  ec   RD: 65003:10022 mac-ip 0050.7966.6807 192.168.22.31
                                 10.100.0.3            -       100     0       65000 65003 i
 * >     RD: 65001:10011 mac-ip 0050.7966.6808
                                 -                     -       -       0       i
 * >     RD: 65001:10011 mac-ip 0050.7966.6808 192.168.11.12
                                 -                     -       -       0       i
 * >Ec   RD: 65002:10011 mac-ip 0050.7966.6809
                                 10.100.0.2            -       100     0       65000 65002 i
 *  ec   RD: 65002:10011 mac-ip 0050.7966.6809
                                 10.100.0.2            -       100     0       65000 65002 i
 * >Ec   RD: 65002:10011 mac-ip 0050.7966.6809 192.168.11.21
                                 10.100.0.2            -       100     0       65000 65002 i
 *  ec   RD: 65002:10011 mac-ip 0050.7966.6809 192.168.11.21
                                 10.100.0.2            -       100     0       65000 65002 i
 * >Ec   RD: 65002:10022 mac-ip 0050.7966.680a
                                 10.100.0.2            -       100     0       65000 65002 i
 *  ec   RD: 65002:10022 mac-ip 0050.7966.680a
                                 10.100.0.2            -       100     0       65000 65002 i
 * >Ec   RD: 65002:10022 mac-ip 0050.7966.680a 192.168.22.22
                                 10.100.0.2            -       100     0       65000 65002 i
 *  ec   RD: 65002:10022 mac-ip 0050.7966.680a 192.168.22.22
                                 10.100.0.2            -       100     0       65000 65002 i
 * >Ec   RD: 65003:10022 mac-ip 0050.7966.680b
                                 10.100.0.3            -       100     0       65000 65003 i
 *  ec   RD: 65003:10022 mac-ip 0050.7966.680b
                                 10.100.0.3            -       100     0       65000 65003 i
 * >Ec   RD: 65003:10022 mac-ip 0050.7966.680b 192.168.22.32
                                 10.100.0.3            -       100     0       65000 65003 i
 *  ec   RD: 65003:10022 mac-ip 0050.7966.680b 192.168.22.32
                                 10.100.0.3            -       100     0       65000 65003 i
```
```
leaf-1#show vxlan address-table
          Vxlan Mac Address Table
----------------------------------------------------------------------

VLAN  Mac Address     Type      Prt  VTEP             Moves   Last Move
----  -----------     ----      ---  ----             -----   ---------
  11  0050.7966.6809  EVPN      Vx1  10.100.0.2       1       0:02:40 ago
  22  0050.7966.6807  EVPN      Vx1  10.100.0.3       1       0:01:37 ago
  22  0050.7966.680a  EVPN      Vx1  10.100.0.2       1       0:01:44 ago
  22  0050.7966.680b  EVPN      Vx1  10.100.0.3       1       0:01:32 ago
Total Remote Mac Addresses for this criterion: 4
```
```
leaf-1#sh interfaces vxlan 1
Vxlan1 is up, line protocol is up (connected)
  Hardware is Vxlan
  Source interface is Loopback100 and is active with 10.100.0.1
  Replication/Flood Mode is headend with Flood List Source: EVPN
  Remote MAC learning via EVPN
  VNI mapping to VLANs
  Static VLAN to VNI mapping is
    [11, 10011]       [22, 10022]
  Dynamic VLAN to VNI mapping for 'evpn' is
    [4093, 10001]
  Note: All Dynamic VLANs used by VCS are internal VLANs.
        Use 'show vxlan vni' for details.
  Static VRF to VNI mapping is
   [OTUS-PROD, 10001]
  Headend replication flood vtep list is:
    11 10.100.0.2      10.100.0.3
    22 10.100.0.2      10.100.0.3
  Shared Router MAC is 0000.0000.0000
```
```  
leaf-1#show vxlan vni
VNI to VLAN Mapping for Vxlan1
VNI         VLAN       Source       Interface       802.1Q Tag
----------- ---------- ------------ --------------- ----------
10011       11         static       Ethernet3       untagged
                                    Ethernet4       untagged
                                    Vxlan1          11
10022       22         static       Vxlan1          22

VNI to dynamic VLAN Mapping for Vxlan1
VNI         VLAN       VRF             Source
----------- ---------- --------------- ------------
10001       4093       OTUS-PROD       evpn
```

- #### leaf-2

```
leaf-2#sh ip route vrf OTUS-PROD

 C        192.168.11.0/24 is directly connected, Vlan11
 C        192.168.22.0/24 is directly connected, Vlan22
```
```
leaf-2#sh bgp evpn route-type mac-ip
BGP routing table information for VRF default
Router identifier 10.1.0.2, local AS number 65002
Route status codes: s - suppressed, * - valid, > - active, E - ECMP head, e - ECMP
                    S - Stale, c - Contributing to ECMP, b - backup
                    % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec   RD: 65001:10011 mac-ip 0050.7966.6806
                                 10.100.0.1            -       100     0       65000 65001 i
 *  ec   RD: 65001:10011 mac-ip 0050.7966.6806
                                 10.100.0.1            -       100     0       65000 65001 i
 * >Ec   RD: 65001:10011 mac-ip 0050.7966.6806 192.168.11.11
                                 10.100.0.1            -       100     0       65000 65001 i
 *  ec   RD: 65001:10011 mac-ip 0050.7966.6806 192.168.11.11
                                 10.100.0.1            -       100     0       65000 65001 i
 * >Ec   RD: 65003:10022 mac-ip 0050.7966.6807
                                 10.100.0.3            -       100     0       65000 65003 i
 *  ec   RD: 65003:10022 mac-ip 0050.7966.6807
                                 10.100.0.3            -       100     0       65000 65003 i
 * >Ec   RD: 65003:10022 mac-ip 0050.7966.6807 192.168.22.31
                                 10.100.0.3            -       100     0       65000 65003 i
 *  ec   RD: 65003:10022 mac-ip 0050.7966.6807 192.168.22.31
                                 10.100.0.3            -       100     0       65000 65003 i
 * >Ec   RD: 65001:10011 mac-ip 0050.7966.6808
                                 10.100.0.1            -       100     0       65000 65001 i
 *  ec   RD: 65001:10011 mac-ip 0050.7966.6808
                                 10.100.0.1            -       100     0       65000 65001 i
 * >Ec   RD: 65001:10011 mac-ip 0050.7966.6808 192.168.11.12
                                 10.100.0.1            -       100     0       65000 65001 i
 *  ec   RD: 65001:10011 mac-ip 0050.7966.6808 192.168.11.12
                                 10.100.0.1            -       100     0       65000 65001 i
 * >     RD: 65002:10011 mac-ip 0050.7966.6809
                                 -                     -       -       0       i
 * >     RD: 65002:10011 mac-ip 0050.7966.6809 192.168.11.21
                                 -                     -       -       0       i
 * >     RD: 65002:10022 mac-ip 0050.7966.680a
                                 -                     -       -       0       i
 * >     RD: 65002:10022 mac-ip 0050.7966.680a 192.168.22.22
                                 -                     -       -       0       i
 * >Ec   RD: 65003:10022 mac-ip 0050.7966.680b
                                 10.100.0.3            -       100     0       65000 65003 i
 *  ec   RD: 65003:10022 mac-ip 0050.7966.680b
                                 10.100.0.3            -       100     0       65000 65003 i
 * >Ec   RD: 65003:10022 mac-ip 0050.7966.680b 192.168.22.32
                                 10.100.0.3            -       100     0       65000 65003 i
 *  ec   RD: 65003:10022 mac-ip 0050.7966.680b 192.168.22.32
                                 10.100.0.3            -       100     0       65000 65003 i
```
```
leaf-2#show vxlan address-table
          Vxlan Mac Address Table
----------------------------------------------------------------------

VLAN  Mac Address     Type      Prt  VTEP             Moves   Last Move
----  -----------     ----      ---  ----             -----   ---------
  11  0050.7966.6806  EVPN      Vx1  10.100.0.1       1       0:03:00 ago
  11  0050.7966.6808  EVPN      Vx1  10.100.0.1       1       0:03:00 ago
  22  0050.7966.6807  EVPN      Vx1  10.100.0.3       1       0:01:50 ago
  22  0050.7966.680b  EVPN      Vx1  10.100.0.3       1       0:01:45 ago
Total Remote Mac Addresses for this criterion: 4
```
```
leaf-2#sh interfaces vxlan 1
Vxlan1 is up, line protocol is up (connected)
  Hardware is Vxlan
  Source interface is Loopback100 and is active with 10.100.0.2
  Replication/Flood Mode is headend with Flood List Source: EVPN
  Remote MAC learning via EVPN
  VNI mapping to VLANs
  Static VLAN to VNI mapping is
    [11, 10011]       [22, 10022]
  Dynamic VLAN to VNI mapping for 'evpn' is
    [4091, 10001]
  Note: All Dynamic VLANs used by VCS are internal VLANs.
        Use 'show vxlan vni' for details.
  Static VRF to VNI mapping is
   [OTUS-PROD, 10001]
  Headend replication flood vtep list is:
    11 10.100.0.1      10.100.0.3
    22 10.100.0.1      10.100.0.3
  Shared Router MAC is 0000.0000.0000

```
```  
leaf-2#show vxlan vni
VNI to VLAN Mapping for Vxlan1
VNI         VLAN       Source       Interface       802.1Q Tag
----------- ---------- ------------ --------------- ----------
10011       11         static       Ethernet3       untagged
                                    Vxlan1          11
10022       22         static       Ethernet4       untagged
                                    Vxlan1          22

VNI to dynamic VLAN Mapping for Vxlan1
VNI         VLAN       VRF             Source
----------- ---------- --------------- ------------
10001       4091       OTUS-PROD       evpn
```

- #### leaf-3

```
leaf-3#sh ip route vrf OTUS-PROD

Gateway of last resort is not set

 C        192.168.11.0/24 is directly connected, Vlan11
 C        192.168.22.0/24 is directly connected, Vlan22
```
```
leaf-3#sh bgp evpn route-type mac-ip
BGP routing table information for VRF default
Router identifier 10.1.0.3, local AS number 65003
Route status codes: s - suppressed, * - valid, > - active, E - ECMP head, e - ECMP
                    S - Stale, c - Contributing to ECMP, b - backup
                    % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec   RD: 65001:10011 mac-ip 0050.7966.6806
                                 10.100.0.1            -       100     0       65000 65001 i
 *  ec   RD: 65001:10011 mac-ip 0050.7966.6806
                                 10.100.0.1            -       100     0       65000 65001 i
 * >Ec   RD: 65001:10011 mac-ip 0050.7966.6806 192.168.11.11
                                 10.100.0.1            -       100     0       65000 65001 i
 *  ec   RD: 65001:10011 mac-ip 0050.7966.6806 192.168.11.11
                                 10.100.0.1            -       100     0       65000 65001 i
 * >     RD: 65003:10022 mac-ip 0050.7966.6807
                                 -                     -       -       0       i
 * >     RD: 65003:10022 mac-ip 0050.7966.6807 192.168.22.31
                                 -                     -       -       0       i
 * >Ec   RD: 65001:10011 mac-ip 0050.7966.6808
                                 10.100.0.1            -       100     0       65000 65001 i
 *  ec   RD: 65001:10011 mac-ip 0050.7966.6808
                                 10.100.0.1            -       100     0       65000 65001 i
 * >Ec   RD: 65001:10011 mac-ip 0050.7966.6808 192.168.11.12
                                 10.100.0.1            -       100     0       65000 65001 i
 *  ec   RD: 65001:10011 mac-ip 0050.7966.6808 192.168.11.12
                                 10.100.0.1            -       100     0       65000 65001 i
 * >Ec   RD: 65002:10011 mac-ip 0050.7966.6809
                                 10.100.0.2            -       100     0       65000 65002 i
 *  ec   RD: 65002:10011 mac-ip 0050.7966.6809
                                 10.100.0.2            -       100     0       65000 65002 i
 * >Ec   RD: 65002:10011 mac-ip 0050.7966.6809 192.168.11.21
                                 10.100.0.2            -       100     0       65000 65002 i
 *  ec   RD: 65002:10011 mac-ip 0050.7966.6809 192.168.11.21
                                 10.100.0.2            -       100     0       65000 65002 i
 * >Ec   RD: 65002:10022 mac-ip 0050.7966.680a
                                 10.100.0.2            -       100     0       65000 65002 i
 *  ec   RD: 65002:10022 mac-ip 0050.7966.680a
                                 10.100.0.2            -       100     0       65000 65002 i
 * >Ec   RD: 65002:10022 mac-ip 0050.7966.680a 192.168.22.22
                                 10.100.0.2            -       100     0       65000 65002 i
 *  ec   RD: 65002:10022 mac-ip 0050.7966.680a 192.168.22.22
                                 10.100.0.2            -       100     0       65000 65002 i
 * >     RD: 65003:10022 mac-ip 0050.7966.680b
                                 -                     -       -       0       i
 * >     RD: 65003:10022 mac-ip 0050.7966.680b 192.168.22.32
                                 -                     -       -       0       i
```
```
leaf-3#sh bgp evpn route-type ip-prefix ipv4
BGP routing table information for VRF default
Router identifier 10.1.0.3, local AS number 65003
Route status codes: s - suppressed, * - valid, > - active, E - ECMP head, e - ECMP
                    S - Stale, c - Contributing to ECMP, b - backup
                    % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
```
```
leaf-3#show vxlan address-table
          Vxlan Mac Address Table
----------------------------------------------------------------------

VLAN  Mac Address     Type      Prt  VTEP             Moves   Last Move
----  -----------     ----      ---  ----             -----   ---------
  11  0050.7966.6806  EVPN      Vx1  10.100.0.1       1       0:02:56 ago
  11  0050.7966.6808  EVPN      Vx1  10.100.0.1       1       0:02:55 ago
  11  0050.7966.6809  EVPN      Vx1  10.100.0.2       1       0:02:49 ago
  22  0050.7966.680a  EVPN      Vx1  10.100.0.2       1       0:01:53 ago
Total Remote Mac Addresses for this criterion: 4
```
```
leaf-3#sh interfaces vxlan 1
Vxlan1 is up, line protocol is up (connected)
  Hardware is Vxlan
  Source interface is Loopback100 and is active with 10.100.0.3
  Replication/Flood Mode is headend with Flood List Source: EVPN
  Remote MAC learning via EVPN
  VNI mapping to VLANs
  Static VLAN to VNI mapping is
    [11, 10011]       [22, 10022]
  Dynamic VLAN to VNI mapping for 'evpn' is
    [4094, 10001]
  Note: All Dynamic VLANs used by VCS are internal VLANs.
        Use 'show vxlan vni' for details.
  Static VRF to VNI mapping is
   [OTUS-PROD, 10001]
  Headend replication flood vtep list is:
    11 10.100.0.1      10.100.0.2
    22 10.100.0.1      10.100.0.2
  Shared Router MAC is 0000.0000.0000
```
```
leaf-3#show vxlan vni
VNI to VLAN Mapping for Vxlan1
VNI         VLAN       Source       Interface       802.1Q Tag
----------- ---------- ------------ --------------- ----------
10011       11         static       Vxlan1          11
10022       22         static       Ethernet3       untagged
                                    Ethernet4       untagged
                                    Vxlan1          22

VNI to dynamic VLAN Mapping for Vxlan1
VNI         VLAN       VRF             Source
----------- ---------- --------------- ------------
10001       4094       OTUS-PROD       evpn
```

- #### client-1

```
client-1> ping 192.168.11.12 -c 2

84 bytes from 192.168.11.12 icmp_seq=1 ttl=64 time=7.469 ms
84 bytes from 192.168.11.12 icmp_seq=2 ttl=64 time=3.091 ms

client-1> ping 192.168.11.21 -c 2

84 bytes from 192.168.11.21 icmp_seq=1 ttl=64 time=14.904 ms
84 bytes from 192.168.11.21 icmp_seq=2 ttl=64 time=12.151 ms

client-1> ping 192.168.22.22 -c 2

84 bytes from 192.168.22.22 icmp_seq=1 ttl=63 time=53.579 ms
84 bytes from 192.168.22.22 icmp_seq=2 ttl=63 time=13.193 ms

client-1> ping 192.168.22.31 -c 2

84 bytes from 192.168.22.31 icmp_seq=1 ttl=63 time=59.388 ms
84 bytes from 192.168.22.31 icmp_seq=2 ttl=63 time=15.440 ms

client-1> ping 192.168.22.32 -c 2

84 bytes from 192.168.22.32 icmp_seq=1 ttl=63 time=56.171 ms
84 bytes from 192.168.22.32 icmp_seq=2 ttl=63 time=15.548 ms
```

- #### client-2

```
client-2> ping 192.168.11.11 -c 2

84 bytes from 192.168.11.11 icmp_seq=1 ttl=64 time=4.506 ms
84 bytes from 192.168.11.11 icmp_seq=2 ttl=64 time=5.306 ms

client-2> ping 192.168.11.21 -c 2

84 bytes from 192.168.11.21 icmp_seq=1 ttl=64 time=11.980 ms
84 bytes from 192.168.11.21 icmp_seq=2 ttl=64 time=16.794 ms

client-2> ping 192.168.22.22 -c 2

84 bytes from 192.168.22.22 icmp_seq=1 ttl=63 time=51.609 ms
84 bytes from 192.168.22.22 icmp_seq=2 ttl=63 time=16.343 ms

client-2> ping 192.168.22.31 -c 2

84 bytes from 192.168.22.31 icmp_seq=1 ttl=63 time=90.012 ms
84 bytes from 192.168.22.31 icmp_seq=2 ttl=63 time=17.467 ms

client-2> ping 192.168.22.32 -c 2

84 bytes from 192.168.22.32 icmp_seq=1 ttl=63 time=47.908 ms
84 bytes from 192.168.22.32 icmp_seq=2 ttl=63 time=15.831 ms
```
- #### client-2

```
client-3> ping 192.168.11.11 -c 2

84 bytes from 192.168.11.11 icmp_seq=1 ttl=64 time=13.465 ms
84 bytes from 192.168.11.11 icmp_seq=2 ttl=64 time=12.574 ms

client-3> ping 192.168.11.12 -c 2

84 bytes from 192.168.11.12 icmp_seq=1 ttl=64 time=16.501 ms
84 bytes from 192.168.11.12 icmp_seq=2 ttl=64 time=12.417 ms

client-3> ping 192.168.22.22 -c 2

84 bytes from 192.168.22.22 icmp_seq=1 ttl=63 time=15.387 ms
84 bytes from 192.168.22.22 icmp_seq=2 ttl=63 time=17.354 ms

client-3> ping 192.168.22.31 -c 2

84 bytes from 192.168.22.31 icmp_seq=1 ttl=63 time=21.182 ms
84 bytes from 192.168.22.31 icmp_seq=2 ttl=63 time=20.368 ms

client-3> ping 192.168.22.32 -c 2

84 bytes from 192.168.22.32 icmp_seq=1 ttl=63 time=20.032 ms
84 bytes from 192.168.22.32 icmp_seq=2 ttl=63 time=19.057 ms
```