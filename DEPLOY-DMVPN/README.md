# DMVPN NSO PACKAGE - ALEX GITTINGS

This is a package to deploy a Phase 2 DMVPN Network using NSO. This was developed as part of my Final Year Dissertation looking at the Orchestration of Network Services through the use of Ansible and NSO. 


# Dissertation Presentation Guide

This section will outline the steps required to demonstrate this package during my third year dissertation presentation. 

## Overview of the files

Show the underlying files both YANG and XML. This can be seen on [GITHUB](https://github.com/minitriga/NSO-Final-Year/tree/master/DEPLOY-DMVPN) or within the NSO VM at `/var/opt/ncs/packages/DEPLOY-DMVPN`

## Demonstrate Deploying the Service CLI

```
services DEPLOY-DMVPN ENT
Cisco1234
HUB SPOKE-1
10.1.1.1
255.255.255.0
GigabitEthernet
0/0
100
eigrp 10.0.0.0
0.255.255.255
eigrp 192.168.0.0
0.0.255.255

SPOKE SPOKE-2
10.1.1.2
255.255.255.0
10.1.1.1
65.100.123.3
GigabitEthernet
0/0
102
eigrp 10.0.0.0
0.255.255.255
eigrp 192.168.0.0
0.0.255.255

SPOKE SPOKE-3
10.1.1.3
255.255.255.0
10.1.1.1
65.100.123.3
GigabitEthernet
0/0
103
eigrp 10.0.0.0
0.255.255.255
eigrp 192.168.0.0
0.0.255.255

SPOKE SPOKE-4
10.1.1.4
255.255.255.0
10.1.1.1
65.100.123.3
GigabitEthernet
0/0
104
eigrp 10.0.0.0
0.255.255.255
eigrp 192.168.0.0
0.0.255.255

```

Varify what we want to commit with:
```
commit dry-run outformat native
commit dry-run outformat cli
commit dry-run outformat xml
```

Commit the service with `commit`

Varify Phase 2 DMVPN on any SPOKE with:
```
show dmvpn
ping <loopback>
traceroute <loopback>

show crypto isakmp sa 
```

## Adding Device to the Service

```
services DEPLOY-DMVPN ENT

SPOKE SPOKE-5
10.1.1.5
255.255.255.0
10.1.1.1
65.100.123.3
GigabitEthernet
0/0
104
eigrp 10.0.0.0
0.255.255.255
eigrp 192.168.0.0
0.0.255.255
```

Varify what we want to commit with:
```
commit dry-run outformat native
commit dry-run outformat cli
commit dry-run outformat xml
```

Commit the service with `commit`

Varify Phase 2 DMVPN on any SPOKE with:
```
show dmvpn
ping <loopback>
traceroute <loopback>

show crypto isakmp sa 
```
## Remove Device from the Service

```
services DEPLOY-DMVPN ENT
no SPOKE SPOKE-5
commit
```

# Add SPOKE-5 though API

To add a device through API you need to have some headers set:
- Content-Type - application/vnd.yang.data+json
- Authorization - application/vnd.yang.data+json

Send this **POST** request to `7.5.17.78:8080/api/config/devices`
```
{
    "tailf-ncs:device": {
        "name": "SPOKE-5",
        "address": "172.16.10.7",
        "port": 22,
        "authgroup": "Routers",
        "device-type": {
            "cli": [
            {
            "ned-id": "tailf-ned-cisco-ios-id:cisco-ios",
          "protocol": "ssh"
          }
          ]
        },
        "state": {
            "admin-state": "unlocked"
        }
    }
}
```

## Add SPOKE to Device-Group

To add a device through API you need to have some headers set:
- Content-Type - application/vnd.yang.data+json
- Authorization - application/vnd.yang.data+json

Send this **PATCH** request to `7.5.17.78:8080/api/config/devices/device-group`

```
{
    "tailf-ncs:device-group": {
        "name": "SPOKES",
        "device-name": [
        	"SPOKE-5"
        ]
    }
}
```

## Delete SPOKE from Device-Group

Send this **DELETE** request to `7.5.17.78:8080/api/config/devices/device/SPOKE-5`

## Delete Service

Send this **DELETE** request to `7.5.17.78:8080/api/running/services/DEPLOY-DMVPN:DEPLOY-DMVPN/ENT`



# Scale DMVPN Network Through API

This section will show you how to scale the DMVPN Service up with multiple devices. 

## Add Devices

To add SPOKE-5 to SPOKE-15 run the `ADD ALL DEVICES` in POSTMAN.

## Add New Devices to Group

Run the `ADD ALL TO GROUP` in POSTMAN.

## Verify in NSO UI

## Create Service with New Devices

To add a device through API you need to have some headers set:
- Content-Type - application/vnd.yang.data+json
- Authorization - application/vnd.yang.data+json

Send this **POST** request to `7.5.17.78:8080/api/config/services/`

```
{
    "DEPLOY-DMVPN:DEPLOY-DMVPN": {
        "name": "ENT",
        "ipsec-secret": "Cisco1234",
        "HUB": [
            {
                "device": "SPOKE-1",
                "hub-tunnel-address": "10.1.1.1",
                "hub-tunnel-mask": "255.255.255.0",
                "interface-type": "GigabitEthernet",
                "interface-number": "0/0",
                "sub-interface-number": 100,
                "eigrp": [
                    {
                        "network": "10.0.0.0",
                        "network-mask": "0.255.255.255"
                    },
                    {
                        "network": "192.168.0.0",
                        "network-mask": "0.0.255.255"
                    }
                ]
            }
        ],
        "SPOKE": [
            {
                "device": "SPOKE-2",
                "spoke-tunnel-address": "10.1.1.2",
                "spoke-tunnel-mask": "255.255.255.0",
                "hub-tunnel-address": "10.1.1.1",
                "hub-public-address": "65.100.123.3",
                "interface-type": "GigabitEthernet",
                "interface-number": "0/0",
                "sub-interface-number": 102,
                "eigrp": [
                    {
                        "network": "10.0.0.0",
                        "network-mask": "0.255.255.255"
                    },
                    {
                        "network": "192.168.0.0",
                        "network-mask": "0.0.255.255"
                    }
                ]
            },
            {
                "device": "SPOKE-3",
                "spoke-tunnel-address": "10.1.1.3",
                "spoke-tunnel-mask": "255.255.255.0",
                "hub-tunnel-address": "10.1.1.1",
                "hub-public-address": "65.100.123.3",
                "interface-type": "GigabitEthernet",
                "interface-number": "0/0",
                "sub-interface-number": 103,
                "eigrp": [
                    {
                        "network": "10.0.0.0",
                        "network-mask": "0.255.255.255"
                    },
                    {
                        "network": "192.168.0.0",
                        "network-mask": "0.0.255.255"
                    }
                ]
            },
            {
                "device": "SPOKE-4",
                "spoke-tunnel-address": "10.1.1.4",
                "spoke-tunnel-mask": "255.255.255.0",
                "hub-tunnel-address": "10.1.1.1",
                "hub-public-address": "65.100.123.3",
                "interface-type": "GigabitEthernet",
                "interface-number": "0/0",
                "sub-interface-number": 104,
                "eigrp": [
                    {
                        "network": "10.0.0.0",
                        "network-mask": "0.255.255.255"
                    },
                    {
                        "network": "192.168.0.0",
                        "network-mask": "0.0.255.255"
                    }
                ]
            },
            {
                "device": "SPOKE-5",
                "spoke-tunnel-address": "10.1.1.5",
                "spoke-tunnel-mask": "255.255.255.0",
                "hub-tunnel-address": "10.1.1.1",
                "hub-public-address": "65.100.123.3",
                "interface-type": "GigabitEthernet",
                "interface-number": "0/0",
                "sub-interface-number": 105,
                "eigrp": [
                    {
                        "network": "10.0.0.0",
                        "network-mask": "0.255.255.255"
                    },
                    {
                        "network": "192.168.0.0",
                        "network-mask": "0.0.255.255"
                    }
                ]
            },
            {
                "device": "SPOKE-6",
                "spoke-tunnel-address": "10.1.1.6",
                "spoke-tunnel-mask": "255.255.255.0",
                "hub-tunnel-address": "10.1.1.1",
                "hub-public-address": "65.100.123.3",
                "interface-type": "GigabitEthernet",
                "interface-number": "0/0",
                "sub-interface-number": 106,
                "eigrp": [
                    {
                        "network": "10.0.0.0",
                        "network-mask": "0.255.255.255"
                    },
                    {
                        "network": "192.168.0.0",
                        "network-mask": "0.0.255.255"
                    }
                ]
            },
            {
                "device": "SPOKE-7",
                "spoke-tunnel-address": "10.1.1.7",
                "spoke-tunnel-mask": "255.255.255.0",
                "hub-tunnel-address": "10.1.1.1",
                "hub-public-address": "65.100.123.3",
                "interface-type": "GigabitEthernet",
                "interface-number": "0/0",
                "sub-interface-number": 107,
                "eigrp": [
                    {
                        "network": "10.0.0.0",
                        "network-mask": "0.255.255.255"
                    },
                    {
                        "network": "192.168.0.0",
                        "network-mask": "0.0.255.255"
                    }
                ]
            },
            {
                "device": "SPOKE-8",
                "spoke-tunnel-address": "10.1.1.8",
                "spoke-tunnel-mask": "255.255.255.0",
                "hub-tunnel-address": "10.1.1.1",
                "hub-public-address": "65.100.123.3",
                "interface-type": "GigabitEthernet",
                "interface-number": "0/0",
                "sub-interface-number": 108,
                "eigrp": [
                    {
                        "network": "10.0.0.0",
                        "network-mask": "0.255.255.255"
                    },
                    {
                        "network": "192.168.0.0",
                        "network-mask": "0.0.255.255"
                    }
                ]
            },
            {
                "device": "SPOKE-9",
                "spoke-tunnel-address": "10.1.1.9",
                "spoke-tunnel-mask": "255.255.255.0",
                "hub-tunnel-address": "10.1.1.1",
                "hub-public-address": "65.100.123.3",
                "interface-type": "GigabitEthernet",
                "interface-number": "0/0",
                "sub-interface-number": 109,
                "eigrp": [
                    {
                        "network": "10.0.0.0",
                        "network-mask": "0.255.255.255"
                    },
                    {
                        "network": "192.168.0.0",
                        "network-mask": "0.0.255.255"
                    }
                ]
            },
            {
                "device": "SPOKE-10",
                "spoke-tunnel-address": "10.1.1.10",
                "spoke-tunnel-mask": "255.255.255.0",
                "hub-tunnel-address": "10.1.1.1",
                "hub-public-address": "65.100.123.3",
                "interface-type": "GigabitEthernet",
                "interface-number": "0/0",
                "sub-interface-number": 110,
                "eigrp": [
                    {
                        "network": "10.0.0.0",
                        "network-mask": "0.255.255.255"
                    },
                    {
                        "network": "192.168.0.0",
                        "network-mask": "0.0.255.255"
                    }
                ]
            },
            {
                "device": "SPOKE-11",
                "spoke-tunnel-address": "10.1.1.11",
                "spoke-tunnel-mask": "255.255.255.0",
                "hub-tunnel-address": "10.1.1.1",
                "hub-public-address": "65.100.123.3",
                "interface-type": "GigabitEthernet",
                "interface-number": "0/0",
                "sub-interface-number": 111,
                "eigrp": [
                    {
                        "network": "10.0.0.0",
                        "network-mask": "0.255.255.255"
                    },
                    {
                        "network": "192.168.0.0",
                        "network-mask": "0.0.255.255"
                    }
                ]
            },
            {
                "device": "SPOKE-12",
                "spoke-tunnel-address": "10.1.1.12",
                "spoke-tunnel-mask": "255.255.255.0",
                "hub-tunnel-address": "10.1.1.1",
                "hub-public-address": "65.100.123.3",
                "interface-type": "GigabitEthernet",
                "interface-number": "0/0",
                "sub-interface-number": 112,
                "eigrp": [
                    {
                        "network": "10.0.0.0",
                        "network-mask": "0.255.255.255"
                    },
                    {
                        "network": "192.168.0.0",
                        "network-mask": "0.0.255.255"
                    }
                ]
            },
            {
                "device": "SPOKE-13",
                "spoke-tunnel-address": "10.1.1.13",
                "spoke-tunnel-mask": "255.255.255.0",
                "hub-tunnel-address": "10.1.1.1",
                "hub-public-address": "65.100.123.3",
                "interface-type": "GigabitEthernet",
                "interface-number": "0/0",
                "sub-interface-number": 113,
                "eigrp": [
                    {
                        "network": "10.0.0.0",
                        "network-mask": "0.255.255.255"
                    },
                    {
                        "network": "192.168.0.0",
                        "network-mask": "0.0.255.255"
                    }
                ]
            },
            {
                "device": "SPOKE-14",
                "spoke-tunnel-address": "10.1.1.14",
                "spoke-tunnel-mask": "255.255.255.0",
                "hub-tunnel-address": "10.1.1.1",
                "hub-public-address": "65.100.123.3",
                "interface-type": "GigabitEthernet",
                "interface-number": "0/0",
                "sub-interface-number": 114,
                "eigrp": [
                    {
                        "network": "10.0.0.0",
                        "network-mask": "0.255.255.255"
                    },
                    {
                        "network": "192.168.0.0",
                        "network-mask": "0.0.255.255"
                    }
                ]
            },
            {
                "device": "SPOKE-15",
                "spoke-tunnel-address": "10.1.1.15",
                "spoke-tunnel-mask": "255.255.255.0",
                "hub-tunnel-address": "10.1.1.1",
                "hub-public-address": "65.100.123.3",
                "interface-type": "GigabitEthernet",
                "interface-number": "0/0",
                "sub-interface-number": 115,
                "eigrp": [
                    {
                        "network": "10.0.0.0",
                        "network-mask": "0.255.255.255"
                    },
                    {
                        "network": "192.168.0.0",
                        "network-mask": "0.0.255.255"
                    }
                ]
            }
        ]
    }
}
```

## Verify Service

To see all services:
Send this **GET** request to `7.5.17.78:8080/api/running/services/DEPLOY-DMVPN`

To see one service in detail
Send this **GET** request to `7.5.17.78:8080/api/running/services/DEPLOY-DMVPN/ENT?deep`



