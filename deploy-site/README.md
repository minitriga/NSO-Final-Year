# L3 MPLS NSO PACKAGE - ALEX GITTINGS

This is a package to deploy a L3 MPLS Network using NSO. This was developed as part of my Final Year Dissertation looking at the Orchestration of Network Services through the use of Ansible and NSO. 

## Show Devices In NSO Already

## Configure Tenant in NSO CLI

```
services deploy-site Kamazoy
1
2894
true
false
false

pe CE-1-A
CSR1K-1-PE-1
192.0.2.11
GigabitEthernet
1
100.64.1.1
255.255.255.252
2001:db8:1:310::1/64
100.64.1.2
2001:db8:1:310::2
64512
65413

pe CE-1-B
CSR1K-1-PE-3
192.0.2.13
GigabitEthernet
1
100.64.2.1
255.255.255.252
2001:db8:1:311::1/64
100.64.2.2
2001:db8:1:311::2
64512
65413

cpe CE-1-A-1
CPE-1-A-1
GigabitEthernet
1
100.64.1.2
255.255.255.252
2001:db8:1:310::2/64
100.64.1.1
2001:db8:1:310::1
65413
64512

cpe CE-1-B-1
CPE-1-B-1
GigabitEthernet
1
100.64.2.2
255.255.255.252
2001:db8:1:311::2/64
100.64.2.1
2001:db8:1:311::1
65413
64512
```

### Verify Configuration
We would like to verify the configuration before applying to the devices.

```
commit dry-run outformat native 
commit dry-run outformat cli
commit dry-run outformat xml
```

### Commit the Tenant

To commit the config enter `commit`

### Verify Connectivity to Other Site

CE-1-B-1:
`ping 172.16.1.1 source lo0 repeat 10000`
`traceroute 172.16.1.1 source lo0`

CSR1K-1-P:
`show mpls forwarding-table`
`show mpls ldp neighbor`

CSR1K-1-PE-1:
`show ip bgp vpnv4 all summary` 
`show mpls forwarding-table `

### Add New Site

```
services deploy-site Kamazoy
pe CE-1-A-2
CSR1K-1-PE-2
192.0.2.12
GigabitEthernet
1
100.64.3.1
255.255.255.252
2001:db8:1:312::1/64
100.64.3.2
2001:db8:1:312::2
64512
65413

cpe CE-1-A-3
CPE-1-A-3
ge
0/0/1
100.64.3.2
255.255.255.252
2001:db8:1:312::2/64
100.64.3.1
2001:db8:1:312::1
65413
64512
ipv4-prefix /30
```

### Verify New Site Connectivity
NSO:
`show running-config services deploy-site`


CE-1-B-1:
`ping 172.16.3.1 source lo0 repeat 10000`
`traceroute 172.16.3.1 source lo0`

### Rollback Functionality

Do you want to roll back the configuration enter `rollback configuration`

## Service Deployment Rest API

See postman scripts
