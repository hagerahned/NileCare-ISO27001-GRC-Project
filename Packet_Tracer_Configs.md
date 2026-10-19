# NileCare - Network Design and Packet Tracer Configurations

Fictional lab for a GRC / network security practice project. 3 sites: Branch 1 (HQ), Branch 2, Branch 3. All names, IPs and passwords are for lab use only.

![NileCare topology](NileCare_Network_PacketTracer_style.png)

Illustration of the topology in Packet Tracer style. Build it in Packet Tracer with the devices and configs below and replace this picture with a real screenshot.


## 1. Devices (Packet Tracer models)

FW-HQ: ASA 5506-X | Core-SW1: 3650-24PS | SW-ACCESS, SW-SERVERS, SW-B2, SW-B3: 2960-24TT | ISP, R-BRANCH2, R-BRANCH3: 2911 | 3 servers, PCs, 3 Access Points. Use straight-through copper cables.


## 2. Addressing plan

| VLAN | Name | Branch 1 (HQ) | Branch 2 | Branch 3 |
|---|---|---|---|---|
| 10 | Clinical | 10.1.10.0/24 | 10.2.10.0/24 | 10.3.10.0/24 |
| 20 | Admin | 10.1.20.0/24 | 10.2.20.0/24 | 10.3.20.0/24 |
| 30 | POS | 10.1.30.0/24 | 10.2.30.0/24 | 10.3.30.0/24 |
| 40 | Servers | 10.1.40.0/24 | - | - |
| 50 | Guest Wi-Fi | 10.1.50.0/24 | 10.2.50.0/24 | 10.3.50.0/24 |
| 99 | Management | 10.1.99.0/24 | - | - |

WAN links: Core to FW 10.1.0.0/30 (.1 / .2) | FW to ISP 200.0.1.0/30 (.1 / .2) | ISP to Branch 2 200.0.2.0/30 (.1 / .2) | ISP to Branch 3 200.0.3.0/30 (.1 / .2) | ISP Loopback 8.8.8.8 (test internet).

| Device | IP / gateway |
|---|---|
| EMR-SRV | 10.1.40.10 /24, gw 10.1.40.1 |
| AD-SRV | 10.1.40.11 /24, gw 10.1.40.1 |
| BKP-NAS | 10.1.40.12 /24, gw 10.1.40.1 |
| All PCs | DHCP (VLAN 10, 20, 30, 50) |
| Guest-AP | SSID NileCare-Guest, wired into VLAN 50 port |


## 3. Cabling

| From | Port | To | Port |
|---|---|---|---|
| Core-SW1 | Gi1/0/1 | SW-ACCESS | Gi0/1 |
| Core-SW1 | Gi1/0/2 | SW-SERVERS | Gi0/1 |
| Core-SW1 | Gi1/0/24 | FW-HQ | Gi1/1 |
| FW-HQ | Gi1/2 | ISP | Gi0/0 |
| ISP | Gi0/1 | R-BRANCH2 | Gi0/0 |
| ISP | Gi0/2 | R-BRANCH3 | Gi0/0 |
| R-BRANCH2 | Gi0/1 | SW-B2 | Gi0/1 |
| R-BRANCH3 | Gi0/1 | SW-B3 | Gi0/1 |
| SW-ACCESS | Fa0/1 to Fa0/4 | PC-Clinical, PC-Admin, POS, Guest-AP | Fa0 |
| SW-SERVERS | Fa0/1 to Fa0/3 | EMR-SRV, AD-SRV, BKP-NAS | Fa0 |
| SW-B2 / SW-B3 | Fa0/1 to Fa0/4 | PC-Clinical, PC-Admin, POS, Guest-AP | Fa0 |


## 4. Baseline for every device (not the ASA)

Paste first, replacing the hostname. Lab password: NileCare@2026. Then apply the device config in the next sections.

```
enable
configure terminal
hostname <DEVICE-NAME>
no ip domain-lookup
service password-encryption
enable secret NileCare@2026
username admin secret NileCare@2026
ip domain-name nilecare.local
crypto key generate rsa general-keys modulus 1024
ip ssh version 2
banner motd # Authorized access only. Activity is logged. #
line console 0
 logging synchronous
 login local
line vty 0 4
 login local
 transport input ssh
end
write memory
```


## 5. ISP router

