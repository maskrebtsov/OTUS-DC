# Домашнее задание
VxLAN. L2 VNI

## Цель:
- настроить Overlay на основе VxLAN EVPN для L2 связанности между клиентами.

## Описание/Пошаговая инструкция выполнения домашнего задания:
В этой самостоятельной работе мы ожидаем, что вы самостоятельно:
- Настроить BGP peering между Leaf и Spine в AF l2vpn evpn
- Настроить связанность между клиентами в первой зоне и убедиться в её наличии
- Зафиксируете в документации - план работы, адресное пространство, схему сети, конфигурацию устройств
- 
# Выполнение: 

## Схема
![LAB01_DIAG.png](LAB01_DIAG.png)

## Целевое распределение адресного пространства

- 10.0.0.0/8 - LLC Private address space  <br>

- DC1 - 10.1.0.0/16
- DC2 - 10.2.0.0/16  <br>

- 10.x.255.0/24 - Lo0 RID (/32) (.1-.10 - S, .11-.250 - L, .251-.254 - BL)
- 10.x.254.0/24 - Lo1 VTEP_ID (/32) (.1-.10 - S, .11-.250 - L, .251-.254 - BL)  <br>

- 10.х.241.0/24 - P2P INTRASITE Underlay Transport from Spine1 to Leafs (/31) 
- 10.х.242.0/24 - P2P INTRASITE Underlay Transport from Spine2 to Leafs (/31)  <br>

- 10.x.251.0/24 - P2P INTRASITE Underlay Transport from Spines to Boarder Leafs (/31) (.0-.63 - to BL1, .64-.127 - to BL2)  <br>

## Вывод команд диагностики

### S1-1
```
S1-1# 
S1-1# sh run bgp

!Command: show running-config bgp
!Running configuration last done at: Mon Oct 19 19:02:50 2026
!Time: Wed Oct 21 10:19:21 2026

version 10.5(3) Bios:version  
feature bgp

router bgp 65001
  router-id 10.1.255.1
  reconnect-interval 12
  log-neighbor-changes
  address-family ipv4 unicast
    network 10.1.255.1/32
    maximum-paths 10
    maximum-paths ibgp 10
  address-family l2vpn evpn
  template peer peer_SPINE_EVPN
    remote-as 65001
    update-source loopback0
    address-family l2vpn evpn
      send-community
      send-community extended
      route-reflector-client
  template peer peer_SPINE_UNDERLAY
    bfd
    remote-as 65001
    timers 3 9
    address-family ipv4 unicast
      route-reflector-client
      next-hop-self all
  neighbor 10.1.241.1
    inherit peer peer_SPINE_UNDERLAY
    description to_L1-1_Underlay
  neighbor 10.1.241.3
    inherit peer peer_SPINE_UNDERLAY
    description to_L1-2_Underlay
  neighbor 10.1.255.11
    inherit peer peer_SPINE_EVPN
    description to_L1-1_EVPN
  neighbor 10.1.255.12
    inherit peer peer_SPINE_EVPN
    description to_L1-2_EVPN


S1-1# show ip bgp summary
BGP summary information for VRF default, address family IPv4 Unicast
BGP router identifier 10.1.255.1, local AS number 65001
BGP table version is 72, IPv4 Unicast config peers 2, capable peers 2
5 network entries and 5 paths using 1820 bytes of memory
BGP attribute entries [2/736], BGP AS path entries [0/0]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/
PfxRcd
10.1.241.1      4 65001      47063      47050       72    0    0 04:24:23 2     
    
10.1.241.3      4 65001      47065      47062       72    0    0 04:24:28 2     
    
S1-1# 
S1-1# sh  ip route bgp-65001 
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

10.1.254.11/32, ubest/mbest: 1/0
    *via 10.1.241.1, [200/0], 04:24:12, bgp-65001, internal, tag 65001
10.1.254.12/32, ubest/mbest: 1/0
    *via 10.1.241.3, [200/0], 04:18:36, bgp-65001, internal, tag 65001
10.1.255.11/32, ubest/mbest: 1/0
    *via 10.1.241.1, [200/0], 04:24:12, bgp-65001, internal, tag 65001
10.1.255.12/32, ubest/mbest: 1/0
    *via 10.1.241.3, [200/0], 04:24:20, bgp-65001, internal, tag 65001

S1-1# sh  ip bgp all
BGP routing table information for VRF default, address family IPv4 Unicast
BGP table version is 72, Local Router ID is 10.1.255.1
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-i
njected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - b
est2

   Network            Next Hop            Metric     LocPrf     Weight Path
*>i10.1.254.11/32     10.1.241.1                        100          0 i
*>i10.1.254.12/32     10.1.241.3                        100          0 i
*>l10.1.255.1/32      0.0.0.0                           100      32768 i
*>i10.1.255.11/32     10.1.241.1                        100          0 i
*>i10.1.255.12/32     10.1.241.3                        100          0 i

BGP routing table information for VRF default, address family L2VPN EVPN
BGP table version is 49, Local Router ID is 10.1.255.1
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-i
njected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - b
est2

   Network            Next Hop            Metric     LocPrf     Weight Path
Route Distinguisher: 10.1.255.11:10010
*>i[2]:[0]:[0]:[48]:[5254.0093.09ed]:[0]:[0.0.0.0]/216
                      10.1.254.11                       100          0 i
*>i[3]:[0]:[32]:[10.1.254.11]/88
                      10.1.254.11                       100          0 i

Route Distinguisher: 10.1.255.12:10010
*>i[2]:[0]:[0]:[48]:[5254.00a3.103d]:[0]:[0.0.0.0]/216
                      10.1.254.12                       100          0 i
*>i[3]:[0]:[32]:[10.1.254.12]/88
                      10.1.254.12                       100          0 i

--- ----------- ----- --------------------------------------- ------- 
S1-1# 

```

