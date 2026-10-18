# Домашнее задание
Underlay. IS-IS

## Цель:
- настроить IS-IS для Underlay сети.

## Описание/Пошаговая инструкция выполнения домашнего задания:
В этой самостоятельной работе мы ожидаем, что вы самостоятельно:
- Настроите ISIS в Underlay сети, для IP связанности между всеми сетевыми устройствами.
- Зафиксируете в документации - план работы, адресное пространство, схему сети, конфигурацию устройств
- Убедитесь в наличии IP связанности между устройствами в ISIS домене

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
S1-1# sh run isis 

!Command: show running-config isis
!Running configuration last done at: Sun Oct 18 13:29:51 2026
!Time: Sun Oct 18 13:30:01 2026

version 10.5(3) Bios:version  
feature isis

router isis UNDERLAY
  net 49.0001.0000.0000.0001.00
  is-type level-2
  address-family ipv4 unicast
    maximum-paths 64

interface loopback0
  ip router isis UNDERLAY

interface loopback1
  ip router isis UNDERLAY

interface Ethernet1/1
  isis network point-to-point
  ip router isis UNDERLAY

interface Ethernet1/2
  isis network point-to-point
  ip router isis UNDERLAY


S1-1#  sh isis adjacency 
IS-IS process: UNDERLAY VRF: default
IS-IS adjacency database:
Legend: '!': No AF level connectivity in given topology
System ID       SNPA            Level  State  Hold Time  Interface
L1-1            N/A             2      UP     00:00:28   Ethernet1/1
L1-2            N/A             2      UP     00:00:26   Ethernet1/2

S1-1# 
S1-1#  sh isis interface brief 
IS-IS process: UNDERLAY VRF: default
Interface    Type  Idx State        Circuit   MTU  Metric  Priority  Adjs/AdjsUp
                                                   L1  L2  L1  L2    L1    L2  
--------------------------------------------------------------------------------
Topology: TopoID: 0
loopback0    Loop  4     Up/Ready   0x01/L2   1500 1   1   64  64    0/0   0/0 
Topology: TopoID: 0
loopback1    Loop  3     Up/Ready   0x01/L2   1500 1   1   64  64    0/0   0/0 
Topology: TopoID: 0
Ethernet1/1  P2P   1     Up/Ready   0x01/L2   1500 40  40  64  64    0/0   1/1 
Topology: TopoID: 0
Ethernet1/2  P2P   2     Up/Ready   0x01/L2   1500 40  40  64  64    0/0   1/1 

S1-1# 
S1-1#  sh isis database 
IS-IS Process: UNDERLAY LSP database VRF: default
IS-IS Level-1 Link State Database
  LSPID                 Seq Number   Checksum  Lifetime   A/P/O/T

IS-IS Level-2 Link State Database
  LSPID                 Seq Number   Checksum  Lifetime   A/P/O/T
  S1-1.00-00          * 0x00000007   0x9B96    1139       0/0/0/3
  S1-2.00-00            0x00000005   0x5EC9    1135       0/0/0/3
  L1-1.00-00            0x00000005   0x8C99    1078       0/0/0/3
  L1-2.00-00            0x00000005   0x24EF    1122       0/0/0/3

S1-1# 
S1-1#  sh routing isis-UNDERLAY 
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

10.1.242.0/31, ubest/mbest: 1/0
    *via 10.1.241.1, Eth1/1, [115/80], 00:10:44, isis-UNDERLAY, L2
10.1.242.2/31, ubest/mbest: 1/0
    *via 10.1.241.3, Eth1/2, [115/80], 00:10:14, isis-UNDERLAY, L2
10.1.254.2/32, ubest/mbest: 2/0
    *via 10.1.241.1, Eth1/1, [115/81], 00:10:43, isis-UNDERLAY, L2
    *via 10.1.241.3, Eth1/2, [115/81], 00:10:14, isis-UNDERLAY, L2
