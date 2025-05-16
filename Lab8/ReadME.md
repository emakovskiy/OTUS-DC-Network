## Домашнее задание №8
### Тестовый стенд
![topology.jpg](topology.jpg)

#### Underlay и Overlay построен на iBGP AS 65000, для роутинга между vrf, к Leaf-4 подключен маршрутизатор R1 c AS65001
#### Хосты PC-1, PC-2, PC-3, PC-4 расположены в vrf SERVICE
#### Хосты PC-5, PC-6 расположены в vrf SERVICE2
#### Конфигурация коммутаторов
<details>
  <summary><b> Spine-1 </b></summary>
  <p> 

  ```
nv overlay evpn
    feature ospf
    feature bgp
    feature isis
    feature bfd
    
    route-map Leaf_AS permit 10
      match as-number 65201-65299 
    vrf context management
    
    
    interface Ethernet1/1
      no switchport
      no ip redirects
      ip address 10.2.1.0/31
      no ipv6 redirects
      no shutdown
    
    interface Ethernet1/2
      no switchport
      no ip redirects
      ip address 10.2.1.2/31
      no ipv6 redirects
      no shutdown
    
    interface Ethernet1/3
      no switchport
      no ip redirects
      ip address 10.2.1.4/31
      no ipv6 redirects
      no shutdown
    
    interface Ethernet1/4
      no switchport
      no ip redirects
      ip address 10.2.1.6/31
      no ipv6 redirects
      no shutdown
  
    interface loopback0
    ip address 10.0.1.0/32
    icam monitor scale
  
    cli alias name wr copy running-config startup-config
    line console
    line vty
    boot nxos bootflash:/nxos64-cs.10.3.5.M.bin 
    router bgp 65000
      router-id 10.0.1.0
      address-family ipv4 unicast
        network 10.0.1.0/32
        maximum-paths 10
      address-family l2vpn evpn
        maximum-paths 10
        maximum-paths ibgp 64
      template peer LEAF
        remote-as 65000
        update-source loopback0
        timers 3 9
        address-family l2vpn evpn
          send-community
          send-community extended
          route-reflector-client
      template peer-policy DC
        send-community
        send-community extended
        route-reflector-client
        next-hop-self all
      template peer-session DC
        remote-as 65000
        timers 3 9
      neighbor 10.1.0.1
        inherit peer LEAF
      neighbor 10.1.0.2
        inherit peer LEAF
      neighbor 10.1.0.3
        inherit peer LEAF
      neighbor 10.1.0.4
        inherit peer LEAF
      neighbor 10.2.0.0/16
        inherit peer DC
        inherit peer-session DC
        address-family ipv4 unicast
          route-reflector-client
          next-hop-self all
```
</p>
  </details>

<details>
  <summary><b> Spine-2 </b></summary>
  <p> 

```

nv overlay evpn
feature ospf
feature bgp
feature isis

interface Ethernet1/1
  no switchport
  no ip redirects
  ip address 10.2.2.0/31
  no ipv6 redirects
  no shutdown

interface Ethernet1/2
  no switchport
  no ip redirects
  ip address 10.2.2.2/31
  no ipv6 redirects
  no shutdown

interface Ethernet1/3
  no switchport
  no ip redirects
  ip address 10.2.2.4/31
  no ipv6 redirects
  no shutdown

interface Ethernet1/4
  no switchport
  no ip redirects
  ip address 10.2.2.6/31
  no ipv6 redirects
  no shutdown

interface loopback0
  ip address 10.0.2.0/32

router bgp 65000
  router-id 10.0.2.0
  address-family ipv4 unicast
    network 10.0.2.0/32
    maximum-paths 10
  address-family l2vpn evpn
    maximum-paths 10
    maximum-paths ibgp 64
  template peer LEAF
    remote-as 65000
    update-source loopback0
    timers 3 9
    address-family l2vpn evpn
      send-community
      send-community extended
      route-reflector-client
  template peer-policy DC
    send-community
    send-community extended
    route-reflector-client
    next-hop-self all
  template peer-session DC
    remote-as 65000
    timers 3 9
  neighbor 10.1.0.1
    inherit peer LEAF
  neighbor 10.1.0.2
    inherit peer LEAF
  neighbor 10.1.0.3
    inherit peer LEAF
  neighbor 10.1.0.4
    inherit peer LEAF
  neighbor 10.2.0.0/16
    inherit peer DC
    inherit peer-session DC
    address-family ipv4 unicast
      route-reflector-client
      next-hop-self all
```
</p>
  </details>

<details>
  <summary><b> Leaf-1 </b></summary>
  <p>

```
nv overlay evpn
feature ospf
feature bgp
feature pim
feature isis
feature fabric forwarding
feature interface-vlan
feature vn-segment-vlan-based
feature lacp
feature vpc
feature bfd
feature nv overlay

fabric forwarding anycast-gateway-mac 0001.0002.0003
vlan 1,10,20,777,999
vlan 10
  name SERVICE10
  vn-segment 101010
vlan 20
  name SERVICE2
  vn-segment 10020
vlan 777
  name anycast_gw
  vn-segment 77777
vlan 999
  name native

route-map RD permit 10
  match interface loopback1 loopback2 
vrf context SERVICE
  vni 77777
  rd auto
  address-family ipv4 unicast
    route-target both auto
    route-target both auto evpn
vrf context management
vpc domain 1
  peer-switch
  role priority 1
  system-mac 00:11:00:22:00:33
  system-priority 100
  peer-keepalive destination 192.168.254.2 source 192.168.254.1
  delay restore 60
  peer-gateway
  layer3 peer-router
  auto-recovery
  fast-convergence
  ip arp synchronize


interface Vlan1
  no ip redirects
  no ipv6 redirects

interface Vlan10
  no shutdown
  vrf member SERVICE
  ip address 192.168.10.1/24
  no ipv6 redirects
  fabric forwarding mode anycast-gateway

interface Vlan20
  no shutdown
  vrf member SERVICE
  no ip redirects
  ip address 192.168.20.1/24
  no ipv6 redirects
  fabric forwarding mode anycast-gateway

interface Vlan777
  no shutdown
  vrf member SERVICE
  no ip redirects
  ip forward
  no ipv6 redirects

interface port-channel1
  description PC1
  switchport mode trunk
  switchport access vlan 10
  spanning-tree port type edge
  vpc 1

interface port-channel2
  description PC2
  switchport mode trunk
  switchport access vlan 20
  spanning-tree port type edge

interface port-channel100
  description vPC_PeerLink
  switchport mode trunk
  switchport trunk native vlan 999
  spanning-tree port type network
  vpc peer-link

interface nve1
  no shutdown
  host-reachability protocol bgp
  advertise virtual-rmac
  source-interface loopback1
  global ingress-replication protocol bgp
  member vni 10010
    ingress-replication protocol bgp
  member vni 10020
  member vni 77777 associate-vrf
  member vni 101010

interface Ethernet1/1
  no switchport
  no ip redirects
  ip address 10.2.1.1/31
  no ipv6 redirects
  no shutdown

interface Ethernet1/2
  no switchport
  no ip redirects
  ip address 10.2.2.1/31
  no ipv6 redirects
  no shutdown

interface Ethernet1/3
  description PC1
  switchport mode trunk
  switchport access vlan 10
  spanning-tree port type edge
  channel-group 1 mode active

interface Ethernet1/4
  description PC2
  switchport mode trunk
  switchport access vlan 20
  spanning-tree port type edge
  channel-group 2 mode active

interface Ethernet1/5
  description vPC_PeerLink
  switchport mode trunk
  switchport trunk native vlan 999
  channel-group 100 mode active

interface Ethernet1/6
  description vPC_PeerLink
  switchport mode trunk
  switchport trunk native vlan 999
  channel-group 100 mode active

interface mgmt0
  vrf member management
  ip address 192.168.254.1/24

interface loopback1
  ip address 10.1.0.1/32
  ip address 10.1.0.111/32 secondary

interface loopback2
  ip address 10.1.1.1/32

router bgp 65000
  router-id 10.1.0.1
  address-family ipv4 unicast
    redistribute direct route-map RD
    maximum-paths 10
    maximum-paths ibgp 64
  address-family l2vpn evpn
    maximum-paths 10
    maximum-paths ibgp 64
  template peer SPINE
    remote-as 65000
    update-source loopback1
    timers 3 9
    address-family l2vpn evpn
      send-community
      send-community extended
  template peer-policy DC
    send-community
    send-community extended
  template peer-session DC
    remote-as 65000
    timers 3 9
  neighbor 10.0.1.0
    inherit peer SPINE
  neighbor 10.0.2.0
    inherit peer SPINE
  neighbor 10.2.1.0
    inherit peer DC
    inherit peer-session DC
    address-family ipv4 unicast
      no next-hop-self
  neighbor 10.2.2.0
    inherit peer DC
    inherit peer-session DC
    address-family ipv4 unicast
      no next-hop-self
evpn
  vni 10010 l2
    rd auto
    route-target import auto
    route-target export auto

```
</p>
  </details>

<details>
  <summary><b> Leaf-2 </b></summary>
  <p>

