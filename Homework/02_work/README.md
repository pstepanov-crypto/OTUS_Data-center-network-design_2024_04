# Домашнее задание №2
## Underlay. OSPF

### Задача:

- Настроите OSPF в Underlay сети, для IP связанности между всеми сетевыми устройствами
- Убедитесь в наличии IP связанности между устройствами в OSFP домене

## Выполнение:

### Схема сети

![](images/osfp.PNG)


### Конфигурация оборудования

- #### [leaf-1](config/leaf-1.conf)

```
feature ospf
feature bfd

router ospf UNDERLAY
  bfd
  router-id 10.1.0.1
  passive-interface default

interface Ethernet1/1
  description to-spine-1
  no switchport
  bfd interval 100 min_rx 100 multiplier 3
  bfd authentication Keyed-SHA1 key-id 1 hex-key 0ABCD123
  no ip redirects
  ip address 10.6.1.1/31
  ip ospf authentication message-digest
  ip ospf message-digest-key 1 md5 3 bcdd5b8a8b5fb88e
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf UNDERLAY area 0.0.0.30
  ip ospf bfd
  no shutdown

interface Ethernet1/2
  description to-spine-2
  no switchport
  bfd interval 100 min_rx 100 multiplier 3
  bfd authentication Keyed-SHA1 key-id 1 hex-key 0ABCD123
  no ip redirects
  ip address 10.6.2.1/31
  ip ospf authentication message-digest
  ip ospf message-digest-key 1 md5 3 bcdd5b8a8b5fb88e
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf UNDERLAY area 0.0.0.30
  ip ospf bfd
  no shutdown


interface loopback2
  ip address 10.1.0.1/32
  ip router ospf UNDERLAY area 0.0.0.30
```

- #### [leaf-2](config/leaf-2.conf)

```
feature ospf
feature bfd

router ospf Underlay
  bfd
  router-id 10.1.0.2
  passive-interface default

interface Ethernet1/1
  description to-spine-1
  no switchport
  bfd interval 100 min_rx 100 multiplier 3
  bfd authentication Keyed-SHA1 key-id 1 hex-key 0ABCD123
  no ip redirects
  ip address 10.6.1.3/31
  ip ospf authentication message-digest
  ip ospf message-digest-key 1 md5 3 bcdd5b8a8b5fb88e
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf UNDERLAY area 0.0.0.30
  ip ospf bfd
  no shutdown

interface Ethernet1/2
  description to-spine-2
  no switchport
  bfd interval 100 min_rx 100 multiplier 3
  bfd authentication Keyed-SHA1 key-id 1 hex-key 0ABCD123
  no ip redirects
  ip address 10.6.2.3/31
  ip ospf authentication message-digest
  ip ospf message-digest-key 1 md5 3 bcdd5b8a8b5fb88e
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf UNDERLAY area 0.0.0.30
  ip ospf bfd
  no shutdown


interface loopback2
  ip address 10.1.0.2/32
  ip router ospf UNDERLAY area 0.0.0.30
```

- #### [leaf-3](config/leaf-3.conf)

```
feature ospf
feature bfd

router ospf UNDERLAY
  bfd
  router-id 10.1.0.3
  passive-interface default

interface Ethernet1/1
  description to-spine-1
  no switchport
  bfd interval 100 min_rx 100 multiplier 3
  bfd authentication Keyed-SHA1 key-id 1 hex-key 0ABCD123
  no ip redirects
  ip address 10.6.1.5/31
  ip ospf authentication message-digest
  ip ospf message-digest-key 1 md5 3 bcdd5b8a8b5fb88e
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf UNDERLAY area 0.0.0.30
  ip ospf bfd
  no shutdown

interface Ethernet1/2
  description to-spine-2
  no switchport
  bfd interval 100 min_rx 100 multiplier 3
  bfd authentication Keyed-SHA1 key-id 1 hex-key 0ABCD123
  no ip redirects
  ip address 10.6.2.5/31
  ip ospf authentication message-digest
  ip ospf message-digest-key 1 md5 3 bcdd5b8a8b5fb88e
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf UNDERLAY area 0.0.0.30
  ip ospf bfd
  no shutdown


interface loopback2
  ip address 10.1.0.3/32
  ip router ospf UNDERLAY area 0.0.0.30
```

- #### [spine-1](config/spine-1.conf)

