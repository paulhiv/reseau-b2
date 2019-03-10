
# B2 Réseau 2018 - TP2
# I. Mise en place du lab

## 1.Création des cartes réseau
net1 = 10.2.1.0/24

net2= 10.2.2.0/24

net3 = 10.2.12.0/29

## 2. Création des VMs et adressage IP
**client1:**
- configuration cartes réseau:

  `sudo vim /etc/sysconfig/network-scripts/ifcfg-enp0s8`
  ```
  TYPE=Ethernet
  BOOTPROTO=static
  NAME=enp0s8
  DEVICE=enp0s8
  ONBOOT=yes
  IPADDR=10.2.1.10
  NETMASK=255.255.255.0
  ```
  `sudo ifdown enp0s8 && sudo ifup enp0s8`
- configuration des hosts 

  ``sudo vim /etc/hosts``

  ```
  127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
  ::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
  10.2.1.10 client1 client1.net1.b2
  10.2.1.254 router1 router1.net12.b2
  10.2.12.2 router1 router1.net12.b2
  10.2.12.3 router2 router2.net12.b2
  10.2.2.254 router2 router2.net12.b2
  10.2.2.10 server1 server1.net2.b2
  ```
- configuration du hostname

  `echo 'client1.net1.b2' | sudo tee /etc/hostname`

**server1:**
- configuration cartes réseau:

  `sudo vim /etc/sysconfig/network-scripts/ifcfg-enp0s8`
  ```TYPE=Ethernet
  BOOTPROTO=static
  NAME=enp0s8
  DEVICE=enp0s8
  ONBOOT=yes
  IPADDR=10.2.2.10
  NETMASK=255.255.255.0
  ```
  `sudo ifdown enp0s8 && sudo ifup enp0s8`
- configuration des hosts 

  ``sudo vim /etc/hosts``

  ```
  localhost localhost.localdomain localhost4 localhost4.localdomain4
  ::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
  10.2.12.2 router1 router1.net12.b2
  10.2.1.254 router1 router1.net12.b2
  10.2.1.10 client1 client1.net1.b2
  10.2.2.254 router2 router2.net12.b2
  10.2.12.3 router2 router2.net12.b2
  10.2.2.10 server1 server1.net2.b2
  ```
- configuration du hostname

  `echo 'server1.net2.b2' | sudo tee /etc/hostname`

**router1**
- configuration cartes réseau:

  `sudo vim /etc/sysconfig/network-scripts/ifcfg-enp0s8`
  ```TYPE=Ethernet
  BOOTPROTO=static
  NAME=enp0s8
  DEVICE=enp0s8
  ONBOOT=yes
  IPADDR=10.2.1.254
  NETMASK=255.255.255.0
  ```

  `sudo vim /etc/sysconfig/network-scripts/ifcfg-enp0s9`
  ```TYPE=Ethernet
  BOOTPROTO=static
  NAME=enp0s8
  DEVICE=enp0s8
  ONBOOT=yes
  IPADDR=10.2.2.10
  NETMASK=255.255.255.248
  ```

  `sudo systemctl restart network`
- configuration des hosts 

  ``sudo vim /etc/hosts``

  ```
  localhost localhost.localdomain localhost4 localhost4.localdomain4
  ::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
  10.2.12.2 router1 router1.net12.b2
  10.2.1.254 router1 router1.net12.b2
  10.2.1.10 client1 client1.net1.b2
  10.2.2.254 router2 router2.net12.b2
  10.2.12.3 router2 router2.net12.b2
  10.2.2.10 server1 server1.net2.b2
  ```
- configuration du hostname

  `echo 'router1.net12.b2' | sudo tee /etc/hostname`

  **router2**
- configuration cartes réseau:

  `sudo vim /etc/sysconfig/network-scripts/ifcfg-enp0s8`
  ```TYPE=Ethernet
  BOOTPROTO=static
  NAME=enp0s8
  DEVICE=enp0s8
  ONBOOT=yes
  IPADDR=10.2.2.254
  NETMASK=255.255.255.0
  ```

  `sudo vim /etc/sysconfig/network-scripts/ifcfg-enp0s9`
  ```TYPE=Ethernet
  BOOTPROTO=static
  NAME=enp0s8
  DEVICE=enp0s8
  ONBOOT=yes
  IPADDR=10.2.12.3
  NETMASK=255.255.255.248
  ```

  `sudo systemctl restart network`