```
nv overlay evpn
feature ospf
feature bgp
feature pim
feature isis
feature fabric forwarding
feature interface-vlan
feature vn-segment-vlan-based
feature lacp
feature vpc
feature nv overlay

fabric forwarding anycast-gateway-mac 0001.0002.0003
vlan 1,10,20,777,999
vlan 10
  name SERVICE10
  vn-segment 101010
vlan 20
  name SERVICE2
  vn-segment 10020
vlan 777
  name anycast_gw
  vn-segment 77777
vlan 999
  name native

spanning-tree vlan 1-3967 priority 28672
route-map RD permit 10
  match interface loopback1 loopback2 
vrf context SERVICE
  vni 77777
  rd auto
  address-family ipv4 unicast
    route-target both auto
    route-target both auto evpn
vrf context management
vpc domain 1
  peer-switch
  role priority 2
  system-mac 00:11:00:22:00:33
  system-priority 100
  peer-keepalive destination 192.168.254.1 source 192.168.254.2
  delay restore 60
  peer-gateway
  layer3 peer-router
  auto-recovery
  fast-convergence
  ip arp synchronize


interface Vlan1
  no ip redirects
  no ipv6 redirects

interface Vlan10
  no shutdown
  vrf member SERVICE
  no ip redirects
  ip address 192.168.10.1/24
  no ipv6 redirects
  fabric forwarding mode anycast-gateway

interface Vlan20
  no shutdown
  vrf member SERVICE
  no ip redirects
  ip address 192.168.20.1/24
  no ipv6 redirects
  fabric forwarding mode anycast-gateway

interface Vlan777
  no shutdown
  vrf member SERVICE
  no ip redirects
  ip forward
  no ipv6 redirects

interface port-channel1
  description PC1
  switchport mode trunk
  switchport access vlan 10
  spanning-tree port type edge
  vpc 1

interface port-channel2
  description PC2
  switchport mode trunk
  switchport access vlan 20
  spanning-tree port type edge

interface port-channel100
  description vPC_PeerLink
  switchport mode trunk
  switchport trunk native vlan 999
  spanning-tree port type network
  vpc peer-link

interface nve1
  no shutdown
  host-reachability protocol bgp
  advertise virtual-rmac
  source-interface loopback1
  global ingress-replication protocol bgp
  member vni 10010
  member vni 10020
  member vni 77777 associate-vrf
  member vni 101010

interface Ethernet1/1
  no switchport
  no ip redirects
  ip address 10.2.1.3/31
  no ipv6 redirects
  no shutdown

interface Ethernet1/2
  no switchport
  no ip redirects
  ip address 10.2.2.3/31
  no ipv6 redirects
  no shutdown

interface Ethernet1/3
  description PC1
  switchport mode trunk
  switchport access vlan 10
  spanning-tree port type edge
  channel-group 1 mode active

interface Ethernet1/4
  description PC2
  switchport mode trunk
  switchport access vlan 20
  spanning-tree port type edge
  channel-group 2 mode active

interface Ethernet1/5
  description vPC_PeerLink
  switchport mode trunk
  switchport trunk native vlan 999
  channel-group 100 mode active

interface Ethernet1/6
  description vPC_PeerLink
  switchport mode trunk
  switchport trunk native vlan 999
  channel-group 100 mode active

interface mgmt0
  vrf member management
  ip address 192.168.254.2/24

interface loopback1
  ip address 10.1.0.2/32
  ip address 10.1.0.111/32 secondary

interface loopback2
  ip address 10.1.1.2/32

router bgp 65000
  router-id 10.1.0.2
  address-family ipv4 unicast
    redistribute direct route-map RD
    maximum-paths 10
    maximum-paths ibgp 64
  address-family l2vpn evpn
    maximum-paths 10
    maximum-paths ibgp 64
  template peer SPINE
    remote-as 65000
    update-source loopback1
    timers 3 9
    address-family l2vpn evpn
      send-community
      send-community extended
  template peer-policy DC
    send-community
    send-community extended
  template peer-session DC
    remote-as 65000
    timers 3 9
  neighbor 10.0.1.0
    inherit peer SPINE
  neighbor 10.0.2.0
    inherit peer SPINE
  neighbor 10.2.1.2
    inherit peer DC
    inherit peer-session DC
    address-family ipv4 unicast
      no next-hop-self
  neighbor 10.2.2.2
    inherit peer DC
    inherit peer-session DC
    address-family ipv4 unicast
      no next-hop-self
evpn
  vni 10010 l2
    rd auto
    route-target import auto
    route-target export auto

```
</p>
  </details>

<details>
  <summary><b> Leaf-3 </b></summary>
  <p>
    
```
nv overlay evpn
feature ospf
feature bgp
feature isis
feature fabric forwarding
feature interface-vlan
feature vn-segment-vlan-based
feature nv overlay

fabric forwarding anycast-gateway-mac 0001.0002.0003
vlan 1,30,50,777,888
vlan 30
  name Service_3
  vn-segment 10030
vlan 50
  vn-segment 10050
vlan 777
  name anycast_gw
  vn-segment 77777
vlan 888
  vn-segment 88888

route-map REDISTRIBUTE_CONN permit 10
  match interface loopback1 loopback2 
vrf context SERVICE
  vni 77777
  rd auto
  address-family ipv4 unicast
    route-target both auto
    route-target both auto evpn
vrf context SERVICE2
  vni 88888
  rd auto
  address-family ipv4 unicast
    route-target both auto
    route-target both auto evpn
vrf context management


interface Vlan1

interface Vlan30
  no shutdown
  vrf member SERVICE
  ip address 192.168.30.1/24
  fabric forwarding mode anycast-gateway

interface Vlan50
  no shutdown
  vrf member SERVICE2
  ip address 192.168.50.1/24
  fabric forwarding mode anycast-gateway

interface Vlan777
  no shutdown
  vrf member SERVICE
  ip forward

interface Vlan888
  no shutdown
  vrf member SERVICE2
  ip forward

interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback1
  global suppress-arp
  global ingress-replication protocol bgp
  member vni 10010
  member vni 10030
  member vni 10050
  member vni 77777 associate-vrf
  member vni 88888 associate-vrf

interface Ethernet1/1
  no switchport
  no ip redirects
  ip address 10.2.1.5/31
  no ipv6 redirects
  no shutdown

interface Ethernet1/2
  no switchport
  no ip redirects
  ip address 10.2.2.5/31
  no ipv6 redirects
  no shutdown

interface Ethernet1/3
  switchport access vlan 30

interface Ethernet1/4
  switchport access vlan 50

interface loopback1
  ip address 10.1.0.3/32

interface loopback2
  ip address 10.1.1.3/32

router bgp 65000
  router-id 10.1.0.3
  address-family ipv4 unicast
    redistribute direct route-map REDISTRIBUTE_CONN
    maximum-paths 10
    maximum-paths ibgp 64
  address-family l2vpn evpn
    maximum-paths 10
    maximum-paths ibgp 64
  template peer SPINE
    remote-as 65000
    update-source loopback1
    timers 3 9
    address-family l2vpn evpn
      send-community
      send-community extended
  template peer-policy DC
    send-community
    send-community extended
  template peer-session DC
    remote-as 65000
    timers 3 9
  neighbor 10.0.1.0
    inherit peer SPINE
  neighbor 10.0.2.0
    inherit peer SPINE
  neighbor 10.2.1.4
    inherit peer DC
    inherit peer-session DC
    address-family ipv4 unicast
  neighbor 10.2.2.4
    inherit peer DC
    inherit peer-session DC
    address-family ipv4 unicast
evpn
  vni 10010 l2
    rd auto
    route-target import auto
    route-target export auto
```
</p>
  </details>

<details>
  <summary><b> Leaf-4 </b></summary>
  <p>

```
nv overlay evpn
feature ospf
feature bgp
feature isis
feature fabric forwarding
feature interface-vlan
feature vn-segment-vlan-based
feature nv overlay

fabric forwarding anycast-gateway-mac 0001.0002.0003
vlan 1,40,60,111-112,777,888
vlan 40
  name SERVICE40
  vn-segment 10040
vlan 60
  vn-segment 10060
vlan 111
  name recon_SERVICE
vlan 112
  name recon_SERVICE2
vlan 777
  name anycast_gw
  vn-segment 77777
vlan 888
  vn-segment 88888

route-map PERMIT permit 10
route-map REDISTRIBUTE_CONN permit 10
  match interface loopback1 loopback2 
vrf context SERVICE
  vni 77777
  rd auto
  address-family ipv4 unicast
    route-target both auto
    route-target both auto evpn
vrf context SERVICE2
  vni 88888
  rd auto
  address-family ipv4 unicast
    route-target both auto
    route-target both auto evpn

interface Vlan40
  no shutdown
  vrf member SERVICE
  ip address 192.168.40.1/24
  fabric forwarding mode anycast-gateway

interface Vlan60
  no shutdown
  vrf member SERVICE2
  ip address 192.168.60.1/24
  fabric forwarding mode anycast-gateway

interface Vlan111
  description recon_Service
  no shutdown
  vrf member SERVICE
  ip address 10.4.0.1/30

interface Vlan112
  no shutdown
  vrf member SERVICE2
  ip address 10.4.0.5/30

interface Vlan777
  no shutdown
  vrf member SERVICE
  ip forward

interface Vlan888
  no shutdown
  vrf member SERVICE2
  ip forward

interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback1
  global ingress-replication protocol bgp
  member vni 10010
    ingress-replication protocol bgp
  member vni 10040
  member vni 10060
  member vni 77777 associate-vrf
  member vni 88888 associate-vrf

interface Ethernet1/1
  no switchport
  no ip redirects
  ip address 10.2.1.7/31
  no ipv6 redirects
  no shutdown

interface Ethernet1/2
  no switchport
  no ip redirects
  ip address 10.2.2.7/31
  no ipv6 redirects
  no shutdown

interface Ethernet1/3
  switchport access vlan 40

interface Ethernet1/4
  switchport access vlan 60

interface Ethernet1/5

interface Ethernet1/6
  switchport access vlan 111

interface Ethernet1/7
  switchport access vlan 112

interface loopback1
  ip address 10.1.0.4/32

interface loopback2
  ip address 10.1.1.4/32

router bgp 65000
  router-id 10.1.0.4
  address-family ipv4 unicast
    redistribute direct route-map REDISTRIBUTE_CONN
    maximum-paths 10
    maximum-paths ibgp 64
  address-family l2vpn evpn
    maximum-paths 10
    maximum-paths ibgp 64
  template peer SPINE
    remote-as 65000
    update-source loopback1
    timers 3 9
    address-family l2vpn evpn
      send-community
      send-community extended
  template peer-policy DC
    send-community
    send-community extended
  template peer-session DC
    remote-as 65000
    timers 3 9
  neighbor 10.0.1.0
    inherit peer SPINE
  neighbor 10.0.2.0
    inherit peer SPINE
  neighbor 10.2.1.6
    inherit peer DC
    inherit peer-session DC
    address-family ipv4 unicast
  neighbor 10.2.2.6
    inherit peer DC
    inherit peer-session DC
    address-family ipv4 unicast
  vrf SERVICE
    address-family ipv4 unicast
      redistribute hmm route-map PERMIT
      redistribute direct route-map PERMIT
    neighbor 10.4.0.2
      remote-as 65001
      address-family ipv4 unicast
  vrf SERVICE2
    address-family ipv4 unicast
      redistribute hmm route-map PERMIT
      redistribute direct route-map PERMIT
    neighbor 10.4.0.6
      remote-as 65001
      address-family ipv4 unicast
evpn
  vni 10010 l2
    rd auto
    route-target import auto
    route-target export auto
```
</p>
  </details>

