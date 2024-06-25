# Домашнее задание №7

## Настроить отказоустойчивое подключение клиентов с использованием EVPN Multihoming

### Задача:

- Подключить клиентов 2-я линками к различным Leaf
- Настроить агрегированный канал со стороны клиента 
- Настроите multihoming для работы в Overlay сети. Если используете Cisco NXOS - vPC, если иной вендор - то ESI LAG (либо MC-LAG с поддержкой VXLAN)
- Проверить связанность между клиентами

## Выполнение:

### Схема сети

![](images/VPC-evpn.PNG)

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
feature lacp
feature vpc
feature bfd
clock timezone MSK 3 0
feature nv overlay

no password strength-check
username admin password 5 $5$KKHKOP$HZju.WhjeHfDQWoywO8kKFJ5ab5xLYqU6qw0gJdUnE4
 role network-admin
ip domain-lookup
spanning-tree mode mst
copp profile strict

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

spanning-tree port type edge bpduguard default
spanning-tree loopguard default
spanning-tree mst 0-1 priority 4096
spanning-tree mst configuration
  name N-Leafs101-102
  revision 1
  instance 1 vlan 1-4094
vrf context VPC
vrf context main
  vni 2000
  rd auto
  address-family ipv4 unicast
    route-target both auto
    route-target both auto evpn
vrf context management
vpc domain 101
  peer-switch
  role priority 100
  peer-keepalive destination 1.1.1.2 source 1.1.1.1 vrf VPC
  peer-gateway
  layer3 peer-router
  auto-recovery
  delay restore interface-vlan 100
  ip arp synchronize


interface Vlan1
  no ip redirects
  no ipv6 redirects

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

interface port-channel1
  description vpc-per-link
  switchport mode trunk
  spanning-tree port type network
  vpc peer-link

interface port-channel101
  switchport access vlan 100
  vpc 101

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
  spanning-tree guard root
  channel-group 101 mode active

interface Ethernet1/4
  description vpc-per-link
  switchport mode trunk
  channel-group 1 mode active

interface Ethernet1/5
  description vpc-per-link
  switchport mode trunk
  channel-group 1 mode active

interface Ethernet1/6
  description vpc-keepalive
  no switchport
  vrf member VPC
  ip address 1.1.1.1/30
  no shutdown

interface mgmt0
  vrf member management

interface loopback2
  description VTEP
  ip address 10.1.0.1/32
  ip address 10.6.255.200/32 secondary
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



---
Leaf-1# sh interface e1/3
Ethernet1/3 is down (suspended(no LACP PDUs))
admin state is up, Dedicated Interface
  Belongs to Po101
  Hardware: 100/1000/10000 Ethernet, address: 5000.0300.0103 (bia 5000.0300.0103)
  Description: VPC1
  MTU 1500 bytes, BW 1000000 Kbit , DLY 10 usec
  reliability 255/255, txload 1/255, rxload 1/255
  Encapsulation ARPA, medium is broadcast
  Port mode is access
  auto-duplex, auto-speed
  Beacon is turned off
  Auto-Negotiation is turned on  FEC mode is Auto
  Input flow-control is off, output flow-control is off
  Auto-mdix is turned off
  Switchport monitor is off
  EtherType is 0x8100
  EEE (efficient-ethernet) : n/a
    admin fec state is auto, oper fec state is off
  Last link flapped 01:25:24
  Last clearing of "show interface" counters never
  0 interface resets
  Load-Interval #1: 30 seconds
    30 seconds input rate 0 bits/sec, 0 packets/sec
    30 seconds output rate 72 bits/sec, 0 packets/sec
    input rate 0 bps, 0 pps; output rate 72 bps, 0 pps
  Load-Interval #2: 5 minute (300 seconds)
    300 seconds input rate 0 bits/sec, 0 packets/sec
    300 seconds output rate 56 bits/sec, 0 packets/sec
    input rate 0 bps, 0 pps; output rate 56 bps, 0 pps
  RX
    6 unicast packets  7 multicast packets  1 broadcast packets
    14 input packets  1842 bytes
    0 jumbo packets  0 storm suppression packets
    0 runts  0 giants  0 CRC  0 no buffer
    0 input error  0 short frame  0 overrun   0 underrun  0 ignored
    0 watchdog  0 bad etype drop  0 bad proto drop  0 if down drop
    0 input with dribble  0 input discard
    0 Rx pause
  TX
    0 unicast packets  753 multicast packets  0 broadcast packets
    753 output packets  118056 bytes
    0 jumbo packets
    0 output error  0 collision  0 deferred  0 late collision
    0 lost carrier  0 no carrier  0 babble  0 output discard
    0 Tx pause

