# TP1 réseau

## 1 Mise en place

### Création de la première machine virtuelle
#### Configuration virutalbox:
 - file > host network manager
 - create virtual network
-  configure Adapter Manually, name: vboxnet1, IPv4 address: 10.1.1.1, IPv4 address: 255.255.255.0, no DHCP Server.
-   configure Adapter Manually, name: vboxnet2, IPv4 address: 10.1.2.1, IPv4 network mask: 255.255.255.252, no DHCP Server.

-   combien y a-t-il d'adresses disponibles dans un  `/24`  ?
	- 254. 256-2: l’adresse de broadcast et 0 qui est l’adresse du réseau en général ne peuvent être utilisées.
- combien y a-t-il d'adresses disponibles dans un `/30`?
	-  2
-  quelle est l'utilité d'un  `/30`  ?
	- comme seulement deux hôtes peuvent ce connecter ceci est pratique pour établir les tables de routage dans un WAN. Ceci permet de facilement chaîner des routeur entre eux et ne pas gaspiller des adresses IP. 

#### configuration de la machine virtuelle:
---
- a partir de la machine patron
- `sudo cp /etc/sysconfig/network-scripts/ifcg-enp0s8 /etc/sysconfig/network-scripts/ifcg-enp0s9`
- fichier de config de la carte connecté au réseau net1: 
- vi /etc/sysconfig/network-scripts/ifcg-enp0s8
	 > TYPE=Ethernet
BOOTPROTO=static
NAME=enp0s8
DEVICE=enp0s8
ONBOOT=yes
IPADDR=10.1.1.2
NETMASK=255.255.255.0


- fichier de configuration de la carte connecté au réseau net2:
- vi /etc/sysconfig/network-scripts/ifcg-enp0s9
> TYPE=Ethernet
BOOTPROTO=static
NAME=enp0s9
DEVICE=enp0s9
ONBOOT=yes
IPADDR=10.1.2.2
NETMASK=255.255.255.252

- `sudo ifdown enps0s8 && sudo ifdown enp0s9`
	`sudo ifup enp0s8 && sudo ifup enp0s9`
- alternativement: `sudo systemctl restart network`

changer le nom de domaine
- `echo 'client1.tp1.b2' | sudo tee /etc/hostname`
`sudo reboot now`
`hostname --fqdn`
> client1.tp1.b2
## 2 Basics
#### Routes

- `ip route show`
>10.1.1.0/24 dev enp0s8 proto kernel scope link src 10.1.1.2 	metric 100
		10.1.2.0/30 dev enp0s9 proto kernel scope link src 10.1.2.2 metric 101

- Chaque ligne représente une route et comment elle est accédé. Par example la première ligne veut dire:
	- Le réseau 10.1.1.0 avec un netmask de 	255.255.255.0 est accessible via la carte enp0s8 avec un protocole qui provient du kernel et est et accessible par l'IP 10.1.1.2


- ` sudo ip route del 10.1.2.2/30 `
	`ping -c 2 10.1.2.2`
	> PING 10.1.2.2 (10.1.2.2) 56(84) bytes of data.
64 bytes from 10.1.2.2: icmp_seq=1 ttl=64 time=0.080 ms
64 bytes from 10.1.2.2: icmp_seq=2 ttl=64 time=0.093 ms
--- 10.1.2.2 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1000ms
rtt min/avg/max/mdev = 0.080/0.086/0.093/0.011 ms

- `sudo ip route add 10.1.2.0/30 via 10.1.2.2 dev enps0s9`
`ping -c 2 10.1.2.2`
> PING 10.1.2.2 (10.1.2.2) 56(84) bytes of data.
64 bytes from 10.1.2.2: icmp_seq=1 ttl=64 time=0.084 ms
64 bytes from 10.1.2.2: icmp_seq=2 ttl=64 time=0.091 ms
--- 10.1.2.2 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1261ms
rtt min/avg/max/mdev = 0.084/0.087/0.091/0.010 ms