```
hostname ISP
interface Loopback0
 description Test "internet" target
 ip address 8.8.8.8 255.255.255.255
interface GigabitEthernet0/0
 description To FW-HQ
 ip address 200.0.1.2 255.255.255.252
 no shutdown
interface GigabitEthernet0/1
 description To R-BRANCH2
 ip address 200.0.2.1 255.255.255.252
 no shutdown
interface GigabitEthernet0/2
 description To R-BRANCH3
 ip address 200.0.3.1 255.255.255.252
 no shutdown
end
write memory
```


## 6. Core-SW1 (3650)

If the switch rejects a command, check the port name with show ip interface brief. The 3650 does not need the trunk encapsulation command.

```
hostname Core-SW1
ip routing
vlan 10
 name CLINICAL
vlan 20
 name ADMIN
vlan 30
 name POS
vlan 40
 name SERVERS
vlan 50
 name GUEST
vlan 99
 name MGMT
! Trunks to access and server switches
interface range GigabitEthernet1/0/1 - 2
 switchport mode trunk
 no shutdown
! Routed port to the firewall
interface GigabitEthernet1/0/24
 description To FW-HQ
 no switchport
 ip address 10.1.0.1 255.255.255.252
 no shutdown
! Gateways (SVIs)
interface Vlan10
 ip address 10.1.10.1 255.255.255.0
interface Vlan20
 ip address 10.1.20.1 255.255.255.0
interface Vlan30
 ip address 10.1.30.1 255.255.255.0
 ip access-group POS_IN in
interface Vlan40
 ip address 10.1.40.1 255.255.255.0
interface Vlan50
 ip address 10.1.50.1 255.255.255.0
 ip access-group GUEST_IN in
interface Vlan99
 ip address 10.1.99.1 255.255.255.0
ip route 0.0.0.0 0.0.0.0 10.1.0.2
! Isolation: guest and POS cannot reach internal networks, internet allowed
ip access-list extended GUEST_IN
 deny ip 10.1.50.0 0.0.0.255 10.0.0.0 0.255.255.255
 permit ip any any
ip access-list extended POS_IN
 deny ip 10.1.30.0 0.0.0.255 10.0.0.0 0.255.255.255
 permit ip any any
! DHCP
ip dhcp excluded-address 10.1.10.1 10.1.10.9
ip dhcp excluded-address 10.1.20.1 10.1.20.9
ip dhcp excluded-address 10.1.30.1 10.1.30.9
ip dhcp excluded-address 10.1.50.1 10.1.50.9
ip dhcp pool CLINICAL
 network 10.1.10.0 255.255.255.0
 default-router 10.1.10.1
 dns-server 8.8.8.8
ip dhcp pool ADMIN
 network 10.1.20.0 255.255.255.0
 default-router 10.1.20.1
 dns-server 8.8.8.8
ip dhcp pool POS
 network 10.1.30.0 255.255.255.0
 default-router 10.1.30.1
 dns-server 8.8.8.8
ip dhcp pool GUEST
 network 10.1.50.0 255.255.255.0
 default-router 10.1.50.1
 dns-server 8.8.8.8
end
write memory
```


## 7. SW-ACCESS (2960)

```
hostname SW-ACCESS
vlan 10
 name CLINICAL
vlan 20
 name ADMIN
vlan 30
 name POS
vlan 50
 name GUEST
vlan 99
 name MGMT
interface GigabitEthernet0/1
 description Trunk to Core-SW1
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,50,99
interface FastEthernet0/1
 description PC-Clinical
 switchport mode access
 switchport access vlan 10
 spanning-tree portfast
interface FastEthernet0/2
 description PC-Admin
 switchport mode access
 switchport access vlan 20
 spanning-tree portfast
interface FastEthernet0/3
 description POS-Terminal
 switchport mode access
 switchport access vlan 30
 spanning-tree portfast
interface FastEthernet0/4
 description Guest-AP
 switchport mode access
 switchport access vlan 50
interface range FastEthernet0/1 - 3
 switchport port-security
 switchport port-security maximum 2
 switchport port-security violation restrict
! Shut unused ports
interface range FastEthernet0/5 - 24
 shutdown
interface Vlan99
 ip address 10.1.99.11 255.255.255.0
 no shutdown
ip default-gateway 10.1.99.1
end
write memory
```


## 8. SW-SERVERS (2960)

```
hostname SW-SERVERS
vlan 40
 name SERVERS
vlan 99
 name MGMT
interface GigabitEthernet0/1
 description Trunk to Core-SW1
 switchport mode trunk
 switchport trunk allowed vlan 40,99
interface range FastEthernet0/1 - 3
 switchport mode access
 switchport access vlan 40
 spanning-tree portfast
interface range FastEthernet0/4 - 24
 shutdown
interface Vlan99
 ip address 10.1.99.12 255.255.255.0
 no shutdown
ip default-gateway 10.1.99.1
end
write memory
```


