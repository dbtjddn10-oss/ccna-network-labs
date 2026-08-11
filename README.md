<p align="center">
  <img src="./labs/05-ipv4-subnetting/topology.png" alt="IPv4 Subnetting Lab" width="1000">
</p>

<p align="center">
  <b>Lab 05 — IPv4 Subnetting / Proxy ARP Troubleshooting</b>
</p>

<br>

<p align="center">
  <img src="./labs/04-static-routing/topology.png" alt="Static Routing Lab" width="1000">
</p>

<p align="center">
  <b>Lab 04 — Static Routing / Routing Troubleshooting</b>
</p>

<br>

<p align="center">
  <img src="./labs/03-router-on-a-stick/topology.png" alt="Router-on-a-Stick Lab" width="1000">
</p>

<p align="center">
  <b>Lab 03 — VLAN / Router-on-a-Stick</b>
</p>

<br>

# CCNA Network Labs

Cisco Packet Tracer와 Cisco IOS CLI를 활용하여  
**CCNA 네트워크 개념을 직접 구성하고 검증한 실습 포트폴리오**입니다.

단순히 명령어를 암기하는 방식보다,

**Configuration → Verification → Troubleshooting → Recovery**

과정을 반복하면서 네트워크가 실제로 어떻게 동작하는지 이해하는 것을 목표로 하고 있습니다.

각 Lab에는 가능한 경우 다음 자료를 함께 기록합니다.

- Packet Tracer 실습 파일
- Network Topology 이미지
- Cisco IOS 설정 명령어
- 통신 검증 결과
- 장애 발생 원인
- Troubleshooting 과정
- 실습을 통해 배운 내용

---

# Lab Progress

| Lab | 주제 | 주요 내용 | 상태 |
|---|---|---|---|
| Lab 01 | Basic Switching | Ethernet, ARP, MAC Address Table, ICMP | ✅ 완료 |
| Lab 02 | VLAN Segmentation | VLAN 10/20, Access Port, Broadcast Domain | ✅ 완료 |
| [Lab 03](./labs/03-router-on-a-stick/README.md) | Router-on-a-Stick | 802.1Q Trunk, Subinterface, Inter-VLAN Routing | ✅ 완료 |
| [Lab 04](./labs/04-static-routing/README.md) | Static Routing | Static Route, Next Hop, /30, CDP, Routing Troubleshooting | ✅ 완료 |
| [Lab 05](./labs/05-ipv4-subnetting/README.md) | IPv4 Subnetting | CIDR, /26 Subnetting, Proxy ARP, Subnet Troubleshooting | ✅ 완료 |
| Lab 06 | STP / RSTP | Loop 방지, Root Bridge, Port Role | ⏳ 예정 |
| Lab 07 | EtherChannel | LACP, Link Aggregation | ⏳ 예정 |
| Lab 08 | OSPF | Dynamic Routing, Neighbor, Route Learning | ⏳ 예정 |
| Lab 09 | DHCP | DHCP Server, Address Allocation, DHCP Relay | ⏳ 예정 |
| Lab 10 | NAT / PAT | Private/Public IP, Address Translation | ⏳ 예정 |
| Lab 11 | ACL | Standard / Extended ACL, Traffic Filtering | ⏳ 예정 |
| Lab 12 | IPv6 | IPv6 Addressing 및 Routing | ⏳ 예정 |
| Final Lab | 종합 네트워크 구축 | VLAN, Routing, DHCP, NAT, ACL, Troubleshooting | ⏳ 예정 |

---

# 실습 환경

- Cisco Packet Tracer 9.0.1
- Cisco IOS CLI
- CCNA 200-301
- Windows
- GitHub

---

# 현재까지 학습한 기술

## Switching / Layer 2

- Ethernet Switching
- MAC Address
- MAC Address Table
- ARP
- ICMP
- VLAN
- Broadcast Domain
- Access Port
- Trunk Port
- IEEE 802.1Q

---

## Routing / Layer 3

