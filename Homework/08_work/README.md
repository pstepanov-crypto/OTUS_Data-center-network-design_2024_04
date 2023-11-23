# Домашнее задание №8

## Overlay. VxLAN Оптимизация таблиц маршрутизации 

### Задача:

- Анонсировать суммарные префиксы клиентов в Overlay сети
- Настроить маршрутизацию между клиентами через суммарный префикс
- Проверить связанность между клиентами

## Выполнение:

### Схема сети

![](images/Overlay_ROUTE.png)

### Конфигурация оборудования

- #### [leaf-1](config/leaf-1.conf)

```
vlan 11

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

interface Vxlan1
   vxlan source-interface Loopback100
   vxlan udp-port 4789
   vxlan vlan 11 vni 10011
   vxlan vrf OTUS-PROD vni 10001

ip virtual-router mac-address 00:00:00:00:00:01

ip routing vrf OTUS-PROD

router bgp 65001

   vlan 11
      rd 65001:10011
      route-target both 11:11
      redistribute learned

   vrf OTUS-PROD
      rd 65001:10001
      route-target import evpn 10001:10001
      route-target export evpn 10001:10001
      redistribute connected
```

- #### [leaf-2](config/leaf-2.conf)

```
vlan 11,22

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
   vxlan vrf OTUS-PROD vni 10001

ip virtual-router mac-address 00:00:00:00:00:02

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

   vrf OTUS-PROD
      rd 65002:10001
      route-target import evpn 10001:10001
      route-target export evpn 10001:10001
      redistribute connected
```

- #### [leaf-3](config/leaf-3.conf)

```
vlan 13,23

vrf instance OTUS-PROD

interface Ethernet3
   description to-clien-5
   switchport access vlan 13

interface Ethernet4
   description to-clien-6
   switchport access vlan 23

interface Vlan13
   vrf OTUS-PROD
   ip address virtual 192.168.13.254/24

interface Vlan23
   vrf OTUS-PROD
   ip address virtual 192.168.23.254/24

interface Vxlan1
   vxlan source-interface Loopback100
   vxlan udp-port 4789
   vxlan vlan 13 vni 10013
   vxlan vlan 23 vni 10023
   vxlan vrf OTUS-PROD vni 10001

ip virtual-router mac-address 00:00:00:00:00:03

ip routing vrf OTUS-PROD

router bgp 65003

   vlan 13
      rd 65003:10013
      route-target both 13:13
      redistribute learned
   !
   vlan 23
      rd 65003:10023
      route-target both 23:23
      redistribute learned

   vrf OTUS-PROD
      rd 65003:10001
      route-target import evpn 10001:10001
      route-target export evpn 10001:10001
      redistribute connected
```

- #### [router](config/router.conf)

```
interface LoopBack0
 ip address 10.200.0.1 255.255.255.255

interface LoopBack1
 ip address 8.8.8.8 255.255.255.255

interface LoopBack2
 ip address 8.8.4.4 255.255.255.255

interface GigabitEthernet1/0.1000
 description OTUS-PROD
 ip address 172.20.0.254 255.255.255.0
 vlan-type dot1q vid 1000

bgp 64999
 router-id 10.200.0.1
 peer 172.20.0.3 as-number 65003
 
 address-family ipv4 unicast
  default-route imported
  import-route direct
  import-route static
  network 8.8.8.8 255.255.255.255
  peer 172.20.0.3 enable

ip route-static 0.0.0.0 0 NULL0
```
---

### Проверка связанности клиентов по L3

- #### leaf-1