- configuration des hosts 

  ``sudo vim /etc/hosts``

  ```
  localhost localhost.localdomain	localhost4 localhost4.localdomain4
  ::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
  10.2.12.3 router2 router2.net12.b2
  10.2.2.254 router2 router2.net12.b2
  10.2.2.10 server1 server1.net2.b2
  10.2.12.2 router1 router1.net12.b2
  10.2.1.254 router1 router1.net12.b2
  10.2.1.10 client1 client1.net1.b2

  ```
- configuration du hostname

  `echo 'router2.net12.b2' | sudo tee /etc/hostname`

## 3. Routage statique

- activer le forwarding ip

  sur router1 et router2:
  `echo 'net.ipv4.conf.all.forwarding=1' | sudo tee --append /etc/sysctl.conf`

- configurer les routes statiques
  
  **router1:** `sudo vim /etc/sysconfig/network-scripts/route-enp0s9`

  ```
  10.2.2.0/24 via 10.2.12.3 dev enp0s9
  ```

  **router2:** `sudo vim /etc/sysconfig/network-scripts/route-enp0s9` 
  ```
  10.2.1.0/24 via 10.2.12.2 dev enp0s9
  ```

  **client1:** `sudo vim /etc/sysconfig/network-scripts/route-enp0s8`
  ```
  10.2.2.0/24 via 10.2.1.254 dev enp0s8
  ```
  **client2:** `sudo vim /etc/sysconfig/network-scripts/route-enp0s8`
  ```
  10.2.1.0/24 via 10.2.2.254 dev enp0s8
  ```

- vérification

  ```
  [paul@client1 ~]$ ping -c 4 server1
  PING server1 (10.2.2.10) 56(84) bytes of data.
  64 bytes from server1 (10.2.2.10): icmp_seq=1 ttl=62 time=2.21 ms
  64 bytes from server1 (10.2.2.10): icmp_seq=2 ttl=62 time=2.64 ms
  64 bytes from server1 (10.2.2.10): icmp_seq=3 ttl=62 time=2.43 ms
  64 bytes from server1 (10.2.2.10): icmp_seq=4 ttl=62 time=2.95 ms

  --- server1 ping statistics ---
  4 packets transmitted, 4 received, 0% packet loss, time 3251ms
  rtt min/avg/max/mdev = 2.217/2.563/2.952/0.271 ms
  ```

## 4. Visualisation du routage avec Wireshark

- [capture wireshark sur net2](./routage-net2.pcap)

- [capture wireshark sur net12](./routage-net12.pcap)

- les différences
  en observant les trames de ping sur les differentes carte on observe une difference des adresses mac ceci montre que L'IP fowarding est en place

  **net12**
  ```
  Ethernet II, Src: PcsCompu_88:b2:e5 (08:00:27:88:b2:e5), Dst: PcsCompu_09:5d:9a (08:00:27:09:5d:9a)
    Destination: PcsCompu_09:5d:9a (08:00:27:09:5d:9a)
    Source: PcsCompu_88:b2:e5 (08:00:27:88:b2:e5)
    Type: IPv4 (0x0800)

  ```
  **net2**
  ```
  Ethernet II, Src: PcsCompu_04:17:56 (08:00:27:04:17:56), Dst: PcsCompu_ea:dd:fc (08:00:27:ea:dd:fc)
    Destination: PcsCompu_ea:dd:fc (08:00:27:ea:dd:fc)
    Source: PcsCompu_04:17:56 (08:00:27:04:17:56)
    Type: IPv4 (0x0800)
  ```

# II. NAT et services d'infra

## 1. Mise en place du NAT

**sur router1**

- configuration de la carte NAT :
`sudo vim /etc/sysconfig/network-scripts/ifcfg-enp0s3`

  ```
  TYPE=Ethernet
  BOOTPROTO=dhcp
  NAME=enp0s3
  DEVICE=enp0s3
  ONBOOT=yes
  ZONE=public
  ```

- mise en place des zones:

  `echo 'ZONE=public' | sudo tee --append/etc/sysconfig/network-scripts/ifcfg-enp0s8`

  `echo 'ZONE=internal' | sudo tee --append/etc/sysconfig/network-scripts/ifcfg-enp0s9`