---

```

- #### [leaf-2](config/leaf-2.conf)

```
cfs eth distribute
nv overlay evpn
feature ospf
feature bgp
feature fabric forwarding
feature interface-vlan
feature vn-segment-vlan-based
feature lacp
feature vpc
feature bfd
clock timezone MSK 3 0
feature nv overlay

no password strength-check
username admin password 5 $5$KIOBMB$/BR2tcIpAyJuzrRj/KMxrGo5rFx9.mAPt0SKsyc8YU/
 role network-admin
ip domain-lookup
spanning-tree mode mst
copp profile strict

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

spanning-tree port type edge bpduguard default
spanning-tree loopguard default
spanning-tree mst 0-1 priority 4096
spanning-tree mst configuration
  name N-Leafs101-102
  revision 1
  instance 1 vlan 1-4094
vrf context VPC
vrf context main
  vni 2000
  rd auto
  address-family ipv4 unicast
    route-target both auto
    route-target both auto evpn
vrf context management
vpc domain 101
  peer-switch
  role priority 50
  peer-keepalive destination 1.1.1.1 source 1.1.1.2 vrf VPC
  peer-gateway
  layer3 peer-router
  auto-recovery
  delay restore interface-vlan 100
  ip arp synchronize


interface Vlan1
  no ip redirects
  no ipv6 redirects

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

interface port-channel1
  description vpc-per-link
  switchport mode trunk
  spanning-tree port type network
  vpc peer-link

interface port-channel101
  switchport access vlan 100
  vpc 101

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
  description VPC1
  switchport access vlan 100
  spanning-tree guard root
  channel-group 101 mode active

interface Ethernet1/4
  description vpc-per-link
  switchport mode trunk
  channel-group 1 mode active

interface Ethernet1/5
  description vpc-per-link
  switchport mode trunk
  channel-group 1 mode active

interface Ethernet1/6
  description vpc-keepalive
  no switchport
  vrf member VPC
  ip address 1.1.1.2/30
  no shutdown

interface mgmt0
  vrf member management

interface loopback2
  description VTEP
  ip address 10.1.0.2/32
  ip address 10.6.255.200/32 secondary
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

- #### [server-1](config/server-1.conf)

```
#bond0
auto bond0
iface bond0 inet static
    address 172.16.100.10
    netmask 255.255.255.0
    gateway 172.16.100.1
    bond-slaves ens3 ens4
    bond-mode 802.3ad
    bond-miimon 100

user@debian:~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 50:69:1e:00:06:00 brd ff:ff:ff:ff:ff:ff
3: ens4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 50:69:1e:00:06:01 brd ff:ff:ff:ff:ff:ff
4: ens5: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 50:00:00:06:00:fe brd ff:ff:ff:ff:ff:ff
    inet 169.254.2.101/30 brd 169.254.2.103 scope link dynamic noprefixroute ens5
       valid_lft 85316sec preferred_lft 85316sec
    inet6 fec0::9de:a56e:ca2a:7041/64 scope site temporary dynamic
       valid_lft 86311sec preferred_lft 14311sec
    inet6 fec0::5200:ff:fe06:fe/64 scope site dynamic mngtmpaddr noprefixroute
       valid_lft 86311sec preferred_lft 14311sec
    inet6 fe80::5200:ff:fe06:fe/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
5: bond0: <NO-CARRIER,BROADCAST,MULTICAST,MASTER,UP> mtu 1500 qdisc noqueue state DOWN group default qlen 1000
    link/ether 82:cd:ba:d4:ca:45 brd ff:ff:ff:ff:ff:ff
    inet 172.16.100.10/24 brd 172.16.100.255 scope global bond0
       valid_lft forever preferred_lft forever
    inet6 fe80::5269:1eff:fe00:600/64 scope link
       valid_lft forever preferred_lft forever
```
---

### Проверка связанности клиентов по L3

- #### leaf-1