```
leaf-1#sh ip route vrf OTUS-PROD

 B E      0.0.0.0/0 [200/0] via VTEP 10.100.0.3 VNI 10001 router-mac 50:00:00:c3:da:3f local-interface Vxlan1

 B E      8.8.4.4/32 [200/0] via VTEP 10.100.0.3 VNI 10001 router-mac 50:00:00:c3:da:3f local-interface Vxlan1
 B E      8.8.8.8/32 [200/0] via VTEP 10.100.0.3 VNI 10001 router-mac 50:00:00:c3:da:3f local-interface Vxlan1
 B E      10.200.0.1/32 [200/0] via VTEP 10.100.0.3 VNI 10001 router-mac 50:00:00:c3:da:3f local-interface Vxlan1
 B E      172.20.0.0/24 [200/0] via VTEP 10.100.0.3 VNI 10001 router-mac 50:00:00:c3:da:3f local-interface Vxlan1
 B E      192.168.11.21/32 [200/0] via VTEP 10.100.0.2 VNI 10001 router-mac 50:00:00:5b:6f:f5 local-interface Vxlan1
 C        192.168.11.0/24 is directly connected, Vlan11
 B E      192.168.22.22/32 [200/0] via VTEP 10.100.0.2 VNI 10001 router-mac 50:00:00:5b:6f:f5 local-interface Vxlan1
 B E      192.168.22.31/32 [200/0] via VTEP 10.100.0.3 VNI 10001 router-mac 50:00:00:c3:da:3f local-interface Vxlan1
 B E      192.168.22.32/32 [200/0] via VTEP 10.100.0.3 VNI 10001 router-mac 50:00:00:c3:da:3f local-interface Vxlan1
 B E      192.168.22.0/24 [200/0] via VTEP 10.100.0.2 VNI 10001 router-mac 50:00:00:5b:6f:f5 local-interface Vxlan1
                                  via VTEP 10.100.0.3 VNI 10001 router-mac 50:00:00:c3:da:3f local-interface Vxlan1
```
```
leaf-1#sh bgp evpn route-type ip-prefix ipv4
BGP routing table information for VRF default
Router identifier 10.1.0.1, local AS number 65001
Route status codes: s - suppressed, * - valid, > - active, E - ECMP head, e - ECMP
                    S - Stale, c - Contributing to ECMP, b - backup
                    % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec   RD: 65003:10001 ip-prefix 0.0.0.0/0
                                 10.100.0.3            -       100     0       65000 65003 64999 ?
 *  ec   RD: 65003:10001 ip-prefix 0.0.0.0/0
                                 10.100.0.3            -       100     0       65000 65003 64999 ?
 * >Ec   RD: 65003:10001 ip-prefix 8.8.4.4/32
                                 10.100.0.3            -       100     0       65000 65003 64999 ?
 *  ec   RD: 65003:10001 ip-prefix 8.8.4.4/32
                                 10.100.0.3            -       100     0       65000 65003 64999 ?
 * >Ec   RD: 65003:10001 ip-prefix 8.8.8.8/32
                                 10.100.0.3            -       100     0       65000 65003 64999 i
 *  ec   RD: 65003:10001 ip-prefix 8.8.8.8/32
                                 10.100.0.3            -       100     0       65000 65003 64999 i
 * >Ec   RD: 65003:10001 ip-prefix 10.200.0.1/32
                                 10.100.0.3            -       100     0       65000 65003 64999 ?
 *  ec   RD: 65003:10001 ip-prefix 10.200.0.1/32
                                 10.100.0.3            -       100     0       65000 65003 64999 ?
 * >Ec   RD: 65003:10001 ip-prefix 172.20.0.0/24
                                 10.100.0.3            -       100     0       65000 65003 i
 *  ec   RD: 65003:10001 ip-prefix 172.20.0.0/24
                                 10.100.0.3            -       100     0       65000 65003 i
 * >     RD: 65001:10001 ip-prefix 192.168.11.0/24
                                 -                     -       -       0       i
 * >Ec   RD: 65002:10001 ip-prefix 192.168.11.0/24
                                 10.100.0.2            -       100     0       65000 65002 i
 *  ec   RD: 65002:10001 ip-prefix 192.168.11.0/24
                                 10.100.0.2            -       100     0       65000 65002 i
 * >Ec   RD: 65002:10001 ip-prefix 192.168.22.0/24
                                 10.100.0.2            -       100     0       65000 65002 i
 *  ec   RD: 65002:10001 ip-prefix 192.168.22.0/24
                                 10.100.0.2            -       100     0       65000 65002 i
 * >Ec   RD: 65003:10001 ip-prefix 192.168.22.0/24
                                 10.100.0.3            -       100     0       65000 65003 i
 *  ec   RD: 65003:10001 ip-prefix 192.168.22.0/24
                                 10.100.0.3            -       100     0       65000 65003 i
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
    [11, 10011]
  Dynamic VLAN to VNI mapping for 'evpn' is
    [4094, 10001]
  Note: All Dynamic VLANs used by VCS are internal VLANs.
        Use 'show vxlan vni' for details.
  Static VRF to VNI mapping is
   [OTUS-PROD, 10001]
  Headend replication flood vtep list is:
    11 10.100.0.2
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

VNI to dynamic VLAN Mapping for Vxlan1
VNI         VLAN       VRF             Source
----------- ---------- --------------- ------------
10001       4094       OTUS-PROD       evpn
```

