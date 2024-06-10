# Домашнее задание №5

## Overlay. VxLAN EVPN L2

### Задача:

- Настроить Overlay на основе VxLAN EVPN для L2 связанности между клиентами
  
## Выполнение:

### Схема сети

![](images/IBGP_EVPN.PNG)

### Конфигурация оборудования

- #### [leaf-1](config/leaf-1.conf)

```
Leaf-1# sh run

!Command: show running-config
!Running configuration last done at: Mon Jun 10 21:18:12 2024
!Time: Mon Jun 10 21:25:12 2024

version 9.3(6) Bios:version
hostname Leaf-1
vdc Leaf-1 id 1
  limit-resource vlan minimum 16 maximum 4094
  limit-resource vrf minimum 2 maximum 4096
  limit-resource port-channel minimum 0 maximum 511
  limit-resource u4route-mem minimum 248 maximum 248
  limit-resource u6route-mem minimum 96 maximum 96
  limit-resource m4route-mem minimum 58 maximum 58
  limit-resource m6route-mem minimum 8 maximum 8

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

no password strength-check
username admin password 5 $5$JPBBJC$39zZxY8lfCntMPJjDj1hYfg9PQhrNJ1ljfDwJa16x7D
 role network-admin
ip domain-lookup
copp profile strict
snmp-server user admin network-admin auth md5 0x895d2007842db7a4c451eb526dbceb97
 priv 0x895d2007842db7a4c451eb526dbceb97 localizedkey
rmon event 1 log trap public description FATAL(1) owner PMON@FATAL
rmon event 2 log trap public description CRITICAL(2) owner PMON@CRITICAL
rmon event 3 log trap public description ERROR(3) owner PMON@ERROR
rmon event 4 log trap public description WARNING(4) owner PMON@WARNING
rmon event 5 log trap public description INFORMATION(5) owner PMON@INFO

vlan 1,100,200
vlan 100
  name Hosts
  vn-segment 100
vlan 200
  name Servers
  vn-segment 200

vrf context management

interface Vlan1

interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback2
  member vni 100
    ingress-replication protocol bgp
  member vni 200
    ingress-replication protocol bgp

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
Leaf-2# sh run

!Command: show running-config
!Running configuration last done at: Mon Jun 10 21:18:32 2024
!Time: Mon Jun 10 21:25:56 2024

version 9.3(6) Bios:version
hostname Leaf-2
vdc Leaf-2 id 1
  limit-resource vlan minimum 16 maximum 4094
  limit-resource vrf minimum 2 maximum 4096
  limit-resource port-channel minimum 0 maximum 511
  limit-resource u4route-mem minimum 248 maximum 248
  limit-resource u6route-mem minimum 96 maximum 96
  limit-resource m4route-mem minimum 58 maximum 58
  limit-resource m6route-mem minimum 8 maximum 8

nv overlay evpn
feature ospf
feature bgp
feature fabric forwarding
feature interface-vlan
feature vn-segment-vlan-based
feature bfd
clock timezone MSK 3 0
feature nv overlay

no password strength-check
username admin password 5 $5$DEAFHG$Zk.XbwUQTiB2PvouRV2K/aLYocC1uuf0Fcki8pZpiU.
 role network-admin
ip domain-lookup
copp profile strict
snmp-server user admin network-admin auth md5 0xc481d9a6d678df710f38b255962f8b3a
 priv 0xc481d9a6d678df710f38b255962f8b3a localizedkey
rmon event 1 log trap public description FATAL(1) owner PMON@FATAL
rmon event 2 log trap public description CRITICAL(2) owner PMON@CRITICAL
rmon event 3 log trap public description ERROR(3) owner PMON@ERROR
rmon event 4 log trap public description WARNING(4) owner PMON@WARNING
rmon event 5 log trap public description INFORMATION(5) owner PMON@INFO

vlan 1,100,200
vlan 100
  name Hosts
  vn-segment 100
vlan 200
  name Servers
  vn-segment 200

vrf context management

interface Vlan1

interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback2
  member vni 100
    ingress-replication protocol bgp
  member vni 200
    ingress-replication protocol bgp

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
Leaf-3# sh run

!Command: show running-config
!Running configuration last done at: Mon Jun 10 21:19:32 2024
!Time: Mon Jun 10 21:27:09 2024

version 9.3(6) Bios:version
hostname Leaf-3
vdc Leaf-3 id 1
  limit-resource vlan minimum 16 maximum 4094
  limit-resource vrf minimum 2 maximum 4096
  limit-resource port-channel minimum 0 maximum 511
  limit-resource u4route-mem minimum 248 maximum 248
  limit-resource u6route-mem minimum 96 maximum 96
  limit-resource m4route-mem minimum 58 maximum 58
  limit-resource m6route-mem minimum 8 maximum 8

nv overlay evpn
feature ospf
feature bgp
feature fabric forwarding
feature interface-vlan
feature vn-segment-vlan-based
feature bfd
clock timezone MSK 3 0
feature nv overlay

no password strength-check
username admin password 5 $5$JMJNCB$OOFbfAItbvnY2saWifJ3sb8F9ixXOOJDD7Vo8PKL98A
 role network-admin
ip domain-lookup
copp profile strict
snmp-server user admin network-admin auth md5 0x137a914874e02521bb89c5e3d99e5bb1
 priv 0x137a914874e02521bb89c5e3d99e5bb1 localizedkey
rmon event 1 log trap public description FATAL(1) owner PMON@FATAL
rmon event 2 log trap public description CRITICAL(2) owner PMON@CRITICAL
rmon event 3 log trap public description ERROR(3) owner PMON@ERROR
rmon event 4 log trap public description WARNING(4) owner PMON@WARNING
rmon event 5 log trap public description INFORMATION(5) owner PMON@INFO

vlan 1,100,200
vlan 100
  name Hosts
  vn-segment 100
vlan 200
  name Servers
  vn-segment 200

vrf context management

interface Vlan1

interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback2
  member vni 100
    ingress-replication protocol bgp
  member vni 200
    ingress-replication protocol bgp

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

```