```
Leaf-1#  sh vpc
Legend:
                (*) - local vPC is down, forwarding via vPC peer-link

vPC domain id                     : 101
Peer status                       : peer adjacency formed ok
vPC keep-alive status             : peer is alive
Configuration consistency status  : success
Per-vlan consistency status       : success
Type-2 consistency status         : success
vPC role                          : secondary
Number of vPCs configured         : 1
Peer Gateway                      : Enabled
Dual-active excluded VLANs        : -
Graceful Consistency Check        : Enabled
Auto-recovery status              : Enabled, timer is off.(timeout = 240s)
Delay-restore status              : Timer is off.(timeout = 30s)
Delay-restore SVI status          : Timer is off.(timeout = 100s)
Operational Layer3 Peer-router    : Enabled
Virtual-peerlink mode             : Disabled

vPC Peer-link status
---------------------------------------------------------------------
id    Port   Status Active vlans
--    ----   ------ -------------------------------------------------
1     Po1    up     1,100,200,2000


vPC status
----------------------------------------------------------------------------
Id    Port          Status Consistency Reason                Active vlans
--    ------------  ------ ----------- ------                ---------------
101   Po101         down*  success     success               -


```
```
Leaf-2#  sh vpc
Legend:
                (*) - local vPC is down, forwarding via vPC peer-link

vPC domain id                     : 101
Peer status                       : peer adjacency formed ok
vPC keep-alive status             : peer is alive
Configuration consistency status  : success
Per-vlan consistency status       : success
Type-2 consistency status         : success
vPC role                          : primary
Number of vPCs configured         : 1
Peer Gateway                      : Enabled
Dual-active excluded VLANs        : -
Graceful Consistency Check        : Enabled
Auto-recovery status              : Enabled, timer is off.(timeout = 240s)
Delay-restore status              : Timer is off.(timeout = 30s)
Delay-restore SVI status          : Timer is off.(timeout = 100s)
Operational Layer3 Peer-router    : Enabled
Virtual-peerlink mode             : Disabled

vPC Peer-link status
---------------------------------------------------------------------
id    Port   Status Active vlans
--    ----   ------ -------------------------------------------------
1     Po1    up     1,100,200,2000


vPC status
----------------------------------------------------------------------------
Id    Port          Status Consistency Reason                Active vlans
--    ------------  ------ ----------- ------                ---------------
101   Po101         down*  success     success               -


```
```
Leaf-1# sh bgp l2vpn evpn
BGP routing table information for VRF default, address family L2VPN EVPN
BGP table version is 39, Local Router ID is 10.1.0.1
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-i
njected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - b
est2

   Network            Next Hop            Metric     LocPrf     Weight Path
Route Distinguisher: 10.1.0.1:32867    (L2VNI 100)
*>i[3]:[0]:[32]:[10.1.0.3]/88
                      10.1.0.3                          100          0 i
*>l[3]:[0]:[32]:[10.6.255.200]/88
                      10.6.255.200                      100      32768 i

Route Distinguisher: 10.1.0.1:32967    (L2VNI 200)
*>i[2]:[0]:[0]:[48]:[0050.7966.6808]:[0]:[0.0.0.0]/216
                      10.1.0.3                          100          0 i
*>i[3]:[0]:[32]:[10.1.0.3]/88
                      10.1.0.3                          100          0 i
*>l[3]:[0]:[32]:[10.6.255.200]/88
                      10.6.255.200                      100      32768 i

Route Distinguisher: 10.1.0.3:3
*>i[2]:[0]:[0]:[48]:[5000.0500.1b08]:[0]:[0.0.0.0]/216
                      10.1.0.3                          100          0 i
* i                   10.1.0.3                          100          0 i

Route Distinguisher: 10.1.0.3:32867
*>i[3]:[0]:[32]:[10.1.0.3]/88
                      10.1.0.3                          100          0 i
* i                   10.1.0.3                          100          0 i

Route Distinguisher: 10.1.0.3:32967
*>i[2]:[0]:[0]:[48]:[0050.7966.6808]:[0]:[0.0.0.0]/216
                      10.1.0.3                          100          0 i
* i                   10.1.0.3                          100          0 i
*>i[3]:[0]:[32]:[10.1.0.3]/88
                      10.1.0.3                          100          0 i
* i                   10.1.0.3                          100          0 i

Route Distinguisher: 10.1.0.1:4    (L3VNI 2000)
*>l[2]:[0]:[0]:[48]:[5000.0300.1b08]:[0]:[0.0.0.0]/216
                      10.6.255.200                      100      32768 i
*>i[2]:[0]:[0]:[48]:[5000.0500.1b08]:[0]:[0.0.0.0]/216
                      10.1.0.3                          100          0 i


```
```
Leaf-1# sh ip route vrf main
172.16.100.0/24, ubest/mbest: 1/0, attached
    *via 172.16.100.1, Vlan100, [0/0], 03:21:38, direct
172.16.100.1/32, ubest/mbest: 1/0, attached
    *via 172.16.100.1, Vlan100, [0/0], 03:21:38, local
172.16.100.20/32, ubest/mbest: 1/0
    *via 10.1.0.3%default, [200/0], 00:01:17, bgp-65200, internal, tag 65200, se
gid: 2000 tunnelid: 0xa010003 encap: VXLAN

172.16.200.0/24, ubest/mbest: 1/0, attached
    *via 172.16.200.1, Vlan200, [0/0], 03:21:38, direct
172.16.200.1/32, ubest/mbest: 1/0, attached
    *via 172.16.200.1, Vlan200, [0/0], 03:21:38, local
172.16.200.20/32, ubest/mbest: 1/0
    *via 10.1.0.3%default, [200/0], 00:01:28, bgp-65200, internal, tag 65200, se
gid: 2000 tunnelid: 0xa010003 encap: VXLAN

```
```
Leaf-1# sh nve peers
Interface Peer-IP                                 State LearnType Uptime   Route
r-Mac
--------- --------------------------------------  ----- --------- -------- -----
------------
nve1      10.1.0.3                                Up    CP        00:35:29 5000.
0500.1b08


```
```
Leaf-1# show vpc consistency-parameters global

    Legend:
        Type 1 : vPC will be suspended in case of mismatch

Name                        Type  Local Value            Peer Value
-------------               ----  ---------------------- -----------------------
STP MST Simulate PVST       1     Enabled                Enabled
STP Port Type, Edge         1     Normal, Disabled,      Normal, Disabled,
BPDUFilter, Edge BPDUGuard        Enabled                Enabled
STP MST Region Name         1     N-Leafs101-102         N-Leafs101-102
STP Disabled                1     None                   None
STP Mode                    1     MST                    MST
STP Bridge Assurance        1     Enabled                Enabled
STP Loopguard               1     Enabled                Enabled
STP MST Region Instance to  1
 VLAN Mapping
STP MST Region Revision     1     1                      1
Interface-vlan admin up     2     100,200,2000           100,200,2000
Interface-vlan routing      2     1,100,200,2000         1,100,200,2000
capability
Nve1 Adm St, Src Adm St,    1     Up, Up, 10.6.255.200,  Up, Up, 10.6.255.200,
Sec IP, Host Reach, VMAC          CP, TRUE, Disabled,    CP, TRUE, Disabled,
Adv, SA,mcast l2, mcast           0.0.0.0, 0.0.0.0,      0.0.0.0, 0.0.0.0,
l3, IR BGP,MS Adm St, Reo         Disabled, Down,        Disabled, Down,
                                  0.0.0.0                0.0.0.0
Xconnect Vlans              1
QoS (Cos)                   2     ([0-7], [], [], [],    ([0-7], [], [], [],
                                  [], [], [], [])        [], [], [], [])
Network QoS (MTU)           2     (1500, 1500, 1500,     (1500, 1500, 1500,
                                  1500, 0, 0, 0, 0)      1500, 0, 0, 0, 0)
Network Qos (Pause:         2     (F, F, F, F, F, F, F,  (F, F, F, F, F, F, F,
T->Enabled, F->Disabled)          F)                     F)
Input Queuing (Bandwidth)   2     (0, 0, 0, 0, 0, 0, 0,  (0, 0, 0, 0, 0, 0, 0,
                                  0)                     0)
Input Queuing (Absolute     2     (F, F, F, F, F, F, F,  (F, F, F, F, F, F, F,
Priority: T->Enabled,             F)                     F)
F->Disabled)
Output Queuing (Bandwidth   2     (100, 0, 0, 0, 0, 0,   (100, 0, 0, 0, 0, 0,
Remaining)                        0, 0)                  0, 0)
Output Queuing (Absolute    2     (F, F, F, T, F, F, F,  (F, F, F, T, F, F, F,
Priority: T->Enabled,             F)                     F)
F->Disabled)
Allowed VLANs               -     1,100,200,2000         1,100,200,2000
Local suspended VLANs       -     -                      -


```

