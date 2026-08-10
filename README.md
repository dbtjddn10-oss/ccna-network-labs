# CCNA Network Labs

Cisco Packet Tracer를 활용해 CCNA 네트워크 개념을 직접 구성하고 검증한 실습 기록입니다.

단순 명령어 암기보다 네트워크가 실제로 어떻게 동작하는지 이해하고,  
구성 과정에서 발생한 문제를 직접 확인하고 해결하는 것을 목표로 합니다.

---

## Environment

- Cisco Packet Tracer 9.0.1
- Cisco IOS CLI
- CCNA 200-301 Study
- Windows

---

# Lab 01 - Basic Switching, ARP & MAC Address Table

## Objective

PC 2대와 Cisco 2960 Switch를 연결하고 기본적인 Layer 2 통신 과정을 확인했습니다.

## Topology

```text
PC0 ---------------- Switch0 ---------------- PC1

192.168.1.10                              192.168.1.20
```

## PC Configuration

### PC0

```text
IP Address : 192.168.1.10
Subnet Mask: 255.255.255.0
```

### PC1

```text
IP Address : 192.168.1.20
Subnet Mask: 255.255.255.0
```

두 PC는 같은 `192.168.1.0/24` 네트워크에 위치하므로 Router 없이 Switch만으로 통신할 수 있습니다.

## Ping Test

PC0에서 PC1으로 Ping을 실행했습니다.

```text
ping 192.168.1.20
```

정상적으로 ICMP Echo Reply가 돌아오는 것을 확인했습니다.

## ARP Verification

PC에서 ARP Table을 확인했습니다.

```text
arp -a
```

ARP를 통해 IPv4 주소에 대응하는 MAC Address가 학습되는 것을 확인했습니다.

## Switch MAC Address Table

Switch CLI에서 다음 명령어를 사용했습니다.

```text
enable
show mac address-table
```

실제 결과:

```text
Vlan    Mac Address       Type        Ports
----    -----------       --------    -----
1       0001.c9e5.e39c    DYNAMIC     Fa0/2
1       0006.2a85.73bd    DYNAMIC     Fa0/1
```

Switch가 각 PC의 MAC Address를 해당 Switch Port와 매핑하여 학습하는 것을 확인했습니다.

## Interface Status

```text
show interfaces status
```

주요 결과:

```text
Fa0/1    connected    1    a-full    a-100
Fa0/2    connected    1    a-full    a-100
```

## VLAN Verification

```text
show vlan brief
```

초기 상태에서는 PC0과 PC1 모두 기본 VLAN인 VLAN 1에 속해 있었습니다.

## Key Concepts

- ARP는 IPv4 Address에 대응하는 MAC Address를 찾는다.
- ARP Request는 Broadcast로 전송된다.
- ARP Reply는 일반적으로 Unicast로 전송된다.
- Ping은 ICMP Echo Request / Echo Reply를 사용한다.
- Switch는 수신한 Ethernet Frame의 **Source MAC Address**를 보고 MAC Address Table을 학습한다.
- 같은 Subnet에 있는 장비끼리는 Router 없이 Layer 2 Switch를 통해 통신할 수 있다.

---

# Lab 02 - VLAN Segmentation

## Objective

하나의 Switch에서 PC0과 PC1을 서로 다른 VLAN으로 분리하여 Broadcast Domain이 분리되는 것을 확인했습니다.

## VLAN Configuration

VLAN 10과 VLAN 20을 생성했습니다.

```text
enable
configure terminal

vlan 10
 name PC0_VLAN
 exit

vlan 20
 name PC1_VLAN
 exit
```

## Access Port Configuration

PC0이 연결된 `Fa0/1`을 VLAN 10에 할당했습니다.

```text
interface fastethernet 0/1
 switchport mode access
 switchport access vlan 10
 exit
```

PC1이 연결된 `Fa0/2`를 VLAN 20에 할당했습니다.

```text
interface fastethernet 0/2
 switchport mode access
 switchport access vlan 20
 exit
```

## Verification

```text
show vlan brief
```

결과:

```text
10   PC0_VLAN    active    Fa0/1
20   PC1_VLAN    active    Fa0/2
```

현재 구조는 다음과 같습니다.

```text
PC0
 │
Fa0/1
 │
VLAN 10
 │
Switch0
 │
VLAN 20
 │
Fa0/2
 │
PC1
```

## Result