<details>
  <summary><b> R1 </b></summary>
  <p>
    
```
interface GigabitEthernet0/0
 ip address 10.4.0.2 255.255.255.252
 duplex auto
 speed auto
 media-type rj45
!
interface GigabitEthernet0/1
 ip address 10.4.0.6 255.255.255.252
 duplex auto
 speed auto
 media-type rj45

router bgp 65001
 bgp log-neighbor-changes
 neighbor 10.4.0.1 remote-as 65000
 neighbor 10.4.0.1 as-override
 neighbor 10.4.0.5 remote-as 65000
 neighbor 10.4.0.5 as-override

```
</p>
  </details>
  
#### Таблицы маршрутизации
<details>
  <summary><b> R1 </b></summary>
  <p>
    
```
R1#sh ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override, p - overrides from PfR

Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 4 subnets, 2 masks
C        10.4.0.0/30 is directly connected, GigabitEthernet0/0
L        10.4.0.2/32 is directly connected, GigabitEthernet0/0
C        10.4.0.4/30 is directly connected, GigabitEthernet0/1
L        10.4.0.6/32 is directly connected, GigabitEthernet0/1
      192.168.10.0/32 is subnetted, 1 subnets
B        192.168.10.10 [20/0] via 10.4.0.1, 00:26:52
      192.168.20.0/32 is subnetted, 1 subnets
B        192.168.20.10 [20/0] via 10.4.0.1, 00:26:52
      192.168.30.0/32 is subnetted, 1 subnets
B        192.168.30.10 [20/0] via 10.4.0.1, 00:26:52
      192.168.40.0/24 is variably subnetted, 2 subnets, 2 masks
B        192.168.40.0/24 [20/0] via 10.4.0.1, 00:26:52
B        192.168.40.10/32 [20/0] via 10.4.0.1, 00:25:45
      192.168.50.0/32 is subnetted, 1 subnets
B        192.168.50.10 [20/0] via 10.4.0.5, 00:26:52
      192.168.60.0/24 is variably subnetted, 2 subnets, 2 masks
B        192.168.60.0/24 [20/0] via 10.4.0.5, 00:26:52
B        192.168.60.10/32 [20/0] via 10.4.0.5, 00:25:50

```
</p>
  </details>

<details>
  <summary><b> Leaf-1 </b></summary>
  <p>
    
