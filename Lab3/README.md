## Домашнее задание №3
### Схема сети и IP адресация
![lab1.jpg](lab1.jpg)

### Тестовый стенд 
![eve_lab2.jpg](eve_lab2.jpg)

#### Net выбирался на основе Loopback интерфейсов
```
AFI - 49
Area ID - 0001
System ID - 1001.0000 (Loopback0 - 10.0.1.0)
Selector - 00
```
#### Конфигурация коммутаторов
<details>
  <summary><b> Spine-1 </b></summary>
  <p> 

  ```
  interface Ethernet1/1
  no switchport
  ip address 10.2.1.0/31
  isis network point-to-point
  ip router isis 1
  ip ospf network point-to-point
  ip router ospf 1 area 0.0.0.0
  no shutdown

interface Ethernet1/2
  no switchport
  ip address 10.2.1.2/31
  isis network point-to-point
  ip router isis 1
  ip ospf network point-to-point
  ip router ospf 1 area 0.0.0.0
  no shutdown

interface Ethernet1/3
  no switchport
  ip address 10.2.1.4/31
  isis network point-to-point
  ip router isis 1
  ip ospf network point-to-point
  ip router ospf 1 area 0.0.0.0
  no shutdown

interface loopback0
  ip address 10.0.1.0/32
  ip router isis 1
  ip router ospf 1 area 0.0.0.0

  router isis 1
  net 49.0001.1001.0000.00
  is-type level-2
  ```
</p>
</details>

<details>
  <summary><b> Spine-2 </b></summary>
  <p> 

```
interface Ethernet1/1
  no switchport
  ip address 10.2.2.0/31
  isis network point-to-point
  ip router isis 1
  ip ospf network point-to-point
  ip router ospf 1 area 0.0.0.0
  no shutdown

interface Ethernet1/2
  no switchport
  ip address 10.2.2.2/31
  isis network point-to-point
  ip router isis 1
  ip ospf network point-to-point
  ip router ospf 1 area 0.0.0.0
  no shutdown

interface Ethernet1/3
  no switchport
  ip address 10.2.2.4/31
  isis network point-to-point
  ip router isis 1
  ip ospf network point-to-point
  ip router ospf 1 area 0.0.0.0
  no shutdown

interface loopback0
  ip address 10.0.2.0/32
  ip router isis 1
  ip router ospf 1 area 0.0.0.0

router isis 1
  net 49.0001.1002.0000.00
  is-type level-2

```
</p>
</details>

<details>
  <summary><b> Leaf-1 </b></summary>
  <p>

```
  interface Ethernet1/1
  no switchport
  ip address 10.2.1.1/31
  isis network point-to-point
  ip router isis 1
  ip ospf network point-to-point
  ip router ospf 1 area 0.0.0.0
  no shutdown

interface Ethernet1/2
  no switchport
  ip address 10.2.2.1/31
  isis network point-to-point
  ip router isis 1
  ip ospf network point-to-point
  ip router ospf 1 area 0.0.0.0
  no shutdown

  interface loopback1
  ip address 10.1.0.1/32
  ip router isis 1
  ip router ospf 1 area 0.0.0.0

  router isis 1
  net 49.0001.1010.1000.00
  is-type level-2
```
</p>
</details>

<details>
  <summary><b> Leaf-2 </b></summary>
  <p>
  
  ```
  interface Ethernet1/1
  no switchport
  ip address 10.2.1.3/31
  isis network point-to-point
  ip router isis 1
  ip ospf network point-to-point
  ip router ospf 1 area 0.0.0.0
  no shutdown

interface Ethernet1/2
  no switchport
  ip address 10.2.2.3/31
  isis network point-to-point
  ip router isis 1
  ip ospf network point-to-point
  ip router ospf 1 area 0.0.0.0
  no shutdown

interface loopback1
  ip address 10.1.0.2/32
  ip router isis 1
  ip router ospf 1 area 0.0.0.0

router isis 1
  net 49.0001.1010.2000.00
  is-type level-2
```
</p>
</details>

<details>
  <summary><b> Leaf-3 </b></summary>
  <p>

```
interface Ethernet1/1
  no switchport
  ip address 10.2.1.5/31
  isis network point-to-point
  ip router isis 1
  ip ospf network point-to-point
  ip router ospf 1 area 0.0.0.0
  no shutdown

interface Ethernet1/2
  no switchport
  ip address 10.2.2.5/31
  isis network point-to-point
  ip router isis 1
  ip ospf network point-to-point
  ip router ospf 1 area 0.0.0.0
  no shutdown

interface loopback1
  ip address 10.1.0.3/32
  ip router isis 1
  ip router ospf 1 area 0.0.0.0

router isis 1
  net 49.0001.1010.3000.00
  is-type level-2
```
</p>
</details>