- #### Проверка связанности Spine-1

```


Spine-1# sh bgp l2vpn evpn summary
Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.1.0.1        4 65200     112     110      130    0    0 01:37:36 5
10.1.0.2        4 65200      93      97      130    0    0 00:00:04 3
10.1.0.3        4 65200     105      93      130    0    0 01:22:20 7
10.2.1.0        4 65200     116      84      130    0    0 01:17:17 15



```

- #### Проверка связанности Spine-2

```


Spine-2# sh bgp l2vpn evpn summary
Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.1.0.1        4 65200      97     112      167    0    0 01:17:08 5
10.1.0.2        4 65200      78      95      167    0    0 00:00:34 3
10.1.0.3        4 65200     106      98      167    0    0 01:18:20 7
10.1.1.0        4 65200     146     109      167    0    0 01:17:14 15




```

- #### Проверка связанности Leaf-1

```

Leaf-1# sh bgp l2vpn evpn
   Network            Next Hop            Metric     LocPrf     Weight Path
Route Distinguisher: 10.1.0.1:32867    (L2VNI 100)
*>l[2]:[0]:[0]:[48]:[0050.7966.6812]:[0]:[0.0.0.0]/216
                      10.1.0.1                          100      32768 i
*>i[2]:[0]:[0]:[48]:[0050.7966.6814]:[0]:[0.0.0.0]/216
                      10.1.0.3                          100          0 i
*>l[2]:[0]:[0]:[48]:[0050.7966.6812]:[32]:[172.16.100.10]/272
                      10.1.0.1                          100      32768 i
*>i[2]:[0]:[0]:[48]:[0050.7966.6814]:[32]:[172.16.100.20]/272
                      10.1.0.3                          100          0 i
*>l[3]:[0]:[32]:[10.1.0.1]/88
                      10.1.0.1                          100      32768 i
*>i[3]:[0]:[32]:[10.1.0.2]/88
                      10.1.0.2                          100          0 i
*>i[3]:[0]:[32]:[10.1.0.3]/88
                      10.1.0.3                          100          0 i

Route Distinguisher: 10.1.0.1:32967    (L2VNI 200)
*>i[2]:[0]:[0]:[48]:[0050.7966.6813]:[0]:[0.0.0.0]/216
                      10.1.0.2                          100          0 i
*>i[2]:[0]:[0]:[48]:[0050.7966.6815]:[0]:[0.0.0.0]/216
                      10.1.0.3                          100          0 i
*>i[2]:[0]:[0]:[48]:[0050.7966.6815]:[32]:[172.16.200.20]/272
                      10.1.0.3                          100          0 i
*>l[3]:[0]:[32]:[10.1.0.1]/88
                      10.1.0.1                          100      32768 i
*>i[3]:[0]:[32]:[10.1.0.2]/88
                      10.1.0.2                          100          0 i
*>i[3]:[0]:[32]:[10.1.0.3]/88
                      10.1.0.3                          100          0 i

Route Distinguisher: 10.1.0.2:3
*>i[2]:[0]:[0]:[48]:[5000.1100.1b08]:[0]:[0.0.0.0]/216
                      10.1.0.2                          100          0 i
* i                   10.1.0.2                          100          0 i

Route Distinguisher: 10.1.0.2:32867
*>i[3]:[0]:[32]:[10.1.0.2]/88
                      10.1.0.2                          100          0 i
* i                   10.1.0.2                          100          0 i

Route Distinguisher: 10.1.0.2:32967
* i[2]:[0]:[0]:[48]:[0050.7966.6813]:[0]:[0.0.0.0]/216
                      10.1.0.2                          100          0 i
*>i                   10.1.0.2                          100          0 i
*>i[3]:[0]:[32]:[10.1.0.2]/88
                      10.1.0.2                          100          0 i
* i                   10.1.0.2                          100          0 i

Route Distinguisher: 10.1.0.3:3
* i[2]:[0]:[0]:[48]:[5000.1000.1b08]:[0]:[0.0.0.0]/216
                      10.1.0.3                          100          0 i
*>i                   10.1.0.3                          100          0 i

Route Distinguisher: 10.1.0.3:32867
* i[2]:[0]:[0]:[48]:[0050.7966.6814]:[0]:[0.0.0.0]/216
                      10.1.0.3                          100          0 i
*>i                   10.1.0.3                          100          0 i
*>i[2]:[0]:[0]:[48]:[0050.7966.6814]:[32]:[172.16.100.20]/272
                      10.1.0.3                          100          0 i
* i                   10.1.0.3                          100          0 i
* i[3]:[0]:[32]:[10.1.0.3]/88
                      10.1.0.3                          100          0 i
*>i                   10.1.0.3                          100          0 i

Route Distinguisher: 10.1.0.3:32967
* i[2]:[0]:[0]:[48]:[0050.7966.6815]:[0]:[0.0.0.0]/216
                      10.1.0.3                          100          0 i
*>i                   10.1.0.3                          100          0 i
*>i[2]:[0]:[0]:[48]:[0050.7966.6815]:[32]:[172.16.200.20]/272
                      10.1.0.3                          100          0 i
* i                   10.1.0.3                          100          0 i
*>i[3]:[0]:[32]:[10.1.0.3]/88
                      10.1.0.3                          100          0 i
* i                   10.1.0.3                          100          0 i

Route Distinguisher: 10.1.0.1:3    (L3VNI 2000)
*>l[2]:[0]:[0]:[48]:[5000.0e00.1b08]:[0]:[0.0.0.0]/216
                      10.1.0.1                          100      32768 i
*>i[2]:[0]:[0]:[48]:[5000.1000.1b08]:[0]:[0.0.0.0]/216
                      10.1.0.3                          100          0 i
*>i[2]:[0]:[0]:[48]:[5000.1100.1b08]:[0]:[0.0.0.0]/216
                      10.1.0.2                          100          0 i
*>i[2]:[0]:[0]:[48]:[0050.7966.6814]:[32]:[172.16.100.20]/272
                      10.1.0.3                          100          0 i
*>i[2]:[0]:[0]:[48]:[0050.7966.6815]:[32]:[172.16.200.20]/272
                      10.1.0.3                          100          0 i

```