```
feature ospf
feature bfd

router ospf UNDERLAY
  bfd
  router-id 10.1.1.0
  passive-interface default

interface Ethernet1/1
  description to-leaf-1
  no switchport
  bfd interval 100 min_rx 100 multiplier 3
  bfd authentication Keyed-SHA1 key-id 1 hex-key 0ABCD123
  no ip redirects
  ip address 10.6.1.0/31
  ip ospf authentication message-digest
  ip ospf message-digest-key 1 md5 3 bcdd5b8a8b5fb88e
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf UNDERLAY area 0.0.0.30
  ip ospf bfd
  no shutdown

interface Ethernet1/2
  description to-leaf-2
  no switchport
  bfd interval 100 min_rx 100 multiplier 3
  bfd authentication Keyed-SHA1 key-id 1 hex-key 0ABCD123
  no ip redirects
  ip address 10.6.1.2/31
  ip ospf authentication message-digest
  ip ospf message-digest-key 1 md5 3 bcdd5b8a8b5fb88e
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf UNDERLAY area 0.0.0.30
  ip ospf bfd
  no shutdown

interface Ethernet1/3
  description to-leaf-3
  no switchport
  bfd interval 100 min_rx 100 multiplier 3
  bfd authentication Keyed-SHA1 key-id 1 hex-key 0ABCD123
  no ip redirects
  ip address 10.6.1.4/31
  ip ospf authentication message-digest
  ip ospf message-digest-key 1 md5 3 bcdd5b8a8b5fb88e
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf UNDERLAY area 0.0.0.30
  ip ospf bfd
  no shutdown


interface loopback1
  ip address 10.1.1.0/32
  ip router ospf UNDERLAY area 0.0.0.30
```

- #### [spine-2](config/spine-2.conf)

```
feature ospf
feature bfd

router ospf UNDERLAY
  bfd
  router-id 10.1.2.0
  passive-interface default

interface Ethernet1/1
  description to-leaf-1
  no switchport
  bfd interval 100 min_rx 100 multiplier 3
  bfd authentication Keyed-SHA1 key-id 1 hex-key 0ABCD123
  no ip redirects
  ip address 10.6.2.0/31
  ip ospf authentication message-digest
  ip ospf message-digest-key 1 md5 3 bcdd5b8a8b5fb88e
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf UNDERLAY area 0.0.0.30
  ip ospf bfd
  no shutdown

interface Ethernet1/2
  description to-leaf-2
  no switchport
  bfd interval 100 min_rx 100 multiplier 3
  bfd authentication Keyed-SHA1 key-id 1 hex-key 0ABCD123
  no ip redirects
  ip address 10.6.2.2/31
  ip ospf authentication message-digest
  ip ospf message-digest-key 1 md5 3 bcdd5b8a8b5fb88e
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf UNDERLAY area 0.0.0.30
  ip ospf bfd
  no shutdown

interface Ethernet1/3
  description to-leaf-3
  no switchport
  bfd interval 100 min_rx 100 multiplier 3
  bfd authentication Keyed-SHA1 key-id 1 hex-key 0ABCD123
  no ip redirects
  ip address 10.6.2.4/31
  ip ospf authentication message-digest
  ip ospf message-digest-key 1 md5 3 bcdd5b8a8b5fb88e
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf UNDERLAY area 0.0.0.30
  ip ospf bfd
  no shutdown


interface loopback1
  ip address 10.1.2.0/32
  ip router ospf UNDERLAY area 0.0.0.30
```
---
### Проверка связанности устройств по протоколу OSPF

- #### spine-1

```
Spine-1# sh ip ospf neighbors
 OSPF Process ID UNDERLAY VRF default
 Total number of neighbors: 3
 Neighbor ID     Pri State            Up Time  Address         Interface
 10.1.0.1          1 FULL/ -          00:32:46 10.6.1.1        Eth1/1
 10.1.0.2          1 FULL/ -          00:31:06 10.6.1.3        Eth1/2
 10.1.0.3          1 FULL/ -          00:30:30 10.6.1.5        Eth1/3
```

- #### spine-2

```
Spine-2# sh ip ospf neighbors
 OSPF Process ID UNDERLAY VRF default
 Total number of neighbors: 3
 Neighbor ID     Pri State            Up Time  Address         Interface
 10.1.0.1          1 FULL/ -          00:32:55 10.6.2.1        Eth1/1
 10.1.0.2          1 FULL/ -          00:31:19 10.6.2.3        Eth1/2
 10.1.0.3          1 FULL/ -          00:30:43 10.6.2.5        Eth1/3
```