PC0과 PC1이 서로 다른 VLAN에 속하면서 기존에 성공하던 Ping이 실패하는 것을 확인했습니다.

VLAN 10에서 발생한 ARP Broadcast가 VLAN 20까지 전달되지 않기 때문입니다.

```text
PC0
 │
 │ ARP Broadcast
 ▼
VLAN 10
 │
 X  VLAN Boundary
 │
VLAN 20
 │
PC1
```

## Key Concepts

- VLAN은 하나의 물리 Switch를 여러 개의 논리적인 네트워크처럼 분리한다.
- VLAN은 Broadcast Domain을 분리한다.
- 서로 다른 VLAN 사이의 통신에는 Layer 3 Routing이 필요하다.
- Access Port는 일반적으로 하나의 VLAN에 소속된다.

---

# Lab 03 - Router-on-a-Stick

## Objective

VLAN 10과 VLAN 20 사이의 Inter-VLAN Routing을 Router-on-a-Stick 방식으로 구성했습니다.

하나의 Router Physical Interface에서 여러 VLAN을 처리하기 위해 Subinterface와 IEEE 802.1Q Tagging을 사용했습니다.

## Final Topology

```text
PC0
192.168.10.10/24
GW: 192.168.10.1
VLAN 10
    │
    │ Fa0/1
    ▼
+-----------+
|  Switch0  |
+-----------+
    │
    │ Gi0/1
    │ 802.1Q Trunk
    │ VLAN 10,20
    ▼
+------------------------------+
|           Router0            |
|                              |
| G0/0.10 = 192.168.10.1/24    |
| VLAN 10                      |
|                              |
| G0/0.20 = 192.168.20.1/24    |
| VLAN 20                      |
+------------------------------+
    │
    │ Routing
    ▼
+-----------+
|  Switch0  |
+-----------+
    │
    │ Fa0/2
    ▼
PC1
192.168.20.20/24
GW: 192.168.20.1
VLAN 20
```

---

## IP Addressing

### PC0

```text
IP Address      : 192.168.10.10
Subnet Mask     : 255.255.255.0
Default Gateway : 192.168.10.1
```

### PC1

```text
IP Address      : 192.168.20.20
Subnet Mask     : 255.255.255.0
Default Gateway : 192.168.20.1
```

### Router0

```text
G0/0.10 : 192.168.10.1/24
G0/0.20 : 192.168.20.1/24
```

---

## Switch VLAN Configuration

```text
enable
configure terminal

vlan 10
 name PC0_VLAN
 exit

vlan 20
 name PC1_VLAN
 exit
```

## Switch Access Port Configuration

```text
interface fastethernet 0/1
 switchport mode access
 switchport access vlan 10
 exit

interface fastethernet 0/2
 switchport mode access
 switchport access vlan 20
 exit
```

---

## Switch Trunk Configuration

Switch와 Router 사이의 `GigabitEthernet0/1` 포트를 Trunk로 설정했습니다.

```text
interface gigabitethernet 0/1
 switchport mode trunk
 switchport trunk allowed vlan 10,20
 exit
```

확인 명령어:

```text
show interfaces trunk
```

실제 결과:

```text
Port        Mode         Encapsulation  Status        Native vlan
Gig0/1      on           802.1q         trunking      1

Port        Vlans allowed on trunk
Gig0/1      10,20

Port        Vlans allowed and active in management domain
Gig0/1      10,20
```

`Gig0/1`이 IEEE 802.1Q Trunk 상태이며 VLAN 10과 VLAN 20이 허용된 것을 확인했습니다.

---

## Router Physical Interface Configuration

Router의 `GigabitEthernet0/0` 인터페이스를 활성화했습니다.

```text
enable
configure terminal

interface gigabitethernet 0/0
 no shutdown
 exit
```

확인:

```text
show ip interface brief
```

결과:

```text
GigabitEthernet0/0     unassigned     up     up
```

---

## Router Subinterface Configuration

VLAN 10용 Subinterface를 생성했습니다.

```text
interface gigabitethernet 0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
 exit
```

VLAN 20용 Subinterface를 생성했습니다.

```text
interface gigabitethernet 0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
 exit
```

확인:

```text
show ip interface brief
```

실제 결과:

```text
GigabitEthernet0/0     unassigned       up    up
GigabitEthernet0/0.10  192.168.10.1     up    up
GigabitEthernet0/0.20  192.168.20.1     up    up
```