```


```

- #### Проверка связанности Leaf-2

```

Leaf-2# sh bgp l2vpn evpn
BGP routing table information for VRF default, address family L2VPN EVPN
BGP table version is 62, Local Router ID is 10.1.0.2
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-i
njected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - b
est2

   Network            Next Hop            Metric     LocPrf     Weight Path
Route Distinguisher: 10.1.0.1:3
*>i[2]:[0]:[0]:[48]:[5000.0e00.1b08]:[0]:[0.0.0.0]/216
                      10.1.0.1                          100          0 i
* i                   10.1.0.1                          100          0 i

Route Distinguisher: 10.1.0.1:32867
*>i[2]:[0]:[0]:[48]:[0050.7966.6812]:[0]:[0.0.0.0]/216
                      10.1.0.1                          100          0 i
* i                   10.1.0.1                          100          0 i
*>i[2]:[0]:[0]:[48]:[0050.7966.6812]:[32]:[172.16.100.10]/272
                      10.1.0.1                          100          0 i
* i                   10.1.0.1                          100          0 i
*>i[3]:[0]:[32]:[10.1.0.1]/88
                      10.1.0.1                          100          0 i
* i                   10.1.0.1                          100          0 i

Route Distinguisher: 10.1.0.1:32967
*>i[3]:[0]:[32]:[10.1.0.1]/88
                      10.1.0.1                          100          0 i
* i                   10.1.0.1                          100          0 i

Route Distinguisher: 10.1.0.2:32867    (L2VNI 100)
*>i[2]:[0]:[0]:[48]:[0050.7966.6812]:[0]:[0.0.0.0]/216
                      10.1.0.1                          100          0 i
*>i[2]:[0]:[0]:[48]:[0050.7966.6814]:[0]:[0.0.0.0]/216
                      10.1.0.3                          100          0 i
*>i[2]:[0]:[0]:[48]:[0050.7966.6812]:[32]:[172.16.100.10]/272
                      10.1.0.1                          100          0 i
*>i[2]:[0]:[0]:[48]:[0050.7966.6814]:[32]:[172.16.100.20]/272
                      10.1.0.3                          100          0 i
*>i[3]:[0]:[32]:[10.1.0.1]/88
                      10.1.0.1                          100          0 i
*>l[3]:[0]:[32]:[10.1.0.2]/88
                      10.1.0.2                          100      32768 i
*>i[3]:[0]:[32]:[10.1.0.3]/88
                      10.1.0.3                          100          0 i

Route Distinguisher: 10.1.0.2:32967    (L2VNI 200)
*>l[2]:[0]:[0]:[48]:[0050.7966.6813]:[0]:[0.0.0.0]/216
                      10.1.0.2                          100      32768 i
*>i[2]:[0]:[0]:[48]:[0050.7966.6815]:[0]:[0.0.0.0]/216
                      10.1.0.3                          100          0 i
*>i[2]:[0]:[0]:[48]:[0050.7966.6815]:[32]:[172.16.200.20]/272
                      10.1.0.3                          100          0 i
*>i[3]:[0]:[32]:[10.1.0.1]/88
                      10.1.0.1                          100          0 i
*>l[3]:[0]:[32]:[10.1.0.2]/88
                      10.1.0.2                          100      32768 i
*>i[3]:[0]:[32]:[10.1.0.3]/88
                      10.1.0.3                          100          0 i

Route Distinguisher: 10.1.0.3:3
*>i[2]:[0]:[0]:[48]:[5000.1000.1b08]:[0]:[0.0.0.0]/216
                      10.1.0.3                          100          0 i
* i                   10.1.0.3                          100          0 i

Route Distinguisher: 10.1.0.3:32867
*>i[2]:[0]:[0]:[48]:[0050.7966.6814]:[0]:[0.0.0.0]/216
                      10.1.0.3                          100          0 i
* i                   10.1.0.3                          100          0 i
*>i[2]:[0]:[0]:[48]:[0050.7966.6814]:[32]:[172.16.100.20]/272
                      10.1.0.3                          100          0 i
* i                   10.1.0.3                          100          0 i
*>i[3]:[0]:[32]:[10.1.0.3]/88
                      10.1.0.3                          100          0 i
* i                   10.1.0.3                          100          0 i

Route Distinguisher: 10.1.0.3:32967
*>i[2]:[0]:[0]:[48]:[0050.7966.6815]:[0]:[0.0.0.0]/216
                      10.1.0.3                          100          0 i
* i                   10.1.0.3                          100          0 i
*>i[2]:[0]:[0]:[48]:[0050.7966.6815]:[32]:[172.16.200.20]/272
                      10.1.0.3                          100          0 i
* i                   10.1.0.3                          100          0 i
*>i[3]:[0]:[32]:[10.1.0.3]/88
                      10.1.0.3                          100          0 i
* i                   10.1.0.3                          100          0 i

Route Distinguisher: 10.1.0.2:3    (L3VNI 2000)
*>i[2]:[0]:[0]:[48]:[5000.0e00.1b08]:[0]:[0.0.0.0]/216
                      10.1.0.1                          100          0 i
*>i[2]:[0]:[0]:[48]:[5000.1000.1b08]:[0]:[0.0.0.0]/216
                      10.1.0.3                          100          0 i
*>l[2]:[0]:[0]:[48]:[5000.1100.1b08]:[0]:[0.0.0.0]/216
                      10.1.0.2                          100      32768 i
*>i[2]:[0]:[0]:[48]:[0050.7966.6812]:[32]:[172.16.100.10]/272
                      10.1.0.1                          100          0 i
*>i[2]:[0]:[0]:[48]:[0050.7966.6814]:[32]:[172.16.100.20]/272
                      10.1.0.3                          100          0 i
*>i[2]:[0]:[0]:[48]:[0050.7966.6815]:[32]:[172.16.200.20]/272
                      10.1.0.3                          100          0 i
```