- IPv4 Addressing
- CIDR Prefix
- Subnet Mask
- IPv4 Subnetting
- Network Address
- Host Range
- Broadcast Address
- Default Gateway
- Connected Route
- Local Route
- Static Route
- Next Hop
- Routing Table
- Inter-VLAN Routing
- Router-on-a-Stick
- Router Subinterface
- `/30` Point-to-Point Network
- Proxy ARP

---

## Verification / Troubleshooting

- `ping`
- `tracert`
- `arp -a`
- `arp -d`
- `show mac address-table`
- `show interfaces status`
- `show vlan brief`
- `show interfaces trunk`
- `show ip interface brief`
- `show ip interface`
- `show ip route`
- `show cdp neighbors`
- `show running-config`
- Interface Mapping 확인
- Routing Table 분석
- Subnet Mask 오류 분석
- Proxy ARP 동작 확인
- Overlapping Network 오류 해결

---

# 대표 실습

## Lab 03 - Router-on-a-Stick

<p align="center">
  <img src="./labs/03-router-on-a-stick/topology.png" alt="Router-on-a-Stick Topology" width="850">
</p>

서로 다른 VLAN에 위치한 PC 간 통신을 위해  
**Router-on-a-Stick 방식의 Inter-VLAN Routing**을 구성했습니다.

구성:

```text
PC0
VLAN 10
192.168.10.10/24
      │
      │ Access Port
      ▼
    Switch
      │
      │ 802.1Q Trunk
      ▼
    Router
 G0/0.10
 G0/0.20
      │
      ▼
    Switch
      │
      │ Access Port
      ▼
PC1
VLAN 20
192.168.20.20/24
```

Router Subinterface:

```cisco
interface gigabitethernet 0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0

interface gigabitethernet 0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
```

주요 학습 내용:

- VLAN 10 / VLAN 20
- Access Port
- Trunk Port
- IEEE 802.1Q
- Router Subinterface
- Default Gateway
- Inter-VLAN Routing
- Connected Route
- Local Route
- ARP / ICMP

➡️ [Lab 03 상세 README](./labs/03-router-on-a-stick/README.md)

➡️ [Packet Tracer 파일](./labs/03-router-on-a-stick/03-router-on-a-stick.pkt)

---

# Lab 04 - Static Routing

<p align="center">
  <img src="./labs/04-static-routing/topology.png" alt="Static Routing Topology" width="850">
</p>

서로 다른 두 LAN을 두 대의 Router로 연결하고  
**Static Route를 이용하여 Remote Network 간 통신**을 구현했습니다.

```text
PC0
192.168.10.10/24
        │
     Switch0
        │
Router0
G0/0 192.168.10.1/24
G0/1 10.0.0.1/30
        │
        │
   10.0.0.0/30
        │
        │
Router1
G0/0 10.0.0.2/30
G0/1 192.168.20.1/24
        │
     Switch1
        │
PC1
192.168.20.20/24
```

Router0 Static Route:

```cisco
ip route 192.168.20.0 255.255.255.0 10.0.0.2
```

Router1 Static Route:

```cisco
ip route 192.168.10.0 255.255.255.0 10.0.0.1
```

Routing Table에서 다음과 같은 Route를 확인했습니다.

```text
C = Connected
L = Local
S = Static
```

주요 학습 내용:

- Router-to-Router Network
- `/30` Point-to-Point Network
- Static Routing
- Next Hop
- Connected Route
- Local Route
- Static Route
- Routing Table
- Administrative Distance
- ICMP
- Traceroute
- CDP
- Interface Troubleshooting

➡️ [Lab 04 상세 README](./labs/04-static-routing/README.md)

➡️ [Packet Tracer 파일](./labs/04-static-routing/04-static-routing.pkt)

---

# Lab 05 - IPv4 Subnetting

<p align="center">
  <img src="./labs/05-ipv4-subnetting/topology.png" alt="IPv4 Subnetting Topology" width="850">
</p>

하나의:

```text
192.168.10.0/24
```

Network를 `/26`으로 Subnetting하여 서로 다른 두 LAN을 만들고  
Router를 이용하여 Subnet 간 통신을 구성했습니다.

사용한 Subnet:

```text
Subnet 1
192.168.10.0/26

Network   : 192.168.10.0
Host      : 192.168.10.1 ~ 192.168.10.62
Broadcast : 192.168.10.63
```