---

## Routing Table Verification

Router에서 다음 명령어를 사용했습니다.

```text
show ip route
```

실제 결과:

```text
Gateway of last resort is not set

192.168.10.0/24 is variably subnetted, 2 subnets, 2 masks

C    192.168.10.0/24 is directly connected, GigabitEthernet0/0.10
L    192.168.10.1/32 is directly connected, GigabitEthernet0/0.10

192.168.20.0/24 is variably subnetted, 2 subnets, 2 masks

C    192.168.20.0/24 is directly connected, GigabitEthernet0/0.20
L    192.168.20.1/32 is directly connected, GigabitEthernet0/0.20
```

### Routing Code

```text
C = Connected
L = Local
```

`C`는 Router에 직접 연결된 Network를 의미합니다.

```text
192.168.10.0/24
192.168.20.0/24
```

`L`은 Router Interface 자신의 IP Address를 의미합니다.

```text
192.168.10.1/32
192.168.20.1/32
```

---

## Connectivity Test

PC0에서 다음 Ping을 수행했습니다.

```text
ping 192.168.10.1
ping 192.168.20.1
ping 192.168.20.20
```

PC1에서도 PC0 방향으로 Ping을 수행했습니다.

```text
ping 192.168.10.10
```

PC0과 PC1 사이의 양방향 통신이 정상적으로 이루어지는 것을 확인했습니다.

---

# Packet Flow

PC0에서 PC1으로 통신할 경우 패킷의 흐름은 다음과 같습니다.

```text
PC0
192.168.10.10
    │
    ▼
Switch Fa0/1
VLAN 10
    │
    ▼
Switch Gi0/1
802.1Q Trunk
    │
    ▼
Router G0/0.10
192.168.10.1
    │
    │ Routing Table Lookup
    ▼
Router G0/0.20
192.168.20.1
    │
    ▼
802.1Q Trunk
    │
    ▼
Switch Fa0/2
VLAN 20
    │
    ▼
PC1
192.168.20.20
```

---

# Troubleshooting Notes

이번 실습 과정에서 발생한 문제와 해결 과정을 기록했습니다.

## 1. Router Interface Shutdown

처음 Router와 Switch를 연결했지만 Trunk가 정상적으로 동작하지 않았습니다.

Router의 Ethernet Interface가 기본적으로 Administratively Down 상태였습니다.

확인:

```text
show ip interface brief
```

해결:

```text
configure terminal
interface gigabitethernet 0/0
no shutdown
```

---

## 2. Wrong Cisco IOS Mode

다음 명령어를 입력했을 때 오류가 발생했습니다.

```text
Router#interface g0/0.10
```

결과:

```text
% Invalid input detected at '^' marker.
```

`interface` 명령은 Privileged EXEC Mode가 아니라 Global Configuration Mode에서 실행해야 합니다.

잘못된 상태:

```text
Router#
```

정상 상태:

```text
Router(config)#
```

해결:

```text
configure terminal
interface g0/0.10
```

---

## 3. Simulation Mode Ping Timeout

Packet Tracer Simulation Mode에서 일반 Ping 명령을 실행했을 때 Timeout처럼 보이는 현상이 발생했습니다.

Simulation Mode에서는 Packet 진행을 직접 제어하기 때문에 일반적인 통신 테스트는 Realtime Mode가 더 편리했습니다.

해결:

```text
Simulation Mode
↓
Realtime Mode
```

---

## 4. Cisco IOS Command Typo

다음과 같이 명령어 오타가 발생했습니다.

```text
show vlan breif
```

오류:

```text
% Invalid input detected at '^' marker.
```

정상 명령어:

```text
show vlan brief
```

또한:

```text
show ip interface breif
```

가 아니라:

```text
show ip interface brief
```

를 사용해야 합니다.

---

## 5. IOS DNS Lookup

잘못된 명령어를 입력했을 때 IOS가 해당 문자열을 Hostname으로 판단하고 DNS Lookup을 시도하는 현상이 발생했습니다.

예:

```text
Translating "ResetSimulation"...domain server (255.255.255.255)
```

DNS Lookup을 비활성화했습니다.

```text
configure terminal
no ip domain lookup
end
```

---

# Useful Cisco IOS Commands

## Switch

```text
enable
configure terminal

show vlan brief
show mac address-table
show interfaces status
show interfaces trunk
show running-config

clear mac address-table dynamic
```

