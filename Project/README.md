# Проектная работа

## Тема: Построение фабрики VXLAN/EVPN для двух POD с использованием технологии multisite

### Цели:

- Построение двух POD VXLAN/EVPN
- Подключение резервного ЦОД с использованием технологии “multisite”

## План:

- Разработка отказоустойчивой и масштабируемой топологии CLOS
- Проектирование DCI с помощью “multisite”
- Проектирование адресного пространства
- Проектирование Underlay и Overlay сетей

### [Презинтация проекта](file/presentation.pptx)

### Схема проектируемой сети

![](images/project-multisite1.PNG)

### Конфигурация оборудования

#### POD 1
- [POD1-BGW-Spine-1](config/POD-1-Spine-1.conf)
- [POD1-BGW-Spine-2](config/POD-1-Spine-2.conf)
- [POD1-Leaf-1](config/POD-1-Leaf-1.conf)
- [POD1-Leaf-2](config/POD-1-Leaf-2.conf)
- [POD1-Leaf-3](config/POD-1-Leaf-3.conf)

#### POD 2
- [POD2-BGW-Spine-m1](config/POD-2-Spine-m1.conf)
- [POD2-BGW-Spine-m2](config/POD-2-Spine-m2.conf)
- [POD2-Leaf-m1](config/POD-2-Leaf-m1.conf)
- [POD2-Leaf-m2](config/POD-2-Leaf-m2.conf)
- [POD2-Leaf-m3](config/POD-2-Leaf-m3.conf)


### Проверка (Underlay. POD 1)
```
Spine-1# sh ip ospf neighbors
 OSPF Process ID UNDERLAY VRF default
 Total number of neighbors: 4
 Neighbor ID     Pri State            Up Time  Address         Interface
 10.1.0.1          1 FULL/ -          01:34:20 10.6.1.1        Eth1/1
 10.1.0.2          1 FULL/ -          01:33:08 10.6.1.3        Eth1/2
 10.1.0.3          1 FULL/ -          01:32:34 10.6.1.5        Eth1/3
 10.2.2.0          1 FULL/ -          00:36:41 172.17.1.2      Eth1/4

```
```
Spine-2# sh ip ospf neighbors
 OSPF Process ID UNDERLAY VRF default
 Total number of neighbors: 4
 Neighbor ID     Pri State            Up Time  Address         Interface
 10.1.0.1          1 FULL/ -          01:50:44 10.6.2.1        Eth1/1
 10.1.0.2          1 FULL/ -          01:49:26 10.6.2.3        Eth1/2
 10.1.0.3          1 FULL/ -          01:48:56 10.6.2.5        Eth1/3
 10.2.3.0          1 FULL/ -          00:00:33 172.18.1.2      Eth1/4

```
### Проверка (Overlay. POD 1)
```
Spine-1# sh bgp l2vpn evpn summary
Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.1.0.1        4 65200     118     117       17    0    0 01:52:16 0
10.1.0.2        4 65200     115     115       17    0    0 01:49:50 0
10.1.0.3        4 65200     115     115       17    0    0 01:49:46 0
10.2.1.0        4 65200     121     116       17    0    0 01:50:45 5
10.2.2.0        4 65202      64      59       17    0    0 00:53:09 5

```

```
Spine-2# sh bgp l2vpn evpn summary
Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.1.0.1        4 65200     277     277       27    0    0 04:32:19 0
10.1.0.2        4 65200     115     114       27    0    0 01:50:02 0
10.1.0.3        4 65200     115     114       27    0    0 01:49:58 0
10.1.1.0        4 65200     121     116       27    0    0 01:51:20 5
10.2.3.0        4 65202      64      60       27    0    0 00:54:12 5
```