- #### leaf-2

```
Leaf-2# sh vpc
Legend:
                (*) - local vPC is down, forwarding via vPC peer-link

vPC domain id                     : 101
Peer status                       : peer adjacency formed ok
vPC keep-alive status             : peer is alive
Configuration consistency status  : success
Per-vlan consistency status       : success
Type-2 consistency status         : success
vPC role                          : primary
Number of vPCs configured         : 1
Peer Gateway                      : Enabled
Dual-active excluded VLANs        : -
Graceful Consistency Check        : Enabled
Auto-recovery status              : Enabled, timer is off.(timeout = 240s)
Delay-restore status              : Timer is off.(timeout = 30s)
Delay-restore SVI status          : Timer is off.(timeout = 100s)
Operational Layer3 Peer-router    : Enabled
Virtual-peerlink mode             : Disabled

vPC Peer-link status
---------------------------------------------------------------------
id    Port   Status Active vlans
--    ----   ------ -------------------------------------------------
1     Po1    up     1,100,200,2000


vPC status
----------------------------------------------------------------------------
Id    Port          Status Consistency Reason                Active vlans
--    ------------  ------ ----------- ------                ---------------
101   Po101         down*  success     success               -

```
```
leaf-2#sh mlag interfaces detail
                                                            local/remote
   mlag             state       local       remote        oper        config       last change    changes
---------- ----------------- ----------- ------------ ----------- ------------- ----------------- -------
      5       active-full         Po5          Po5       up/up       ena/ena       0:26:56 ago          8
```
```
Leaf-2# show vpc consistency-parameters global

    Legend:
        Type 1 : vPC will be suspended in case of mismatch

Name                        Type  Local Value            Peer Value
-------------               ----  ---------------------- -----------------------
STP MST Simulate PVST       1     Enabled                Enabled
STP Port Type, Edge         1     Normal, Disabled,      Normal, Disabled,
BPDUFilter, Edge BPDUGuard        Enabled                Enabled
STP MST Region Name         1     N-Leafs101-102         N-Leafs101-102
STP Disabled                1     None                   None
STP Mode                    1     MST                    MST
STP Bridge Assurance        1     Enabled                Enabled
STP Loopguard               1     Enabled                Enabled
STP MST Region Instance to  1
 VLAN Mapping
STP MST Region Revision     1     1                      1
Interface-vlan admin up     2     100,200,2000           100,200,2000
Interface-vlan routing      2     1,100,200,2000         1,100,200,2000
capability
Nve1 Adm St, Src Adm St,    1     Up, Up, 10.6.255.200,  Up, Up, 10.6.255.200,
Sec IP, Host Reach, VMAC          CP, TRUE, Disabled,    CP, TRUE, Disabled,
Adv, SA,mcast l2, mcast           0.0.0.0, 0.0.0.0,      0.0.0.0, 0.0.0.0,
l3, IR BGP,MS Adm St, Reo         Disabled, Down,        Disabled, Down,
                                  0.0.0.0                0.0.0.0
Xconnect Vlans              1
QoS (Cos)                   2     ([0-7], [], [], [],    ([0-7], [], [], [],
                                  [], [], [], [])        [], [], [], [])
Network QoS (MTU)           2     (1500, 1500, 1500,     (1500, 1500, 1500,
                                  1500, 0, 0, 0, 0)      1500, 0, 0, 0, 0)
Network Qos (Pause:         2     (F, F, F, F, F, F, F,  (F, F, F, F, F, F, F,
T->Enabled, F->Disabled)          F)                     F)
Input Queuing (Bandwidth)   2     (0, 0, 0, 0, 0, 0, 0,  (0, 0, 0, 0, 0, 0, 0,
                                  0)                     0)
Input Queuing (Absolute     2     (F, F, F, F, F, F, F,  (F, F, F, F, F, F, F,
Priority: T->Enabled,             F)                     F)
F->Disabled)
Output Queuing (Bandwidth   2     (100, 0, 0, 0, 0, 0,   (100, 0, 0, 0, 0, 0,
Remaining)                        0, 0)                  0, 0)
Output Queuing (Absolute    2     (F, F, F, T, F, F, F,  (F, F, F, T, F, F, F,
Priority: T->Enabled,             F)                     F)
F->Disabled)
Allowed VLANs               -     1,100,200,2000         1,100,200,2000
Local suspended VLANs       -     -                      -

```
```
Leaf-2# sh bgp l2vpn evpn
BGP routing table information for VRF default, address family L2VPN EVPN
BGP table version is 55, Local Router ID is 10.1.0.2
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-i
njected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - b
est2

   Network            Next Hop            Metric     LocPrf     Weight Path
Route Distinguisher: 10.1.0.2:32867    (L2VNI 100)
*>i[2]:[0]:[0]:[48]:[0050.7966.6807]:[0]:[0.0.0.0]/216
                      10.1.0.3                          100          0 i
*>i[2]:[0]:[0]:[48]:[0050.7966.6807]:[32]:[172.16.100.20]/272
                      10.1.0.3                          100          0 i
*>i[3]:[0]:[32]:[10.1.0.3]/88
                      10.1.0.3                          100          0 i
*>l[3]:[0]:[32]:[10.6.255.200]/88
                      10.6.255.200                      100      32768 i

Route Distinguisher: 10.1.0.2:32967    (L2VNI 200)
*>i[2]:[0]:[0]:[48]:[0050.7966.6808]:[0]:[0.0.0.0]/216
                      10.1.0.3                          100          0 i
*>i[2]:[0]:[0]:[48]:[0050.7966.6808]:[32]:[172.16.200.20]/272
                      10.1.0.3                          100          0 i
*>i[3]:[0]:[32]:[10.1.0.3]/88
                      10.1.0.3                          100          0 i
*>l[3]:[0]:[32]:[10.6.255.200]/88
                      10.6.255.200                      100      32768 i

Route Distinguisher: 10.1.0.3:3
* i[2]:[0]:[0]:[48]:[5000.0500.1b08]:[0]:[0.0.0.0]/216
                      10.1.0.3                          100          0 i
*>i                   10.1.0.3                          100          0 i

Route Distinguisher: 10.1.0.3:32867
* i[2]:[0]:[0]:[48]:[0050.7966.6807]:[0]:[0.0.0.0]/216
                      10.1.0.3                          100          0 i
*>i                   10.1.0.3                          100          0 i
*>i[2]:[0]:[0]:[48]:[0050.7966.6807]:[32]:[172.16.100.20]/272
                      10.1.0.3                          100          0 i
* i                   10.1.0.3                          100          0 i
* i[3]:[0]:[32]:[10.1.0.3]/88
                      10.1.0.3                          100          0 i
*>i                   10.1.0.3                          100          0 i

Route Distinguisher: 10.1.0.3:32967
*>i[2]:[0]:[0]:[48]:[0050.7966.6808]:[0]:[0.0.0.0]/216
                      10.1.0.3                          100          0 i
* i                   10.1.0.3                          100          0 i
* i[2]:[0]:[0]:[48]:[0050.7966.6808]:[32]:[172.16.200.20]/272
                      10.1.0.3                          100          0 i
*>i                   10.1.0.3                          100          0 i
* i[3]:[0]:[32]:[10.1.0.3]/88
                      10.1.0.3                          100          0 i
*>i                   10.1.0.3                          100          0 i

Route Distinguisher: 10.1.0.2:4    (L3VNI 2000)
*>l[2]:[0]:[0]:[48]:[5000.0400.1b08]:[0]:[0.0.0.0]/216
                      10.6.255.200                      100      32768 i
*>i[2]:[0]:[0]:[48]:[5000.0500.1b08]:[0]:[0.0.0.0]/216
                      10.1.0.3                          100          0 i
*>i[2]:[0]:[0]:[48]:[0050.7966.6807]:[32]:[172.16.100.20]/272
                      10.1.0.3                          100          0 i
*>i[2]:[0]:[0]:[48]:[0050.7966.6808]:[32]:[172.16.200.20]/272
                      10.1.0.3                          100          0 i

```
```
Leaf-2# sh ip route vrf main
172.16.100.0/24, ubest/mbest: 1/0, attached
    *via 172.16.100.1, Vlan100, [0/0], 02:28:08, direct
172.16.100.1/32, ubest/mbest: 1/0, attached
    *via 172.16.100.1, Vlan100, [0/0], 02:28:08, local
172.16.100.20/32, ubest/mbest: 1/0
    *via 10.1.0.3%default, [200/0], 00:08:02, bgp-65200, internal, tag 65200, se
gid: 2000 tunnelid: 0xa010003 encap: VXLAN

172.16.200.0/24, ubest/mbest: 1/0, attached
    *via 172.16.200.1, Vlan200, [0/0], 02:28:08, direct
172.16.200.1/32, ubest/mbest: 1/0, attached
    *via 172.16.200.1, Vlan200, [0/0], 02:28:08, local
172.16.200.20/32, ubest/mbest: 1/0
    *via 10.1.0.3%default, [200/0], 00:08:13, bgp-65200, internal, tag 65200, se
gid: 2000 tunnelid: 0xa010003 encap: VXLAN

```
```
Leaf-2# sh l2route mac-ip all detail
Flags -(Rmac):Router MAC (Stt):Static (L):Local (R):Remote (V):vPC link
(Dup):Duplicate (Spl):Split (Rcv):Recv(D):Del Pending (S):Stale (C):Clear
(Ps):Peer Sync (Ro):Re-Originated (Orp):Orphan
Topology    Mac Address    Host IP                                 Prod   Flags
        Seq No     Next-Hops
----------- -------------- --------------------------------------- ------ ------
---- ---------- ---------------------------------------
100         0050.7966.6807 172.16.100.20                           BGP    --
        0         10.1.0.3 (Label: 100)
            encap-type:1
200         0050.7966.6808 172.16.200.20                           BGP    --
        0         10.1.0.3 (Label: 200)
            encap-type:1

```
```
Leaf-2# sh nve peers
Interface Peer-IP                                 State LearnType Uptime   Route
r-Mac
--------- --------------------------------------  ----- --------- -------- -----
------------
nve1      10.1.0.3                                Up    CP        00:54:35 5000.
0500.1b08

```
```  
Leaf-2# show nve vni
Codes: CP - Control Plane        DP - Data Plane
       UC - Unconfigured         SA - Suppress ARP
       SU - Suppress Unknown Unicast
       Xconn - Crossconnect
       MS-IR - Multisite Ingress Replication

Interface VNI      Multicast-group   State Mode Type [BD/VRF]      Flags
--------- -------- ----------------- ----- ---- ------------------ -----
nve1      100      UnicastBGP        Up    CP   L2 [100]
nve1      200      UnicastBGP        Up    CP   L2 [200]
nve1      2000     n/a               Up    CP   L3 [main]


```