- #### leaf-2

```
leaf-2#sh ip route vrf OTUS-PROD

 B E      0.0.0.0/0 [200/0] via VTEP 10.100.0.3 VNI 10001 router-mac 50:00:00:c3:da:3f local-interface Vxlan1

 B E      8.8.4.4/32 [200/0] via VTEP 10.100.0.3 VNI 10001 router-mac 50:00:00:c3:da:3f local-interface Vxlan1
 B E      8.8.8.8/32 [200/0] via VTEP 10.100.0.3 VNI 10001 router-mac 50:00:00:c3:da:3f local-interface Vxlan1
 B E      10.200.0.1/32 [200/0] via VTEP 10.100.0.3 VNI 10001 router-mac 50:00:00:c3:da:3f local-interface Vxlan1
 B E      172.20.0.0/24 [200/0] via VTEP 10.100.0.3 VNI 10001 router-mac 50:00:00:c3:da:3f local-interface Vxlan1
 B E      192.168.11.11/32 [200/0] via VTEP 10.100.0.1 VNI 10001 router-mac 50:00:00:be:a1:e3 local-interface Vxlan1
 B E      192.168.11.12/32 [200/0] via VTEP 10.100.0.1 VNI 10001 router-mac 50:00:00:be:a1:e3 local-interface Vxlan1
 C        192.168.11.0/24 is directly connected, Vlan11
 B E      192.168.22.31/32 [200/0] via VTEP 10.100.0.3 VNI 10001 router-mac 50:00:00:c3:da:3f local-interface Vxlan1
 B E      192.168.22.32/32 [200/0] via VTEP 10.100.0.3 VNI 10001 router-mac 50:00:00:c3:da:3f local-interface Vxlan1
 C        192.168.22.0/24 is directly connected, Vlan22
```
```
leaf-2#sh bgp evpn route-type ip-prefix ipv4
BGP routing table information for VRF default
Router identifier 10.1.0.2, local AS number 65002
Route status codes: s - suppressed, * - valid, > - active, E - ECMP head, e - ECMP
                    S - Stale, c - Contributing to ECMP, b - backup
                    % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec   RD: 65003:10001 ip-prefix 0.0.0.0/0
                                 10.100.0.3            -       100     0       65000 65003 64999 ?
 *  ec   RD: 65003:10001 ip-prefix 0.0.0.0/0
                                 10.100.0.3            -       100     0       65000 65003 64999 ?
 * >Ec   RD: 65003:10001 ip-prefix 8.8.4.4/32
                                 10.100.0.3            -       100     0       65000 65003 64999 ?
 *  ec   RD: 65003:10001 ip-prefix 8.8.4.4/32
                                 10.100.0.3            -       100     0       65000 65003 64999 ?
 * >Ec   RD: 65003:10001 ip-prefix 8.8.8.8/32
                                 10.100.0.3            -       100     0       65000 65003 64999 i
 *  ec   RD: 65003:10001 ip-prefix 8.8.8.8/32
                                 10.100.0.3            -       100     0       65000 65003 64999 i
 * >Ec   RD: 65003:10001 ip-prefix 10.200.0.1/32
                                 10.100.0.3            -       100     0       65000 65003 64999 ?
 *  ec   RD: 65003:10001 ip-prefix 10.200.0.1/32
                                 10.100.0.3            -       100     0       65000 65003 64999 ?
 * >Ec   RD: 65003:10001 ip-prefix 172.20.0.0/24
                                 10.100.0.3            -       100     0       65000 65003 i
 *  ec   RD: 65003:10001 ip-prefix 172.20.0.0/24
                                 10.100.0.3            -       100     0       65000 65003 i
 * >Ec   RD: 65001:10001 ip-prefix 192.168.11.0/24
                                 10.100.0.1            -       100     0       65000 65001 i
 *  ec   RD: 65001:10001 ip-prefix 192.168.11.0/24
                                 10.100.0.1            -       100     0       65000 65001 i
 * >     RD: 65002:10001 ip-prefix 192.168.11.0/24
                                 -                     -       -       0       i
 * >     RD: 65002:10001 ip-prefix 192.168.22.0/24
                                 -                     -       -       0       i
 * >Ec   RD: 65003:10001 ip-prefix 192.168.22.0/24
                                 10.100.0.3            -       100     0       65000 65003 i
 *  ec   RD: 65003:10001 ip-prefix 192.168.22.0/24
                                 10.100.0.3            -       100     0       65000 65003 i
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
    [4094, 10001]
  Note: All Dynamic VLANs used by VCS are internal VLANs.
        Use 'show vxlan vni' for details.
  Static VRF to VNI mapping is
   [OTUS-PROD, 10001]
  Headend replication flood vtep list is:
    11 10.100.0.1
    22 10.100.0.3
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
10001       4094       OTUS-PROD       evpn
```