- configuration du firewall
  
  `sudo firewall-cmd --add-interface=enp0s8 --zone=public --permanent`

   `sudo firewall-cmd --add-interface=enp0s3 --zone=public --permanent`

   `sudo firewall-cmd --add-interface=enp0s9 --zone=internal --permanent`

  `sudo firewall-cmd --add-masquerade --zone=public --permanent`

  `sudo firewall-cmd --reload`

- mise en place des routes de default

  **sur router2**
  `sudo vim /etc/sysconfig/network`
  ```
  GATEWAY=10.2.12.2
  GATEWAYDEV=enp0s9
  ```
  `sudo ifdown enp0s9 && sudo ifup enp0s9`

  **sur client1**
  `sudo vim /etc/sysconfig/network`
  ```
  GATEWAY=10.2.1.254
  GATEWAYDEV=enp0s8
  ```
   `sudo ifdown enp0s9 && sudo ifup enp0s8`

## 2. DHCP server

- reconfiguration du hostname: `echo 'dhcp-server.net1.b2' | sudo tee /etc/hostname'`


  `sudo vim /etc/dhcp/dhcpd.conf`
  ```
  # dhcpd.conf

  # option definitions common to all supported networks
  option domain-name "net1.tp2";

  default-lease-time 600;
  max-lease-time 7200;

  # If this DHCP server is the official DHCP server for the local
  # network, the authoritative directive should be uncommented.
  authoritative;

  # Use this to send dhcp log messages to a different log file (you also
  # have to hack syslog.conf to complete the redirection).
  log-facility local7;

  subnet 10.2.1.0 netmask 255.255.255.0 {
    range 10.2.1.50 10.2.1.70;
    option domain-name "net1.tp2";
    option routers 10.2.1.254;
    option broadcast-address 10.2.1.255;
    option domain-name-servers 8.8.8.8;
  }
  ```
  `sudo systemctl start dhcpd`

- test dhcp
  **clone**
  
   client1.net1.b1 = client2.net1.b1

  `echo "client2.net1.b2" | sudo tee /etc/hostname`
  
  `sudo vim /etc/sysconfig/network-scripts/ifcfg-enp0s8`
  ```
  TYPE=Ethernet
  BOOTPROTO=dhcp
  NAME=enp0s8
  DEVICE=enp0s8
  ONBOOT=yes
  ```

- dhclient
  ```
  [paul@client2 ~]$ sudo dhclient -v
  [sudo] password for paul: 
  Internet Systems Consortium DHCP Client 4.2.5
  Copyright 2004-2013 Internet Systems Consortium.
  All rights reserved.
  For info, please visit https://www.isc.org/software/dhcp/

  Listening on LPF/enp0s8/08:00:27:6c:3d:87
  Sending on   LPF/enp0s8/08:00:27:6c:3d:87
  Sending on   Socket/fallback
  DHCPDISCOVER on enp0s8 to 255.255.255.255 port 67 interval 4 (xid=0x6d379eb)
  DHCPREQUEST on enp0s8 to 255.255.255.255 port 67 (xid=0x6d379eb)
  DHCPOFFER from 10.2.1.10
  DHCPACK from 10.2.1.10 (xid=0x6d379eb)
  bound to 10.2.1.50 -- renewal in 266 seconds.
  ```

## 3. NTP server

**sur router1, router2, server1:**

