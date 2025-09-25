# VLAN-VTP-STP-Router-on-a-Stick

![Cisco](https://img.shields.io/badge/Cisco-Networking-blue?style=for-the-badge&logo=cisco&logoColor=white)
![VLAN](https://img.shields.io/badge/VLAN-Segmentation-green?style=for-the-badge)
![VTP](https://img.shields.io/badge/VTP-Domain-orange?style=for-the-badge)
![STP](https://img.shields.io/badge/STP-Protocol-red?style=for-the-badge)

---

## 📝 Deskripsi
Repository ini berisi **implementasi lengkap jaringan** dengan teknologi:
- **VLAN** (Virtual LAN) untuk segmentasi jaringan
- **VTP** (VLAN Trunking Protocol) untuk manajemen VLAN terpusat
- **STP** (Spanning Tree Protocol) untuk pencegahan loop
- **Router-on-a-Stick** untuk inter-VLAN routing
- **DHCP** untuk alokasi IP otomatis

---

## 🏗️ Topologi Jaringan
| Perangkat | Peran | VLAN | IP Address | Koneksi |
|-----------|------|------|------------|---------|
| **Router** | Inter-VLAN Routing | - | Fa0/0: Trunk | Ke SW-SERVER |
| **SW-SERVER** | VTP Server | 10,20,30 | - | Ke Router & Client |
| **SW-CLIENT1** | VTP Client | 10,20 | - | Ke SW-SERVER |
| **SW-CLIENT2** | VTP Client | 30 | - | Ke SW-SERVER |
| **SW-CLIENT3** | VTP Client | 20 | - | Ke SW-SERVER |

---

## ⚙️ Konfigurasi VLAN
| VLAN ID | Nama | Pengguna | IP Subnet | DHCP |
|---------|------|----------|-----------|------|
| 10 | HRD | 3 Komputer (kiri) | 192.168.10.0/26 | Tidak |
| 20 | IT | 5 Komputer (kanan) | 192.168.20.0/27 | Ya |
| 30 | Marketing | 3 Komputer (kanan bawah) | 192.168.30.0/24 | Tidak |

---

## 🔧 Konfigurasi Perangkat
### Router (Inter-VLAN Routing & DHCP)
```cisco
enable
configure terminal
interface fa0/0
no shutdown
interface fa0/0.10
encapsulation dot1Q 10
ip address 192.168.10.1 255.255.255.192
interface fa0/0.20
encapsulation dot1Q 20
ip address 192.168.20.1 255.255.255.224
interface fa0/0.30
encapsulation dot1Q 30
ip address 192.168.30.1 255.255.255.0
exit
ip dhcp pool VLAN20
network 192.168.20.0 255.255.255.224
default-router 192.168.20.1
dns-server 8.8.8.8
end
```

### SW-SERVER (VTP Server)
```cisco
enable
configure terminal
vlan 10
name HRD
vlan 20
name IT
vlan 30
name Marketing
exit
vtp mode server
vtp domain CampusNetwork
vtp password securepass
interface range fa0/1-4
switchport mode trunk
exit
spanning-tree vlan 10 root primary
spanning-tree mode pvst
interface fa0/5
switchport mode trunk
end
```

### SW-CLIENT1 (VLAN 10 & 20)
```cisco
enable
configure terminal
interface range fa0/1-2
switchport mode trunk
exit
vtp mode client
vtp domain CampusNetwork
vtp password securepass
interface range fa0/3-12
switchport mode access
switchport access vlan 10
exit
interface range fa0/13-24
switchport mode access
switchport access vlan 20
exit
interface range fa0/1-2
switchport mode trunk
switchport trunk allowed vlan 10,20,30
end
```

### SW-CLIENT2 (VLAN 30)
```cisco
enable
configure terminal
vtp mode client
vtp domain CampusNetwork
vtp password securepass
interface range fa0/1-4
switchport mode trunk
exit
spanning-tree mode pvst
interface range fa0/5-24
switchport mode access
switchport access vlan 30
end
```

### SW-CLIENT3 (VLAN 20)
```cisco
enable
configure terminal
vtp mode client
vtp domain CampusNetwork
vtp password securepass
interface range fa0/1-4
switchport mode trunk
exit
spanning-tree mode pvst
interface range fa0/5-24
switchport mode access
switchport access vlan 20
end
```

---

## 📊 Output Konfigurasi
### Verifikasi VLAN
```
SW-SERVER# show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/6, Fa0/7, Fa0/8, Fa0/9
                                                Fa0/10, Fa0/11, Fa0/12, Fa0/13
                                                Fa0/14, Fa0/15, Fa0/16, Fa0/17
                                                Fa0/18, Fa0/19, Fa0/20, Fa0/21
                                                Fa0/22, Fa0/23, Fa0/24, Gig1/1
                                                Gig1/2
10   HRD                              active    Fa0/3, Fa0/4, Fa0/5
20   IT                               active    Fa0/6, Fa0/7, Fa0/8
30   Marketing                        active    Fa0/9, Fa0/10, Fa0/11
```

### Verifikasi VTP
```
SW-SERVER# show vtp status

VTP Version                     : 2
Configuration Revision          : 3
Maximum VLANs supported locally : 255
Number of existing VLANs       : 8
VTP Operating Mode              : Server
VTP Domain Name                 : CampusNetwork
VTP Pruning Mode                : Disabled
VTP V2 Mode                     : Disabled
VTP Traps Generation            : Disabled
MD5 digest                      : 0x3A 0x2C 0x4B 0x1D 0x5E 0x3F 0x7A 0x8B
```

### Verifikasi STP
```
SW-SERVER# show spanning-tree vlan 10

VLAN0010
  Spanning tree enabled protocol ieee
  Root ID    Priority    24586
             Address     0060.70DB.C5E3
             This bridge is the root
             
  Bridge ID  Priority    24586  (priority 24576 sys-id-ext 10)
             Address     0060.70DB.C5E3
             
Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/1            Desg FWD 19        128.1    P2p
Fa0/2            Desg FWD 19        128.2    P2p
```

### Verifikasi DHCP
```
Router# show ip dhcp pool

Pool VLAN20 :
 Utilization mark (high/low)    : 100 / 0
 Subnet size (first/next)       : 0 / 0 
 Total addresses                : 25
 Leased addresses               : 0
 Pending event                  : none
 1 subnet is currently in the pool
 Current index        IP address range                    Leased/Excluded/Total
 192.168.20.1        192.168.20.1    - 192.168.20.30      0 / 1 / 25
```

<img width="827" height="346" alt="image" src="https://github.com/user-attachments/assets/87ca282c-2494-437a-95d5-00fd77a11571" />

---

**luqmanaru**  