```text
Subnet 2
192.168.10.64/26

Network   : 192.168.10.64
Host      : 192.168.10.65 ~ 192.168.10.126
Broadcast : 192.168.10.127
```

구성:

```text
PC0
192.168.10.10/26
GW 192.168.10.1
        │
     Switch0
        │
Router0 G0/0
192.168.10.1/26
        │
     Router0
        │
Router0 G0/1
192.168.10.65/26
        │
     Switch1
        │
PC1
192.168.10.70/26
GW 192.168.10.65
```

Router는 두 Network에 직접 연결되어 있기 때문에  
별도의 Static Route 없이 Connected Route를 이용해 통신할 수 있습니다.

```text
C 192.168.10.0/26
C 192.168.10.64/26
```

주요 학습 내용:

- CIDR
- Subnet Mask
- Block Size
- Network Address
- First Host
- Last Host
- Broadcast Address
- `/24` → `/26` Subnetting
- Connected Routing
- Default Gateway
- Proxy ARP
- Subnet Mask Troubleshooting

➡️ [Lab 05 상세 README](./labs/05-ipv4-subnetting/README.md)

➡️ [Packet Tracer 파일](./labs/05-ipv4-subnetting/05-ipv4-subnetting.pkt)

---

# Troubleshooting 경험

정상적인 Network를 구성하는 것뿐만 아니라  
실습 중 발생하거나 의도적으로 만든 장애 상황의 원인을 직접 확인하고 복구했습니다.

---

## 1. Router 간 Ping 실패

Static Routing Lab에서 Router0 → Router1 Ping 테스트가 실패했습니다.

```text
Success rate is 0 percent (0/5)
```

먼저:

```cisco
show ip interface brief
```

를 확인했지만 Interface는 모두:

```text
up / up
```

상태였습니다.

따라서 물리적인 Link Down이나 `shutdown` 문제가 아닌 것으로 판단했습니다.

이후:

```cisco
show cdp neighbors
```

명령어를 이용해 실제 Router 간 연결 Interface를 확인했습니다.

확인 결과 Router1의 IP Address가 실제 케이블 연결 구조와 반대로 설정되어 있었습니다.

잘못된 구성:

```text
Router1 G0/0 → 192.168.20.1/24
Router1 G0/1 → 10.0.0.2/30
```

실제 연결 구조:

```text
Router1 G0/0 → Router0
Router1 G0/1 → Switch1
```

따라서 다음과 같이 수정했습니다.

```text
Router1 G0/0 → 10.0.0.2/30
Router1 G0/1 → 192.168.20.1/24
```

수정 후 Router 간 Ping이 정상적으로 성공했습니다.

### 배운 점

`up/up`은 Interface와 Line Protocol이 동작하고 있다는 의미이지,  
**의도한 상대 장비 또는 올바른 Network가 해당 Interface에 연결되어 있다는 의미는 아닙니다.**

실제 연결 관계를 확인할 때:

```cisco
show cdp neighbors
```

를 활용할 수 있다는 것을 확인했습니다.

---

## 2. Overlapping Network 오류

Router1 Interface의 IP Address를 변경하는 과정에서:

```text
% 10.0.0.0 overlaps with GigabitEthernet0/1
```

오류가 발생했습니다.

기존 Interface에 이미:

```text
10.0.0.0/30
```

Network에 속하는 IP가 존재했기 때문입니다.

기존 IP를:

```cisco
no ip address
```

로 먼저 제거한 후 올바른 Interface에 다시 설정하여 해결했습니다.

### 배운 점

Router의 서로 다른 Routed Interface에  
서로 겹치는 IPv4 Network를 설정해서는 안 됩니다.

---

## 3. 잘못된 Subnet Mask와 Proxy ARP

IPv4 Subnetting Lab에서 PC0의 정상 설정:

```text
192.168.10.10/26
```

을 의도적으로:

```text
192.168.10.10/24
```

로 잘못 변경했습니다.

PC0는 목적지:

```text
192.168.10.70
```

을 같은 Local Network에 있다고 잘못 판단하게 됩니다.