- #### leaf-3

```
leaf-3#sh ip route vrf OTUS-PROD

 B E      0.0.0.0/0 [200/0] via 172.20.0.254, Vlan1000

 B E      8.8.4.4/32 [200/0] via 172.20.0.254, Vlan1000
 B E      8.8.8.8/32 [200/0] via 172.20.0.254, Vlan1000
 B E      10.200.0.1/32 [200/0] via 172.20.0.254, Vlan1000
 C        172.20.0.0/24 is directly connected, Vlan1000
 B E      192.168.11.11/32 [200/0] via VTEP 10.100.0.1 VNI 10001 router-mac 50:00:00:be:a1:e3 local-interface Vxlan1
 B E      192.168.11.12/32 [200/0] via VTEP 10.100.0.1 VNI 10001 router-mac 50:00:00:be:a1:e3 local-interface Vxlan1
 B E      192.168.11.21/32 [200/0] via VTEP 10.100.0.2 VNI 10001 router-mac 50:00:00:5b:6f:f5 local-interface Vxlan1
 B E      192.168.11.0/24 [200/0] via VTEP 10.100.0.2 VNI 10001 router-mac 50:00:00:5b:6f:f5 local-interface Vxlan1
                                  via VTEP 10.100.0.1 VNI 10001 router-mac 50:00:00:be:a1:e3 local-interface Vxlan1
 B E      192.168.22.22/32 [200/0] via VTEP 10.100.0.2 VNI 10001 router-mac 50:00:00:5b:6f:f5 local-interface Vxlan1
 C        192.168.22.0/24 is directly connected, Vlan22
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
 * >     RD: 65003:10001 ip-prefix 0.0.0.0/0
                                 -                     0       100     0       64999 ?
 * >     RD: 65003:10001 ip-prefix 8.8.4.4/32
                                 -                     0       100     0       64999 ?
 * >     RD: 65003:10001 ip-prefix 8.8.8.8/32
                                 -                     0       100     0       64999 i
 * >     RD: 65003:10001 ip-prefix 10.200.0.1/32
                                 -                     0       100     0       64999 ?
 * >     RD: 65003:10001 ip-prefix 172.20.0.0/24
                                 -                     -       -       0       i
 *       RD: 65003:10001 ip-prefix 172.20.0.0/24
                                 -                     0       100     0       64999 ?
 * >Ec   RD: 65001:10001 ip-prefix 192.168.11.0/24
                                 10.100.0.1            -       100     0       65000 65001 i
 *  ec   RD: 65001:10001 ip-prefix 192.168.11.0/24
                                 10.100.0.1            -       100     0       65000 65001 i
 * >Ec   RD: 65002:10001 ip-prefix 192.168.11.0/24
                                 10.100.0.2            -       100     0       65000 65002 i
 *  ec   RD: 65002:10001 ip-prefix 192.168.11.0/24
                                 10.100.0.2            -       100     0       65000 65002 i
 * >Ec   RD: 65002:10001 ip-prefix 192.168.22.0/24
                                 10.100.0.2            -       100     0       65000 65002 i
 *  ec   RD: 65002:10001 ip-prefix 192.168.22.0/24
                                 10.100.0.2            -       100     0       65000 65002 i
 * >     RD: 65003:10001 ip-prefix 192.168.22.0/24
                                 -                     -       -       0       i
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
    [22, 10022]
  Dynamic VLAN to VNI mapping for 'evpn' is
    [4094, 10001]
  Note: All Dynamic VLANs used by VCS are internal VLANs.
        Use 'show vxlan vni' for details.
  Static VRF to VNI mapping is
   [OTUS-PROD, 10001]
  Headend replication flood vtep list is:
    22 10.100.0.2
  Shared Router MAC is 0000.0000.0000
```
```
leaf-3#show vxlan vni
VNI to VLAN Mapping for Vxlan1
VNI         VLAN       Source       Interface       802.1Q Tag
----------- ---------- ------------ --------------- ----------
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

84 bytes from 192.168.11.12 icmp_seq=1 ttl=64 time=6.718 ms
84 bytes from 192.168.11.12 icmp_seq=2 ttl=64 time=3.473 ms

client-1> ping 192.168.11.21 -c 2

84 bytes from 192.168.11.21 icmp_seq=1 ttl=64 time=12.843 ms
84 bytes from 192.168.11.21 icmp_seq=2 ttl=64 time=11.981 ms

client-1> ping 192.168.22.22 -c 2

84 bytes from 192.168.22.22 icmp_seq=1 ttl=62 time=17.394 ms
84 bytes from 192.168.22.22 icmp_seq=2 ttl=62 time=12.500 ms

client-1> ping 192.168.22.31 -c 2

84 bytes from 192.168.22.31 icmp_seq=1 ttl=62 time=18.353 ms
84 bytes from 192.168.22.31 icmp_seq=2 ttl=62 time=11.603 ms

client-1> ping 192.168.22.32 -c 2

84 bytes from 192.168.22.32 icmp_seq=1 ttl=62 time=16.225 ms
84 bytes from 192.168.22.32 icmp_seq=2 ttl=62 time=12.242 ms

client-1> ping 8.8.8.8 -c 2

84 bytes from 8.8.8.8 icmp_seq=1 ttl=253 time=13.354 ms
84 bytes from 8.8.8.8 icmp_seq=2 ttl=253 time=15.227 ms
```

