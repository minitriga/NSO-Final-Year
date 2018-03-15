#Deploy Tennant Script

This will allow you to be able to deploy a tennant using NSO with options to enable BFD, BGP SOO and IPv6 support
```
services deploy-site Volvo
5
2894
true
true
true
pe CE-5-A
CSR1
192.0.2.9
GigabitEthernet
1
100.64.1.1
255.255.255.252
2001:db8:1:310::1/64
100.64.1.2
2001:db8:1:310::2
64086.59904
65412

cpe CE-5-A
CSR2
GigabitEthernet
1
100.64.1.2
255.255.255.252
2001:db8:1:310::2/64
100.64.1.1
2001:db8:1:310::1
65412
64086.59904
commit dry-run outformat native 

```
