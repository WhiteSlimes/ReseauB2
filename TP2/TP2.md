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

Topologie
```
+-----+        +-------+        +-----+
| PC1 +--------+  IOU1 +--------+ PC2 |
+-----+        +-------+        +-----+
```
Plan d'adressage

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

Topologie
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
Plan d'adressage

Machine | `net1`
--- | ---
`PC1` | `10.2.2.1/24`
`PC2` | `10.2.2.2/24`
`PC3` | `10.2.2.3/24`

🌞 Mettre en place la topologie ci-dessus

