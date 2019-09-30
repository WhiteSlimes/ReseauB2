# TP 1 : Back to basics

# Sommaire

* [I. Gather informations](#i-gather-informations)
* [II. Edit configurtion](#ii-edit-configuration)
  * [1. Configuration cartes réseau](#1-configuration-cartes-réseau)
  * [2. Serveur SSH](#2-serveur-ssh)
* [III. Routage simple](#iii-routage-simple)
* [IV. Autres applications et métrologie](#iv-autres-applications-et-métrologie)
  * [1. Commandes](#1-commandes)
  * [2. Cockpit](#2-cockpit)
  * [3. Netdata](#3-netdata)

# I. Gather informations
🌞 Pour récupérer une **liste des cartes réseau** on utilise la commande `nmcli` qui nous affiche donc toutes les cartes réseau avec leur IP, etc ...
 ```[root@localhost ~]# nmcli
    enp0s3: connected to enp0s3
            "Intel 82540EM"
            ethernet (e1000), 08:00:27:67:2E:F7, hw, mtu 1500
            ip4 default
            inet4 10.0.2.15/24
            route4 0.0.0.0/0
            route4 10.0.2.0/24
            inet6 fe80::8dad:52af:ea13:1766/64
            route6 fe80::/64
            route6 ff00::/8
    
    enp0s8: connected to Wired connection 1
            "Intel 82540EM"
            ethernet (e1000), 08:00:27:A0:1F:7F, hw, mtu 1500
            inet4 192.168.2.3/24
            route4 192.168.2.0/24
            inet6 fe80::d03f:e1d4:7c2d:4204/64
            route6 fe80::/64
            route6 ff00::/8
```
🌞 Pour déterminez si les cartes on récupéré une **IP en DHCP**, on se rend dans ``etc\sysconfig\network-scripts`` et on ouvre le fichier **ifcfg-enp0s3** à l'aide de la commande `vi`.
Et on obtient ceci :
```TYPE="Ethernet"
   PROXY_METHOD="none"
   BROWSER_ONLY="no"
   BOOTPROTO="dhcp"
   DEFROUTE="yes"
   IPV4_FAILURE_FATAL="no"
   IPV6INIT="yes"
   IPV6_AUTOCONF="yes"
   IPV6_DEFROUTE="yes"
   IPV6_FAILURE_FATAL="no"
   IPV6_ADDR_GEN_MODE="stable-privacy"
   NAME="enp0s3"
   UUID="0d176d65-d83f-4da7-a7f2-108337c262ef"
   DEVICE="enp0s3"
   ONBOOT="yes"
   ~              
```
On remarque bien le **BOOTPROTO** en **dhcp**.

Voici le bail **DHCP** de enp0s3:
```[root@localhost NetworkManager]# nmcli -f DHCP4 con show enp0s3
   DHCP4.OPTION[1]:                        domain_name = auvence.co
   DHCP4.OPTION[2]:                        domain_name_servers = 10.33.10.20 10.33.10.2 8.8.8.8 8.8.4.4
   DHCP4.OPTION[3]:                        expiry = 1569569326
   DHCP4.OPTION[4]:                        ip_address = 10.0.2.15
   DHCP4.OPTION[5]:                        requested_broadcast_address = 1
   DHCP4.OPTION[6]:                        requested_dhcp_server_identifier = 1
   DHCP4.OPTION[7]:                        requested_domain_name = 1
   DHCP4.OPTION[8]:                        requested_domain_name_servers = 1
   DHCP4.OPTION[9]:                        requested_domain_search = 1
   DHCP4.OPTION[10]:                       requested_host_name = 1
   DHCP4.OPTION[11]:                       requested_interface_mtu = 1
   DHCP4.OPTION[12]:                       requested_ms_classless_static_routes = 1
   DHCP4.OPTION[13]:                       requested_nis_domain = 1
   DHCP4.OPTION[14]:                       requested_nis_servers = 1
   DHCP4.OPTION[15]:                       requested_ntp_servers = 1
   DHCP4.OPTION[16]:                       requested_rfc3442_classless_static_routes = 1
   DHCP4.OPTION[17]:                       requested_routers = 1
   DHCP4.OPTION[18]:                       requested_static_routes = 1
   DHCP4.OPTION[19]:                       requested_subnet_mask = 1
   DHCP4.OPTION[20]:                       requested_time_offset = 1
   DHCP4.OPTION[21]:                       requested_wpad = 1
   DHCP4.OPTION[22]:                       routers = 10.0.2.2
   DHCP4.OPTION[23]:                       subnet_mask = 255.255.255.0
```

🌞 Pour afficher la **table de routage** on utilise la commande `ip route` et on obtient la **table de routage** :
```
[root@localhost network-scripts]# ip route
   default via 10.0.2.2 dev enp0s3 proto dhcp metric 100
   10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.15 metric 100
   192.168.2.0/24 dev enp0s8 proto kernel scope link src 192.168.2.3 metric 101
``` 
`10.0.2.0`, cette route est celle de la carte **NAT**, elle est utilisée pour une connexion **externe**,  la passerelle de cette route est à l'IP 10.0.2.2 et cette IP est portée par 10.0.2.15.

`192.168.2.0`, cette route est celle du **Host Only**, elle est utilisée pour une connexion **local**, la passerelle de cette route est à l'IP 192.168.2.2 et cette IP est portée par 192.168.2.3.


  Pour afficher la **table ARP** on utilise la commande `ip neigh` qui nous retourne directement la **table ARP**
```
[root@localhost network-scripts]# ip neigh
   192.168.2.2 dev enp0s8 lladdr 08:00:27:9e:7d:39 STALE
   192.168.2.1 dev enp0s8 lladdr 0a:00:27:00:00:12 REACHABLE
   10.0.2.2 dev enp0s3 lladdr 52:54:00:12:35:02 STALE
```
🌞 Pour récupérer **la liste des ports en écoute** on éxécute la commande `ss -t` qui nous retourne la liste des ports **TCP** :
```
[root@localhost ~]# ss -t
   State         Recv-Q          Send-Q                    Local Address:Port                   Peer Address:Port
   ESTAB         0               36                          192.168.2.3:ssh                     192.168.2.1:51838
```
Et pour les ports **UDP** `ss -u` qui retourne tous des applications qui utilisent les ports **UDP**.

Cette commande ne retourne rien car il n'y a aucune application qui utilise les ports **UDP**

🌞 Pour récupérer **la liste des DNS utilisés par la machine** on éxécute la commande `cat /etc/resolv.conf` et il nous affiche la listes des **DNS** utilisés :
```
[root@localhost etc]# cat resolv.conf
# Generated by NetworkManager
search auvence.co
nameserver 10.33.10.20
nameserver 10.33.10.2
nameserver 8.8.8.8
```
J'installe **bind-utils** avec la commande `yum install bind-utils` puis je fais la requête vers `www.reddit.com` grâce à la commande : `dig www.reddit.com`
```
[root@localhost admin]# dig www.reddit.com

; <<>> DiG 9.11.4-P2-RedHat-9.11.4-17.P2.el8_0.1 <<>> www.reddit.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 7288
;; flags: qr rd ra; QUERY: 1, ANSWER: 5, AUTHORITY: 13, ADDITIONAL: 27

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;www.reddit.com.                        IN      A

;; ANSWER SECTION:
www.reddit.com.         16      IN      CNAME   reddit.map.fastly.net.
reddit.map.fastly.net.  29      IN      A       151.101.65.140
reddit.map.fastly.net.  29      IN      A       151.101.193.140
reddit.map.fastly.net.  29      IN      A       151.101.129.140
reddit.map.fastly.net.  29      IN      A       151.101.1.140

;; AUTHORITY SECTION:
net.                    164437  IN      NS      a.gtld-servers.net.
net.                    164437  IN      NS      d.gtld-servers.net.
net.                    164437  IN      NS      m.gtld-servers.net.
net.                    164437  IN      NS      e.gtld-servers.net.
net.                    164437  IN      NS      k.gtld-servers.net.
net.                    164437  IN      NS      b.gtld-servers.net.
net.                    164437  IN      NS      g.gtld-servers.net.
net.                    164437  IN      NS      j.gtld-servers.net.
net.                    164437  IN      NS      i.gtld-servers.net.
net.                    164437  IN      NS      h.gtld-servers.net.
net.                    164437  IN      NS      l.gtld-servers.net.
net.                    164437  IN      NS      c.gtld-servers.net.
net.                    164437  IN      NS      f.gtld-servers.net.

;; ADDITIONAL SECTION:
a.gtld-servers.net.     164437  IN      A       192.5.6.30
a.gtld-servers.net.     164437  IN      AAAA    2001:503:a83e::2:30
b.gtld-servers.net.     164437  IN      A       192.33.14.30
b.gtld-servers.net.     164437  IN      AAAA    2001:503:231d::2:30
c.gtld-servers.net.     164437  IN      A       192.26.92.30
c.gtld-servers.net.     164437  IN      AAAA    2001:503:83eb::30
d.gtld-servers.net.     164437  IN      A       192.31.80.30
d.gtld-servers.net.     164437  IN      AAAA    2001:500:856e::30
e.gtld-servers.net.     164437  IN      A       192.12.94.30
e.gtld-servers.net.     164437  IN      AAAA    2001:502:1ca1::30
f.gtld-servers.net.     164437  IN      A       192.35.51.30
f.gtld-servers.net.     164437  IN      AAAA    2001:503:d414::30
g.gtld-servers.net.     164437  IN      A       192.42.93.30
g.gtld-servers.net.     164437  IN      AAAA    2001:503:eea3::30
h.gtld-servers.net.     164437  IN      A       192.54.112.30
h.gtld-servers.net.     164437  IN      AAAA    2001:502:8cc::30
i.gtld-servers.net.     164437  IN      A       192.43.172.30
i.gtld-servers.net.     164437  IN      AAAA    2001:503:39c1::30
j.gtld-servers.net.     164437  IN      A       192.48.79.30
j.gtld-servers.net.     164437  IN      AAAA    2001:502:7094::30
k.gtld-servers.net.     164437  IN      A       192.52.178.30
k.gtld-servers.net.     164437  IN      AAAA    2001:503:d2d::30
l.gtld-servers.net.     164437  IN      A       192.41.162.30
l.gtld-servers.net.     164437  IN      AAAA    2001:500:d937::30
m.gtld-servers.net.     164437  IN      A       192.55.83.30
m.gtld-servers.net.     164437  IN      AAAA    2001:501:b1f9::30

;; Query time: 78 msec
;; SERVER: 10.33.10.20#53(10.33.10.20)
;; WHEN: Mon Sep 30 14:10:18 CEST 2019
;; MSG SIZE  rcvd: 935
```
Il utilise le Server en **10.33.10.20** ce qui est correcte.

🌞 Pour afficher l'état actuel du **firewall** on rentre la commande `systemctl status firewalld` qui nous retourne :
```
[root@localhost admin]# systemctl status firewalld
● firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; enabled; vendor preset: enabled)
   Active: active (running) since Mon 2019-09-30 14:02:18 CEST; 15min ago
     Docs: man:firewalld(1)
 Main PID: 737 (firewalld)
    Tasks: 2 (limit: 5062)
   Memory: 33.9M
   CGroup: /system.slice/firewalld.service
           └─737 /usr/libexec/platform-python -s /usr/sbin/firewalld --nofork --nopid

Sep 30 14:02:16 localhost.localdomain systemd[1]: Starting firewalld - dynamic firewall daemon...
Sep 30 14:02:18 localhost.localdomain systemd[1]: Started firewalld - dynamic firewall daemon.
```
Pour afficher les interfaces filtrer on entre la commande `firewall-cmd --list-interfaces` et qui retourne la liste des interfaces comme-ci dessous :
```
[root@localhost admin]# firewall-cmd --list-interfaces
enp0s3 enp0s8
```

# II. Edit Configuration
 1. Configuration cartes réseau
 On commence par définir une adresse IP statique toute neuve avec : `sudo vim /etc/sysconfig/network-scripts/ifcfg-enp0s8` et on modifie le **BOOTPROTO** en **static** et on définit une **adresse ip static** :
 ```
TYPE=Ethernet
BOOTPROTO=static
NAME=enp0s8
DEVICE=enp0s8
UUID=6aecd646-fccc-3945-9877-2d19731ca4c8
ONBOOT=yes


IPADDR=192.168.2.89
NETMASK=255.255.255.0
```
On refresh : `sudo ifdown enp0s8` et `sudo ifup enp0s8`

J'ai créer une nouvelle interface **Host Only** qui porte l'adresse ip : `192.168.157.1`

Voici la config de la nouvelle carte réseau :
```
TYPE=Ethernet
BOOTPROTO=static

IPADDR=192.168.157.89
NETMASK=255.255.255.0
ONBOOT=yes
```
**Les nouvelles cartes IP** :
```
[root@localhost network-scripts]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:67:2e:f7 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic noprefixroute enp0s3
       valid_lft 85791sec preferred_lft 85791sec
    inet6 fe80::8dad:52af:ea13:1766/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:a0:1f:7f brd ff:ff:ff:ff:ff:ff
    inet 192.168.2.89/24 brd 192.168.2.255 scope global noprefixroute enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fea0:1f7f/64 scope link
       valid_lft forever preferred_lft forever
4: enp0s9: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:09:45:8e brd ff:ff:ff:ff:ff:ff
    inet 192.168.157.89/24 brd 192.168.157.255 scope global noprefixroute enp0s9
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe09:458e/64 scope link
       valid_lft forever preferred_lft forever
```
**Table ARP** :
```
[root@localhost network-scripts]# ip neigh
192.168.2.1 dev enp0s8 lladdr 0a:00:27:00:00:13 DELAY
10.0.2.2 dev enp0s3 lladdr 52:54:00:12:35:02 STALE
```