10.1.254.11/32, ubest/mbest: 1/0
    *via 10.1.241.1, Eth1/1, [115/41], 00:10:44, isis-UNDERLAY, L2
10.1.254.12/32, ubest/mbest: 1/0
    *via 10.1.241.3, Eth1/2, [115/41], 00:10:14, isis-UNDERLAY, L2
10.1.255.2/32, ubest/mbest: 2/0
    *via 10.1.241.1, Eth1/1, [115/81], 00:10:43, isis-UNDERLAY, L2
    *via 10.1.241.3, Eth1/2, [115/81], 00:10:14, isis-UNDERLAY, L2
10.1.255.11/32, ubest/mbest: 1/0
    *via 10.1.241.1, Eth1/1, [115/41], 00:10:44, isis-UNDERLAY, L2
10.1.255.12/32, ubest/mbest: 1/0
    *via 10.1.241.3, Eth1/2, [115/41], 00:10:14, isis-UNDERLAY, L2
```

### S1-2
```
S1-2#  sh run isis

!Command: show running-config isis
!Running configuration last done at: Sun Oct 18 13:33:22 2026
!Time: Sun Oct 18 13:33:44 2026

version 10.5(3) Bios:version  
feature isis

router isis UNDERLAY
  net 49.0001.0000.0000.0002.00
  is-type level-2
  address-family ipv4 unicast
    maximum-paths 64

interface loopback0
  ip router isis UNDERLAY

interface loopback1
  ip router isis UNDERLAY

interface Ethernet1/1
  isis network point-to-point
  ip router isis UNDERLAY

interface Ethernet1/2
  isis network point-to-point
  ip router isis UNDERLAY


S1-2#  sh isis adjacency 
IS-IS process: UNDERLAY VRF: default
IS-IS adjacency database:
Legend: '!': No AF level connectivity in given topology
System ID       SNPA            Level  State  Hold Time  Interface
L1-1            N/A             2      UP     00:00:23   Ethernet1/1
L1-2            N/A             2      UP     00:00:28   Ethernet1/2

S1-2#  
S1-2#  sh isis interface brief 
IS-IS process: UNDERLAY VRF: default
Interface    Type  Idx State        Circuit   MTU  Metric  Priority  Adjs/AdjsUp
                                                   L1  L2  L1  L2    L1    L2  
--------------------------------------------------------------------------------
Topology: TopoID: 0
loopback0    Loop  1     Up/Ready   0x01/L2   1500 1   1   64  64    0/0   0/0 
Topology: TopoID: 0
loopback1    Loop  2     Up/Ready   0x01/L2   1500 1   1   64  64    0/0   0/0 
Topology: TopoID: 0
Ethernet1/1  P2P   3     Up/Ready   0x01/L2   1500 40  40  64  64    0/0   1/1 
Topology: TopoID: 0
Ethernet1/2  P2P   4     Up/Ready   0x01/L2   1500 40  40  64  64    0/0   1/1 

S1-2# 
S1-2#  sh isis database 
IS-IS Process: UNDERLAY LSP database VRF: default
IS-IS Level-1 Link State Database
  LSPID                 Seq Number   Checksum  Lifetime   A/P/O/T

IS-IS Level-2 Link State Database
  LSPID                 Seq Number   Checksum  Lifetime   A/P/O/T
  S1-1.00-00            0x00000007   0x9B96    926        0/0/0/3
  S1-2.00-00          * 0x00000005   0x5EC9    925        0/0/0/3
  L1-1.00-00            0x00000005   0x8C99    866        0/0/0/3
  L1-2.00-00            0x00000005   0x24EF    910        0/0/0/3

S1-2# 
S1-2#  sh routing isis-UNDERLAY 
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

10.1.241.0/31, ubest/mbest: 1/0
    *via 10.1.242.1, Eth1/1, [115/80], 00:14:13, isis-UNDERLAY, L2