```

Leaf-1# sh ip route
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

10.0.1.0/32, ubest/mbest: 1/0
    *via 10.2.1.0, [200/0], 03:49:50, bgp-65000, internal, tag 65000
10.0.2.0/32, ubest/mbest: 1/0
    *via 10.2.2.0, [200/0], 03:49:54, bgp-65000, internal, tag 65000
10.1.0.1/32, ubest/mbest: 2/0, attached
    *via 10.1.0.1, Lo1, [0/0], 03:46:45, local
    *via 10.1.0.1, Lo1, [0/0], 03:46:45, direct
10.1.0.2/32, ubest/mbest: 2/0
    *via 10.2.1.0, [200/0], 03:46:45, bgp-65000, internal, tag 65000
    *via 10.2.2.0, [200/0], 03:46:45, bgp-65000, internal, tag 65000
10.1.0.3/32, ubest/mbest: 2/0
    *via 10.2.1.0, [200/0], 02:32:10, bgp-65000, internal, tag 65000
    *via 10.2.2.0, [200/0], 02:32:10, bgp-65000, internal, tag 65000
10.1.0.4/32, ubest/mbest: 2/0
    *via 10.2.1.0, [200/0], 01:16:05, bgp-65000, internal, tag 65000
    *via 10.2.2.0, [200/0], 01:16:05, bgp-65000, internal, tag 65000
10.1.0.111/32, ubest/mbest: 2/0, attached
    *via 10.1.0.111, Lo1, [0/0], 03:46:45, local
    *via 10.1.0.111, Lo1, [0/0], 03:46:45, direct
10.1.1.1/32, ubest/mbest: 2/0, attached
    *via 10.1.1.1, Lo2, [0/0], 03:51:53, local
    *via 10.1.1.1, Lo2, [0/0], 03:51:53, direct
10.1.1.2/32, ubest/mbest: 2/0
    *via 10.2.1.0, [200/0], 03:49:50, bgp-65000, internal, tag 65000
    *via 10.2.2.0, [200/0], 03:49:54, bgp-65000, internal, tag 65000
10.1.1.3/32, ubest/mbest: 2/0
    *via 10.2.1.0, [200/0], 02:32:05, bgp-65000, internal, tag 65000
    *via 10.2.2.0, [200/0], 02:32:05, bgp-65000, internal, tag 65000
10.1.1.4/32, ubest/mbest: 2/0
    *via 10.2.1.0, [200/0], 01:16:05, bgp-65000, internal, tag 65000
    *via 10.2.2.0, [200/0], 01:16:05, bgp-65000, internal, tag 65000
10.2.1.0/31, ubest/mbest: 1/0, attached
    *via 10.2.1.1, Eth1/1, [0/0], 03:50:20, direct
10.2.1.1/32, ubest/mbest: 1/0, attached
    *via 10.2.1.1, Eth1/1, [0/0], 03:50:20, local
10.2.2.0/31, ubest/mbest: 1/0, attached
    *via 10.2.2.1, Eth1/2, [0/0], 03:50:20, direct
10.2.2.1/32, ubest/mbest: 1/0, attached
    *via 10.2.2.1, Eth1/2, [0/0], 03:50:20, local

Leaf-1# sh ip route vrf SERVICE
IP Route Table for VRF "SERVICE"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

10.4.0.0/30, ubest/mbest: 1/0
    *via 10.1.0.4%default, [200/0], 01:13:38, bgp-65000, internal, tag 65000, se
gid: 77777 tunnelid: 0xa010004 encap: VXLAN
 
10.4.0.4/30, ubest/mbest: 1/0
    *via 10.1.0.4%default, [200/0], 00:28:21, bgp-65000, internal, tag 65001, se
gid: 77777 tunnelid: 0xa010004 encap: VXLAN
 
192.168.10.0/24, ubest/mbest: 1/0, attached
    *via 192.168.10.1, Vlan10, [0/0], 03:50:20, direct
192.168.10.1/32, ubest/mbest: 1/0, attached
    *via 192.168.10.1, Vlan10, [0/0], 03:50:20, local
192.168.10.10/32, ubest/mbest: 1/0, attached
    *via 192.168.10.10, Vlan10, [190/0], 00:50:18, hmm
192.168.20.0/24, ubest/mbest: 1/0, attached
    *via 192.168.20.1, Vlan20, [0/0], 03:50:20, direct
192.168.20.1/32, ubest/mbest: 1/0, attached
    *via 192.168.20.1, Vlan20, [0/0], 03:50:20, local
192.168.20.10/32, ubest/mbest: 1/0, attached
    *via 192.168.20.10, Vlan20, [190/0], 00:20:18, hmm
192.168.30.10/32, ubest/mbest: 1/0
    *via 10.1.0.3%default, [200/0], 02:32:58, bgp-65000, internal, tag 65000, se
gid: 77777 tunnelid: 0xa010003 encap: VXLAN
 
192.168.40.0/24, ubest/mbest: 1/0
    *via 10.1.0.4%default, [200/0], 01:13:38, bgp-65000, internal, tag 65000, se
gid: 77777 tunnelid: 0xa010004 encap: VXLAN
 
192.168.40.10/32, ubest/mbest: 1/0
    *via 10.1.0.4%default, [200/0], 00:27:14, bgp-65000, internal, tag 65000, se
gid: 77777 tunnelid: 0xa010004 encap: VXLAN
 
192.168.50.10/32, ubest/mbest: 1/0
    *via 10.1.0.4%default, [200/0], 00:28:21, bgp-65000, internal, tag 65001, se
gid: 77777 tunnelid: 0xa010004 encap: VXLAN
 
192.168.60.0/24, ubest/mbest: 1/0
    *via 10.1.0.4%default, [200/0], 00:28:21, bgp-65000, internal, tag 65001, se
gid: 77777 tunnelid: 0xa010004 encap: VXLAN
 
192.168.60.10/32, ubest/mbest: 1/0
    *via 10.1.0.4%default, [200/0], 00:27:19, bgp-65000, internal, tag 65001, se
gid: 77777 tunnelid: 0xa010004 encap: VXLAN

#### L2 таблица маршрутизации
Leaf-1# sh bgp l2vpn evpn 
BGP routing table information for VRF default, address family L2VPN EVPN
BGP table version is 76, Local Router ID is 10.1.0.1
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-i
njected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - b
est2

   Network            Next Hop            Metric     LocPrf     Weight Path
Route Distinguisher: 10.1.0.1:32777    (L2VNI 101010)
*>l[2]:[0]:[0]:[48]:[0050.0000.0700]:[0]:[0.0.0.0]/216
                      10.1.0.111                        100      32768 i
*>l[2]:[0]:[0]:[48]:[0050.0000.0700]:[32]:[192.168.10.10]/272
                      10.1.0.111                        100      32768 i
*>l[3]:[0]:[32]:[10.1.0.111]/88
                      10.1.0.111                        100      32768 i

Route Distinguisher: 10.1.0.1:32787    (L2VNI 10020)
*>l[2]:[0]:[0]:[48]:[0050.0000.0a00]:[0]:[0.0.0.0]/216
                      10.1.0.111                        100      32768 i
*>l[2]:[0]:[0]:[48]:[0050.0000.0a00]:[32]:[192.168.20.10]/272
                      10.1.0.111                        100      32768 i
*>l[3]:[0]:[32]:[10.1.0.111]/88
                      10.1.0.111                        100      32768 i

Route Distinguisher: 10.1.0.3:32797
* i[2]:[0]:[0]:[48]:[0050.0000.0800]:[32]:[192.168.30.10]/272
                      10.1.0.3                          100          0 i
*>i                   10.1.0.3                          100          0 i

Route Distinguisher: 10.1.0.4:4
* i[5]:[0]:[0]:[24]:[192.168.40.0]/224
                      10.1.0.4                 0        100          0 ?
*>i                   10.1.0.4                 0        100          0 ?
*>i[5]:[0]:[0]:[24]:[192.168.60.0]/224
                      10.1.0.4                          100          0 65001 650
01 ?
* i                   10.1.0.4                          100          0 65001 650
01 ?
* i[5]:[0]:[0]:[30]:[10.4.0.0]/224
                      10.1.0.4                 0        100          0 ?
*>i                   10.1.0.4                 0        100          0 ?
*>i[5]:[0]:[0]:[30]:[10.4.0.4]/224
                      10.1.0.4                          100          0 65001 650
01 ?
* i                   10.1.0.4                          100          0 65001 650
01 ?
* i[5]:[0]:[0]:[32]:[192.168.40.10]/224
                      10.1.0.4                 0        100          0 ?
*>i                   10.1.0.4                 0        100          0 ?
*>i[5]:[0]:[0]:[32]:[192.168.50.10]/224
                      10.1.0.4                          100          0 65001 650
01 i
* i                   10.1.0.4                          100          0 65001 650
01 i
* i[5]:[0]:[0]:[32]:[192.168.60.10]/224
                      10.1.0.4                          100          0 65001 650
01 ?
*>i                   10.1.0.4                          100          0 65001 650
01 ?

Route Distinguisher: 10.1.0.4:32807
* i[2]:[0]:[0]:[48]:[0050.0000.0b00]:[32]:[192.168.40.10]/272
                      10.1.0.4                          100          0 i
*>i                   10.1.0.4                          100          0 i

Route Distinguisher: 10.1.0.1:4    (L3VNI 77777)
*>i[2]:[0]:[0]:[48]:[0050.0000.0800]:[32]:[192.168.30.10]/272
                      10.1.0.3                          100          0 i
*>i[2]:[0]:[0]:[48]:[0050.0000.0b00]:[32]:[192.168.40.10]/272
                      10.1.0.4                          100          0 i
*>i[5]:[0]:[0]:[24]:[192.168.40.0]/224
                      10.1.0.4                 0        100          0 ?
*>i[5]:[0]:[0]:[24]:[192.168.60.0]/224
                      10.1.0.4                          100          0 65001 650
01 ?
*>i[5]:[0]:[0]:[30]:[10.4.0.0]/224
                      10.1.0.4                 0        100          0 ?
*>i[5]:[0]:[0]:[30]:[10.4.0.4]/224
                      10.1.0.4                          100          0 65001 650
01 ?
*>i[5]:[0]:[0]:[32]:[192.168.40.10]/224
                      10.1.0.4                 0        100          0 ?
*>i[5]:[0]:[0]:[32]:[192.168.50.10]/224
                      10.1.0.4                          100          0 65001 650
01 i
*>i[5]:[0]:[0]:[32]:[192.168.60.10]/224
                      10.1.0.4                          100          0 65001 650
01 ?

```
</p>
  </details>

<details>
  <summary><b> Leaf-2 </b></summary>
  <p>
    