- reconfiguration des zones
  
    **sur router1:**
    
      `sudo vim /etc/sysconfig/network-scripts/ifcfg-enp0s9`
    ```
    TYPE=Ethernet
    BOOTPROTO=static
    NAME=enp0s9
    DEVICE=enp0s9
    ONBOOT=yes
    IPADDR=10.2.12.2
    NETMASK=255.255.255.248
    ZONE=trusted
    ```

    `sudo vim /etc/sysconfig/network-scripts/ifcfg-enp0s8`
    
    ```
    TYPE=Ethernet
    BOOTPROTO=static
    NAME=enp0s8
    DEVICE=enp0s8
    ONBOOT=yes
    IPADDR=10.2.1.254
    NETMASK=255.255.255.0
    ZONE=trusted
    ```

    `sudo firewall-cmd --add-masquerade --zone=trusted --permanent`

    `sudo firewall-cmd --reload`

    `sudo systemctl restart network`
    **sur router2**
    `sudo vim /etc/sysconfig/network`
    ```
    GATEWAY=10.2.2.254
    GATEWAYDEV=enp0s8
    ```
    `sudo vim /etc/sysconfig/network-scripts/ifcfg-enp0s9`
    ```
    TYPE=Ethernet
    BOOTPROTO=static
    NAME=enp0s8
    DEVICE=enp0s8
    ONBOOT=yes
    IPADDR=10.2.2.254
    NETMASK=255.255.255.0
    ZONE=trusted
    ```

    `sudo vim /etc/sysconfig/network-scripts/ifcfg-enp0s9`
   
    ```
    TYPE=Ethernet
    BOOTPROTO=static
    NAME=enp0s9
    DEVICE=enp0s9
    ONBOOT=yes
    IPADDR=10.2.12.3
    NETMASK=255.255.255.248
    ZONE=trusted
    ```

    `sudo firewall-cmd --add-masquerade --zone=trusted --permanent`

    `sudo firewall-cmd --reload`
    
    `sudo systemctl restart network`

- mise en place chrony

  **router1**

    `sudo yum install chrony`

    `sudo vim /etc/chrony.conf`
    ```
    # Use public servers from the pool.ntp.org project.
    # Please consider joining the pool (http://www.pool.ntp.org/join.html).

    server 0.fr.pool.ntp.org
    server 1.fr.pool.ntp.org
    server 2.fr.pool.ntp.org
    server 3.fr.pool.ntp.org

    # Record the rate at which the system clock gains/losses time.
    driftfile /var/lib/chrony/drift

    # Allow the system clock to be  stepped in the first three updates
    # if its offset is larger than 1 second.
    makestep 1.0 3

    # Enable kernel synchronization of the real-time clock (RTC).
    rtcsync
    
    # Enable hardware timestamping on all interfaces that support it.
    
    #hwtimestamp *

    # Increase the minimum number of selectable sources required to adjust
    # the system clock.
    #minsources 2

    # Allow NTP client access from local network.
    #allow 192.168.0.0/16

    # Serve time even if not synchronized to a time source.
    #local stratum 10

    # Specify file containing keys for NTP authentication.
    #keyfile /etc/chrony.keys

    # Specify directory for log files.
    logdir /var/log/chrony

    # Select which information is logged.
    #log measurements statistics tracking
    ```
    `sudo firewall-cmd --add-port=123/udp --permanent`

    `sudo systemctl restart chronyd`

    `sudo firewall-cmd --add-masquerade --zone=public --permanent`

    `sudo firewall-cmd --reload`
    - vérification
    ```
    [paul@router1 ~]$ chronyc sources
    210 Number of sources = 4
    MS Name/IP address         Stratum Poll Reach LastRx Last sample               
    ===============================================================================
    ^+ ntp4.rbx-fr.hosts.301-mo>     2   6    77     4  -2972us[-2972us] +/-   25ms
    ^* ntp-2.arkena.net              2   6    77     5   +366us[ +660us] +/-   29ms
    ^+ electabuzz.felixc.at          3   6    77     4  -7188us[-7188us] +/-   29ms
    ^+ cp01.webhd.nl                 3   6    77     5   +215us[ +215us] +/-   39ms
    [paul@router1 ~]$ chronyc tracking
    Reference ID    : 5F51AD4A (ntp-2.arkena.net)
    Stratum         : 3
    Ref time (UTC)  : Sun Mar 10 17:43:19 2019
    System time     : 0.000290660 seconds fast of NTP time
    Last offset     : +0.000294080 seconds
    RMS offset      : 0.003901408 seconds
    Frequency       : 11.221 ppm fast
    Residual freq   : +0.476 ppm
    Skew            : 123.052 ppm
    Root delay      : 0.028785819 seconds
    Root dispersion : 0.016339561 seconds
    Update interval : 64.3 seconds
    Leap status     : Normal
    ```
  **router2**

    `sudo yum install chrony`

    `sudo vim /etc/chrony.conf`
  ```
  # Server to syncrhonize with
  server router1.tp2.b2 prefer
  #
  initstepslew 20 router1.tp2.b2

  # # Record the rate at which the system clock gains/losses time.
  driftfile /var/lib/chrony/drift

  # # Allow the system clock to be stepped in the first three updates
  # # if its offset is larger than 1 second.
  makestep 1.0 3

  # # Enable kernel synchronization of the real-time clock (RTC).
  rtcsync

  # # Serve time even if not synchronized to a time source.
  local stratum 10

  # # Specify file containing keys for NTP authentication.
  keyfile /etc/chrony.keys

  # # Specify directory for log files.
  logdir /var/log/chrony
  # # Select which information is logged.
  log measurements statistics tracking
  [paul@router2 ~]$ cat vim /etc/chrony.conf 
  cat: vim: No such file or directory
  # Server to syncrhonize with
  server router1.tp2.b2 prefer
  #
  initstepslew 20 router1.tp2.b2

  # # Record the rate at which the system clock gains/losses time.
  driftfile /var/lib/chrony/drift

  # # Allow the system clock to be stepped in the first three updates
  # # if its offset is larger than 1 second.
  makestep 1.0 3

  # # Enable kernel synchronization of the real-time clock (RTC).
  rtcsync

  # # Serve time even if not synchronized to a time source.
  local stratum 10

  # # Specify file containing keys for NTP authentication.
  keyfile /etc/chrony.keys

  # # Specify directory for log files.
  logdir /var/log/chrony
  # # Select which information is logged.
  log measurements statistics tracking

  ```
   `sudo firewall-cmd --add-port=123/udp --zone=trusted --permanent`

    `sudo systemctl restart chronyd`

    `sudo firewall-cmd --add-masquerade --zone=public --permanent`

    `sudo firewall-cmd --reload`