10.1.241.2/31, ubest/mbest: 1/0
    *via 10.1.242.3, Eth1/2, [115/80], 00:13:50, isis-UNDERLAY, L2
10.1.254.1/32, ubest/mbest: 2/0
    *via 10.1.242.1, Eth1/1, [115/81], 00:14:08, isis-UNDERLAY, L2
    *via 10.1.242.3, Eth1/2, [115/81], 00:13:37, isis-UNDERLAY, L2
10.1.254.11/32, ubest/mbest: 1/0
    *via 10.1.242.1, Eth1/1, [115/41], 00:14:13, isis-UNDERLAY, L2
10.1.254.12/32, ubest/mbest: 1/0
    *via 10.1.242.3, Eth1/2, [115/41], 00:13:50, isis-UNDERLAY, L2
10.1.255.1/32, ubest/mbest: 2/0
    *via 10.1.242.1, Eth1/1, [115/81], 00:14:08, isis-UNDERLAY, L2
    *via 10.1.242.3, Eth1/2, [115/81], 00:13:37, isis-UNDERLAY, L2
10.1.255.11/32, ubest/mbest: 1/0
    *via 10.1.242.1, Eth1/1, [115/41], 00:14:13, isis-UNDERLAY, L2
10.1.255.12/32, ubest/mbest: 1/0
    *via 10.1.242.3, Eth1/2, [115/41], 00:13:50, isis-UNDERLAY, L2

S1-2# 
```

### L1-1
```
L1-1#  sh run isis 

!Command: show running-config isis
!Running configuration last done at: Sun Oct 18 13:34:38 2026
!Time: Sun Oct 18 13:34:45 2026

version 10.5(3) Bios:version  
feature isis

router isis UNDERLAY
  net 49.0001.0000.0000.0011.00
  is-type level-2
  address-family ipv4 unicast
    maximum-paths 64

interface loopback0
  ip router isis UNDERLAY

interface loopback1
  ip router isis UNDERLAY

interface Ethernet1/1
  isis network point-to-point
  ip router isis UNDERLAY

interface Ethernet1/2
  isis network point-to-point
  ip router isis UNDERLAY


L1-1#  sh isis adjacency 
IS-IS process: UNDERLAY VRF: default
IS-IS adjacency database:
Legend: '!': No AF level connectivity in given topology
System ID       SNPA            Level  State  Hold Time  Interface
S1-1            N/A             2      UP     00:00:24   Ethernet1/1
S1-2            N/A             2      UP     00:00:25   Ethernet1/2

L1-1#   sh isis interface brief 
IS-IS process: UNDERLAY VRF: default
Interface    Type  Idx State        Circuit   MTU  Metric  Priority  Adjs/AdjsUp
                                                   L1  L2  L1  L2    L1    L2  
--------------------------------------------------------------------------------
Topology: TopoID: 0
loopback0    Loop  1     Up/Ready   0x01/L2   1500 1   1   64  64    0/0   0/0 
Topology: TopoID: 0
loopback1    Loop  2     Up/Ready   0x01/L2   1500 1   1   64  64    0/0   0/0 
Topology: TopoID: 0
Ethernet1/1  P2P   3     Up/Ready   0x01/L2   1500 40  40  64  64    0/0   1/1 
Topology: TopoID: 0
Ethernet1/2  P2P   4     Up/Ready   0x01/L2   1500 40  40  64  64    0/0   1/1 

L1-1#   sh isis database 
IS-IS Process: UNDERLAY LSP database VRF: default
IS-IS Level-1 Link State Database
  LSPID                 Seq Number   Checksum  Lifetime   A/P/O/T

IS-IS Level-2 Link State Database
  LSPID                 Seq Number   Checksum  Lifetime   A/P/O/T
  S1-1.00-00            0x00000007   0x9B96    864        0/0/0/3
  S1-2.00-00            0x00000005   0x5EC9    862        0/0/0/3
  L1-1.00-00          * 0x00000005   0x8C99    805        0/0/0/3
  L1-2.00-00            0x00000005   0x24EF    847        0/0/0/3