```

Leaf-2# sh ip route
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

10.0.1.0/32, ubest/mbest: 1/0
    *via 10.2.1.2, [200/0], 03:53:37, bgp-65000, internal, tag 65000
10.0.2.0/32, ubest/mbest: 1/0
    *via 10.2.2.2, [200/0], 03:53:34, bgp-65000, internal, tag 65000
10.1.0.1/32, ubest/mbest: 2/0
    *via 10.2.1.2, [200/0], 03:50:25, bgp-65000, internal, tag 65000
    *via 10.2.2.2, [200/0], 03:50:25, bgp-65000, internal, tag 65000
10.1.0.2/32, ubest/mbest: 2/0, attached
    *via 10.1.0.2, Lo1, [0/0], 03:50:25, local
    *via 10.1.0.2, Lo1, [0/0], 03:50:25, direct
10.1.0.3/32, ubest/mbest: 2/0
    *via 10.2.1.2, [200/0], 02:35:49, bgp-65000, internal, tag 65000
    *via 10.2.2.2, [200/0], 02:35:49, bgp-65000, internal, tag 65000
10.1.0.4/32, ubest/mbest: 2/0
    *via 10.2.1.2, [200/0], 01:19:45, bgp-65000, internal, tag 65000
    *via 10.2.2.2, [200/0], 01:19:45, bgp-65000, internal, tag 65000
10.1.0.111/32, ubest/mbest: 2/0, attached
    *via 10.1.0.111, Lo1, [0/0], 03:50:25, local
    *via 10.1.0.111, Lo1, [0/0], 03:50:25, direct
10.1.1.1/32, ubest/mbest: 2/0
    *via 10.2.1.2, [200/0], 03:53:29, bgp-65000, internal, tag 65000
    *via 10.2.2.2, [200/0], 03:53:29, bgp-65000, internal, tag 65000
10.1.1.2/32, ubest/mbest: 2/0, attached
    *via 10.1.1.2, Lo2, [0/0], 03:55:37, local
    *via 10.1.1.2, Lo2, [0/0], 03:55:37, direct
10.1.1.3/32, ubest/mbest: 2/0
    *via 10.2.1.2, [200/0], 02:35:45, bgp-65000, internal, tag 65000
    *via 10.2.2.2, [200/0], 02:35:45, bgp-65000, internal, tag 65000
10.1.1.4/32, ubest/mbest: 2/0
    *via 10.2.1.2, [200/0], 01:19:45, bgp-65000, internal, tag 65000
    *via 10.2.2.2, [200/0], 01:19:45, bgp-65000, internal, tag 65000
10.2.1.2/31, ubest/mbest: 1/0, attached
    *via 10.2.1.3, Eth1/1, [0/0], 03:54:08, direct
10.2.1.3/32, ubest/mbest: 1/0, attached
    *via 10.2.1.3, Eth1/1, [0/0], 03:54:08, local
10.2.2.2/31, ubest/mbest: 1/0, attached
    *via 10.2.2.3, Eth1/2, [0/0], 03:54:08, direct
10.2.2.3/32, ubest/mbest: 1/0, attached
    *via 10.2.2.3, Eth1/2, [0/0], 03:54:08, local


Leaf-2# sh ip route vrf SERVICE 
IP Route Table for VRF "SERVICE"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

10.4.0.0/30, ubest/mbest: 1/0
    *via 10.1.0.4%default, [200/0], 01:16:45, bgp-65000, internal, tag 65000, se
gid: 77777 tunnelid: 0xa010004 encap: VXLAN
 
10.4.0.4/30, ubest/mbest: 1/0
    *via 10.1.0.4%default, [200/0], 00:31:28, bgp-65000, internal, tag 65001, se
gid: 77777 tunnelid: 0xa010004 encap: VXLAN
 
192.168.10.0/24, ubest/mbest: 1/0, attached
    *via 192.168.10.1, Vlan10, [0/0], 03:53:31, direct
192.168.10.1/32, ubest/mbest: 1/0, attached
    *via 192.168.10.1, Vlan10, [0/0], 03:53:31, local
192.168.10.10/32, ubest/mbest: 1/0, attached
    *via 192.168.10.10, Vlan10, [190/0], 00:53:24, hmm
192.168.20.0/24, ubest/mbest: 1/0, attached
    *via 192.168.20.1, Vlan20, [0/0], 03:53:31, direct
192.168.20.1/32, ubest/mbest: 1/0, attached
    *via 192.168.20.1, Vlan20, [0/0], 03:53:31, local
192.168.20.10/32, ubest/mbest: 1/0, attached
    *via 192.168.20.10, Vlan20, [190/0], 00:23:25, hmm
192.168.30.10/32, ubest/mbest: 1/0
    *via 10.1.0.3%default, [200/0], 02:36:03, bgp-65000, internal, tag 65000, se
gid: 77777 tunnelid: 0xa010003 encap: VXLAN
 
192.168.40.0/24, ubest/mbest: 1/0
    *via 10.1.0.4%default, [200/0], 01:16:45, bgp-65000, internal, tag 65000, se
gid: 77777 tunnelid: 0xa010004 encap: VXLAN
 
192.168.40.10/32, ubest/mbest: 1/0
    *via 10.1.0.4%default, [200/0], 00:30:20, bgp-65000, internal, tag 65000, se
gid: 77777 tunnelid: 0xa010004 encap: VXLAN
 
192.168.50.10/32, ubest/mbest: 1/0
    *via 10.1.0.4%default, [200/0], 00:31:28, bgp-65000, internal, tag 65001, se
gid: 77777 tunnelid: 0xa010004 encap: VXLAN
 
192.168.60.0/24, ubest/mbest: 1/0
    *via 10.1.0.4%default, [200/0], 00:31:28, bgp-65000, internal, tag 65001, se
gid: 77777 tunnelid: 0xa010004 encap: VXLAN
 
192.168.60.10/32, ubest/mbest: 1/0
    *via 10.1.0.4%default, [200/0], 00:30:26, bgp-65000, internal, tag 65001, se
gid: 77777 tunnelid: 0xa010004 encap: VXLAN

#### L2 таблица маршрутизации
Leaf-2# sh bgp l2vpn evpn 
BGP routing table information for VRF default, address family L2VPN EVPN
BGP table version is 81, Local Router ID is 10.1.0.2
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-i
njected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - b
est2

   Network            Next Hop            Metric     LocPrf     Weight Path
Route Distinguisher: 10.1.0.2:32777    (L2VNI 101010)
*>l[2]:[0]:[0]:[48]:[0050.0000.0700]:[0]:[0.0.0.0]/216
                      10.1.0.111                        100      32768 i
*>l[2]:[0]:[0]:[48]:[0050.0000.0700]:[32]:[192.168.10.10]/272
                      10.1.0.111                        100      32768 i
*>l[3]:[0]:[32]:[10.1.0.111]/88
                      10.1.0.111                        100      32768 i

Route Distinguisher: 10.1.0.2:32787    (L2VNI 10020)
*>l[2]:[0]:[0]:[48]:[0050.0000.0a00]:[0]:[0.0.0.0]/216
                      10.1.0.111                        100      32768 i
*>l[2]:[0]:[0]:[48]:[0050.0000.0a00]:[32]:[192.168.20.10]/272
                      10.1.0.111                        100      32768 i
*>l[3]:[0]:[32]:[10.1.0.111]/88
                      10.1.0.111                        100      32768 i

Route Distinguisher: 10.1.0.3:32797
* i[2]:[0]:[0]:[48]:[0050.0000.0800]:[32]:[192.168.30.10]/272
                      10.1.0.3                          100          0 i
*>i                   10.1.0.3                          100          0 i

Route Distinguisher: 10.1.0.4:4
* i[5]:[0]:[0]:[24]:[192.168.40.0]/224
                      10.1.0.4                 0        100          0 ?
*>i                   10.1.0.4                 0        100          0 ?
*>i[5]:[0]:[0]:[24]:[192.168.60.0]/224
                      10.1.0.4                          100          0 65001 650
01 ?
* i                   10.1.0.4                          100          0 65001 650
01 ?
* i[5]:[0]:[0]:[30]:[10.4.0.0]/224
                      10.1.0.4                 0        100          0 ?
*>i                   10.1.0.4                 0        100          0 ?
*>i[5]:[0]:[0]:[30]:[10.4.0.4]/224
                      10.1.0.4                          100          0 65001 650
01 ?
* i                   10.1.0.4                          100          0 65001 650
01 ?
* i[5]:[0]:[0]:[32]:[192.168.40.10]/224
                      10.1.0.4                 0        100          0 ?
*>i                   10.1.0.4                 0        100          0 ?
*>i[5]:[0]:[0]:[32]:[192.168.50.10]/224
                      10.1.0.4                          100          0 65001 650
01 i
* i                   10.1.0.4                          100          0 65001 650
01 i
* i[5]:[0]:[0]:[32]:[192.168.60.10]/224
                      10.1.0.4                          100          0 65001 650
01 ?
*>i                   10.1.0.4                          100          0 65001 650
01 ?

Route Distinguisher: 10.1.0.4:32807
* i[2]:[0]:[0]:[48]:[0050.0000.0b00]:[32]:[192.168.40.10]/272
                      10.1.0.4                          100          0 i
*>i                   10.1.0.4                          100          0 i

Route Distinguisher: 10.1.0.2:4    (L3VNI 77777)
*>i[2]:[0]:[0]:[48]:[0050.0000.0800]:[32]:[192.168.30.10]/272
                      10.1.0.3                          100          0 i
*>i[2]:[0]:[0]:[48]:[0050.0000.0b00]:[32]:[192.168.40.10]/272
                      10.1.0.4                          100          0 i
*>i[5]:[0]:[0]:[24]:[192.168.40.0]/224
                      10.1.0.4                 0        100          0 ?
*>i[5]:[0]:[0]:[24]:[192.168.60.0]/224
                      10.1.0.4                          100          0 65001 650
01 ?
*>i[5]:[0]:[0]:[30]:[10.4.0.0]/224
                      10.1.0.4                 0        100          0 ?
*>i[5]:[0]:[0]:[30]:[10.4.0.4]/224
                      10.1.0.4                          100          0 65001 650
01 ?
*>i[5]:[0]:[0]:[32]:[192.168.40.10]/224
                      10.1.0.4                 0        100          0 ?
*>i[5]:[0]:[0]:[32]:[192.168.50.10]/224
                      10.1.0.4                          100          0 65001 650
01 i
*>i[5]:[0]:[0]:[32]:[192.168.60.10]/224
                      10.1.0.4                          100          0 65001 650
01 ?
```
</p>
  </details>

<details>
  <summary><b> Leaf-3 </b></summary>
  <p>
    