```


```

- #### Проверка связанности Leaf-3

```

Leaf-3# sh bgp l2vpn evpn
BGP routing table information for VRF default, address family L2VPN EVPN
BGP table version is 229, Local Router ID is 10.1.0.3
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-i
njected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - b
est2

   Network            Next Hop            Metric     LocPrf     Weight Path
Route Distinguisher: 10.1.0.1:3
* i[2]:[0]:[0]:[48]:[5000.0e00.1b08]:[0]:[0.0.0.0]/216
                      10.1.0.1                          100          0 i
*>i                   10.1.0.1                          100          0 i

Route Distinguisher: 10.1.0.1:32867
* i[2]:[0]:[0]:[48]:[0050.7966.6812]:[0]:[0.0.0.0]/216
                      10.1.0.1                          100          0 i
*>i                   10.1.0.1                          100          0 i
*>i[2]:[0]:[0]:[48]:[0050.7966.6812]:[32]:[172.16.100.10]/272
                      10.1.0.1                          100          0 i
* i                   10.1.0.1                          100          0 i
* i[3]:[0]:[32]:[10.1.0.1]/88
                      10.1.0.1                          100          0 i
*>i                   10.1.0.1                          100          0 i

Route Distinguisher: 10.1.0.1:32967
* i[3]:[0]:[32]:[10.1.0.1]/88
                      10.1.0.1                          100          0 i
*>i                   10.1.0.1                          100          0 i

Route Distinguisher: 10.1.0.2:3
*>i[2]:[0]:[0]:[48]:[5000.1100.1b08]:[0]:[0.0.0.0]/216
                      10.1.0.2                          100          0 i
* i                   10.1.0.2                          100          0 i

Route Distinguisher: 10.1.0.2:32867
*>i[3]:[0]:[32]:[10.1.0.2]/88
                      10.1.0.2                          100          0 i
* i                   10.1.0.2                          100          0 i

Route Distinguisher: 10.1.0.2:32967
* i[2]:[0]:[0]:[48]:[0050.7966.6813]:[0]:[0.0.0.0]/216
                      10.1.0.2                          100          0 i
*>i                   10.1.0.2                          100          0 i
*>i[3]:[0]:[32]:[10.1.0.2]/88
                      10.1.0.2                          100          0 i
* i                   10.1.0.2                          100          0 i

Route Distinguisher: 10.1.0.3:32867    (L2VNI 100)
*>i[2]:[0]:[0]:[48]:[0050.7966.6812]:[0]:[0.0.0.0]/216
                      10.1.0.1                          100          0 i
*>l[2]:[0]:[0]:[48]:[0050.7966.6814]:[0]:[0.0.0.0]/216
                      10.1.0.3                          100      32768 i
*>i[2]:[0]:[0]:[48]:[0050.7966.6812]:[32]:[172.16.100.10]/272
                      10.1.0.1                          100          0 i
*>l[2]:[0]:[0]:[48]:[0050.7966.6814]:[32]:[172.16.100.20]/272
                      10.1.0.3                          100      32768 i
*>i[3]:[0]:[32]:[10.1.0.1]/88
                      10.1.0.1                          100          0 i
*>i[3]:[0]:[32]:[10.1.0.2]/88
                      10.1.0.2                          100          0 i
*>l[3]:[0]:[32]:[10.1.0.3]/88
                      10.1.0.3                          100      32768 i

Route Distinguisher: 10.1.0.3:32967    (L2VNI 200)
*>i[2]:[0]:[0]:[48]:[0050.7966.6813]:[0]:[0.0.0.0]/216
                      10.1.0.2                          100          0 i
*>l[2]:[0]:[0]:[48]:[0050.7966.6815]:[0]:[0.0.0.0]/216
                      10.1.0.3                          100      32768 i
*>l[2]:[0]:[0]:[48]:[0050.7966.6815]:[32]:[172.16.200.20]/272
                      10.1.0.3                          100      32768 i
*>i[3]:[0]:[32]:[10.1.0.1]/88
                      10.1.0.1                          100          0 i
*>i[3]:[0]:[32]:[10.1.0.2]/88
                      10.1.0.2                          100          0 i
*>l[3]:[0]:[32]:[10.1.0.3]/88
                      10.1.0.3                          100      32768 i

Route Distinguisher: 10.1.0.3:3    (L3VNI 2000)
*>i[2]:[0]:[0]:[48]:[5000.0e00.1b08]:[0]:[0.0.0.0]/216
                      10.1.0.1                          100          0 i
*>l[2]:[0]:[0]:[48]:[5000.1000.1b08]:[0]:[0.0.0.0]/216
                      10.1.0.3                          100      32768 i
*>i[2]:[0]:[0]:[48]:[5000.1100.1b08]:[0]:[0.0.0.0]/216
                      10.1.0.2                          100          0 i
*>i[2]:[0]:[0]:[48]:[0050.7966.6812]:[32]:[172.16.100.10]/272
                      10.1.0.1                          100          0 i



```
- #### Проверка связанности хостов