원래라면 목적지 Host의 MAC Address를 직접 찾기 위해 ARP를 수행하고  
Router 반대편에 있는 PC1과의 통신에 문제가 발생할 수 있습니다.

그러나 예상과 다르게 Ping이 성공했습니다.

Router Interface를 확인했습니다.

```cisco
show ip interface gigabitethernet 0/0
```

확인 결과:

```text
Proxy ARP is enabled
```

상태였습니다.

Router가 목적지 대신 자신의 MAC Address로 ARP Reply를 보내면서  
잘못된 Subnet Mask 설정에서도 통신이 가능했습니다.

Proxy ARP를 비활성화했습니다.

```cisco
interface gigabitethernet 0/0
 no ip proxy-arp
```

PC0의 ARP Cache도 삭제했습니다.

```text
arp -d
```

다시 Ping을 수행하자 잘못된 `/24` 상태에서는 통신이 실패했습니다.

PC0의 Subnet Mask를 정상적인 `/26`으로 복구한 후  
다시 통신이 정상적으로 성공하는 것을 확인했습니다.

실습 후 Proxy ARP도 다시 활성화했습니다.

```cisco
interface gigabitethernet 0/0
 ip proxy-arp
```

### 배운 점

**Ping이 성공한다는 사실만으로 Network 설정이 모두 올바르다고 판단해서는 안 됩니다.**

Proxy ARP와 같은 기능이 잘못된 설정을 겉으로 가릴 수 있기 때문에  
실제 IP Address, Subnet Mask, Gateway, ARP Table, Routing Table까지 함께 확인해야 합니다.

---

# Subnetting Quick Reference

| CIDR | Subnet Mask | 전체 주소 | 일반 사용 가능 Host | Block Size |
|---|---|---:|---:|---:|
| /24 | 255.255.255.0 | 256 | 254 | 256 |
| /25 | 255.255.255.128 | 128 | 126 | 128 |
| /26 | 255.255.255.192 | 64 | 62 | 64 |
| /27 | 255.255.255.224 | 32 | 30 | 32 |
| /28 | 255.255.255.240 | 16 | 14 | 16 |
| /29 | 255.255.255.248 | 8 | 6 | 8 |
| /30 | 255.255.255.252 | 4 | 2 | 4 |

Block Size 계산:

```text
Block Size = 256 - Subnet Mask의 해당 Octet
```

예:

```text
/26
255.255.255.192

256 - 192 = 64
```

따라서 Network 시작점:

```text
0
64
128
192
```

Broadcast Address:

```text
다음 Network Address - 1
```

Host Range:

```text
First Host = Network Address + 1
Last Host  = Broadcast Address - 1
```

---

# 주요 Cisco IOS 명령어

## Switch 확인

```cisco
show mac address-table
show interfaces status
show vlan brief
show interfaces trunk
show running-config
```

## Router 확인

```cisco
show ip interface brief
show ip interface
show ip route
show cdp neighbors
show running-config
```

## Interface 설정

```cisco
enable
configure terminal

interface <interface>
 ip address <IP Address> <Subnet Mask>
 no shutdown
```

## Static Route

```cisco
ip route <Destination Network> <Subnet Mask> <Next Hop>
```

예:

```cisco
ip route 192.168.20.0 255.255.255.0 10.0.0.2
```

## IP 제거

```cisco
interface <interface>
 no ip address
```

## DNS Lookup 비활성화

```cisco
no ip domain lookup
```

## Proxy ARP

비활성화:

```cisco
interface <interface>
 no ip proxy-arp
```

활성화:

```cisco
interface <interface>
 ip proxy-arp
```

## End Device 확인

```text
ping <IP Address>
tracert <IP Address>
arp -a
arp -d
```

---

# 학습 진행 상황

## Day 1

- [x] Ethernet Connection
- [x] ARP
- [x] MAC Address Table
- [x] ICMP Ping
- [x] VLAN
- [x] Broadcast Domain
- [x] Access Port
- [x] IEEE 802.1Q
- [x] Trunk Port
- [x] Default Gateway
- [x] Router Subinterface
- [x] Inter-VLAN Routing
- [x] Router-on-a-Stick
- [x] Connected Route
- [x] Local Route

