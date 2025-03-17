# Cisco-IPSEC-IKEv2-Site-to-Site-VPN
Site-to-Site IPSec IKEv2 VPN Tunnel is used to allow the secure transmission of data, voice and video between two sites (e.g offices or branches). The VPN tunnel is created over the Internet public network and encrypted using a number of advanced encryption algorithms to provide confidentiality of the data transmitted between the two sites.
# Cisco IPSEC IKEv2 Site-to-Site VPN Configuration
<img src="https://i.imgur.com/E3ZLWkl.png" height="80%" alt="Topology" />
## Overview
This project demonstrates the configuration of an IPSEC IKEv2 Site-to-Site VPN between two Cisco Adaptive Security Appliances (ASA) using EVE-NG. The VPN tunnel allows secure data transmission between two remote sites over an encrypted connection.

## Lab Setup
The topology consists of:
- **ASAv1** - Represents Site 1
- **ASAv2** - Represents Site 2
- **Host-PC1** - A client at Site 1
- **Host-PC2** - A client at Site 2

### **Image Versions Used**
- **ASA Version:** `ASAv 9.14.2`
- **Router Version:** `i86bi_LinuxL3-AdvEnterpriseK9-M2_157_3_May_2018`
- **Switch Version:** `i86bi_linux_l2-adventerprisek9-ms.SSA.high_iron_20190423.bin`
- **VPCS Node** - Simulated PC hosts

### **Objective**
- Configure a **Site-to-Site VPN using IKEv2** between ASAv1 and ASAv2.
- Allow **Host-PC1 to communicate securely with Host-PC2**.

## Configuration Steps

### 1. Configure ASAv Interfaces and Default Routes
#### **ASAv1 (Site 1)**
```shell
hostname ASAv1
enable password cisco

interface GigabitEthernet0/0
 no shut
 nameif outside
 security-level 0
 ip address 20.1.1.1 255.255.255.0

interface GigabitEthernet0/1
 no shut
 nameif inside
 security-level 100
 ip address 192.168.101.254 255.255.255.0

route outside 0.0.0.0 0.0.0.0 20.1.1.254 1
```

#### **ASAv2 (Site 2)**
```shell
hostname ASAv2
enable password cisco

interface GigabitEthernet0/0
 no shut
 nameif outside
 security-level 0
 ip address 30.1.1.1 255.255.255.0

interface GigabitEthernet0/1
 no shut
 nameif inside
 security-level 100
 ip address 192.168.102.254 255.255.255.0

route outside 0.0.0.0 0.0.0.0 30.1.1.254 1
```

### 2. Creating ASAv Objects
#### **ASAv1 (Site 1)**
```shell
object network remote-network
 subnet 192.168.102.0 255.255.255.0
object network inside-network
 subnet 192.168.101.0 255.255.255.0
```

#### **ASAv2 (Site 2)**
```shell
object network remote-network
 subnet 192.168.101.0 255.255.255.0
object network inside-network
 subnet 192.168.102.0 255.255.255.0
```

### 3. Configure DHCP Pools for Inside Network
#### **ASAv1**
```shell
dhcpd address 192.168.101.10-192.168.101.20 inside
dhcpd enable inside
```

#### **ASAv2**
```shell
dhcpd address 192.168.102.10-192.168.102.20 inside
dhcpd enable inside
```

### 4. Create Access Lists for VPN Traffic
#### **ASAv1**
```shell
access-list outside_cryptomap extended permit ip 192.168.101.0 255.255.255.0 192.168.102.0 255.255.255.0
```

#### **ASAv2**
```shell
access-list outside_cryptomap extended permit ip 192.168.102.0 255.255.255.0 192.168.101.0 255.255.255.0
```

### 5. Configure NAT Exemption
#### **ASAv1**
```shell
nat (inside,outside) source static inside-network inside-network destination static remote-network remote-network no-proxy-arp route-lookup
```

#### **ASAv2**
```shell
nat (inside,outside) source static inside-network inside-network destination static remote-network remote-network no-proxy-arp route-lookup
```

### 6. Configure IPSEC IKEv2 VPN
#### **ASAv1**
```shell
crypto ipsec ikev2 ipsec-proposal AES256
 protocol esp encryption aes-256
 protocol esp integrity sha-1

crypto map outside_map 1 match address outside_cryptomap
crypto map outside_map 1 set peer 30.1.1.1
crypto map outside_map 1 set ikev2 ipsec-proposal AES256
crypto map outside_map interface outside

crypto ikev2 enable outside
tunnel-group 30.1.1.1 type ipsec-l2l
tunnel-group 30.1.1.1 ipsec-attributes
 ikev2 remote-authentication pre-shared-key eve
 ikev2 local-authentication pre-shared-key eve
```

#### **ASAv2**
```shell
crypto ipsec ikev2 ipsec-proposal AES256
 protocol esp encryption aes-256
 protocol esp integrity sha-1

crypto map outside_map 1 match address outside_cryptomap
crypto map outside_map 1 set peer 20.1.1.1
crypto map outside_map 1 set ikev2 ipsec-proposal AES256
crypto map outside_map interface outside

crypto ikev2 enable outside
tunnel-group 20.1.1.1 type ipsec-l2l
tunnel-group 20.1.1.1 ipsec-attributes
 ikev2 remote-authentication pre-shared-key eve
 ikev2 local-authentication pre-shared-key eve
```

## Verification

### 1. Ping from Host-PC1 to Host-PC2
```shell
ping 192.168.102.1
```
### 2. Verify Traffic Encryption
#### **Check IKEv2 SA**
```shell
show crypto ikev2 sa
```
#### **Check IPSec SA**
```shell
show ipsec sa
```

## Repository Structure
```
├── configs/
│   ├── ASAv1_config.txt   # Configuration for ASAv1
│   ├── ASAv2_config.txt   # Configuration for ASAv2
├── README.md              # Documentation
├── screenshots/           # Screenshots of verification outputs
└── topology.png           # Network diagram
```

## Conclusion
This project demonstrates a **Cisco IPSEC IKEv2 Site-to-Site VPN** setup using Cisco ASAv devices. It provides a robust and secure way to connect remote sites and is a valuable skill for network and security engineers.

---
**Author:** Travis Johnson  
**Company:** 10Digit Solutions LLC  
**GitHub Repository:** [Add link when available]
