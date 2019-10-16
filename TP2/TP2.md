# TP2 : Network low-level, Switching

# Sommaire

* [Intro](#intro)
* [0. Etapes préliminaires](#0-etapes-préliminaires)
* [I. Simplest setup](#i-simplest-setup)
  * [Topologie](#topologie)
  * [Plan d'adressage](#plan-dadressage)
  * [ToDo](#todo)
* [II. More switches](#ii-more-switches)
  * [Topologie](#topologie-1)
  * [Plan d'adressage](#plan-dadressage-1)
  * [ToDo](#todo-1)
  * [Mise en évidence du Spanning Tree Protocol](#mise-en-évidence-du-spanning-tree-protocol)
* [III. Isolation](#iii-isolation)
  * [1. Simple](#1-simple)
    * [Topologie](#topologie-2)
    * [Plan d'adressage](#plan-dadressage-2)
    * [ToDo](#todo-2)
  * [2. Avec trunk](#2-avec-trunk)
    * [Topologie](#topologie-3)
    * [Plan d'adressage](#plan-dadressage-3)
    * [ToDo](#todo-3)
* [IV. Need perfs](#iv-need-perfs)
  * [Topologie](#topologie-4)
  * [Plan d'adressage](#plan-dadressage-4)
  * [ToDo](#todo-4)

# Intro
# 1. Simplest setup

#### Topologie
```
+-----+        +-------+        +-----+
| PC1 +--------+  IOU1 +--------+ PC2 |
+-----+        +-------+        +-----+
```
#### Plan d'adressage

Machine | `net1`
--- | ---
`PC1` | `10.2.1.1/24`
`PC2` | `10.2.1.2/24`

🌞 Mettre en place la topologie ci-dessus.
```
PC1> ip 10.2.1.1/24
```
```
PC2> ip 10.2.1.2/24
```

🌞 Faire communiquer les deux PCs
```
PC1> ping 10.2.1.2
84 bytes from 10.2.1.2 icmp_seq=1 ttl=64 time=0.323 ms
84 bytes from 10.2.1.2 icmp_seq=2 ttl=64 time=0.543 ms
84 bytes from 10.2.1.2 icmp_seq=3 ttl=64 time=0.306 ms
84 bytes from 10.2.1.2 icmp_seq=4 ttl=64 time=0.331 ms
84 bytes from 10.2.1.2 icmp_seq=5 ttl=64 time=0.284 ms
```
```
PC2> ping 10.2.1.1
84 bytes from 10.2.1.1 icmp_seq=1 ttl=64 time=0.801 ms
84 bytes from 10.2.1.1 icmp_seq=2 ttl=64 time=0.329 ms
84 bytes from 10.2.1.1 icmp_seq=3 ttl=64 time=0.474 ms
84 bytes from 10.2.1.1 icmp_seq=4 ttl=64 time=0.287 ms
84 bytes from 10.2.1.1 icmp_seq=5 ttl=64 time=0.426 ms
```
```
PC1> show arp
00:50:79:66:68:01  10.2.1.2 expires in 81 seconds
```
```
PC2> show arp
00:50:79:66:68:00  10.2.1.1 expires in 49 seconds
```

Pour les captures Wireshark j'ai lancé la capture entre le **PC1** et IOU1 et si on ping avec le **PC2** le **PC1** on a donc la capture suivante: 

![alt text](https://github.com/WhiteSlimes/ReseauB2/blob/master/TP2/captures/Capture.png)

# II. More switches

#### Topologie
```
                        +-----+
                        | PC2 |
                        +--+--+
                           |
                           |
                       +---+---+
                   +---+  SW2  +----+
                   |   +-------+    |
                   |                |
                   |                |
+-----+        +---+---+        +---+---+        +-----+
| PC1 +--------+  SW1  +--------+  SW3  +--------+ PC3 |
+-----+        +-------+        +-------+        +-----+
```
#### Plan d'adressage

Machine | `net1`
--- | ---
`PC1` | `10.2.2.1/24`
`PC2` | `10.2.2.2/24`
`PC3` | `10.2.2.3/24`

🌞 Mettre en place la topologie ci-dessus


```
PC1> ip 10.2.2.1/24
PC2> ip 10.2.2.2/24
PC3> ip 10.2.2.3/24
```
🌞 faire communiquer les trois PCs

```
PC1> ping 10.2.2.2
84 bytes from 10.2.2.2 icmp_seq=1 ttl=64 time=0.269 ms
84 bytes from 10.2.2.2 icmp_seq=2 ttl=64 time=0.379 ms
84 bytes from 10.2.2.2 icmp_seq=3 ttl=64 time=0.433 ms
84 bytes from 10.2.2.2 icmp_seq=4 ttl=64 time=0.436 ms
84 bytes from 10.2.2.2 icmp_seq=5 ttl=64 time=0.349 ms

PC1> ping 10.2.2.3
84 bytes from 10.2.2.3 icmp_seq=1 ttl=64 time=0.307 ms
84 bytes from 10.2.2.3 icmp_seq=2 ttl=64 time=0.391 ms
84 bytes from 10.2.2.3 icmp_seq=3 ttl=64 time=0.407 ms
84 bytes from 10.2.2.3 icmp_seq=4 ttl=64 time=0.380 ms
84 bytes from 10.2.2.3 icmp_seq=5 ttl=64 time=0.415 ms
```
```
PC2> ping 10.2.2.1
84 bytes from 10.2.2.1 icmp_seq=1 ttl=64 time=0.259 ms
84 bytes from 10.2.2.1 icmp_seq=2 ttl=64 time=0.366 ms
84 bytes from 10.2.2.1 icmp_seq=3 ttl=64 time=0.442 ms
84 bytes from 10.2.2.1 icmp_seq=4 ttl=64 time=0.373 ms
84 bytes from 10.2.2.1 icmp_seq=5 ttl=64 time=0.387 ms

PC2> ping 10.2.2.3
84 bytes from 10.2.2.3 icmp_seq=1 ttl=64 time=0.385 ms
84 bytes from 10.2.2.3 icmp_seq=2 ttl=64 time=0.539 ms
84 bytes from 10.2.2.3 icmp_seq=3 ttl=64 time=0.516 ms
84 bytes from 10.2.2.3 icmp_seq=4 ttl=64 time=0.583 ms
84 bytes from 10.2.2.3 icmp_seq=5 ttl=64 time=0.466 ms
```
```
PC3> ping 10.2.2.1
84 bytes from 10.2.2.1 icmp_seq=1 ttl=64 time=0.301 ms
84 bytes from 10.2.2.1 icmp_seq=2 ttl=64 time=0.440 ms
84 bytes from 10.2.2.1 icmp_seq=3 ttl=64 time=0.374 ms
84 bytes from 10.2.2.1 icmp_seq=4 ttl=64 time=0.372 ms
84 bytes from 10.2.2.1 icmp_seq=5 ttl=64 time=0.501 ms

PC3> ping 10.2.2.2
84 bytes from 10.2.2.2 icmp_seq=1 ttl=64 time=0.514 ms
84 bytes from 10.2.2.2 icmp_seq=2 ttl=64 time=0.513 ms
84 bytes from 10.2.2.2 icmp_seq=3 ttl=64 time=0.567 ms
84 bytes from 10.2.2.2 icmp_seq=4 ttl=64 time=0.502 ms
84 bytes from 10.2.2.2 icmp_seq=5 ttl=64 time=0.514 ms
```
🌞 analyser la table MAC d'un switch

On rentre dans la console d'un switch et on tape la commande `show mac address-table`

La console nous retourne ceci :
```
          Mac Address Table
-------------------------------------------

Vlan    Mac Address       Type        Ports
----    -----------       --------    -----
   1    0050.7966.6800    DYNAMIC     Et0/0
   1    0050.7966.6801    DYNAMIC     Et0/1
   1    0050.7966.6802    DYNAMIC     Et0/2
   1    aabb.cc00.0210    DYNAMIC     Et0/1
   1    aabb.cc00.0310    DYNAMIC     Et0/1
   1    aabb.cc00.0320    DYNAMIC     Et0/2
Total Mac Addresses for this criterion: 6
```
Les 3 premières adresse MAC sont les 3 différents switch avec les ports sur lequel ils sont connecté et les 3 dernières sont les adresses MAC des différents postes avec le ports qur lequel ils sont connecté au switch.

#### Mise en évidence du Spanning Tree Protocol

🌞 déterminer les informations STP
```
IOU1#show spanning-tree

VLAN0001
  Spanning tree enabled protocol rstp
  Root ID    Priority    32769
             Address     aabb.cc00.0100
             This bridge is the root
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.0100
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Desg FWD 100       128.1    Shr
Et0/1               Desg FWD 100       128.2    Shr
Et0/2               Desg FWD 100       128.3    Shr
Et0/3               Desg FWD 100       128.4    Shr
Et1/0               Desg FWD 100       128.5    Shr
Et1/1               Desg FWD 100       128.6    Shr
Et1/2               Desg FWD 100       128.7    Shr
Et1/3               Desg FWD 100       128.8    Shr
```
```
IOU2#show spanning-tree

VLAN0001
  Spanning tree enabled protocol rstp
  Root ID    Priority    32769
             Address     aabb.cc00.0100
             Cost        100
             Port        2 (Ethernet0/1)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.0200
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Desg FWD 100       128.1    Shr
Et0/1               Root FWD 100       128.2    Shr
Et0/2               Desg FWD 100       128.3    Shr
Et0/3               Desg FWD 100       128.4    Shr
Et1/0               Desg FWD 100       128.5    Shr
Et1/1               Desg FWD 100       128.6    Shr
Et1/2               Desg FWD 100       128.7    Shr
 --More--
```
```
IOU3#show spanning-tree

VLAN0001
  Spanning tree enabled protocol rstp
  Root ID    Priority    32769
             Address     aabb.cc00.0100
             Cost        100
             Port        3 (Ethernet0/2)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.0300
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Desg FWD 100       128.1    Shr
Et0/1               Altn BLK 100       128.2    Shr
Et0/2               Root FWD 100       128.3    Shr
Et0/3               Desg FWD 100       128.4    Shr
Et1/0               Desg FWD 100       128.5    Shr
Et1/1               Desg FWD 100       128.6    Shr
Et1/2               Desg FWD 100       128.7    Shr
 --More--
```
On remarque sur la console de L'IOU1 qu'il y a une ligne supplémentaire avec écrit ``This root is a bridge``

# III. Isolation

## 1. Simple
 
#### Topologie
```
+-----+        +-------+        +-----+
| PC1 +--------+  SW1  +--------+ PC3 |
+-----+      10+-------+20      +-----+
                 20|
                   |
                +--+--+
                | PC2 |
                +-----+
```

#### Plan d'adressage

Machine | IP `net1` | VLAN
--- | --- | --- 
`PC1` | `10.2.3.1/24` | 10
`PC2` | `10.2.3.2/24` | 20
`PC3` | `10.2.3.3/24` | 20

Mise en place la topologie ci-dessus.
```
PC1> ip 10.2.3.1/24
PC2> ip 10.2.3.2/24
PC3> ip 10.2.3.3/24
```

```
IOU1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
IOU1(config)#vlan 10
IOU1(config-vlan)#name pc1-iou
IOU1(config-vlan)#exit
IOU1(config)#interface Ethernet 0/0
IOU1(config-if)#switchport mode access
IOU1(config-if)#switchport access vlan 10
IOU1(config-if)#exit
IOU1(config)#vlan 20
IOU1(config-vlan)# name pc2-iou-pc3
IOU1(config-vlan)#exit
IOU1(config)#interface Ethernet 0/1
IOU1(config-if)#switchport mode access
IOU1(config-if)#switchport access vlan 20
IOU1(config-if)#exit
IOU1(config)#interface Ethernet 0/2
IOU1(config-if)#switchport mode access
IOU1(config-if)#switchport access vlan 20
IOU1(config-if)#exit
```

```
IOU1#show vlan br

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Et0/3, Et1/0, Et1/1, Et1/2
                                                Et1/3, Et2/0, Et2/1, Et2/2
                                                Et2/3, Et3/0, Et3/1, Et3/2
                                                Et3/3
10   pc1-iou                          active    Et0/0
20   pc2-iou-pc3                      active    Et0/1, Et0/2
1002 fddi-default                     act/unsup
1003 token-ring-default               act/unsup
1004 fddinet-default                  act/unsup
1005 trnet-default                    act/unsup
```

Vérifier que `PC2` ne peut joindre que `PC3`.
```
PC2> ping 10.2.3.3
84 bytes from 10.2.3.3 icmp_seq=1 ttl=64 time=0.167 ms
84 bytes from 10.2.3.3 icmp_seq=2 ttl=64 time=0.277 ms
84 bytes from 10.2.3.3 icmp_seq=3 ttl=64 time=0.279 ms
84 bytes from 10.2.3.3 icmp_seq=4 ttl=64 time=0.314 ms
84 bytes from 10.2.3.3 icmp_seq=5 ttl=64 time=0.270 ms

PC2> ping 10.2.3.1
host (10.2.3.1) not reachable
```

Vérifier que `PC1` ne peut joindre personne alors qu'il est dans le même réseau.
```
PC1> ping 10.2.3.2
host (10.2.3.2) not reachable

PC1> ping 10.2.3.3
host (10.2.3.3) not reachable
```