### Таблица маршрутизации Leaf-1
```
10.0.1.0/32, ubest/mbest: 1/0
    *via 10.2.1.0, Eth1/1, [115/41], 03:02:28, isis-1, L2
10.0.2.0/32, ubest/mbest: 1/0
    *via 10.2.2.0, Eth1/2, [115/41], 03:02:06, isis-1, L2
10.1.0.1/32, ubest/mbest: 2/0, attached
    *via 10.1.0.1, Lo1, [0/0], 07:52:17, local
    *via 10.1.0.1, Lo1, [0/0], 07:52:17, direct
10.1.0.2/32, ubest/mbest: 2/0
    *via 10.2.1.0, Eth1/1, [115/81], 03:02:23, isis-1, L2
    *via 10.2.2.0, Eth1/2, [115/81], 03:02:06, isis-1, L2
10.1.0.3/32, ubest/mbest: 2/0
    *via 10.2.1.0, Eth1/1, [115/81], 02:57:18, isis-1, L2
    *via 10.2.2.0, Eth1/2, [115/81], 02:57:18, isis-1, L2
10.2.1.0/31, ubest/mbest: 1/0, attached
    *via 10.2.1.1, Eth1/1, [0/0], 07:47:23, direct
10.2.1.1/32, ubest/mbest: 1/0, attached
    *via 10.2.1.1, Eth1/1, [0/0], 07:47:23, local
10.2.1.2/31, ubest/mbest: 1/0
    *via 10.2.1.0, Eth1/1, [115/80], 03:02:28, isis-1, L2
10.2.1.4/31, ubest/mbest: 1/0
    *via 10.2.1.0, Eth1/1, [115/80], 03:02:28, isis-1, L2
10.2.2.0/31, ubest/mbest: 1/0, attached
    *via 10.2.2.1, Eth1/2, [0/0], 07:47:12, direct
10.2.2.1/32, ubest/mbest: 1/0, attached
    *via 10.2.2.1, Eth1/2, [0/0], 07:47:12, local
10.2.2.2/31, ubest/mbest: 1/0
    *via 10.2.2.0, Eth1/2, [115/80], 03:02:06, isis-1, L2
10.2.2.4/31, ubest/mbest: 1/0
    *via 10.2.2.0, Eth1/2, [115/80], 03:02:06, isis-1, L2
```
### Проверка доступности остальных коммутаторов с Leaf-1
##### Leaf-1 -> Spine-1
```
Leaf-1# ping 10.0.1.0 source-interface loopback 1
PING 10.0.1.0 (10.0.1.0): 56 data bytes
64 bytes from 10.0.1.0: icmp_seq=0 ttl=254 time=4.125 ms
64 bytes from 10.0.1.0: icmp_seq=1 ttl=254 time=2.188 ms
64 bytes from 10.0.1.0: icmp_seq=2 ttl=254 time=1.975 ms
64 bytes from 10.0.1.0: icmp_seq=3 ttl=254 time=1.916 ms
64 bytes from 10.0.1.0: icmp_seq=4 ttl=254 time=1.205 ms
```
##### Leaf-1 -> Spine-2
```
Leaf-1# ping 10.0.2.0 source-interface loopback 1
PING 10.0.2.0 (10.0.2.0): 56 data bytes
64 bytes from 10.0.2.0: icmp_seq=0 ttl=254 time=3.393 ms
64 bytes from 10.0.2.0: icmp_seq=1 ttl=254 time=3.138 ms
64 bytes from 10.0.2.0: icmp_seq=2 ttl=254 time=1.854 ms
64 bytes from 10.0.2.0: icmp_seq=3 ttl=254 time=2.538 ms
64 bytes from 10.0.2.0: icmp_seq=4 ttl=254 time=1.715 ms
```
##### Leaf-1 -> Leaf-2
```
Leaf-1# ping 10.1.0.2 source-interface loopback 1
PING 10.1.0.2 (10.1.0.2): 56 data bytes
64 bytes from 10.1.0.2: icmp_seq=0 ttl=253 time=3.54 ms
64 bytes from 10.1.0.2: icmp_seq=1 ttl=253 time=1.611 ms
64 bytes from 10.1.0.2: icmp_seq=2 ttl=253 time=1.867 ms
64 bytes from 10.1.0.2: icmp_seq=3 ttl=253 time=1.711 ms
64 bytes from 10.1.0.2: icmp_seq=4 ttl=253 time=1.783 ms
```
##### Leaf-1 -> Leaf-3
```
Leaf-1# ping 10.1.0.3 source-interface loopback 1
PING 10.1.0.3 (10.1.0.3): 56 data bytes
64 bytes from 10.1.0.3: icmp_seq=0 ttl=253 time=5.265 ms
64 bytes from 10.1.0.3: icmp_seq=1 ttl=253 time=2.736 ms
64 bytes from 10.1.0.3: icmp_seq=2 ttl=253 time=2.619 ms
64 bytes from 10.1.0.3: icmp_seq=3 ttl=253 time=4.291 ms
64 bytes from 10.1.0.3: icmp_seq=4 ttl=253 time=2.442 ms
```