---

## Day 2

- [x] Router-to-Router Network
- [x] `/30` Network
- [x] Static Routing
- [x] Next Hop
- [x] Routing Table
- [x] Static Route 확인
- [x] CDP
- [x] `tracert`
- [x] Interface Troubleshooting
- [x] Routing Troubleshooting
- [x] Overlapping Network 오류 확인 및 해결

---

## Day 3

- [x] CIDR Prefix
- [x] Subnet Mask
- [x] Block Size 계산
- [x] Network Address 계산
- [x] First / Last Host 계산
- [x] Broadcast Address 계산
- [x] `/24` → `/26` Subnetting
- [x] 서로 다른 Subnet 구성
- [x] Connected Routing
- [x] Default Gateway 동작 이해
- [x] 잘못된 Subnet Mask 테스트
- [x] ARP Cache 확인 및 삭제
- [x] Proxy ARP 확인
- [x] Proxy ARP 비활성화 테스트
- [x] Network Troubleshooting

---

# 다음 학습

- [ ] STP / RSTP
- [ ] Root Bridge
- [ ] STP Port Role
- [ ] EtherChannel / LACP
- [ ] OSPF
- [ ] DHCP
- [ ] DHCP Relay
- [ ] NAT / PAT
- [ ] Standard ACL
- [ ] Extended ACL
- [ ] Port Security
- [ ] IPv6
- [ ] 종합 Network Troubleshooting
- [ ] Final Integrated Lab

---

# Repository 구조

```text
ccna-network-labs/
│
├── README.md
│
└── labs/
    │
    ├── 03-router-on-a-stick/
    │   ├── README.md
    │   ├── topology.png
    │   └── 03-router-on-a-stick.pkt
    │
    ├── 04-static-routing/
    │   ├── README.md
    │   ├── topology.png
    │   └── 04-static-routing.pkt
    │
    └── 05-ipv4-subnetting/
        ├── README.md
        ├── topology.png
        └── 05-ipv4-subnetting.pkt
```

각 Lab은 가능한 경우 다음 형식으로 관리합니다.

```text
README.md
→ 실습 목표, 설정, 검증, Troubleshooting, 학습 내용

topology.png
→ Cisco Packet Tracer Network Topology

*.pkt
→ 실제 Packet Tracer 실습 파일
```

---

# Troubleshooting 접근 방식

Network 장애가 발생했을 때 무작정 설정을 변경하기보다  
다음 순서로 확인하는 습관을 만드는 것을 목표로 합니다.

```text
1. Physical / Interface 상태 확인
        ↓
2. IP Address / Subnet Mask 확인
        ↓
3. 실제 연결 Interface 확인
        ↓
4. VLAN / Trunk 상태 확인
        ↓
5. ARP / MAC Address Table 확인
        ↓
6. Routing Table 확인
        ↓
7. Ping / Traceroute로 경로 검증
        ↓
8. 원인 수정
        ↓
9. 정상 통신 재검증
```

---

# 목표

CCNA 학습 과정에서 Cisco Packet Tracer를 이용해 직접 Network를 구성하고,

**Configuration → Verification → Troubleshooting**

과정을 반복하여 Network 동작 원리를 이해하는 것을 목표로 합니다.

단순히 Cisco IOS 명령어를 입력할 수 있는 수준이 아니라,

- Packet이 어떤 경로로 전달되는지
- Switch가 MAC Address를 어떻게 학습하는지
- ARP가 어떤 상황에서 동작하는지
- VLAN이 Broadcast Domain을 어떻게 분리하는지
- Router가 Routing Table을 어떻게 사용하는지
- Subnet Mask가 Local / Remote Network 판단에 어떤 영향을 주는지
- 장애가 발생했을 때 어떤 명령어로 원인을 좁혀야 하는지

를 직접 설명하고 검증할 수 있는 수준까지 학습하는 것이 목표입니다.

최종적으로 VLAN, STP, EtherChannel, OSPF, DHCP, NAT, ACL, IPv6 등을 통합한  
**종합 Network 환경을 직접 구축하고 Troubleshooting하는 Final Lab**을 완성할 예정입니다.