## 9. FW-HQ (ASA 5506-X) - base

```
hostname FW-HQ
interface GigabitEthernet1/1
 nameif inside
 security-level 100
 ip address 10.1.0.2 255.255.255.252
 no shutdown
interface GigabitEthernet1/2
 nameif outside
 security-level 0
 ip address 200.0.1.1 255.255.255.252
 no shutdown
route outside 0.0.0.0 0.0.0.0 200.0.1.2
route inside 10.1.0.0 255.255.0.0 10.1.0.1
! NAT: HQ networks to the internet
object network HQ-LAN
 subnet 10.1.0.0 255.255.0.0
 nat (inside,outside) dynamic interface
! Allow ping replies back in
policy-map global_policy
 class inspection_default
  inspect icmp
end
write memory
```


## 10. R-BRANCH2 (2911)

```
hostname R-BRANCH2
interface GigabitEthernet0/0
 description WAN to ISP
 ip address 200.0.2.2 255.255.255.252
 ip nat outside
 no shutdown
interface GigabitEthernet0/1
 description LAN trunk to SW-B2
 no ip address
 no shutdown
interface GigabitEthernet0/1.10
 encapsulation dot1Q 10
 ip address 10.2.10.1 255.255.255.0
 ip nat inside
interface GigabitEthernet0/1.20
 encapsulation dot1Q 20
 ip address 10.2.20.1 255.255.255.0
 ip nat inside
interface GigabitEthernet0/1.30
 encapsulation dot1Q 30
 ip address 10.2.30.1 255.255.255.0
 ip nat inside
 ip access-group POS_IN in
interface GigabitEthernet0/1.50
 encapsulation dot1Q 50
 ip address 10.2.50.1 255.255.255.0
 ip nat inside
 ip access-group GUEST_IN in
ip route 0.0.0.0 0.0.0.0 200.0.2.1
ip access-list extended GUEST_IN
 deny ip 10.2.50.0 0.0.0.255 10.0.0.0 0.255.255.255
 permit ip any any
ip access-list extended POS_IN
 deny ip 10.2.30.0 0.0.0.255 10.0.0.0 0.255.255.255
 permit ip any any
! NAT, skipping VPN traffic to HQ
ip access-list extended NAT-ACL
 deny ip 10.2.0.0 0.0.255.255 10.1.0.0 0.0.255.255
 permit ip 10.2.0.0 0.0.255.255 any
ip nat inside source list NAT-ACL interface GigabitEthernet0/0 overload
ip dhcp excluded-address 10.2.10.1 10.2.10.9
ip dhcp excluded-address 10.2.20.1 10.2.20.9
ip dhcp excluded-address 10.2.30.1 10.2.30.9
ip dhcp excluded-address 10.2.50.1 10.2.50.9
ip dhcp pool CLINICAL
 network 10.2.10.0 255.255.255.0
 default-router 10.2.10.1
 dns-server 8.8.8.8
ip dhcp pool ADMIN
 network 10.2.20.0 255.255.255.0
 default-router 10.2.20.1
 dns-server 8.8.8.8
ip dhcp pool POS
 network 10.2.30.0 255.255.255.0
 default-router 10.2.30.1
 dns-server 8.8.8.8
ip dhcp pool GUEST
 network 10.2.50.0 255.255.255.0
 default-router 10.2.50.1
 dns-server 8.8.8.8
end
write memory
```


## 11. SW-B2 (2960)

```
hostname SW-B2
vlan 10
 name CLINICAL
vlan 20
 name ADMIN
vlan 30
 name POS
vlan 50
 name GUEST
interface GigabitEthernet0/1
 description Trunk to R-BRANCH2
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,50
interface FastEthernet0/1
 description PC-Clinical
 switchport mode access
 switchport access vlan 10
 spanning-tree portfast
interface FastEthernet0/2
 description PC-Admin
 switchport mode access
 switchport access vlan 20
 spanning-tree portfast
interface FastEthernet0/3
 description POS-Terminal
 switchport mode access
 switchport access vlan 30
 spanning-tree portfast
interface FastEthernet0/4
 description Guest-AP
 switchport mode access
 switchport access vlan 50
interface range FastEthernet0/1 - 3
 switchport port-security
 switchport port-security maximum 2
 switchport port-security violation restrict
interface range FastEthernet0/5 - 24
 shutdown
end
write memory
```


## 12. R-BRANCH3 (2911)