`ip route s`
>10.1.1.0/24 dev enp0s8 proto kernel scope link src 10.1.1.2 metric 100 
10.1.2.0/30 via 10.1.2.2 dev enp0s9 

#### Table ARP
---
- `ip neigh s`
	>10.1.2.1 dev enp0s9 lladdr 0a:00:27:00:00:02 REACHABLE
10.1.1.1 dev enp0s8 lladdr 0a:00:27:00:00:01 REACHABLE

- `sudo ip neigh f all`
	`ip neigh s`
	>10.1.1.1 dev enp0s8 lladdr 0a:00:27:00:00:01 REACHABLE

- `ping -c 1 10.1.2.1`
`ip neigh s`
			
	> 10.1.1.1 dev enp0s8 lladdr 0a:00:27:00:00:01 REACHABLE
10.1.2.1 dev enp0s9 lladdr 0a:00:27:00:00:02 REACHABLE

- Nous avons vider la table ARP (les voisins) et ensuite nous les avons rajouté lors d'un ping ou la machine a fait une requête ARP et reçu une réponse et ensuite ajouté l'entré dans la table ARP.
#### Capture Réseau
---
- `ssh paul@10.1.1.2` en 2 instances T1 et T2 .
- `ssh paul@10.1.2.2`
`sudo ip neigh f all`

- T1: `sudo tcpdump -i enp0s8 -w ping.pcap` 
T2: `ping -c 4 10.1.2.1`
T1: arreter la capture de packet
	>10 packets received by filter
0 packets dropped by kernel

- `scp paul@10.1.1.2:/home/paul/ping.pcap /home/paul/ynov/reseau/tp1`

[ping.pcap](ping.pcap)

---
# II. Communication simple entre deux machines

## 1. Mise en place

#### configuration de la machine virtuelle:
- fichier de config de la carte connecté au réseau net1: 
- vi /etc/sysconfig/network-scripts/ifcg-enp0s8
	 > TYPE=Ethernet
BOOTPROTO=static
NAME=enp0s8
DEVICE=enp0s8
ONBOOT=yes
IPADDR=10.1.1.3
NETMASK=255.255.255.0
- `echo 'client2.tp1.b2' | sudo tee /etc/hostname`
`sudo reboot now`
---
## 2. Basics

### ping et ARP
---
-  c1 = 10.1.2.2
- c2 = 10.1.1.3
	 - c2: `ping -c 1 10.1.1.3`
	 `ip neigh s`
	>10.1.2.1 dev enp0s9 lladdr 0a:00:27:00:00:02 STALE
10.1.1.3 dev enp0s8 lladdr 08:00:27:6c:53:a8 REACHABLE
10.1.1.1 dev enp0s8 lladdr 0a:00:27:00:00:01 REACHABLE
	- c2: `sudo ip neigh f all`
	- nouveau shell a fermer après la commande: 
	`ssh 10.1.1.2`
	`sudo ip neigh f all`
	- c2: `sudo tcpdump -i enp0s8 -w ping.pcap` 
	- ouvrir un autre shell t2 sur c2
	- c2.t2: `ping -c 4 10.1.1.3`
	- c2: arreter la capture de packet
	>10 packets received by filter
0 packets dropped by kernel
	- `scp paul@10.1.1.3:/home/paul/ping-2.pcap /home/paul/ynov/reseau/tp1`

[ping-2.pcap](ping.pcap)

---
#### UDP
---
- ouvrir deux shell pour chaque client:
	- c1.t1 et c1.t2 sont les shells pour la première machine
	- c2.t1 et c2.t2 sont les shells pour la deuxième machine
- sur chaque machine ajouter le port udp
`sudo firewall-cmd --zone=public --add-port=8888/udp --permanent`
`sudo firewall-cmd --reload`
- sur c1.t1: `nc -u -l 8888`
- sur c2.t1: `nc -u 10.1.1.2 8888`
- sur c1.t2:	`ss -unp`
	>Recv-Q Send-Q Local Address:Port               Peer Address:Port              