### S1-2
```
S1-2# sh run bgp

!Command: show running-config bgp
!Running configuration last done at: Mon Oct 19 19:02:49 2026
!Time: Wed Oct 21 10:21:32 2026

version 10.5(3) Bios:version  
feature bgp

router bgp 65001
  router-id 10.1.255.2
  reconnect-interval 12
  log-neighbor-changes
  address-family ipv4 unicast
    network 10.1.255.2/32
    maximum-paths 10
    maximum-paths ibgp 10
  address-family l2vpn evpn
  template peer peer_SPINE_EVPN
    remote-as 65001
    update-source loopback0
    address-family l2vpn evpn
      send-community
      send-community extended
      route-reflector-client
  template peer peer_SPINE_UNDERLAY
    bfd
    remote-as 65001
    timers 3 9
    address-family ipv4 unicast
      route-reflector-client
      next-hop-self all
  neighbor 10.1.242.1
    inherit peer peer_SPINE_UNDERLAY
    description to_L1-1_Underlay
  neighbor 10.1.242.3
    inherit peer peer_SPINE_UNDERLAY
    description to_L1-2_Underlay
  neighbor 10.1.255.11
    inherit peer peer_SPINE_EVPN
    description to_L1-1_EVPN
  neighbor 10.1.255.12
    inherit peer peer_SPINE_EVPN
    description to_L1-2_EVPN


S1-2#   
S1-2# show ip bgp summary
BGP summary information for VRF default, address family IPv4 Unicast
BGP router identifier 10.1.255.2, local AS number 65001
BGP table version is 69, IPv4 Unicast config peers 2, capable peers 2
5 network entries and 5 paths using 1820 bytes of memory
BGP attribute entries [2/736], BGP AS path entries [0/0]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/
PfxRcd
10.1.242.1      4 65001      47109      47095       69    0    0 04:26:25 2     
    
10.1.242.3      4 65001      47103      47099       69    0    0 04:26:36 2     
    
S1-2#  sh  ip route bgp-65001 
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

10.1.254.11/32, ubest/mbest: 1/0
    *via 10.1.242.1, [200/0], 04:26:23, bgp-65001, internal, tag 65001
10.1.254.12/32, ubest/mbest: 1/0
    *via 10.1.242.3, [200/0], 04:20:48, bgp-65001, internal, tag 65001
10.1.255.11/32, ubest/mbest: 1/0
    *via 10.1.242.1, [200/0], 04:26:23, bgp-65001, internal, tag 65001
10.1.255.12/32, ubest/mbest: 1/0
    *via 10.1.242.3, [200/0], 04:26:31, bgp-65001, internal, tag 65001

S1-2# 
S1-2# sh  ip bgp all
BGP routing table information for VRF default, address family IPv4 Unicast
BGP table version is 69, Local Router ID is 10.1.255.2
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-i
njected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - b
est2

   Network            Next Hop            Metric     LocPrf     Weight Path
*>i10.1.254.11/32     10.1.242.1                        100          0 i
*>i10.1.254.12/32     10.1.242.3                        100          0 i
*>l10.1.255.2/32      0.0.0.0                           100      32768 i
*>i10.1.255.11/32     10.1.242.1                        100          0 i
*>i10.1.255.12/32     10.1.242.3                        100          0 i

BGP routing table information for VRF default, address family L2VPN EVPN
BGP table version is 48, Local Router ID is 10.1.255.2
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-i
njected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - b
est2

   Network            Next Hop            Metric     LocPrf     Weight Path
Route Distinguisher: 10.1.255.11:10010
*>i[2]:[0]:[0]:[48]:[5254.0093.09ed]:[0]:[0.0.0.0]/216
                      10.1.254.11                       100          0 i
*>i[3]:[0]:[32]:[10.1.254.11]/88
                      10.1.254.11                       100          0 i

Route Distinguisher: 10.1.255.12:10010
*>i[2]:[0]:[0]:[48]:[5254.00a3.103d]:[0]:[0.0.0.0]/216
                      10.1.254.12                       100          0 i
*>i[3]:[0]:[32]:[10.1.254.12]/88
                      10.1.254.12                       100          0 i

S1-2# show bgp l2vpn evpn summary
BGP summary information for VRF default, address family L2VPN EVPN
BGP router identifier 10.1.255.2, local AS number 65001
BGP table version is 48, L2VPN EVPN config peers 2, capable peers 2
4 network entries and 4 paths using 1456 bytes of memory
BGP attribute entries [3/1104], BGP AS path entries [0/0]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/
PfxRcd
10.1.255.11     4 65001       2370       2371       48    0    0 04:26:34 2     
    
10.1.255.12     4 65001        430        433       48    0    0 04:26:42 2     
    

Neighbor        T    AS Type-1     Type-2     Type-3     Type-4     Type-5     T
ype-6     Type-7     Type-8     Type-12   
10.1.255.11     I 65001 0          1          1          0          0          0
          0          0          0         
10.1.255.12     I 65001 0          1          1          0          0          0
          0          0          0         
S1-2# 

```
### L1-1
```
L1-1# sh run bgp

!Command: show running-config bgp
!Running configuration last done at: Wed Oct 21 06:44:18 2026
!Time: Wed Oct 21 08:27:25 2026

version 10.5(3) Bios:version  
feature bgp

router bgp 65001
  router-id 10.1.255.11
  reconnect-interval 12
  log-neighbor-changes
  address-family ipv4 unicast
    network 10.1.254.11/32
    network 10.1.255.11/32
    maximum-paths 10
    maximum-paths ibgp 10
  address-family l2vpn evpn
  template peer peer_LEAF_EVPN
    remote-as 65001
    update-source loopback0
    address-family l2vpn evpn
      send-community
      send-community extended
  template peer peer_LEAF_UNDERLAY
    bfd
    remote-as 65001
    timers 3 9
    address-family ipv4 unicast
  neighbor 10.1.241.0
    inherit peer peer_LEAF_UNDERLAY
    description to_Spine1_Underlay
  neighbor 10.1.242.0
    inherit peer peer_LEAF_UNDERLAY
    description to_Spine2_Underlay
  neighbor 10.1.255.1
    inherit peer peer_LEAF_EVPN
    description to_Spine1_EVPN
  neighbor 10.1.255.2
    inherit peer peer_LEAF_EVPN
    description to_Spine2_EVPN
evpn
  vni 10010 l2
    rd 10.1.255.11:10010
    route-target import 65001:10010
    route-target export 65001:10010


L1-1# sh run vl 10

!Command: show running-config vlan 10
!Running configuration last done at: Wed Oct 21 06:44:18 2026
!Time: Wed Oct 21 08:27:31 2026

version 10.5(3) Bios:version  
vlan 10
vlan 10
  vn-segment 10010

L1-1# sh run nv overlay

!Command: show running-config nv overlay
!Running configuration last done at: Wed Oct 21 06:44:18 2026
!Time: Wed Oct 21 08:27:39 2026

version 10.5(3) Bios:version  
feature nv overlay

vlan 10

interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback1
  source-interface hold-down-time 1
  member vni 10010
    ingress-replication protocol bgp

L1-1# show ip bgp summary
BGP summary information for VRF default, address family IPv4 Unicast
BGP router identifier 10.1.255.11, local AS number 65001
BGP table version is 12, IPv4 Unicast config peers 2, capable peers 2
6 network entries and 8 paths using 2392 bytes of memory
BGP attribute entries [4/1472], BGP AS path entries [0/0]
BGP community entries [0/0], BGP clusterlist entries [2/8]

Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/
PfxRcd
10.1.241.0      4 65001       2771       2772       12    0    0 02:18:18 3     
    
10.1.242.0      4 65001       2769       2769       12    0    0 02:18:09 3     
    
L1-1# sh  ip route bgp-65001 
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

10.1.254.12/32, ubest/mbest: 2/0
    *via 10.1.241.0, [200/0], 02:12:31, bgp-65001, internal, tag 65001
    *via 10.1.242.0, [200/0], 02:12:31, bgp-65001, internal, tag 65001
10.1.255.1/32, ubest/mbest: 1/0
    *via 10.1.241.0, [200/0], 02:18:13, bgp-65001, internal, tag 65001
10.1.255.2/32, ubest/mbest: 1/0
    *via 10.1.242.0, [200/0], 02:18:13, bgp-65001, internal, tag 65001
10.1.255.12/32, ubest/mbest: 2/0
    *via 10.1.241.0, [200/0], 02:18:13, bgp-65001, internal, tag 65001
    *via 10.1.242.0, [200/0], 02:18:13, bgp-65001, internal, tag 65001

L1-1# sh  ip bgp all
BGP routing table information for VRF default, address family IPv4 Unicast
BGP table version is 12, Local Router ID is 10.1.255.11
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-i
njected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - b
est2

   Network            Next Hop            Metric     LocPrf     Weight Path
*>l10.1.254.11/32     0.0.0.0                           100      32768 i
*|i10.1.254.12/32     10.1.242.0                        100          0 i
*>i                   10.1.241.0                        100          0 i
*>i10.1.255.1/32      10.1.241.0                        100          0 i
*>i10.1.255.2/32      10.1.242.0                        100          0 i
*>l10.1.255.11/32     0.0.0.0                           100      32768 i
*|i10.1.255.12/32     10.1.242.0                        100          0 i
*>i                   10.1.241.0                        100          0 i

BGP routing table information for VRF default, address family L2VPN EVPN
BGP table version is 41, Local Router ID is 10.1.255.11
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-i
njected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - b
est2

   Network            Next Hop            Metric     LocPrf     Weight Path
Route Distinguisher: 10.1.255.11:10010    (L2VNI 10010)
*>l[2]:[0]:[0]:[48]:[5254.0093.09ed]:[0]:[0.0.0.0]/216
                      10.1.254.11                       100      32768 i
*>i[2]:[0]:[0]:[48]:[5254.00a3.103d]:[0]:[0.0.0.0]/216
                      10.1.254.12                       100          0 i
*>l[3]:[0]:[32]:[10.1.254.11]/88
                      10.1.254.11                       100      32768 i
*>i[3]:[0]:[32]:[10.1.254.12]/88
                      10.1.254.12                       100          0 i

Route Distinguisher: 10.1.255.12:10010
*>i[2]:[0]:[0]:[48]:[5254.00a3.103d]:[0]:[0.0.0.0]/216
                      10.1.254.12                       100          0 i
* i                   10.1.254.12                       100          0 i
*>i[3]:[0]:[32]:[10.1.254.12]/88
                      10.1.254.12                       100          0 i
* i                   10.1.254.12                       100          0 i

L1-1# show bgp l2vpn evpn summary
BGP summary information for VRF default, address family L2VPN EVPN
BGP router identifier 10.1.255.11, local AS number 65001
BGP table version is 41, L2VPN EVPN config peers 2, capable peers 2
6 network entries and 8 paths using 2184 bytes of memory
BGP attribute entries [8/2944], BGP AS path entries [0/0]
BGP community entries [0/0], BGP clusterlist entries [2/8]

Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/
PfxRcd
10.1.255.1      4 65001        149        145       41    0    0 02:18:25 2     
    
10.1.255.2      4 65001        149        145       41    0    0 02:18:26 2     
    

Neighbor        T    AS Type-1     Type-2     Type-3     Type-4     Type-5     T
ype-6     Type-7     Type-8     Type-12   
10.1.255.1      I 65001 0          1          1          0          0          0
          0          0          0         
10.1.255.2      I 65001 0          1          1          0          0          0
          0          0          0         
L1-1#  show l2route evpn imet all

Flags- (F): Originated From Fabric, (W): Originated from WAN
TE-Flags- (ELB): Egress Load Balancing

Topology ID VNI         Prod  IP Addr                                 Flags  
----------- ----------- ----- --------------------------------------- -------
10          10010       BGP   10.1.254.12                             -      
10          10010       VXLAN 10.1.254.11                             -      
L1-1# sh  ip bgp 10.1.255.12
BGP routing table information for VRF default, address family IPv4 Unicast
BGP routing table entry for 10.1.255.12/32, version 6
Paths: (2 available, best #2)
Flags: (0x8000001a) (high32 00000000) on xmit-list, is in urib, is best urib rou
te, is in HW
Multipath: eBGP iBGP

  Path type: internal, path is valid, not best reason: Neighbor Address, multipa
th, no labeled nexthop, in rib
  AS-Path: NONE, path sourced internal to AS
    10.1.242.0 (metric 0) from 10.1.242.0 (10.1.255.2)
      Origin IGP, MED not set, localpref 100, weight 0
      Originator: 10.1.255.12 Cluster list: 10.1.255.2 

  Advertised path-id 1
  Path type: internal, path is valid, is best path, no labeled nexthop, in rib
  AS-Path: NONE, path sourced internal to AS
    10.1.241.0 (metric 0) from 10.1.241.0 (10.1.255.1)
      Origin IGP, MED not set, localpref 100, weight 0
      Originator: 10.1.255.12 Cluster list: 10.1.255.1 

  Path-id 1 not advertised to any peer

L1-1# 

```

