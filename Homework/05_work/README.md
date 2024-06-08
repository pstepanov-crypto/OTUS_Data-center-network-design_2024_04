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