```

Leaf-3# sh ip route
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

10.0.1.0/32, ubest/mbest: 1/0
    *via 10.2.1.4, [200/0], 03:54:57, bgp-65000, internal, tag 65000
10.0.2.0/32, ubest/mbest: 1/0
    *via 10.2.2.4, [200/0], 03:54:54, bgp-65000, internal, tag 65000
10.1.0.1/32, ubest/mbest: 2/0
    *via 10.2.1.4, [200/0], 03:51:45, bgp-65000, internal, tag 65000
    *via 10.2.2.4, [200/0], 03:51:45, bgp-65000, internal, tag 65000
10.1.0.2/32, ubest/mbest: 2/0
    *via 10.2.1.4, [200/0], 03:51:45, bgp-65000, internal, tag 65000
    *via 10.2.2.4, [200/0], 03:51:45, bgp-65000, internal, tag 65000
10.1.0.3/32, ubest/mbest: 2/0, attached
    *via 10.1.0.3, Lo1, [0/0], 03:56:57, local
    *via 10.1.0.3, Lo1, [0/0], 03:56:57, direct
10.1.0.4/32, ubest/mbest: 2/0
    *via 10.2.1.4, [200/0], 01:21:05, bgp-65000, internal, tag 65000
    *via 10.2.2.4, [200/0], 01:21:05, bgp-65000, internal, tag 65000
10.1.0.111/32, ubest/mbest: 2/0
    *via 10.2.1.4, [200/0], 03:51:45, bgp-65000, internal, tag 65000
    *via 10.2.2.4, [200/0], 03:51:45, bgp-65000, internal, tag 65000
10.1.1.1/32, ubest/mbest: 2/0
    *via 10.2.1.4, [200/0], 03:54:50, bgp-65000, internal, tag 65000
    *via 10.2.2.4, [200/0], 03:54:50, bgp-65000, internal, tag 65000
10.1.1.2/32, ubest/mbest: 2/0
    *via 10.2.1.4, [200/0], 03:55:01, bgp-65000, internal, tag 65000
    *via 10.2.2.4, [200/0], 03:54:54, bgp-65000, internal, tag 65000
10.1.1.3/32, ubest/mbest: 2/0, attached
    *via 10.1.1.3, Lo2, [0/0], 03:56:57, local
    *via 10.1.1.3, Lo2, [0/0], 03:56:57, direct
10.1.1.4/32, ubest/mbest: 2/0
    *via 10.2.1.4, [200/0], 01:21:05, bgp-65000, internal, tag 65000
    *via 10.2.2.4, [200/0], 01:21:05, bgp-65000, internal, tag 65000
10.2.1.4/31, ubest/mbest: 1/0, attached
    *via 10.2.1.5, Eth1/1, [0/0], 03:55:31, direct
10.2.1.5/32, ubest/mbest: 1/0, attached
    *via 10.2.1.5, Eth1/1, [0/0], 03:55:31, local
10.2.2.4/31, ubest/mbest: 1/0, attached
    *via 10.2.2.5, Eth1/2, [0/0], 03:55:31, direct
10.2.2.5/32, ubest/mbest: 1/0, attached
    *via 10.2.2.5, Eth1/2, [0/0], 03:55:31, local

Leaf-3# sh ip route vrf SERVICE
IP Route Table for VRF "SERVICE"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

10.4.0.0/30, ubest/mbest: 1/0
    *via 10.1.0.4%default, [200/0], 01:19:18, bgp-65000, internal, tag 65000, se
gid: 77777 tunnelid: 0xa010004 encap: VXLAN
 
10.4.0.4/30, ubest/mbest: 1/0
    *via 10.1.0.4%default, [200/0], 00:34:01, bgp-65000, internal, tag 65001, se
gid: 77777 tunnelid: 0xa010004 encap: VXLAN
 
192.168.10.10/32, ubest/mbest: 1/0
    *via 10.1.0.111%default, [200/0], 02:32:46, bgp-65000, internal, tag 65000, 
segid: 77777 tunnelid: 0xa01006f encap: VXLAN
 
192.168.20.10/32, ubest/mbest: 1/0
    *via 10.1.0.111%default, [200/0], 02:35:34, bgp-65000, internal, tag 65000, 
segid: 77777 tunnelid: 0xa01006f encap: VXLAN
 
192.168.30.0/24, ubest/mbest: 1/0, attached
    *via 192.168.30.1, Vlan30, [0/0], 03:58:27, direct
192.168.30.1/32, ubest/mbest: 1/0, attached
    *via 192.168.30.1, Vlan30, [0/0], 03:58:27, local
192.168.30.10/32, ubest/mbest: 1/0, attached
    *via 192.168.30.10, Vlan30, [190/0], 00:27:03, hmm
192.168.40.0/24, ubest/mbest: 1/0
    *via 10.1.0.4%default, [200/0], 01:19:18, bgp-65000, internal, tag 65000, se
gid: 77777 tunnelid: 0xa010004 encap: VXLAN
 
192.168.40.10/32, ubest/mbest: 1/0
    *via 10.1.0.4%default, [200/0], 00:32:54, bgp-65000, internal, tag 65000, se
gid: 77777 tunnelid: 0xa010004 encap: VXLAN
 
192.168.50.10/32, ubest/mbest: 1/0
    *via 10.1.0.4%default, [200/0], 00:34:01, bgp-65000, internal, tag 65001, se
gid: 77777 tunnelid: 0xa010004 encap: VXLAN
 
192.168.60.0/24, ubest/mbest: 1/0
    *via 10.1.0.4%default, [200/0], 00:34:01, bgp-65000, internal, tag 65001, se
gid: 77777 tunnelid: 0xa010004 encap: VXLAN
 
192.168.60.10/32, ubest/mbest: 1/0
    *via 10.1.0.4%default, [200/0], 00:32:59, bgp-65000, internal, tag 65001, se
gid: 77777 tunnelid: 0xa010004 encap: VXLAN

Leaf-3# sh ip route vrf SERVICE2
IP Route Table for VRF "SERVICE2"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

10.4.0.0/30, ubest/mbest: 1/0
    *via 10.1.0.4%default, [200/0], 00:34:25, bgp-65000, internal, tag 65001, se
gid: 88888 tunnelid: 0xa010004 encap: VXLAN
 
10.4.0.4/30, ubest/mbest: 1/0
    *via 10.1.0.4%default, [200/0], 01:19:38, bgp-65000, internal, tag 65000, se
gid: 88888 tunnelid: 0xa010004 encap: VXLAN
 
192.168.10.10/32, ubest/mbest: 1/0
    *via 10.1.0.4%default, [200/0], 00:34:25, bgp-65000, internal, tag 65001, se
gid: 88888 tunnelid: 0xa010004 encap: VXLAN
 
192.168.20.10/32, ubest/mbest: 1/0
    *via 10.1.0.4%default, [200/0], 00:34:25, bgp-65000, internal, tag 65001, se
gid: 88888 tunnelid: 0xa010004 encap: VXLAN
 
192.168.30.10/32, ubest/mbest: 1/0
    *via 10.1.0.4%default, [200/0], 00:34:25, bgp-65000, internal, tag 65001, se
gid: 88888 tunnelid: 0xa010004 encap: VXLAN
 
192.168.40.0/24, ubest/mbest: 1/0
    *via 10.1.0.4%default, [200/0], 00:34:25, bgp-65000, internal, tag 65001, se
gid: 88888 tunnelid: 0xa010004 encap: VXLAN
 
192.168.40.10/32, ubest/mbest: 1/0
    *via 10.1.0.4%default, [200/0], 00:32:53, bgp-65000, internal, tag 65001, se
gid: 88888 tunnelid: 0xa010004 encap: VXLAN
 
192.168.50.0/24, ubest/mbest: 1/0, attached
    *via 192.168.50.1, Vlan50, [0/0], 02:42:02, direct
192.168.50.1/32, ubest/mbest: 1/0, attached
    *via 192.168.50.1, Vlan50, [0/0], 02:42:02, local
192.168.50.10/32, ubest/mbest: 1/0, attached
    *via 192.168.50.10, Vlan50, [190/0], 00:57:27, hmm
192.168.60.0/24, ubest/mbest: 1/0
    *via 10.1.0.4%default, [200/0], 01:19:38, bgp-65000, internal, tag 65000, se
gid: 88888 tunnelid: 0xa010004 encap: VXLAN
 
192.168.60.10/32, ubest/mbest: 1/0
    *via 10.1.0.4%default, [200/0], 00:33:23, bgp-65000, internal, tag 65000, se
gid: 88888 tunnelid: 0xa010004 encap: VXLAN

#### L2 таблица маршрутизации
Leaf-3# sh bgp l2vpn evpn 
BGP routing table information for VRF default, address family L2VPN EVPN
BGP table version is 142, Local Router ID is 10.1.0.3
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-i
njected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - b
est2

   Network            Next Hop            Metric     LocPrf     Weight Path
Route Distinguisher: 10.1.0.1:32777
* i[2]:[0]:[0]:[48]:[0050.0000.0700]:[32]:[192.168.10.10]/272
                      10.1.0.111                        100          0 i
*>i                   10.1.0.111                        100          0 i

Route Distinguisher: 10.1.0.1:32787
* i[2]:[0]:[0]:[48]:[0050.0000.0a00]:[32]:[192.168.20.10]/272
                      10.1.0.111                        100          0 i
*>i                   10.1.0.111                        100          0 i

Route Distinguisher: 10.1.0.2:32777
*>i[2]:[0]:[0]:[48]:[0050.0000.0700]:[32]:[192.168.10.10]/272
                      10.1.0.111                        100          0 i
* i                   10.1.0.111                        100          0 i

Route Distinguisher: 10.1.0.2:32787
* i[2]:[0]:[0]:[48]:[0050.0000.0a00]:[32]:[192.168.20.10]/272
                      10.1.0.111                        100          0 i
*>i                   10.1.0.111                        100          0 i

Route Distinguisher: 10.1.0.3:32797    (L2VNI 10030)
*>l[2]:[0]:[0]:[48]:[0050.0000.0800]:[0]:[0.0.0.0]/216
                      10.1.0.3                          100      32768 i
*>l[2]:[0]:[0]:[48]:[0050.0000.0800]:[32]:[192.168.30.10]/272
                      10.1.0.3                          100      32768 i
*>l[3]:[0]:[32]:[10.1.0.3]/88
                      10.1.0.3                          100      32768 i

Route Distinguisher: 10.1.0.3:32817    (L2VNI 10050)
*>l[2]:[0]:[0]:[48]:[0050.0000.0d00]:[0]:[0.0.0.0]/216
                      10.1.0.3                          100      32768 i
*>l[2]:[0]:[0]:[48]:[0050.0000.0d00]:[32]:[192.168.50.10]/272
                      10.1.0.3                          100      32768 i
*>l[3]:[0]:[32]:[10.1.0.3]/88
                      10.1.0.3                          100      32768 i

Route Distinguisher: 10.1.0.4:4
* i[5]:[0]:[0]:[24]:[192.168.40.0]/224
                      10.1.0.4                 0        100          0 ?
*>i                   10.1.0.4                 0        100          0 ?
*>i[5]:[0]:[0]:[24]:[192.168.60.0]/224
                      10.1.0.4                          100          0 65001 650
01 ?
* i                   10.1.0.4                          100          0 65001 650
01 ?
* i[5]:[0]:[0]:[30]:[10.4.0.0]/224
                      10.1.0.4                 0        100          0 ?
*>i                   10.1.0.4                 0        100          0 ?
*>i[5]:[0]:[0]:[30]:[10.4.0.4]/224
                      10.1.0.4                          100          0 65001 650
01 ?
* i                   10.1.0.4                          100          0 65001 650
01 ?
* i[5]:[0]:[0]:[32]:[192.168.40.10]/224
                      10.1.0.4                 0        100          0 ?
*>i                   10.1.0.4                 0        100          0 ?
*>i[5]:[0]:[0]:[32]:[192.168.50.10]/224
                      10.1.0.4                          100          0 65001 650
01 i
* i                   10.1.0.4                          100          0 65001 650
01 i
* i[5]:[0]:[0]:[32]:[192.168.60.10]/224
                      10.1.0.4                          100          0 65001 650
01 ?
*>i                   10.1.0.4                          100          0 65001 650
01 ?

Route Distinguisher: 10.1.0.4:5
*>i[5]:[0]:[0]:[24]:[192.168.40.0]/224
                      10.1.0.4                          100          0 65001 650
01 ?
* i                   10.1.0.4                          100          0 65001 650
01 ?
*>i[5]:[0]:[0]:[24]:[192.168.60.0]/224
                      10.1.0.4                 0        100          0 ?
* i                   10.1.0.4                 0        100          0 ?
*>i[5]:[0]:[0]:[30]:[10.4.0.0]/224
                      10.1.0.4                          100          0 65001 650
01 ?
* i                   10.1.0.4                          100          0 65001 650
01 ?
*>i[5]:[0]:[0]:[30]:[10.4.0.4]/224
                      10.1.0.4                 0        100          0 ?
* i                   10.1.0.4                 0        100          0 ?
*>i[5]:[0]:[0]:[32]:[192.168.10.10]/224
                      10.1.0.4                          100          0 65001 650
01 i
* i                   10.1.0.4                          100          0 65001 650
01 i
*>i[5]:[0]:[0]:[32]:[192.168.20.10]/224
                      10.1.0.4                          100          0 65001 650
01 i
* i                   10.1.0.4                          100          0 65001 650
01 i
*>i[5]:[0]:[0]:[32]:[192.168.30.10]/224
                      10.1.0.4                          100          0 65001 650
01 i
* i                   10.1.0.4                          100          0 65001 650
01 i
* i[5]:[0]:[0]:[32]:[192.168.40.10]/224
                      10.1.0.4                          100          0 65001 650
01 ?
*>i                   10.1.0.4                          100          0 65001 650
01 ?
* i[5]:[0]:[0]:[32]:[192.168.60.10]/224
                      10.1.0.4                 0        100          0 ?
*>i                   10.1.0.4                 0        100          0 ?

Route Distinguisher: 10.1.0.4:32807
* i[2]:[0]:[0]:[48]:[0050.0000.0b00]:[32]:[192.168.40.10]/272
                      10.1.0.4                          100          0 i
*>i                   10.1.0.4                          100          0 i

Route Distinguisher: 10.1.0.4:32827
* i[2]:[0]:[0]:[48]:[0050.0000.0e00]:[32]:[192.168.60.10]/272
                      10.1.0.4                          100          0 i
*>i                   10.1.0.4                          100          0 i

Route Distinguisher: 10.1.0.3:4    (L3VNI 77777)
* i[2]:[0]:[0]:[48]:[0050.0000.0700]:[32]:[192.168.10.10]/272
                      10.1.0.111                        100          0 i
*>i                   10.1.0.111                        100          0 i
* i[2]:[0]:[0]:[48]:[0050.0000.0a00]:[32]:[192.168.20.10]/272
                      10.1.0.111                        100          0 i
*>i                   10.1.0.111                        100          0 i
*>i[2]:[0]:[0]:[48]:[0050.0000.0b00]:[32]:[192.168.40.10]/272
                      10.1.0.4                          100          0 i
*>i[5]:[0]:[0]:[24]:[192.168.40.0]/224
                      10.1.0.4                 0        100          0 ?
*>i[5]:[0]:[0]:[24]:[192.168.60.0]/224
                      10.1.0.4                          100          0 65001 650
01 ?
*>i[5]:[0]:[0]:[30]:[10.4.0.0]/224
                      10.1.0.4                 0        100          0 ?
*>i[5]:[0]:[0]:[30]:[10.4.0.4]/224
                      10.1.0.4                          100          0 65001 650
01 ?
*>i[5]:[0]:[0]:[32]:[192.168.40.10]/224
                      10.1.0.4                 0        100          0 ?
*>i[5]:[0]:[0]:[32]:[192.168.50.10]/224
                      10.1.0.4                          100          0 65001 650
01 i
*>i[5]:[0]:[0]:[32]:[192.168.60.10]/224
                      10.1.0.4                          100          0 65001 650
01 ?

Route Distinguisher: 10.1.0.3:5    (L3VNI 88888)
*>i[2]:[0]:[0]:[48]:[0050.0000.0e00]:[32]:[192.168.60.10]/272
                      10.1.0.4                          100          0 i
*>i[5]:[0]:[0]:[24]:[192.168.40.0]/224
                      10.1.0.4                          100          0 65001 650
01 ?
*>i[5]:[0]:[0]:[24]:[192.168.60.0]/224
                      10.1.0.4                 0        100          0 ?
*>i[5]:[0]:[0]:[30]:[10.4.0.0]/224
                      10.1.0.4                          100          0 65001 650
01 ?
*>i[5]:[0]:[0]:[30]:[10.4.0.4]/224
                      10.1.0.4                 0        100          0 ?
*>i[5]:[0]:[0]:[32]:[192.168.10.10]/224
                      10.1.0.4                          100          0 65001 650
01 i
*>i[5]:[0]:[0]:[32]:[192.168.20.10]/224
                      10.1.0.4                          100          0 65001 650
01 i
*>i[5]:[0]:[0]:[32]:[192.168.30.10]/224
                      10.1.0.4                          100          0 65001 650
01 i
*>i[5]:[0]:[0]:[32]:[192.168.40.10]/224
                      10.1.0.4                          100          0 65001 650
01 ?
*>i[5]:[0]:[0]:[32]:[192.168.60.10]/224
                      10.1.0.4                 0        100          0 ?
```
</p>
  </details>