## Router

```text
enable
configure terminal

show ip interface brief
show ip route
show running-config
```

## PC

```text
ping <IP Address>
arp -a
arp -d
```

---

# Key Concepts Learned

## Ethernet Switching

Switch는 수신한 Ethernet Frame의 Source MAC Address를 학습하여 MAC Address Table을 생성합니다.

```text
MAC Address → Switch Port
```

이를 통해 목적지 MAC Address가 알려져 있는 Frame은 필요한 Port로 전달할 수 있습니다.

---

## ARP

ARP는 IPv4 Address와 MAC Address를 매핑하는 데 사용됩니다.

```text
IPv4 Address
      ↓
     ARP
      ↓
MAC Address
```

ARP Request는 Broadcast로 전달되며, 해당 IPv4 주소를 가진 장비가 ARP Reply를 반환합니다.

---

## VLAN

VLAN은 하나의 Physical Switch를 여러 개의 Logical Network로 분리합니다.

```text
VLAN 10 = Broadcast Domain 1
VLAN 20 = Broadcast Domain 2
```

서로 다른 VLAN 사이에서는 Layer 2 Switching만으로 통신할 수 없습니다.

---

## Access Port

Access Port는 일반적으로 하나의 VLAN에 연결되는 Endpoint용 Port입니다.

예:

```text
Fa0/1 → VLAN 10
Fa0/2 → VLAN 20
```

---

## Trunk Port

Trunk Port는 하나의 Physical Link를 통해 여러 VLAN의 Traffic을 전달합니다.

이번 Lab에서는 다음 VLAN을 전달했습니다.

```text
VLAN 10
VLAN 20
```

IEEE 802.1Q Tagging을 사용했습니다.

---

## Inter-VLAN Routing

서로 다른 VLAN 사이에서 통신하기 위해서는 Layer 3 Routing이 필요합니다.

이번 Lab에서는 Router-on-a-Stick 방식을 사용했습니다.

---

## Router-on-a-Stick

하나의 Router Physical Interface에 여러 Subinterface를 생성합니다.

```text
G0/0

├── G0/0.10
│   └── VLAN 10
│       192.168.10.1
│
└── G0/0.20
    └── VLAN 20
        192.168.20.1
```

각 Subinterface는 `encapsulation dot1Q`를 사용하여 특정 VLAN과 연결됩니다.

---

# Study Progress

## Completed

- [x] Cisco Packet Tracer Setup
- [x] Basic Ethernet Connection
- [x] IPv4 Address Configuration
- [x] Subnet Mask Basics
- [x] ICMP Ping
- [x] ARP
- [x] MAC Address Table
- [x] Cisco Switch CLI
- [x] VLAN
- [x] Broadcast Domain
- [x] Access Port
- [x] IEEE 802.1Q
- [x] Trunk Port
- [x] Default Gateway
- [x] Router Subinterface
- [x] Inter-VLAN Routing
- [x] Router-on-a-Stick
- [x] Routing Table Basics
- [x] Connected Route
- [x] Local Route

## Next Labs

- [ ] IPv4 Subnetting
- [ ] Static Routing
- [ ] Default Route
- [ ] STP
- [ ] Rapid STP
- [ ] EtherChannel
- [ ] OSPF
- [ ] DHCP
- [ ] NAT / PAT
- [ ] ACL
- [ ] Port Security
- [ ] IPv6
- [ ] Network Troubleshooting

---

# Repository Structure

앞으로 다음과 같은 구조로 Packet Tracer Lab을 추가할 예정입니다.

```text
ccna-network-labs/
│
├── README.md
│
├── labs/
│   ├── 01-basic-switching/
│   ├── 02-vlan-segmentation/
│   ├── 03-router-on-a-stick/
│   ├── 04-subnetting/
│   ├── 05-static-routing/
│   ├── 06-stp/
│   ├── 07-etherchannel/
│   ├── 08-ospf/
│   ├── 09-dhcp/
│   ├── 10-nat/
│   └── 11-acl/
│
└── notes/
    └── cisco-ios-commands.md
```

---

# Goal

CCNA 자격 취득을 목표로 Cisco Packet Tracer를 이용해 직접 Network를 구성하고,

**Configuration → Verification → Troubleshooting**

과정을 반복하면서 네트워크 동작 원리를 이해하는 것을 목표로 합니다.
