# Lab 03 - Router-on-a-Stick

## Objective

Cisco Packet Tracer에서 VLAN 10과 VLAN 20을 구성하고  
Router-on-a-Stick 방식을 이용해 서로 다른 VLAN 간의 통신을 구현했습니다.

---

## Topology

```text
PC0
192.168.10.10/24
GW 192.168.10.1
VLAN 10
    │
  Fa0/1
    │
+---------+
| Switch0 |
+---------+
    │ Gi0/1
    │ 802.1Q Trunk
    │ VLAN 10,20
    │
+---------+
| Router0 |
+---------+
    │
    ├─ G0/0.10
    │  VLAN 10
    │  192.168.10.1/24
    │
    └─ G0/0.20
       VLAN 20
       192.168.20.1/24

Switch0 Fa0/2
    │
 VLAN 20
    │
PC1
192.168.20.20/24
GW 192.168.20.1
```

---

## Addressing Table

| Device | Interface | VLAN | IP Address | Default Gateway |
|---|---|---:|---|---|
| PC0 | FastEthernet0 | 10 | 192.168.10.10/24 | 192.168.10.1 |
| Router0 | G0/0.10 | 10 | 192.168.10.1/24 | - |
| Router0 | G0/0.20 | 20 | 192.168.20.1/24 | - |
| PC1 | FastEthernet0 | 20 | 192.168.20.20/24 | 192.168.20.1 |

---

## Switch Configuration

### VLAN Creation

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

### Access Ports

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

### 802.1Q Trunk

```text
interface gigabitethernet 0/1
 switchport mode trunk
 switchport trunk allowed vlan 10,20
 exit
```

---

## Router Configuration

### Physical Interface

```text
interface gigabitethernet 0/0
 no shutdown
 exit
```

### VLAN 10 Subinterface

```text
interface gigabitethernet 0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
 exit
```

### VLAN 20 Subinterface

```text
interface gigabitethernet 0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
 exit
```

---

## Verification

### VLAN

```text
show vlan brief
```

확인 결과:

```text
VLAN 10 → Fa0/1
VLAN 20 → Fa0/2
```

### Trunk

```text
show interfaces trunk
```

확인 결과:

```text
Gig0/1
Mode          : on
Encapsulation : 802.1Q
Status        : trunking
Allowed VLANs : 10,20
```

### Router Interfaces

```text
show ip interface brief
```

확인 결과:

```text
GigabitEthernet0/0.10  192.168.10.1  up/up
GigabitEthernet0/0.20  192.168.20.1  up/up
```

### Routing Table

```text
show ip route
```

확인 결과:

```text
C 192.168.10.0/24 is directly connected, GigabitEthernet0/0.10
L 192.168.10.1/32 is directly connected, GigabitEthernet0/0.10

C 192.168.20.0/24 is directly connected, GigabitEthernet0/0.20
L 192.168.20.1/32 is directly connected, GigabitEthernet0/0.20
```

- `C` = Connected Network
- `L` = Router 자신의 Local Interface Address

---

## Connectivity Test

PC0에서:

```text
ping 192.168.10.1
ping 192.168.20.1
ping 192.168.20.20
```

PC1에서:

```text
ping 192.168.10.10
```

### Result

**PC0 ↔ PC1 양방향 Ping 성공**

VLAN 10과 VLAN 20 사이의 Inter-VLAN Routing이 정상적으로 동작하는 것을 확인했습니다.

---

## Troubleshooting

### 1. Router Interface가 Down 상태

초기에 Router의 `G0/0`이 administratively down 상태였습니다.

```text
interface g0/0
no shutdown
```

으로 해결했습니다.

### 2. Wrong IOS Mode

다음 명령을 `Router#`에서 실행해 오류가 발생했습니다.

```text
interface g0/0.10
```

Interface 설정은 Global Configuration Mode에서 실행해야 합니다.

```text
Router# configure terminal
Router(config)# interface g0/0.10
```

### 3. Simulation Mode Ping

Packet Tracer의 Simulation Mode에서 일반 Ping을 실행하면서  
패킷 진행이 멈춰 Timeout처럼 보였습니다.

Realtime Mode로 전환한 뒤 정상 통신을 확인했습니다.

### 4. IOS Command Typo

```text
show vlan breif
```

가 아니라

```text
show vlan brief
```

를 사용해야 합니다.

---

## What I Learned

- VLAN은 Broadcast Domain을 분리한다.
- 서로 다른 VLAN 간 통신에는 Layer 3 Routing이 필요하다.
- Access Port는 하나의 VLAN에 Endpoint를 연결한다.
- Trunk Port는 여러 VLAN의 Traffic을 전달한다.
- IEEE 802.1Q를 통해 VLAN Tagging을 수행한다.
- Router-on-a-Stick은 하나의 Physical Interface에서 여러 Subinterface를 사용한다.
- Router는 Routing Table을 이용해 목적지 Network로 Packet을 전달한다.

---

## Lab File

`03-router-on-a-stick.pkt`

Cisco Packet Tracer에서 직접 열어 실습 구성을 확인할 수 있습니다.