L1-1#   sh routing isis-UNDERLAY 
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

10.1.241.2/31, ubest/mbest: 1/0
    *via 10.1.241.0, Eth1/1, [115/80], 00:15:11, isis-UNDERLAY, L2
10.1.242.2/31, ubest/mbest: 1/0
    *via 10.1.242.0, Eth1/2, [115/80], 00:15:15, isis-UNDERLAY, L2
10.1.254.1/32, ubest/mbest: 1/0
    *via 10.1.241.0, Eth1/1, [115/41], 00:15:11, isis-UNDERLAY, L2
10.1.254.2/32, ubest/mbest: 1/0
    *via 10.1.242.0, Eth1/2, [115/41], 00:15:15, isis-UNDERLAY, L2
10.1.254.12/32, ubest/mbest: 2/0
    *via 10.1.241.0, Eth1/1, [115/81], 00:14:41, isis-UNDERLAY, L2
    *via 10.1.242.0, Eth1/2, [115/81], 00:14:57, isis-UNDERLAY, L2
10.1.255.1/32, ubest/mbest: 1/0
    *via 10.1.241.0, Eth1/1, [115/41], 00:15:11, isis-UNDERLAY, L2
10.1.255.2/32, ubest/mbest: 1/0
    *via 10.1.242.0, Eth1/2, [115/41], 00:15:15, isis-UNDERLAY, L2
10.1.255.12/32, ubest/mbest: 2/0
    *via 10.1.241.0, Eth1/1, [115/81], 00:14:41, isis-UNDERLAY, L2
    *via 10.1.242.0, Eth1/2, [115/81], 00:14:57, isis-UNDERLAY, L2

L1-1#  
L1-1# ping 10.1.255.12 source 10.1.255.11
PING 10.1.255.12 (10.1.255.12) from 10.1.255.11: 56 data bytes
64 bytes from 10.1.255.12: icmp_seq=0 ttl=253 time=14.008 ms
64 bytes from 10.1.255.12: icmp_seq=1 ttl=253 time=13.477 ms
64 bytes from 10.1.255.12: icmp_seq=2 ttl=253 time=14.005 ms
64 bytes from 10.1.255.12: icmp_seq=3 ttl=253 time=10.919 ms
64 bytes from 10.1.255.12: icmp_seq=4 ttl=253 time=12.026 ms

--- 10.1.255.12 ping statistics ---
5 packets transmitted, 5 packets received, 0.00% packet loss
round-trip min/avg/max = 10.919/12.887/14.008 ms
L1-1# 
```

### L1-2
```
L1-2#  sh run isis 

!Command: show running-config isis
!Running configuration last done at: Sun Oct 18 13:36:09 2026
!Time: Sun Oct 18 13:36:17 2026

version 10.5(3) Bios:version  
feature isis

router isis UNDERLAY
  net 49.0001.0000.0000.0012.00
  is-type level-2
  address-family ipv4 unicast
    maximum-paths 64

interface loopback0
  ip router isis UNDERLAY

interface loopback1
  ip router isis UNDERLAY

interface Ethernet1/1
  isis network point-to-point
  ip router isis UNDERLAY

interface Ethernet1/2
  isis network point-to-point
  ip router isis UNDERLAY


L1-2#   sh isis adjacency 
IS-IS process: UNDERLAY VRF: default
IS-IS adjacency database:
Legend: '!': No AF level connectivity in given topology
System ID       SNPA            Level  State  Hold Time  Interface
S1-1            N/A             2      UP     00:00:23   Ethernet1/1
S1-2            N/A             2      UP     00:00:28   Ethernet1/2

L1-2# 
L1-2#  sh isis interface brief 
IS-IS process: UNDERLAY VRF: default
Interface    Type  Idx State        Circuit   MTU  Metric  Priority  Adjs/AdjsUp
                                                   L1  L2  L1  L2    L1    L2  