**server1**

  `sudo yum install chrony`

    `sudo vim /etc/chrony.conf`
  ```
  # Server to syncrhonize with
  server router1.tp2.b2 prefer
  #
  initstepslew 20 router1.tp2.b2

  # # Record the rate at which the system clock gains/losses time.
  driftfile /var/lib/chrony/drift

  # # Allow the system clock to be stepped in the first three updates
  # # if its offset is larger than 1 second.
  makestep 1.0 3

  # # Enable kernel synchronization of the real-time clock (RTC).
  rtcsync

  # # Serve time even if not synchronized to a time source.
  local stratum 10

  # # Specify file containing keys for NTP authentication.
  keyfile /etc/chrony.keys

  # # Specify directory for log files.
  logdir /var/log/chrony
  # # Select which information is logged.
  log measurements statistics tracking
  [paul@router2 ~]$ cat vim /etc/chrony.conf 
  cat: vim: No such file or directory
  # Server to syncrhonize with
  server router1.tp2.b2 prefer
  #
  initstepslew 20 router1.tp2.b2

  # # Record the rate at which the system clock gains/losses time.
  driftfile /var/lib/chrony/drift

  # # Allow the system clock to be stepped in the first three updates
  # # if its offset is larger than 1 second.
  makestep 1.0 3

  # # Enable kernel synchronization of the real-time clock (RTC).
  rtcsync

  # # Serve time even if not synchronized to a time source.
  local stratum 10

  # # Specify file containing keys for NTP authentication.
  keyfile /etc/chrony.keys

  # # Specify directory for log files.
  logdir /var/log/chrony
  # # Select which information is logged.
  log measurements statistics tracking
  ```
   
  `sudo firewall-cmd --add-port=123/udp --zone=trusted --permanent`

  `sudo systemctl restart chronyd`

  `sudo firewall-cmd --add-masquerade --zone=public --permanent`

  `sudo firewall-cmd --reload`

## 4. Web server

- mise en place

  `sudo yum install -y epel-release`

  `sudo yum install -y nginx`

  `sudo firewall-cmd --add-port=80/tcp  --permanent`

  `sudo systemctl start nginx`

  `vim /usr/share/nginx/html/index.html`