### L1-2
```
L1-2# sh run bgp

!Command: show running-config bgp
!Running configuration last done at: Wed Oct 21 06:44:06 2026
!Time: Wed Oct 21 08:30:05 2026

version 10.5(3) Bios:version  
feature bgp

router bgp 65001
  router-id 10.1.255.12
  reconnect-interval 12
  log-neighbor-changes
  address-family ipv4 unicast
    network 10.1.254.12/32
    network 10.1.255.12/32
    maximum-paths 10
    maximum-paths ibgp 10
  address-family l2vpn evpn
  template peer peer_LEAF_EVPN
    remote-as 65001
    update-source loopback0
    address-family l2vpn evpn
      send-community
      send-community extended
  template peer peer_LEAF_UNDERLAY
    bfd
    remote-as 65001
    timers 3 9
    address-family ipv4 unicast
  neighbor 10.1.241.2
    inherit peer peer_LEAF_UNDERLAY
    description to_Spine1_Underlay
  neighbor 10.1.242.2
    inherit peer peer_LEAF_UNDERLAY
    description to_Spine2_Underlay
  neighbor 10.1.255.1
    inherit peer peer_LEAF_EVPN
    description to_Spine1_EVPN
  neighbor 10.1.255.2
    inherit peer peer_LEAF_EVPN
    description to_Spine2_EVPN
evpn
  vni 10010 l2
    rd 10.1.255.12:10010
    route-target import 65001:10010
    route-target export 65001:10010


L1-2#           sh run vl 10

!Command: show running-config vlan 10
!Running configuration last done at: Wed Oct 21 06:44:06 2026
!Time: Wed Oct 21 08:30:09 2026

version 10.5(3) Bios:version  
vlan 10
vlan 10
  vn-segment 10010


L1-2# 
L1-2# sh run nv overlay 

!Command: show running-config nv overlay
!Running configuration last done at: Wed Oct 21 06:44:06 2026
!Time: Wed Oct 21 08:30:15 2026

version 10.5(3) Bios:version  
feature nv overlay

vlan 10

interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback1
  source-interface hold-down-time 1
  member vni 10010
    ingress-replication protocol bgp

L1-2# 
L1-2# show ip bgp summary
BGP summary information for VRF default, address family IPv4 Unicast
BGP router identifier 10.1.255.12, local AS number 65001
BGP table version is 12, IPv4 Unicast config peers 2, capable peers 2
6 network entries and 8 paths using 2392 bytes of memory
BGP attribute entries [4/1472], BGP AS path entries [0/0]
BGP community entries [0/0], BGP clusterlist entries [2/8]

Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/
PfxRcd
10.1.241.2      4 65001       2826       2825       12    0    0 02:21:00 3     
    
10.1.242.2      4 65001       2825       2824       12    0    0 02:20:58 3     
    
L1-2#  sh  ip route bgp-65001 
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

10.1.254.11/32, ubest/mbest: 2/0
    *via 10.1.241.2, [200/0], 02:20:45, bgp-65001, internal, tag 65001
    *via 10.1.242.2, [200/0], 02:20:45, bgp-65001, internal, tag 65001
10.1.255.1/32, ubest/mbest: 1/0
    *via 10.1.241.2, [200/0], 02:21:02, bgp-65001, internal, tag 65001
10.1.255.2/32, ubest/mbest: 1/0
    *via 10.1.242.2, [200/0], 02:21:02, bgp-65001, internal, tag 65001
10.1.255.11/32, ubest/mbest: 2/0
    *via 10.1.241.2, [200/0], 02:20:45, bgp-65001, internal, tag 65001
    *via 10.1.242.2, [200/0], 02:20:45, bgp-65001, internal, tag 65001

L1-2#  sh  ip bgp all
BGP routing table information for VRF default, address family IPv4 Unicast
BGP table version is 12, Local Router ID is 10.1.255.12
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-i
njected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - b
est2

   Network            Next Hop            Metric     LocPrf     Weight Path
*|i10.1.254.11/32     10.1.242.2                        100          0 i
*>i                   10.1.241.2                        100          0 i
*>l10.1.254.12/32     0.0.0.0                           100      32768 i
*>i10.1.255.1/32      10.1.241.2                        100          0 i
*>i10.1.255.2/32      10.1.242.2                        100          0 i
*|i10.1.255.11/32     10.1.242.2                        100          0 i
*>i                   10.1.241.2                        100          0 i
*>l10.1.255.12/32     0.0.0.0                           100      32768 i

BGP routing table information for VRF default, address family L2VPN EVPN
BGP table version is 36, Local Router ID is 10.1.255.12
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-i
njected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - b
est2

   Network            Next Hop            Metric     LocPrf     Weight Path
Route Distinguisher: 10.1.255.11:10010
* i[2]:[0]:[0]:[48]:[5254.0093.09ed]:[0]:[0.0.0.0]/216
                      10.1.254.11                       100          0 i
*>i                   10.1.254.11                       100          0 i
* i[3]:[0]:[32]:[10.1.254.11]/88
                      10.1.254.11                       100          0 i
*>i                   10.1.254.11                       100          0 i

Route Distinguisher: 10.1.255.12:10010    (L2VNI 10010)
*>i[2]:[0]:[0]:[48]:[5254.0093.09ed]:[0]:[0.0.0.0]/216
                      10.1.254.11                       100          0 i
*>l[2]:[0]:[0]:[48]:[5254.00a3.103d]:[0]:[0.0.0.0]/216
                      10.1.254.12                       100      32768 i
*>i[3]:[0]:[32]:[10.1.254.11]/88
                      10.1.254.11                       100          0 i
*>l[3]:[0]:[32]:[10.1.254.12]/88
                      10.1.254.12                       100      32768 i

L1-2# show bgp l2vpn evpn summary
BGP summary information for VRF default, address family L2VPN EVPN
BGP router identifier 10.1.255.12, local AS number 65001
BGP table version is 36, L2VPN EVPN config peers 2, capable peers 2
6 network entries and 8 paths using 2184 bytes of memory
BGP attribute entries [8/2944], BGP AS path entries [0/0]
BGP community entries [0/0], BGP clusterlist entries [2/8]

Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/
PfxRcd
10.1.255.1      4 65001        152        147       36    0    0 02:21:00 2     
    
10.1.255.2      4 65001        152        147       36    0    0 02:21:01 2     
    

Neighbor        T    AS Type-1     Type-2     Type-3     Type-4     Type-5     T
ype-6     Type-7     Type-8     Type-12   
10.1.255.1      I 65001 0          1          1          0          0          0
          0          0          0         
10.1.255.2      I 65001 0          1          1          0          0          0
          0          0          0         
L1-2#  show l2route evpn imet all

Flags- (F): Originated From Fabric, (W): Originated from WAN
TE-Flags- (ELB): Egress Load Balancing

Topology ID VNI         Prod  IP Addr                                 Flags  
----------- ----------- ----- --------------------------------------- -------
10          10010       BGP   10.1.254.11                             -      
10          10010       VXLAN 10.1.254.12                             -      
L1-1# sh  ip bgp 10.1.254.12
BGP routing table information for VRF default, address family IPv4 Unicast
BGP routing table entry for 10.1.254.12/32, version 11
Paths: (2 available, best #2)
Flags: (0x8000001a) (high32 00000000) on xmit-list, is in urib, is best urib rou
te, is in HW
Multipath: eBGP iBGP

  Path type: internal, path is valid, not best reason: Neighbor Address, multipa
th, no labeled nexthop, in rib
  AS-Path: NONE, path sourced internal to AS
    10.1.242.0 (metric 0) from 10.1.242.0 (10.1.255.2)
      Origin IGP, MED not set, localpref 100, weight 0
      Originator: 10.1.255.12 Cluster list: 10.1.255.2 

  Advertised path-id 1
  Path type: internal, path is valid, is best path, no labeled nexthop, in rib
  AS-Path: NONE, path sourced internal to AS
    10.1.241.0 (metric 0) from 10.1.241.0 (10.1.255.1)
      Origin IGP, MED not set, localpref 100, weight 0
      Originator: 10.1.255.12 Cluster list: 10.1.255.1 

  Path-id 1 not advertised to any peer

L1-2# 
	

```

