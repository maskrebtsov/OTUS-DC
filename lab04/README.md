# Домашнее задание
Underlay. BGP

## Цель:
- настроить BGP для Underlay сети.

## Описание/Пошаговая инструкция выполнения домашнего задания:
В этой самостоятельной работе мы ожидаем, что вы самостоятельно:
- Настроите BGP в Underlay сети, для IP связанности между всеми сетевыми устройствами. iBGP или eBGP - решать вам!
- Зафиксируете в документации - план работы, адресное пространство, схему сети, конфигурацию устройств
- Убедитесь в наличии IP связанности между устройствами в BGP домене
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
S1-1# sh ip bgp summary 
BGP summary information for VRF default, address family IPv4 Unicast
BGP router identifier 10.1.255.1, local AS number 65001
BGP table version is 48, IPv4 Unicast config peers 2, capable peers 2
6 network entries and 6 paths using 2184 bytes of memory
BGP attribute entries [2/736], BGP AS path entries [0/0]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/
PfxRcd
10.1.241.1      4 65001        702        711       48    0    0 00:04:45 2     
    
10.1.241.3      4 65001        668        681       48    0    0 00:04:44 2     
    
S1-1#  
S1-1# sh  ip route bgp-65001 
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

10.1.254.11/32, ubest/mbest: 1/0
    *via 10.1.241.1, [200/0], 00:04:49, bgp-65001, internal, tag 65001
10.1.254.12/32, ubest/mbest: 1/0
    *via 10.1.241.3, [200/0], 00:04:49, bgp-65001, internal, tag 65001
10.1.255.11/32, ubest/mbest: 1/0
    *via 10.1.241.1, [200/0], 00:04:49, bgp-65001, internal, tag 65001
10.1.255.12/32, ubest/mbest: 1/0
    *via 10.1.241.3, [200/0], 00:04:49, bgp-65001, internal, tag 65001

S1-1#  
S1-1# sh  ip bgp all
BGP routing table information for VRF default, address family IPv4 Unicast
BGP table version is 48, Local Router ID is 10.1.255.1
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-i
njected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - b
est2

   Network            Next Hop            Metric     LocPrf     Weight Path
*>l10.1.254.1/32      0.0.0.0                           100      32768 i
*>i10.1.254.11/32     10.1.241.1                        100          0 i
*>i10.1.254.12/32     10.1.241.3                        100          0 i
*>l10.1.255.1/32      0.0.0.0                           100      32768 i
*>i10.1.255.11/32     10.1.241.1                        100          0 i
*>i10.1.255.12/32     10.1.241.3                        100          0 i

S1-1#  
```

### S1-2
```
S1-2# 
S1-2# sh run bgp

!Command: show running-config bgp
!Running configuration last done at: Sun Oct 18 16:08:54 2026
!Time: Sun Oct 18 16:16:27 2026

version 10.5(3) Bios:version  
feature bgp

router bgp 65001
  router-id 10.1.255.2
  reconnect-interval 12
  log-neighbor-changes
  address-family ipv4 unicast
    network 10.1.254.2/32
    network 10.1.255.2/32
    maximum-paths 10
    maximum-paths ibgp 10
  template peer peer_SPINE
    bfd
    remote-as 65001
    timers 3 9
    address-family ipv4 unicast
      route-reflector-client
      next-hop-self all
  neighbor 10.1.242.1
    inherit peer peer_SPINE
    description to_L1-1
  neighbor 10.1.242.3
    inherit peer peer_SPINE
    description to_L1-2


S1-2# 
S1-2# sh ip bgp summary 
BGP summary information for VRF default, address family IPv4 Unicast
BGP router identifier 10.1.255.2, local AS number 65001
BGP table version is 43, IPv4 Unicast config peers 2, capable peers 2
6 network entries and 6 paths using 2184 bytes of memory
BGP attribute entries [2/736], BGP AS path entries [0/0]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/
PfxRcd
10.1.242.1      4 65001        749        754       43    0    0 00:07:02 2     
    
10.1.242.3      4 65001        710        724       43    0    0 00:07:04 2     
    
S1-2# 
S1-2# sh  ip route bgp-65001 
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

10.1.254.11/32, ubest/mbest: 1/0
    *via 10.1.242.1, [200/0], 00:07:06, bgp-65001, internal, tag 65001
10.1.254.12/32, ubest/mbest: 1/0
    *via 10.1.242.3, [200/0], 00:07:06, bgp-65001, internal, tag 65001