- #### leaf-3

```
Leaf-3# sh l2route mac-ip all detail
Flags -(Rmac):Router MAC (Stt):Static (L):Local (R):Remote (V):vPC link
(Dup):Duplicate (Spl):Split (Rcv):Recv(D):Del Pending (S):Stale (C):Clear
(Ps):Peer Sync (Ro):Re-Originated (Orp):Orphan
Topology    Mac Address    Host IP                                 Prod   Flags
        Seq No     Next-Hops
----------- -------------- --------------------------------------- ------ ------
---- ---------- ---------------------------------------
100         0050.7966.6807 172.16.100.20                           HMM    L,
        0         Local
            L3-Info: 2000
            Sent To: BGP
200         0050.7966.6808 172.16.200.20                           HMM    L,
        0         Local
            L3-Info: 2000
            Sent To: BGP

```
```
Leaf-3# sh bgp l2vpn evpn
BGP routing table information for VRF default, address family L2VPN EVPN
BGP table version is 41, Local Router ID is 10.1.0.3
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-i
njected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - b
est2

   Network            Next Hop            Metric     LocPrf     Weight Path
Route Distinguisher: 10.1.0.1:4
*>i[2]:[0]:[0]:[48]:[5000.0300.1b08]:[0]:[0.0.0.0]/216
                      10.6.255.200                      100          0 i
* i                   10.6.255.200                      100          0 i

Route Distinguisher: 10.1.0.1:32867
*>i[3]:[0]:[32]:[10.6.255.200]/88
                      10.6.255.200                      100          0 i
* i                   10.6.255.200                      100          0 i

Route Distinguisher: 10.1.0.1:32967
*>i[3]:[0]:[32]:[10.6.255.200]/88
                      10.6.255.200                      100          0 i
* i                   10.6.255.200                      100          0 i

Route Distinguisher: 10.1.0.2:4
* i[2]:[0]:[0]:[48]:[5000.0400.1b08]:[0]:[0.0.0.0]/216
                      10.6.255.200                      100          0 i
*>i                   10.6.255.200                      100          0 i

Route Distinguisher: 10.1.0.2:32867
* i[3]:[0]:[32]:[10.6.255.200]/88
                      10.6.255.200                      100          0 i
*>i                   10.6.255.200                      100          0 i

Route Distinguisher: 10.1.0.2:32967
* i[3]:[0]:[32]:[10.6.255.200]/88
                      10.6.255.200                      100          0 i
*>i                   10.6.255.200                      100          0 i

Route Distinguisher: 10.1.0.3:32867    (L2VNI 100)
*>l[2]:[0]:[0]:[48]:[0050.7966.6807]:[0]:[0.0.0.0]/216
                      10.1.0.3                          100      32768 i
*>l[2]:[0]:[0]:[48]:[0050.7966.6807]:[32]:[172.16.100.20]/272
                      10.1.0.3                          100      32768 i
*>l[3]:[0]:[32]:[10.1.0.3]/88
                      10.1.0.3                          100      32768 i
*>i[3]:[0]:[32]:[10.6.255.200]/88
                      10.6.255.200                      100          0 i
* i                   10.6.255.200                      100          0 i

Route Distinguisher: 10.1.0.3:32967    (L2VNI 200)
*>l[2]:[0]:[0]:[48]:[0050.7966.6808]:[0]:[0.0.0.0]/216
                      10.1.0.3                          100      32768 i
*>l[2]:[0]:[0]:[48]:[0050.7966.6808]:[32]:[172.16.200.20]/272
                      10.1.0.3                          100      32768 i
*>l[3]:[0]:[32]:[10.1.0.3]/88
                      10.1.0.3                          100      32768 i
*>i[3]:[0]:[32]:[10.6.255.200]/88
                      10.6.255.200                      100          0 i
* i                   10.6.255.200                      100          0 i

Route Distinguisher: 10.1.0.3:3    (L3VNI 2000)
*>i[2]:[0]:[0]:[48]:[5000.0300.1b08]:[0]:[0.0.0.0]/216
                      10.6.255.200                      100          0 i
*>i[2]:[0]:[0]:[48]:[5000.0400.1b08]:[0]:[0.0.0.0]/216
                      10.6.255.200                      100          0 i
*>l[2]:[0]:[0]:[48]:[5000.0500.1b08]:[0]:[0.0.0.0]/216
                      10.1.0.3                          100      32768 i

```
```
Leaf-3# sh nve peers
Interface Peer-IP                                 State LearnType Uptime   Route
r-Mac
--------- --------------------------------------  ----- --------- -------- -----
------------
nve1      10.6.255.200                            Up    CP        00:58:37 n/a

```
```
Leaf-3# show interface nve 1
nve1 is up
admin state is up,  Hardware: NVE
  MTU 9216 bytes
  Encapsulation VXLAN
  Auto-mdix is turned off
  RX
    ucast: 0 pkts, 0 bytes - mcast: 2 pkts, 148 bytes
  TX
    ucast: 5 pkts, 604 bytes - mcast: 0 pkts, 0 bytes

```
```
Leaf-3# show nve vni
Codes: CP - Control Plane        DP - Data Plane
       UC - Unconfigured         SA - Suppress ARP
       SU - Suppress Unknown Unicast
       Xconn - Crossconnect
       MS-IR - Multisite Ingress Replication

Interface VNI      Multicast-group   State Mode Type [BD/VRF]      Flags
--------- -------- ----------------- ----- ---- ------------------ -----
nve1      100      UnicastBGP        Up    CP   L2 [100]
nve1      200      UnicastBGP        Up    CP   L2 [200]
nve1      2000     n/a               Up    CP   L3 [main]

```

