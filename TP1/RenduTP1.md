#TP 1 : Back to basics

#Sommaire

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