--------------------------------------------------------------------------------
Topology: TopoID: 0
loopback0    Loop  1     Up/Ready   0x01/L2   1500 1   1   64  64    0/0   0/0 
Topology: TopoID: 0
loopback1    Loop  2     Up/Ready   0x01/L2   1500 1   1   64  64    0/0   0/0 
Topology: TopoID: 0
Ethernet1/1  P2P   3     Up/Ready   0x01/L2   1500 40  40  64  64    0/0   1/1 
Topology: TopoID: 0
Ethernet1/2  P2P   4     Up/Ready   0x01/L2   1500 40  40  64  64    0/0   1/1 

L1-2#  
L1-2#  sh isis database 
IS-IS Process: UNDERLAY LSP database VRF: default
IS-IS Level-1 Link State Database
  LSPID                 Seq Number   Checksum  Lifetime   A/P/O/T

IS-IS Level-2 Link State Database
  LSPID                 Seq Number   Checksum  Lifetime   A/P/O/T
  S1-1.00-00            0x00000007   0x9B96    775        0/0/0/3
  S1-2.00-00            0x00000005   0x5EC9    773        0/0/0/3
  L1-1.00-00            0x00000005   0x8C99    713        0/0/0/3
  L1-2.00-00          * 0x00000005   0x24EF    760        0/0/0/3

L1-2#  
L1-2#  sh routing isis-UNDERLAY 
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

10.1.241.0/31, ubest/mbest: 1/0
    *via 10.1.241.2, Eth1/1, [115/80], 00:16:09, isis-UNDERLAY, L2
10.1.242.0/31, ubest/mbest: 1/0
    *via 10.1.242.2, Eth1/2, [115/80], 00:16:26, isis-UNDERLAY, L2
10.1.254.1/32, ubest/mbest: 1/0
    *via 10.1.241.2, Eth1/1, [115/41], 00:16:09, isis-UNDERLAY, L2
10.1.254.2/32, ubest/mbest: 1/0
    *via 10.1.242.2, Eth1/2, [115/41], 00:16:26, isis-UNDERLAY, L2
10.1.254.11/32, ubest/mbest: 2/0
    *via 10.1.241.2, Eth1/1, [115/81], 00:16:09, isis-UNDERLAY, L2
    *via 10.1.242.2, Eth1/2, [115/81], 00:16:24, isis-UNDERLAY, L2
10.1.255.1/32, ubest/mbest: 1/0
    *via 10.1.241.2, Eth1/1, [115/41], 00:16:09, isis-UNDERLAY, L2
10.1.255.2/32, ubest/mbest: 1/0
    *via 10.1.242.2, Eth1/2, [115/41], 00:16:26, isis-UNDERLAY, L2
10.1.255.11/32, ubest/mbest: 2/0
    *via 10.1.241.2, Eth1/1, [115/81], 00:16:09, isis-UNDERLAY, L2
    *via 10.1.242.2, Eth1/2, [115/81], 00:16:24, isis-UNDERLAY, L2

L1-2# 
L1-2# ping 10.1.254.11 source 10.1.254.12
PING 10.1.254.11 (10.1.254.11) from 10.1.254.12: 56 data bytes
64 bytes from 10.1.254.11: icmp_seq=0 ttl=253 time=14.051 ms
64 bytes from 10.1.254.11: icmp_seq=1 ttl=253 time=12.626 ms
64 bytes from 10.1.254.11: icmp_seq=2 ttl=253 time=14.476 ms
64 bytes from 10.1.254.11: icmp_seq=3 ttl=253 time=18.826 ms
64 bytes from 10.1.254.11: icmp_seq=4 ttl=253 time=12.875 ms

--- 10.1.254.11 ping statistics ---
5 packets transmitted, 5 packets received, 0.00% packet loss
round-trip min/avg/max = 12.626/14.57/18.826 ms
L1-2# 
```