- #### client-4

```
client-4> ping 192.168.11.11 -c 2

84 bytes from 192.168.11.11 icmp_seq=1 ttl=62 time=18.603 ms
84 bytes from 192.168.11.11 icmp_seq=2 ttl=62 time=11.262 ms

client-4> ping 192.168.11.12 -c 2

84 bytes from 192.168.11.12 icmp_seq=1 ttl=62 time=18.301 ms
84 bytes from 192.168.11.12 icmp_seq=2 ttl=62 time=14.211 ms

client-4> ping 192.168.11.21 -c 2

84 bytes from 192.168.11.21 icmp_seq=1 ttl=63 time=10.425 ms
84 bytes from 192.168.11.21 icmp_seq=2 ttl=63 time=5.281 ms

client-4> ping 192.168.22.31 -c 2

84 bytes from 192.168.22.31 icmp_seq=1 ttl=64 time=14.184 ms
84 bytes from 192.168.22.31 icmp_seq=2 ttl=64 time=15.091 ms

client-4> ping 192.168.22.32 -c 2

84 bytes from 192.168.22.32 icmp_seq=1 ttl=64 time=8.892 ms
84 bytes from 192.168.22.32 icmp_seq=2 ttl=64 time=13.229 ms

client-4> ping 8.8.8.8 -c 2

84 bytes from 8.8.8.8 icmp_seq=1 ttl=253 time=14.873 ms
84 bytes from 8.8.8.8 icmp_seq=2 ttl=253 time=13.024 ms
```