10.1.255.11/32, ubest/mbest: 1/0
    *via 10.1.242.1, [200/0], 00:07:06, bgp-65001, internal, tag 65001
10.1.255.12/32, ubest/mbest: 1/0
    *via 10.1.242.3, [200/0], 00:07:06, bgp-65001, internal, tag 65001

S1-2#  
S1-2# sh  ip bgp all 
BGP routing table information for VRF default, address family IPv4 Unicast
BGP table version is 43, Local Router ID is 10.1.255.2
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-i
njected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - b
est2

   Network            Next Hop            Metric     LocPrf     Weight Path
*>l10.1.254.2/32      0.0.0.0                           100      32768 i
*>i10.1.254.11/32     10.1.242.1                        100          0 i
*>i10.1.254.12/32     10.1.242.3                        100          0 i
*>l10.1.255.2/32      0.0.0.0                           100      32768 i
*>i10.1.255.11/32     10.1.242.1                        100          0 i
*>i10.1.255.12/32     10.1.242.3                        100          0 i

S1-2# 
```
### L1-1
```
L1-1# 
L1-1# sh run bgp

!Command: show running-config bgp
!Running configuration last done at: Sun Oct 18 16:09:03 2026
!Time: Sun Oct 18 16:17:39 2026

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
  template peer peer_LEAF
    bfd
    remote-as 65001
    timers 3 9
    address-family ipv4 unicast
  neighbor 10.1.241.0
    inherit peer peer_LEAF
    description to_S1-1
  neighbor 10.1.242.0
    inherit peer peer_LEAF
    description to_S1-2


L1-1# 
L1-1# sh ip bgp summary 
BGP summary information for VRF default, address family IPv4 Unicast
BGP router identifier 10.1.255.11, local AS number 65001
BGP table version is 69, IPv4 Unicast config peers 2, capable peers 2
8 network entries and 10 paths using 3120 bytes of memory
BGP attribute entries [4/1472], BGP AS path entries [0/0]
BGP community entries [0/0], BGP clusterlist entries [2/8]

Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/
PfxRcd
10.1.241.0      4 65001        793        770       69    0    0 00:08:14 4     
    
10.1.242.0      4 65001        787        770       69    0    0 00:08:08 4     
    
L1-1#  
L1-1# sh  ip route bgp-65001 
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

10.1.254.1/32, ubest/mbest: 1/0
    *via 10.1.241.0, [200/0], 00:08:20, bgp-65001, internal, tag 65001
10.1.254.2/32, ubest/mbest: 1/0
    *via 10.1.242.0, [200/0], 00:08:13, bgp-65001, internal, tag 65001
10.1.254.12/32, ubest/mbest: 2/0
    *via 10.1.241.0, [200/0], 00:08:18, bgp-65001, internal, tag 65001
    *via 10.1.242.0, [200/0], 00:08:13, bgp-65001, internal, tag 65001
10.1.255.1/32, ubest/mbest: 1/0
    *via 10.1.241.0, [200/0], 00:08:20, bgp-65001, internal, tag 65001
10.1.255.2/32, ubest/mbest: 1/0
    *via 10.1.242.0, [200/0], 00:08:13, bgp-65001, internal, tag 65001
10.1.255.12/32, ubest/mbest: 2/0
    *via 10.1.241.0, [200/0], 00:08:18, bgp-65001, internal, tag 65001
    *via 10.1.242.0, [200/0], 00:08:13, bgp-65001, internal, tag 65001

L1-1# 
L1-1# sh  ip bgp all 
BGP routing table information for VRF default, address family IPv4 Unicast
BGP table version is 69, Local Router ID is 10.1.255.11
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-i
njected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - b
est2

   Network            Next Hop            Metric     LocPrf     Weight Path
*>i10.1.254.1/32      10.1.241.0                        100          0 i
*>i10.1.254.2/32      10.1.242.0                        100          0 i
*>l10.1.254.11/32     0.0.0.0                           100      32768 i
*|i10.1.254.12/32     10.1.242.0                        100          0 i
*>i                   10.1.241.0                        100          0 i
*>i10.1.255.1/32      10.1.241.0                        100          0 i
*>i10.1.255.2/32      10.1.242.0                        100          0 i
*>l10.1.255.11/32     0.0.0.0                           100      32768 i
*|i10.1.255.12/32     10.1.242.0                        100          0 i
*>i                   10.1.241.0                        100          0 i

L1-1# 
L1-1# sh  ip bgp 10.1.255.12
BGP routing table information for VRF default, address family IPv4 Unicast
BGP routing table entry for 10.1.255.12/32, version 69
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
L1-2# 
L1-2# sh run bgp