<details>
  <summary><b> Leaf-4 </b></summary>
  <p>
    
```
Leaf-4# sh ip route
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

10.0.1.0/32, ubest/mbest: 1/0
    *via 10.2.1.6, [200/0], 01:29:02, bgp-65000, internal, tag 65000
10.0.2.0/32, ubest/mbest: 1/0
    *via 10.2.2.6, [200/0], 01:29:02, bgp-65000, internal, tag 65000
10.1.0.1/32, ubest/mbest: 2/0
    *via 10.2.1.6, [200/0], 01:29:02, bgp-65000, internal, tag 65000
    *via 10.2.2.6, [200/0], 01:29:02, bgp-65000, internal, tag 65000
10.1.0.2/32, ubest/mbest: 2/0
    *via 10.2.1.6, [200/0], 01:29:02, bgp-65000, internal, tag 65000
    *via 10.2.2.6, [200/0], 01:29:02, bgp-65000, internal, tag 65000
10.1.0.3/32, ubest/mbest: 2/0
    *via 10.2.1.6, [200/0], 01:29:02, bgp-65000, internal, tag 65000
    *via 10.2.2.6, [200/0], 01:29:02, bgp-65000, internal, tag 65000
10.1.0.4/32, ubest/mbest: 2/0, attached
    *via 10.1.0.4, Lo1, [0/0], 01:30:51, local
    *via 10.1.0.4, Lo1, [0/0], 01:30:51, direct
10.1.0.111/32, ubest/mbest: 2/0
    *via 10.2.1.6, [200/0], 01:29:02, bgp-65000, internal, tag 65000
    *via 10.2.2.6, [200/0], 01:29:02, bgp-65000, internal, tag 65000
10.1.1.1/32, ubest/mbest: 2/0
    *via 10.2.1.6, [200/0], 01:29:02, bgp-65000, internal, tag 65000
    *via 10.2.2.6, [200/0], 01:29:02, bgp-65000, internal, tag 65000
10.1.1.2/32, ubest/mbest: 2/0
    *via 10.2.1.6, [200/0], 01:29:02, bgp-65000, internal, tag 65000
    *via 10.2.2.6, [200/0], 01:29:02, bgp-65000, internal, tag 65000
10.1.1.3/32, ubest/mbest: 2/0
    *via 10.2.1.6, [200/0], 01:29:02, bgp-65000, internal, tag 65000
    *via 10.2.2.6, [200/0], 01:29:02, bgp-65000, internal, tag 65000
10.1.1.4/32, ubest/mbest: 2/0, attached
    *via 10.1.1.4, Lo2, [0/0], 01:30:51, local
    *via 10.1.1.4, Lo2, [0/0], 01:30:51, direct
10.2.1.6/31, ubest/mbest: 1/0, attached
    *via 10.2.1.7, Eth1/1, [0/0], 01:29:31, direct
10.2.1.7/32, ubest/mbest: 1/0, attached
    *via 10.2.1.7, Eth1/1, [0/0], 01:29:31, local
10.2.2.6/31, ubest/mbest: 1/0, attached
    *via 10.2.2.7, Eth1/2, [0/0], 01:29:31, direct
10.2.2.7/32, ubest/mbest: 1/0, attached
    *via 10.2.2.7, Eth1/2, [0/0], 01:29:31, local

Leaf-4# sh ip route vrf SE
SERVICE    SERVICE2   
Leaf-4# sh ip route vrf SERVICE
IP Route Table for VRF "SERVICE"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

10.4.0.0/30, ubest/mbest: 1/0, attached
    *via 10.4.0.1, Vlan111, [0/0], 01:29:15, direct
10.4.0.1/32, ubest/mbest: 1/0, attached
    *via 10.4.0.1, Vlan111, [0/0], 01:29:15, local
10.4.0.4/30, ubest/mbest: 1/0
    *via 10.4.0.2, [20/0], 00:40:44, bgp-65000, external, tag 65001
192.168.10.10/32, ubest/mbest: 1/0
    *via 10.1.0.111%default, [200/0], 01:26:01, bgp-65000, internal, tag 65000, 
segid: 77777 tunnelid: 0xa01006f encap: VXLAN
 
192.168.20.10/32, ubest/mbest: 1/0
    *via 10.1.0.111%default, [200/0], 01:26:01, bgp-65000, internal, tag 65000, 
segid: 77777 tunnelid: 0xa01006f encap: VXLAN
 
192.168.30.10/32, ubest/mbest: 1/0
    *via 10.1.0.3%default, [200/0], 01:26:01, bgp-65000, internal, tag 65000, se
gid: 77777 tunnelid: 0xa010003 encap: VXLAN
 
192.168.40.0/24, ubest/mbest: 1/0, attached
    *via 192.168.40.1, Vlan40, [0/0], 01:31:06, direct
192.168.40.1/32, ubest/mbest: 1/0, attached
    *via 192.168.40.1, Vlan40, [0/0], 01:31:06, local
192.168.40.10/32, ubest/mbest: 1/0, attached
    *via 192.168.40.10, Vlan40, [190/0], 00:29:48, hmm
192.168.50.10/32, ubest/mbest: 1/0
    *via 10.4.0.2, [20/0], 00:40:44, bgp-65000, external, tag 65001
192.168.60.0/24, ubest/mbest: 1/0
    *via 10.4.0.2, [20/0], 00:40:44, bgp-65000, external, tag 65001
192.168.60.10/32, ubest/mbest: 1/0
    *via 10.4.0.2, [20/0], 00:39:42, bgp-65000, external, tag 65001

Leaf-4# sh ip route vrf SERVICE2
IP Route Table for VRF "SERVICE2"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

10.4.0.0/30, ubest/mbest: 1/0
    *via 10.4.0.6, [20/0], 00:41:04, bgp-65000, external, tag 65001
10.4.0.4/30, ubest/mbest: 1/0, attached
    *via 10.4.0.5, Vlan112, [0/0], 01:29:34, direct
10.4.0.5/32, ubest/mbest: 1/0, attached
    *via 10.4.0.5, Vlan112, [0/0], 01:29:34, local
192.168.10.10/32, ubest/mbest: 1/0
    *via 10.4.0.6, [20/0], 00:41:04, bgp-65000, external, tag 65001
192.168.20.10/32, ubest/mbest: 1/0
    *via 10.4.0.6, [20/0], 00:41:04, bgp-65000, external, tag 65001
192.168.30.10/32, ubest/mbest: 1/0
    *via 10.4.0.6, [20/0], 00:41:04, bgp-65000, external, tag 65001
192.168.40.0/24, ubest/mbest: 1/0
    *via 10.4.0.6, [20/0], 00:41:04, bgp-65000, external, tag 65001
192.168.40.10/32, ubest/mbest: 1/0
    *via 10.4.0.6, [20/0], 00:39:31, bgp-65000, external, tag 65001
192.168.50.10/32, ubest/mbest: 1/0
    *via 10.1.0.3%default, [200/0], 01:26:17, bgp-65000, internal, tag 65000, se
gid: 88888 tunnelid: 0xa010003 encap: VXLAN
 
192.168.60.0/24, ubest/mbest: 1/0, attached
    *via 192.168.60.1, Vlan60, [0/0], 01:31:26, direct
192.168.60.1/32, ubest/mbest: 1/0, attached
    *via 192.168.60.1, Vlan60, [0/0], 01:31:26, local
192.168.60.10/32, ubest/mbest: 1/0, attached
    *via 192.168.60.10, Vlan60, [190/0], 00:00:08, hmm

#### L2 таблица маршрутизации
Leaf-4# sh bgp l2vpn evpn 
BGP routing table information for VRF default, address family L2VPN EVPN
BGP table version is 38, Local Router ID is 10.1.0.4
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-i
njected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - b
est2

   Network            Next Hop            Metric     LocPrf     Weight Path
Route Distinguisher: 10.1.0.1:32777
* i[2]:[0]:[0]:[48]:[0050.0000.0700]:[32]:[192.168.10.10]/272
                      10.1.0.111                        100          0 i
*>i                   10.1.0.111                        100          0 i

Route Distinguisher: 10.1.0.1:32787
* i[2]:[0]:[0]:[48]:[0050.0000.0a00]:[32]:[192.168.20.10]/272
                      10.1.0.111                        100          0 i
*>i                   10.1.0.111                        100          0 i

Route Distinguisher: 10.1.0.2:32777
* i[2]:[0]:[0]:[48]:[0050.0000.0700]:[32]:[192.168.10.10]/272
                      10.1.0.111                        100          0 i
*>i                   10.1.0.111                        100          0 i

Route Distinguisher: 10.1.0.2:32787
* i[2]:[0]:[0]:[48]:[0050.0000.0a00]:[32]:[192.168.20.10]/272
                      10.1.0.111                        100          0 i
*>i                   10.1.0.111                        100          0 i

Route Distinguisher: 10.1.0.3:32797
* i[2]:[0]:[0]:[48]:[0050.0000.0800]:[32]:[192.168.30.10]/272
                      10.1.0.3                          100          0 i
*>i                   10.1.0.3                          100          0 i

Route Distinguisher: 10.1.0.3:32817
* i[2]:[0]:[0]:[48]:[0050.0000.0d00]:[32]:[192.168.50.10]/272
                      10.1.0.3                          100          0 i
*>i                   10.1.0.3                          100          0 i

Route Distinguisher: 10.1.0.4:32807    (L2VNI 10040)
*>l[2]:[0]:[0]:[48]:[0050.0000.0b00]:[0]:[0.0.0.0]/216
                      10.1.0.4                          100      32768 i
*>l[2]:[0]:[0]:[48]:[0050.0000.0b00]:[32]:[192.168.40.10]/272
                      10.1.0.4                          100      32768 i
*>l[3]:[0]:[32]:[10.1.0.4]/88
                      10.1.0.4                          100      32768 i

Route Distinguisher: 10.1.0.4:32827    (L2VNI 10060)
*>l[2]:[0]:[0]:[48]:[0050.0000.0e00]:[0]:[0.0.0.0]/216
                      10.1.0.4                          100      32768 i
*>l[2]:[0]:[0]:[48]:[0050.0000.0e00]:[32]:[192.168.60.10]/272
                      10.1.0.4                          100      32768 i
*>l[3]:[0]:[32]:[10.1.0.4]/88
                      10.1.0.4                          100      32768 i

Route Distinguisher: 10.1.0.4:4    (L3VNI 77777)
* i[2]:[0]:[0]:[48]:[0050.0000.0700]:[32]:[192.168.10.10]/272
                      10.1.0.111                        100          0 i
*>i                   10.1.0.111                        100          0 i
*>i[2]:[0]:[0]:[48]:[0050.0000.0800]:[32]:[192.168.30.10]/272
                      10.1.0.3                          100          0 i
* i[2]:[0]:[0]:[48]:[0050.0000.0a00]:[32]:[192.168.20.10]/272
                      10.1.0.111                        100          0 i
*>i                   10.1.0.111                        100          0 i
*>l[5]:[0]:[0]:[24]:[192.168.40.0]/224
                      10.1.0.4                 0        100      32768 ?
*>l[5]:[0]:[0]:[24]:[192.168.60.0]/224
                      10.1.0.4                                       0 65001 650
01 ?
*>l[5]:[0]:[0]:[30]:[10.4.0.0]/224
                      10.1.0.4                 0        100      32768 ?
*>l[5]:[0]:[0]:[30]:[10.4.0.4]/224
                      10.1.0.4                                       0 65001 650
01 ?
*>l[5]:[0]:[0]:[32]:[192.168.40.10]/224
                      10.1.0.4                 0        100      32768 ?
*>l[5]:[0]:[0]:[32]:[192.168.50.10]/224
                      10.1.0.4                                       0 65001 650
01 i
*>l[5]:[0]:[0]:[32]:[192.168.60.10]/224
                      10.1.0.4                                       0 65001 650
01 ?

Route Distinguisher: 10.1.0.4:5    (L3VNI 88888)
*>i[2]:[0]:[0]:[48]:[0050.0000.0d00]:[32]:[192.168.50.10]/272
                      10.1.0.3                          100          0 i
*>l[5]:[0]:[0]:[24]:[192.168.40.0]/224
                      10.1.0.4                                       0 65001 650
01 ?
*>l[5]:[0]:[0]:[24]:[192.168.60.0]/224
                      10.1.0.4                 0        100      32768 ?
*>l[5]:[0]:[0]:[30]:[10.4.0.0]/224
                      10.1.0.4                                       0 65001 650
01 ?
*>l[5]:[0]:[0]:[30]:[10.4.0.4]/224
                      10.1.0.4                 0        100      32768 ?
*>l[5]:[0]:[0]:[32]:[192.168.10.10]/224
                      10.1.0.4                                       0 65001 650
01 i
*>l[5]:[0]:[0]:[32]:[192.168.20.10]/224
                      10.1.0.4                                       0 65001 650
01 i
*>l[5]:[0]:[0]:[32]:[192.168.30.10]/224
                      10.1.0.4                                       0 65001 650
01 i
*>l[5]:[0]:[0]:[32]:[192.168.40.10]/224
                      10.1.0.4                                       0 65001 650
01 ?
*>l[5]:[0]:[0]:[32]:[192.168.60.10]/224
                      10.1.0.4                 0        100      32768 ?
```
</p>
  </details>