0      0      10.1.1.2:8888               10.1.1.3:43163               users:(("nc",pid=4181,fd=4))

-  c2.t2: `ss -unp`
	> Recv-Q Send-Q Local Address:Port               Peer Address:Port              
0      0      10.1.1.3:43163              10.1.1.2:8888                users:(("nc",pid=4030,fd=3))
 
 - c1.t3 = 10.1.2.2
	 - c1.t1: `nc -u -l 8888`
	 - c2.t2: `nc -u 10.1.1.2 8888`
	 - c1.t3: `sudo tcpdump -i enp0s8 -w nc-udp.pcap`
	- c1.t1: `test` 
	- arrêter la capture de packet
	- `scp 10.1.1.2:/home/paul/nc-udp.pcap /home/paul/ynov/reseau/tp1`

[nc-udp.pcap](nc-udp.pcap)

---
#### TCP
---
- c1.t1 = 10.1.1.2
- c2.t1 = 10.1.1.3
- t2 = deuxième shell de t1
- c1.t3 = 10.1.2.2
	- sur chaque machine ajouter le port TCP
`sudo firewall-cmd --zone=public --add-port=8888/tcp --permanent && sudo firewall-cmd --reload`
	- c1.t1: `nc -l 8888`
	- c2.t1: `nc 10.1.1.2 8888`
	- c1.t2: `ss -tnp`
		>State       Recv-Q Send-Q Local Address:Port               Peer Address:Port              
ESTAB       0      0      10.1.2.2:22                 10.1.2.1:46006              
ESTAB       0      0      10.1.1.2:8888               10.1.1.3:57706               users:(("nc",pid=5451,fd=5))
ESTAB       0      0      10.1.1.2:22                 10.1.1.1:36416   
		
	- c1.t2: `ss -tnp`
		> State       Recv-Q Send-Q Local Address:Port               Peer Address:Port              
ESTAB       0      0      10.1.1.3:22                 10.1.1.1:38068              
ESTAB       0      0      10.1.1.3:22                 10.1.1.1:42006              
ESTAB       0      0      10.1.1.3:57706              10.1.1.2:8888                users:(("nc",pid=4671,fd=3))

- c1.t3: `sudo tcpdump -i enp0s8 -w firewall.pcap`
- c1.t1:   `nc -l 8888`
- c2.t2: `nc 10.1.1.2 8888`
- fermer netcat
- fermer la capture de paquet
- `scp 10.1.1.2:/home/paul/nc-tcp.pcap /home/paul/ynov/reseau/tp1`

[nc-tcp.pcap](nc-tcp.pcap)

---
#### Firewall
---
- c1.t1: `sudo firewall-cmd --remove-port=8888/udp --permanent`
`sudo firewall-cmd --reload`
- c1.t3: `sudo tcpdump -i enp0s8 -w nc-udp.pcap`
- c1.t1: `nc -u -l 8888`
- arrêter la capture de paquet 
- `scp 10.1.1.2:/home/paul/firewall.pcap /home/paul/ynov/reseau/tp1`

[firewall.pcap](firewall.pcap)

---
## 3. Bonus : ARP spoofing




---
# III. Routage statique simple
- client 1: -   `sysctl -w net.ipv4.ip_forward=1`
	>net.ipv4.ip_forward = 1

- client 2: `sudo ip route add 10.1.2.0/30 via 10.1.1.2 dev enp0s8
`
	- `ping -c 2 10.1.2.2`
		> PING 10.1.2.2 (10.1.2.2) 56(84) bytes of data.
64 bytes from 10.1.2.2: icmp_seq=1 ttl=64 time=0.933 ms
64 bytes from 10.1.2.2: icmp_seq=2 ttl=64 time=0.543 ms
--- 10.1.2.2 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 0.543/0.738/0.933/0.195 ms

- `ip route show`
	>10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.15 metric 100 
10.1.1.0/24 dev enp0s8 proto kernel scope link src 10.1.1.3 metric 101 
10.1.2.0/30 via 10.1.1.2 dev enp0s8 