- #### server-1

```
#bond0
auto bond0
iface bond0 inet static
    address 172.16.100.10
    netmask 255.255.255.0
    gateway 172.16.100.1
    bond-slaves ens3 ens4
    bond-mode 802.3ad
    bond-miimon 100

user@debian:~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 50:69:1e:00:06:00 brd ff:ff:ff:ff:ff:ff
3: ens4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 50:69:1e:00:06:01 brd ff:ff:ff:ff:ff:ff
4: ens5: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 50:00:00:06:00:fe brd ff:ff:ff:ff:ff:ff
    inet 169.254.2.101/30 brd 169.254.2.103 scope link dynamic noprefixroute ens5
       valid_lft 85316sec preferred_lft 85316sec
    inet6 fec0::9de:a56e:ca2a:7041/64 scope site temporary dynamic
       valid_lft 86311sec preferred_lft 14311sec
    inet6 fec0::5200:ff:fe06:fe/64 scope site dynamic mngtmpaddr noprefixroute
       valid_lft 86311sec preferred_lft 14311sec
    inet6 fe80::5200:ff:fe06:fe/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
5: bond0: <NO-CARRIER,BROADCAST,MULTICAST,MASTER,UP> mtu 1500 qdisc noqueue state DOWN group default qlen 1000
    link/ether 82:cd:ba:d4:ca:45 brd ff:ff:ff:ff:ff:ff
    inet 172.16.100.10/24 brd 172.16.100.255 scope global bond0
       valid_lft forever preferred_lft forever
    inet6 fe80::5269:1eff:fe00:600/64 scope link
       valid_lft forever preferred_lft forever

```

- #### VPC2

```
VPCS2> ping 172.16.100.20
172.16.100.20 icmp_seq=1 timeout
84 bytes from 172.16.100.20 icmp_seq=2 ttl=63 time=6.425 ms
84 bytes from 172.16.100.20 icmp_seq=3 ttl=63 time=4.216 ms
84 bytes from 172.16.100.20 icmp_seq=4 ttl=63 time=4.107 ms
84 bytes from 172.16.100.20 icmp_seq=5 ttl=63 time=5.393 ms


```

- #### VPC2

```
VPCS2> ping 172.16.200.20

84 bytes from 172.16.200.20 icmp_seq=1 ttl=63 time=10.976 ms
84 bytes from 172.16.200.20 icmp_seq=2 ttl=63 time=4.679 ms
84 bytes from 172.16.200.20 icmp_seq=3 ttl=63 time=7.146 ms
84 bytes from 172.16.200.20 icmp_seq=4 ttl=63 time=4.220 ms
84 bytes from 172.16.200.20 icmp_seq=5 ttl=63 time=4.436 ms

```
