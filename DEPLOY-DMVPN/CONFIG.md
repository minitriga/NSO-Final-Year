# DMVPN NSO PACKAGE - ALEX GITTINGS

This is a package to deploy a Phase 2 DMVPN Network using NSO. This was developed as part of my Final Year Dissertation looking at the Orchestration of Network Services through the use of Ansible and NSO. 

# Device Information

Just a few things to set up the environment. 

## Default Config

```
en
conf t
hostname SPOKE-15
vrf definition MGMT
address-family ipv4 unicast
exit
exit
no ip domain lookup
line vty 0 4
 login local
 transport input all
exit
username script priv 15 secret Cisco1234
ip route vrf MGMT 172.16.10.0 255.255.255.0 172.16.10.1
ip domain name lab.local
crypto key gen rsa mod 2048
ip ssh version 2
int gi0/0
no shut
int gi0/1
description **OOB Management**
 vrf forwarding MGMT
 ip address 172.16.10.17 255.255.255.0
 no shut
int lo0
 ip add	192.168.15.1 255.255.255.0
 no shut
exit
exit
copy run flash:base.cfg

wr

ping vrf MGMT 172.16.10.1
ping 65.100.123.30
```

##Configure SP to SPOKE connection

### SP
```
en
conf t
int gi0/0.115
encap dot 115
ip add	65.100.123.30 255.255.255.254
no shut
```

### SPOKE
```
en
conf t
int gi0/0.115
encap dot 115
ip add	65.100.123.31 255.255.255.254
no shut
exit
ip route 0.0.0.0 0.0.0.0 65.100.123.30
exit
copy run flash:base.cfg


ping 65.100.123.30

wr
```

## DMVPN Config 

### HUB
```
crypto isakmp policy 10
 encr 3des
 hash sha256
 authentication pre-share
crypto isakmp key Cisco1234 address 0.0.0.0        
!
!
crypto ipsec transform-set MINE esp-3des 
 mode tunnel
!
crypto ipsec profile DMVPN
 set transform-set MINE 

router eigrp 1
network 192.168.0.0 0.0.255.255
network 10.0.0.0 
no auto-summary


HUB
int tun 0
 ip address 10.1.1.1 255.255.255.0
 ip nhrp map multicast dynamic
 ip nhrp network 1
 tunnel source gig0/0.100
 tunnel mode gre multipoint 
 ip mtu 1416
 ip hold-time eigrp 1 35
 no ip next-hop-self eigrp 1
 no ip split-horizon eigrp 1
tunnel protection ipsec profile DMVPN
```
### SPOKE
```
crypto isakmp policy 10
 encr 3des
 hash sha256
 authentication pre-share
crypto isakmp key Cisco1234 address 0.0.0.0        

crypto ipsec transform-set MINE esp-3des 
 mode tunnel

crypto ipsec profile DMVPN
 set transform-set MINE 

router eigrp 1
network 192.168.0.0 0.0.255.255
network 10.0.0.0 
no auto-summary

interface Tunnel0
 ip address 10.1.1.4 255.255.255.0
 no ip redirects
 ip mtu 1416
 ip nhrp map 10.1.1.1 65.100.123.3
 ip nhrp map multicast 65.100.123.3
 ip nhrp network-id 1
 ip nhrp nhs 10.1.1.1
 tunnel source GigabitEthernet0/0.104
 tunnel mode gre multipoint
 ip hold-time eigrp 1 35
 no ip next-hop-self eigrp 1
 no ip split-horizon eigrp 1
 tunnel protection ipsec profile DMVPN
```

## Verify
```
show ip route eigrp
show crypto isakmp sa 
show cry ipsec sa
```