!Command: show running-config bgp
!Running configuration last done at: Sun Oct 18 16:08:56 2026
!Time: Sun Oct 18 16:19:28 2026

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
  template peer peer_LEAF
    bfd
    remote-as 65001
    timers 3 9
    address-family ipv4 unicast
  neighbor 10.1.241.2
    inherit peer peer_LEAF
    description to_S1-1
  neighbor 10.1.242.2
    inherit peer peer_LEAF
    description to_S1-2


L1-2# 
L1-2# sh ip bgp summary 
BGP summary information for VRF default, address family IPv4 Unicast
BGP router identifier 10.1.255.12, local AS number 65001
BGP table version is 67, IPv4 Unicast config peers 2, capable peers 2
8 network entries and 10 paths using 3120 bytes of memory
BGP attribute entries [4/1472], BGP AS path entries [0/0]
BGP community entries [0/0], BGP clusterlist entries [2/8]

Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/
PfxRcd
10.1.241.2      4 65001        786        758       67    0    0 00:09:53 4     
    
10.1.242.2      4 65001        782        759       67    0    0 00:09:50 4     
    
L1-2# 
L1-2# sh  ip route bgp-65001 
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

10.1.254.1/32, ubest/mbest: 1/0
    *via 10.1.241.2, [200/0], 00:09:58, bgp-65001, internal, tag 65001
10.1.254.2/32, ubest/mbest: 1/0
    *via 10.1.242.2, [200/0], 00:09:54, bgp-65001, internal, tag 65001
10.1.254.11/32, ubest/mbest: 2/0
    *via 10.1.241.2, [200/0], 00:09:58, bgp-65001, internal, tag 65001
    *via 10.1.242.2, [200/0], 00:09:52, bgp-65001, internal, tag 65001
10.1.255.1/32, ubest/mbest: 1/0
    *via 10.1.241.2, [200/0], 00:09:58, bgp-65001, internal, tag 65001
10.1.255.2/32, ubest/mbest: 1/0
    *via 10.1.242.2, [200/0], 00:09:54, bgp-65001, internal, tag 65001
10.1.255.11/32, ubest/mbest: 2/0
    *via 10.1.241.2, [200/0], 00:09:58, bgp-65001, internal, tag 65001
    *via 10.1.242.2, [200/0], 00:09:52, bgp-65001, internal, tag 65001

L1-2# 
L1-2# sh  ip bgp all 
BGP routing table information for VRF default, address family IPv4 Unicast
BGP table version is 67, Local Router ID is 10.1.255.12
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-i
njected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - b
est2

   Network            Next Hop            Metric     LocPrf     Weight Path
*>i10.1.254.1/32      10.1.241.2                        100          0 i
*>i10.1.254.2/32      10.1.242.2                        100          0 i
*|i10.1.254.11/32     10.1.242.2                        100          0 i
*>i                   10.1.241.2                        100          0 i
*>l10.1.254.12/32     0.0.0.0                           100      32768 i
*>i10.1.255.1/32      10.1.241.2                        100          0 i
*>i10.1.255.2/32      10.1.242.2                        100          0 i
*|i10.1.255.11/32     10.1.242.2                        100          0 i
*>i                   10.1.241.2                        100          0 i
*>l10.1.255.12/32     0.0.0.0                           100      32768 i

L1-2# 
L1-2# sh  ip bgp 10.1.255.11
BGP routing table information for VRF default, address family IPv4 Unicast
BGP routing table entry for 10.1.255.11/32, version 67
Paths: (2 available, best #2)
Flags: (0x8000001a) (high32 00000000) on xmit-list, is in urib, is best urib rou
te, is in HW
Multipath: eBGP iBGP

  Path type: internal, path is valid, not best reason: Neighbor Address, multipa
th, no labeled nexthop, in rib
  AS-Path: NONE, path sourced internal to AS
    10.1.242.2 (metric 0) from 10.1.242.2 (10.1.255.2)
      Origin IGP, MED not set, localpref 100, weight 0
      Originator: 10.1.255.11 Cluster list: 10.1.255.2 

  Advertised path-id 1
  Path type: internal, path is valid, is best path, no labeled nexthop, in rib
  AS-Path: NONE, path sourced internal to AS
    10.1.241.2 (metric 0) from 10.1.241.2 (10.1.255.1)
      Origin IGP, MED not set, localpref 100, weight 0
      Originator: 10.1.255.11 Cluster list: 10.1.255.1 

  Path-id 1 not advertised to any peer

L1-2# 
```