```
hostname R-BRANCH3
interface GigabitEthernet0/0
 description WAN to ISP
 ip address 200.0.3.2 255.255.255.252
 ip nat outside
 no shutdown
interface GigabitEthernet0/1
 description LAN trunk to SW-B3
 no ip address
 no shutdown
interface GigabitEthernet0/1.10
 encapsulation dot1Q 10
 ip address 10.3.10.1 255.255.255.0
 ip nat inside
interface GigabitEthernet0/1.20
 encapsulation dot1Q 20
 ip address 10.3.20.1 255.255.255.0
 ip nat inside
interface GigabitEthernet0/1.30
 encapsulation dot1Q 30
 ip address 10.3.30.1 255.255.255.0
 ip nat inside
 ip access-group POS_IN in
interface GigabitEthernet0/1.50
 encapsulation dot1Q 50
 ip address 10.3.50.1 255.255.255.0
 ip nat inside
 ip access-group GUEST_IN in
ip route 0.0.0.0 0.0.0.0 200.0.3.1
ip access-list extended GUEST_IN
 deny ip 10.3.50.0 0.0.0.255 10.0.0.0 0.255.255.255
 permit ip any any
ip access-list extended POS_IN
 deny ip 10.3.30.0 0.0.0.255 10.0.0.0 0.255.255.255
 permit ip any any
! NAT, skipping VPN traffic to HQ
ip access-list extended NAT-ACL
 deny ip 10.3.0.0 0.0.255.255 10.1.0.0 0.0.255.255
 permit ip 10.3.0.0 0.0.255.255 any
ip nat inside source list NAT-ACL interface GigabitEthernet0/0 overload
ip dhcp excluded-address 10.3.10.1 10.3.10.9
ip dhcp excluded-address 10.3.20.1 10.3.20.9
ip dhcp excluded-address 10.3.30.1 10.3.30.9
ip dhcp excluded-address 10.3.50.1 10.3.50.9
ip dhcp pool CLINICAL
 network 10.3.10.0 255.255.255.0
 default-router 10.3.10.1
 dns-server 8.8.8.8
ip dhcp pool ADMIN
 network 10.3.20.0 255.255.255.0
 default-router 10.3.20.1
 dns-server 8.8.8.8
ip dhcp pool POS
 network 10.3.30.0 255.255.255.0
 default-router 10.3.30.1
 dns-server 8.8.8.8
ip dhcp pool GUEST
 network 10.3.50.0 255.255.255.0
 default-router 10.3.50.1
 dns-server 8.8.8.8
end
write memory
```


## 13. SW-B3 (2960)

```
hostname SW-B3
vlan 10
 name CLINICAL
vlan 20
 name ADMIN
vlan 30
 name POS
vlan 50
 name GUEST
interface GigabitEthernet0/1
 description Trunk to R-BRANCH3
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,50
interface FastEthernet0/1
 description PC-Clinical
 switchport mode access
 switchport access vlan 10
 spanning-tree portfast
interface FastEthernet0/2
 description PC-Admin
 switchport mode access
 switchport access vlan 20
 spanning-tree portfast
interface FastEthernet0/3
 description POS-Terminal
 switchport mode access
 switchport access vlan 30
 spanning-tree portfast
interface FastEthernet0/4
 description Guest-AP
 switchport mode access
 switchport access vlan 50
interface range FastEthernet0/1 - 3
 switchport port-security
 switchport port-security maximum 2
 switchport port-security violation restrict
interface range FastEthernet0/5 - 24
 shutdown
end
write memory
```


## 14. IPsec VPN (do this last, after sections 1 to 13 work)

Hub and spoke: each branch has a tunnel to HQ. Branch-to-branch traffic is not part of this design. Packet Tracer's ASA may not accept the IPsec commands in some versions. If so, replace FW-HQ with a 2911 router using the same pattern as the branch VPN below (peer = the branch WAN address, crypto ACL 10.1.0.0/16 to 10.x.0.0/16).

FW-HQ:

```
object network B2-LAN
 subnet 10.2.0.0 255.255.0.0
object network B3-LAN
 subnet 10.3.0.0 255.255.0.0
! Do not NAT VPN traffic
nat (inside,outside) source static HQ-LAN HQ-LAN destination static B2-LAN B2-LAN
nat (inside,outside) source static HQ-LAN HQ-LAN destination static B3-LAN B3-LAN
access-list VPN-B2 extended permit ip 10.1.0.0 255.255.0.0 10.2.0.0 255.255.0.0
access-list VPN-B3 extended permit ip 10.1.0.0 255.255.0.0 10.3.0.0 255.255.0.0
crypto ikev1 policy 10
 authentication pre-share
 encryption aes
 hash sha
 group 2
 lifetime 86400
crypto ipsec ikev1 transform-set NC-TS esp-aes esp-sha-hmac
crypto map NC-MAP 10 match address VPN-B2
crypto map NC-MAP 10 set peer 200.0.2.2
crypto map NC-MAP 10 set ikev1 transform-set NC-TS
crypto map NC-MAP 20 match address VPN-B3
crypto map NC-MAP 20 set peer 200.0.3.2
crypto map NC-MAP 20 set ikev1 transform-set NC-TS
crypto map NC-MAP interface outside
crypto ikev1 enable outside
tunnel-group 200.0.2.2 type ipsec-l2l
tunnel-group 200.0.2.2 ipsec-attributes
 ikev1 pre-shared-key NileCare-VPN-Lab-Key
tunnel-group 200.0.3.2 type ipsec-l2l
tunnel-group 200.0.3.2 ipsec-attributes
 ikev1 pre-shared-key NileCare-VPN-Lab-Key
end
write memory
```

R-BRANCH2:

```
! Step 1 - enable the security license, then reload (save first)
license boot module c2900 technology-package securityk9
write memory
reload
! Step 2 - after reload
crypto isakmp policy 10
 encryption aes
 authentication pre-share
 group 2
 hash sha
 lifetime 86400
crypto isakmp key NileCare-VPN-Lab-Key address 200.0.1.1
crypto ipsec transform-set NC-TS esp-aes esp-sha-hmac
ip access-list extended VPN-TRAFFIC
 permit ip 10.2.0.0 0.0.255.255 10.1.0.0 0.0.255.255
crypto map NC-MAP 10 ipsec-isakmp
 set peer 200.0.1.1
 set transform-set NC-TS
 match address VPN-TRAFFIC
interface GigabitEthernet0/0
 crypto map NC-MAP
end
write memory
```

R-BRANCH3:

```
! Step 1 - enable the security license, then reload (save first)
license boot module c2900 technology-package securityk9
write memory
reload
! Step 2 - after reload
crypto isakmp policy 10
 encryption aes
 authentication pre-share
 group 2
 hash sha
 lifetime 86400
crypto isakmp key NileCare-VPN-Lab-Key address 200.0.1.1
crypto ipsec transform-set NC-TS esp-aes esp-sha-hmac
ip access-list extended VPN-TRAFFIC
 permit ip 10.3.0.0 0.0.255.255 10.1.0.0 0.0.255.255
crypto map NC-MAP 10 ipsec-isakmp
 set peer 200.0.1.1
 set transform-set NC-TS
 match address VPN-TRAFFIC
interface GigabitEthernet0/0
 crypto map NC-MAP
end
write memory
```


## 15. Tests

| Test (from) | Command | Expected |
|---|---|---|
| PC-Clinical HQ | ping 10.1.40.10 | Success |
| PC-Clinical HQ | ping 8.8.8.8 | Success (NAT) |
| POS-Terminal HQ | ping 10.1.40.10 | Fails (ACL POS_IN) |
| POS-Terminal HQ | ping 8.8.8.8 | Success |
| PC-Clinical Branch 2 | ping 10.1.40.10 | Success (VPN) |
| PC-Clinical Branch 3 | ping 10.1.40.10 | Success (VPN) |
| POS-Terminal Branch 2 | ping 10.2.10.1 or any 10.x host | Fails (ACL POS_IN) |

Useful commands: show vlan brief | show ip interface brief | show ip route | show ip dhcp binding | show access-lists | show ip nat translations | show crypto isakmp sa | show crypto ipsec sa | ASA: show crypto ikev1 sa


## 16. Mapping to the gap assessment

| Design element | ISO 27001:2022 Annex A |
|---|---|
| VLANs per function, guest and POS isolated by ACLs | A.8.22 Segregation of networks |
| Firewall, NAT, routed inside/outside zones | A.8.20 Networks security |
| IPsec VPN between sites | A.8.24 Use of cryptography; A.8.21 Network services |
| Local accounts, SSH only, no Telnet, password encryption | A.8.5 Secure authentication; A.8.9 Configuration management |
| Port security, unused ports shut down | A.7.2 Physical entry (network access); A.8.20 |
| Management VLAN 99 | A.8.9 Configuration management |

Closing these gaps in the lab (segmentation, firewall, encryption, secure management) addresses risks R-01, R-09, R-10 and R-11 in the risk register.