```
VPCS1> ping 172.16.200.10
84 bytes from 172.16.200.10 icmp_seq=1 ttl=62 time=15.258 ms
84 bytes from 172.16.200.10 icmp_seq=2 ttl=62 time=14.683 ms
84 bytes from 172.16.200.10 icmp_seq=3 ttl=62 time=13.479 ms
84 bytes from 172.16.200.10 icmp_seq=4 ttl=62 time=13.968 ms
84 bytes from 172.16.200.10 icmp_seq=5 ttl=62 time=17.691 ms

VPCS2> ping 172.16.100.10
84 bytes from 172.16.100.10 icmp_seq=1 ttl=62 time=11.168 ms
84 bytes from 172.16.100.10 icmp_seq=2 ttl=62 time=14.478 ms
84 bytes from 172.16.100.10 icmp_seq=3 ttl=62 time=12.703 ms
84 bytes from 172.16.100.10 icmp_seq=4 ttl=62 time=14.105 ms
84 bytes from 172.16.100.10 icmp_seq=5 ttl=62 time=12.260 ms

VPCS3> ping 172.16.200.10
84 bytes from 172.16.200.10 icmp_seq=1 ttl=62 time=48.669 ms
84 bytes from 172.16.200.10 icmp_seq=2 ttl=62 time=23.180 ms
84 bytes from 172.16.200.10 icmp_seq=3 ttl=62 time=28.885 ms
84 bytes from 172.16.200.10 icmp_seq=4 ttl=62 time=16.711 ms
84 bytes from 172.16.200.10 icmp_seq=5 ttl=62 time=14.302 ms

VPCS4> ping 172.16.100.10
84 bytes from 172.16.100.10 icmp_seq=1 ttl=62 time=44.282 ms
84 bytes from 172.16.100.10 icmp_seq=2 ttl=62 time=18.590 ms
84 bytes from 172.16.100.10 icmp_seq=3 ttl=62 time=12.054 ms
84 bytes from 172.16.100.10 icmp_seq=4 ttl=62 time=44.098 ms
84 bytes from 172.16.100.10 icmp_seq=5 ttl=62 time=15.935 ms
  
```
```