```
```
### Проверка (Route. POD 1)
```
Leaf-1# sh ip route vrf main
172.16.100.0/24, ubest/mbest: 1/0, attached
    *via 172.16.100.1, Vlan100, [0/0], 15:42:01, direct
172.16.100.1/32, ubest/mbest: 1/0, attached
    *via 172.16.100.1, Vlan100, [0/0], 15:42:01, local
172.16.100.30/32, ubest/mbest: 1/0, attached
    *via 172.16.100.30, Vlan100, [190/0], 00:24:01, hmm
172.16.100.40/32, ubest/mbest: 1/0
    *via 10.2.0.1%default, [200/0], 00:04:14, bgp-65200, internal, tag 65202, se
gid: 2000 tunnelid: 0xa020001 encap: VXLAN

172.16.100.50/32, ubest/mbest: 1/0, attached
    *via 172.16.100.50, Vlan100, [190/0], 00:24:01, hmm (no-redist)
172.16.200.0/24, ubest/mbest: 1/0, attached
    *via 172.16.200.1, Vlan200, [0/0], 15:42:01, direct
172.16.200.1/32, ubest/mbest: 1/0, attached
    *via 172.16.200.1, Vlan200, [0/0], 15:42:01, local
172.16.200.20/32, ubest/mbest: 1/0
    *via 10.1.0.3%default, [200/0], 00:04:58, bgp-65200, internal, tag 65200, se
gid: 2000 tunnelid: 0xa010003 encap: VXLAN


```
```
<IRF-01>disp ip routing-table vpn-instance DEV

Destinations : 13       Routes : 13

Destination/Mask   Proto   Pre Cost        NextHop         Interface
0.0.0.0/32         Direct  0   0           127.0.0.1       InLoop0
127.0.0.0/8        Direct  0   0           127.0.0.1       InLoop0
127.0.0.1/32       Direct  0   0           127.0.0.1       InLoop0
127.255.255.255/32 Direct  0   0           127.0.0.1       InLoop0
192.168.20.0/24    Direct  0   0           192.168.20.254  Vsi20
192.168.20.2/32    BGP     255 0           10.12.3.0       Vsi2
192.168.20.6/32    BGP     255 0           10.12.5.0       Vsi2
192.168.20.8/32    BGP     255 0           10.12.5.0       Vsi2
192.168.20.254/32  Direct  0   0           127.0.0.1       InLoop0
192.168.20.255/32  Direct  0   0           192.168.20.254  Vsi20
224.0.0.0/4        Direct  0   0           0.0.0.0         NULL0
224.0.0.0/24       Direct  0   0           0.0.0.0         NULL0
255.255.255.255/32 Direct  0   0           127.0.0.1       InLoop0
```
```
<IRF-01>dis bgp l2vpn evpn

 BGP local router ID is 10.12.1.0
 Status codes: * - valid, > - best, d - dampened, h - history,
               s - suppressed, S - stale, i - internal, e - external
               Origin: i - IGP, e - EGP, ? - incomplete

 Total number of routes from all PEs: 20

 Route distinguisher: 1:1010
 Total number of routes: 8

     Network            NextHop         MED        LocPrf     PrefVal Path/Ogn

* >  [2][0][48][cc3e-5f81-85b7][32][192.168.10.1]/136
                        0.0.0.0         0          100        32768   i
* >i [2][0][48][cc3e-5f82-332c][0][0.0.0.0]/104
                        10.12.3.0       0          100        0       i
*  i                    10.12.3.0       0          100        0       i
* >i [2][0][48][cc3e-5f82-332c][32][192.168.10.2]/136
                        10.12.3.0       0          100        0       i
*  i                    10.12.3.0       0          100        0       i
* >  [3][0][32][10.12.1.0]/80
                        0.0.0.0         0          100        32768   i
* >i [3][0][32][10.12.3.0]/80
                        10.12.3.0       0          100        0       i
*  i                    10.12.3.0       0          100        0       i

 Route distinguisher: 1:2010
 Total number of routes: 6

     Network            NextHop         MED        LocPrf     PrefVal Path/Ogn

* >  [2][0][48][cc3e-5f81-85b7][32][192.168.20.1]/136
                        0.0.0.0         0          100        32768   i
* >i [2][0][48][cc3e-5f82-332c][32][192.168.20.2]/136
                        10.12.3.0       0          100        0       i
*  i                    10.12.3.0       0          100        0       i
* >  [3][0][32][10.12.1.0]/80
                        0.0.0.0         0          100        32768   i
* >i [3][0][32][10.12.3.0]/80
                        10.12.3.0       0          100        0       i
*  i                    10.12.3.0       0          100        0       i

 Route distinguisher: 65100:1(PROD)
 Total number of routes: 8

     Network            NextHop         MED        LocPrf     PrefVal Path/Ogn

* >i [2][0][48][cc3e-5f82-332c][32][192.168.10.2]/136
                        10.12.3.0       0          100        0       i
* >  [5][0][24][192.168.10.0]/80
                        0.0.0.0         0          100        32768   i
* >i [5][0][24][192.168.30.0]/80
                        10.12.5.0                  100        0       65200i
*  i                    10.12.5.0                  100        0       65200i
* >i [5][0][32][192.168.10.5]/80
                        10.12.5.0                  100        0       65200i
*  i                    10.12.5.0                  100        0       65200i
* >i [5][0][32][192.168.10.7]/80
                        10.12.5.0                  100        0       65200i
*  i                    10.12.5.0                  100        0       65200i

 Route distinguisher: 65100:2(DEV)
 Total number of routes: 6

     Network            NextHop         MED        LocPrf     PrefVal Path/Ogn

* >i [2][0][48][cc3e-5f82-332c][32][192.168.20.2]/136
                        10.12.3.0       0          100        0       i
* >  [5][0][24][192.168.20.0]/80
                        0.0.0.0         0          100        32768   i
* >i [5][0][32][192.168.20.6]/80
                        10.12.5.0                  100        0       65200i
*  i                    10.12.5.0                  100        0       65200i
* >i [5][0][32][192.168.20.8]/80
                        10.12.5.0                  100        0       65200i
*  i                    10.12.5.0                  100        0       65200i
```
```
<IRF-03>disp bgp peer ipv4 vpn-instance PROD

 BGP local router ID: 10.13.5.1
 Local AS number: 65100
 Total number of peers: 1                 Peers in established state: 1

  * - Dynamically created peer
  Peer                    AS  MsgRcvd  MsgSent OutQ PrefRcv Up/Down  State

  10.23.1.1            65200      192      196    0       2 02:33:46 Established
<IRF-03>
<IRF-03>
<IRF-03>
<IRF-03>disp bgp peer ipv4 vpn-instance DEV

 BGP local router ID: 10.13.5.2
 Local AS number: 65100
 Total number of peers: 1                 Peers in established state: 1

  * - Dynamically created peer
  Peer                    AS  MsgRcvd  MsgSent OutQ PrefRcv Up/Down  State

  10.23.1.2            65200      181      184    0       2 02:30:01 Established
```
```
<server-01>ping -vpn-instance PROD 192.168.10.2
Ping 192.168.10.2 (192.168.10.2): 56 data bytes, press CTRL+C to break
56 bytes from 192.168.10.2: icmp_seq=0 ttl=253 time=6.729 ms
56 bytes from 192.168.10.2: icmp_seq=1 ttl=253 time=2.904 ms
56 bytes from 192.168.10.2: icmp_seq=2 ttl=253 time=3.650 ms
56 bytes from 192.168.10.2: icmp_seq=3 ttl=253 time=2.115 ms
56 bytes from 192.168.10.2: icmp_seq=4 ttl=253 time=2.893 ms

--- Ping statistics for 192.168.10.2 in VPN instance PROD ---
5 packet(s) transmitted, 5 packet(s) received, 0.0% packet loss
round-trip min/avg/max/std-dev = 2.115/3.658/6.729/1.610 ms


<server-01>ping -vpn-instance DEV 192.168.20.2
Ping 192.168.20.2 (192.168.20.2): 56 data bytes, press CTRL+C to break
56 bytes from 192.168.20.2: icmp_seq=0 ttl=253 time=3.569 ms
56 bytes from 192.168.20.2: icmp_seq=1 ttl=253 time=2.257 ms
56 bytes from 192.168.20.2: icmp_seq=2 ttl=253 time=1.925 ms
56 bytes from 192.168.20.2: icmp_seq=3 ttl=253 time=1.925 ms
56 bytes from 192.168.20.2: icmp_seq=4 ttl=253 time=2.471 ms

--- Ping statistics for 192.168.20.2 in VPN instance DEV ---
5 packet(s) transmitted, 5 packet(s) received, 0.0% packet loss
round-trip min/avg/max/std-dev = 1.925/2.429/3.569/0.606 ms
```
### Проверка (Underlay. POD 2)
```
Spine-m1# sh ip ospf neighbors
 OSPF Process ID UNDERLAY VRF default
 Total number of neighbors: 4
 Neighbor ID     Pri State            Up Time  Address         Interface
 10.2.0.1          1 FULL/ -          02:09:45 10.5.1.1        Eth1/1
 10.2.0.2          1 FULL/ -          01:29:12 10.5.1.3        Eth1/2
 10.2.0.3          1 FULL/ -          02:07:10 10.5.1.5        Eth1/3
 10.1.1.0          1 FULL/ -          01:27:28 172.17.1.1      Eth1/4

```
```
Spine-m2# sh ip ospf neighbors
 OSPF Process ID UNDERLAY VRF default
 Total number of neighbors: 4
 Neighbor ID     Pri State            Up Time  Address         Interface
 10.2.0.1          1 FULL/ -          02:10:25 10.5.2.1        Eth1/1
 10.2.0.2          1 FULL/ -          01:29:52 10.5.2.3        Eth1/2
 10.2.0.3          1 FULL/ -          02:07:47 10.5.2.5        Eth1/3
 10.2.1.0          1 INIT/DROTHER     00:34:48 172.18.1.1      Eth1/4


```
### Проверка (Overlay. POD 2)
```
Spine-m1# sh bgp l2vpn evpn summary
Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.1.1.0        4 65200      93      94       21    0    0 01:28:05 0
10.2.0.1        4 65202     141     136       21    0    0 02:09:01 5
10.2.0.2        4 65202      96      96       21    0    0 01:30:13 0
10.2.0.3        4 65202     134     134       21    0    0 02:08:10 0
10.2.3.0        4 65202     143     136       21    0    0 02:10:51 5

```
```
Spine-m2# sh bgp l2vpn evpn summary
Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.2.0.1        4 65202     144     138       21    0    0 02:11:34 5
10.2.0.2        4 65202      96      96       21    0    0 01:31:06 0
10.2.0.3        4 65202     134     134       21    0    0 02:08:15 0
10.2.1.0        4 65200      95      94       21    0    0 01:29:07 0
10.2.2.0        4 65202     143     137       21    0    0 02:11:26 5

```
```
dc02-bgw01#show bgp evpn summary
BGP summary information for VRF default
Router identifier 10.23.1.0, local AS number 65200
Neighbor Status Codes: m - Under maintenance
  Neighbor         V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.21.1.0        4 65200           5950      5904    0    0 04:12:04 Estab   17     17
  10.21.2.0        4 65200           5659      5638    0    0 04:00:17 Estab   17     17
```
```
dc02-bgw02#show bgp evpn summary
BGP summary information for VRF default
Router identifier 10.23.2.0, local AS number 65200
Neighbor Status Codes: m - Under maintenance
  Neighbor         V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.21.1.0        4 65200           5955      5892    0    0 04:12:22 Estab   22     22
  10.21.2.0        4 65200           5679      5637    0    0 04:00:38 Estab   22     22
```
```
dc02-bgw01#sh mlag
MLAG Configuration:
domain-id                          :               mlag1
local-interface                    :            Vlan4094
peer-address                       :           10.26.0.1
peer-link                          :       Port-Channel1
peer-config                        :          consistent

MLAG Status:
state                              :              Active
negotiation status                 :           Connected
peer-link status                   :                  Up
local-int status                   :                  Up
system-id                          :   52:00:00:76:bd:1d
dual-primary detection             :            Disabled
dual-primary interface errdisabled :               False

MLAG Ports:
Disabled                           :                   0
Configured                         :                   0
Inactive                           :                   0
Active-partial                     :                   0
Active-full                        :                   0
```
### Проверка (Route. POD 1)
```
dc02-le01#show ip route vrf PROD

 B I      10.23.1.1/32 [200/0] via VTEP 10.23.1.254 VNI 109999 router-mac 50:00:00:76:bd:1d local-interface Vxlan1
 B I      10.27.1.1/32 [200/0] via VTEP 10.23.1.254 VNI 109999 router-mac 50:00:00:ba:b7:36 local-interface Vxlan1
 C        192.168.10.0/24 is directly connected, Vlan10
 B I      192.168.30.0/24 [200/0] via VTEP 10.22.2.254 VNI 109999 router-mac 50:00:00:d0:ea:86 local-interface Vxlan1

dc02-le01#show ip route vrf DEV

 B I      10.27.1.3/32 [200/0] via VTEP 10.23.1.254 VNI 209999 router-mac 50:00:00:ba:b7:36 local-interface Vxlan1
 B I      192.168.20.8/32 [200/0] via VTEP 10.22.2.254 VNI 209999 router-mac 50:00:00:d0:ea:86 local-interface Vxlan1
 C        192.168.20.0/24 is directly connected, Vlan20
```
```
dc02-bgw01#sh ip route vrf PROD

 S        10.13.5.1/32 [1/0] via 10.27.1.1, Vlan10
 C        10.23.1.1/32 is directly connected, Loopback1
 C        10.27.1.0/31 is directly connected, Vlan10
 B I      192.168.10.0/24 [200/0] via VTEP 10.22.1.254 VNI 109999 router-mac 50:00:00:5f:63:6e local-interface Vxlan1
                                  via VTEP 10.22.2.254 VNI 109999 router-mac 50:00:00:d0:ea:86 local-interface Vxlan1
 B I      192.168.30.0/24 [200/0] via VTEP 10.22.2.254 VNI 109999 router-mac 50:00:00:d0:ea:86 local-interface Vxlan1

dc02-bgw01#sh ip route vrf DEV

 S        10.13.5.2/32 [1/0] via 10.27.1.3, Vlan20
 C        10.23.1.2/32 is directly connected, Loopback2
 C        10.27.1.2/31 is directly connected, Vlan20
 B I      192.168.20.6/32 [200/0] via VTEP 10.22.1.254 VNI 209999 router-mac 50:00:00:5f:63:6e local-interface Vxlan1
 B I      192.168.20.8/32 [200/0] via VTEP 10.22.2.254 VNI 209999 router-mac 50:00:00:d0:ea:86 local-interface Vxlan1
 B I      192.168.20.0/24 [200/0] via VTEP 10.22.2.254 VNI 209999 router-mac 50:00:00:d0:ea:86 local-interface Vxlan1
                                  via VTEP 10.22.1.254 VNI 209999 router-mac 50:00:00:5f:63:6e local-interface Vxlan1
```
```

```
```
dc02-bgw01#show vxlan vni
VNI to VLAN Mapping for Vxlan1
VNI          VLAN       Source       Interface       802.1Q Tag
------------ ---------- ------------ --------------- ----------
100010       10         static       Ethernet5       10
                                     Vxlan1          10
200020       20         static       Ethernet5       20
                                     Vxlan1          20

VNI to dynamic VLAN Mapping for Vxlan1
VNI          VLAN       VRF        Source
------------ ---------- ---------- ------------
109999       4090       PROD       evpn
209999       4091       DEV        evpn
```
```
dc02-le01#show vxlan vni
VNI to VLAN Mapping for Vxlan1
VNI          VLAN       Source       Interface       802.1Q Tag
------------ ---------- ------------ --------------- ----------
100010       10         static       Ethernet3       untagged
                                     Vxlan1          10
200020       20         static       Ethernet4       untagged
                                     Vxlan1          20

VNI to dynamic VLAN Mapping for Vxlan1
VNI          VLAN       VRF        Source
------------ ---------- ---------- ------------
109999       4093       PROD       evpn
209999       4094       DEV        evpn
```
```
VPCS-1m> ping 172.16.200.20
84 bytes from 172.16.200.20 icmp_seq=1 ttl=62 time=25.278 ms
84 bytes from 172.16.200.20 icmp_seq=2 ttl=62 time=17.525 ms
84 bytes from 172.16.200.20 icmp_seq=3 ttl=62 time=18.453 ms
84 bytes from 172.16.200.20 icmp_seq=4 ttl=62 time=19.638 ms
84 bytes from 172.16.200.20 icmp_seq=5 ttl=62 time=20.356 ms


```
```
VPCS-2> ping 172.16.100.40
84 bytes from 172.16.100.40 icmp_seq=1 ttl=62 time=23.922 ms
84 bytes from 172.16.100.40 icmp_seq=2 ttl=62 time=20.967 ms
84 bytes from 172.16.100.40 icmp_seq=3 ttl=62 time=17.761 ms
84 bytes from 172.16.100.40 icmp_seq=4 ttl=62 time=19.283 ms
84 bytes from 172.16.100.40 icmp_seq=5 ttl=62 time=19.140 ms


```