```
  <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.1//EN" "http://www.w3.org/TR/xhtml11/DTD/xhtml11.dtd">

<html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en">
    <head>
        <title>Test Page for the Nginx HTTP Server on Fedora</title>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
        <style type="text/css">
            /*<![CDATA[*/
            body {
                background-color: #fff;
                color: #000;
                font-size: 0.9em;
                font-family: sans-serif,helvetica;
                margin: 0;
                padding: 0;
            }
            :link {
                color: #c00;
            }
            :visited {
                color: #c00;
            }
            a:hover {
                color: #f50;
            }
            h1 {
                text-align: center;
                margin: 0;
                padding: 0.6em 2em 0.4em;
                background-color: #294172;
                color: #fff;
                font-weight: normal;
                font-size: 1.75em;
                border-bottom: 2px solid #000;
            }
            h1 strong {
                font-weight: bold;
                font-size: 1.5em;
            }
            h2 {
                text-align: center;
                background-color: #3C6EB4;
                font-size: 1.1em;
                font-weight: bold;
                color: #fff;
                margin: 0;
                padding: 0.5em;
                border-bottom: 2px solid #294172;
            }
            hr {
                display: none;
            }
            .content {
                padding: 1em 5em;
            }
            .alert {
                border: 2px solid #000;
            }

            img {
                border: 2px solid #fff;
                padding: 2px;
                margin: 2px;
            }
            a:hover img {
                border: 2px solid #294172;
            }
            .logos {
                margin: 1em;
                text-align: center;
            }
            /*]]>*/
        </style>
    </head>
    <body>
        <h1>Welcome t <strong>nginx</strong> on Fedora!</h1>
	<h1> it works</h1>
    </body>
</html>
```
` sudo systemctl restart nginx`

- vérification
```
    [paul@dhcp-server ~]$ curl 10.2.2.10
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.1//EN" "http://www.w3.org/TR/xhtml11/DTD/xhtml11.dtd">

<html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en">
    <head>
        <title>Test Page for the Nginx HTTP Server on Fedora</title>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
        <style type="text/css">
            /*<![CDATA[*/
            body {
                background-color: #fff;
                color: #000;
                font-size: 0.9em;
                font-family: sans-serif,helvetica;
                margin: 0;
                padding: 0;
            }
            :link {
                color: #c00;
            }
            :visited {
                color: #c00;
            }
            a:hover {
                color: #f50;
            }
            h1 {
                text-align: center;
                margin: 0;
                padding: 0.6em 2em 0.4em;
                background-color: #294172;
                color: #fff;
                font-weight: normal;
                font-size: 1.75em;
                border-bottom: 2px solid #000;
            }
            h1 strong {
                font-weight: bold;
                font-size: 1.5em;
            }
            h2 {
                text-align: center;
                background-color: #3C6EB4;
                font-size: 1.1em;
                font-weight: bold;
                color: #fff;
                margin: 0;
                padding: 0.5em;
                border-bottom: 2px solid #294172;
            }
            hr {
                display: none;
            }
            .content {
                padding: 1em 5em;
            }
            .alert {
                border: 2px solid #000;
            }

            img {
                border: 2px solid #fff;
                padding: 2px;
                margin: 2px;
            }
            a:hover img {
                border: 2px solid #294172;
            }
            .logos {
                margin: 1em;
                text-align: center;
            }
            /*]]>*/
        </style>
    </head>

    <body>
        <h1>Welcome t <strong>nginx</strong> on Fedora!</h1>
	<h1> it works</h1>
        <div class="content">
            <p>This page is used to test the proper operation of the
            <strong>nginx</strong> HTTP server after it has been
            installed. If you can read this page, it means that the
            web server installed at this site is working
            properly.</p>

            <div class="alert">
                <h2>Website Administrator</h2>
                <div class="content">
                    <p>This is the default <tt>index.html</tt> page that
                    is distributed with <strong>nginx</strong> on
                    Fedora.  It is located in
                    <tt>/usr/share/nginx/html</tt>.</p>

                    <p>You should now put your content in a location of
                    your choice and edit the <tt>root</tt> configuration
                    directive in the <strong>nginx</strong>
                    configuration file
                    <tt>/etc/nginx/nginx.conf</tt>.</p>

                </div>
            </div>

            <div class="logos">
                <a href="http://nginx.net/"><img
                    src="nginx-logo.png" 
                    alt="[ Powered by nginx ]"
                    width="121" height="32" /></a>

                <a href="http://fedoraproject.org/"><img 
                    src="poweredby.png" 
                    alt="[ Powered by Fedora ]" 
                    width="88" height="31" /></a>
            </div>
        </div>
    </body>
</html>
```

# Bilan
:fire: :fire: :fire: :fire: :fire:
