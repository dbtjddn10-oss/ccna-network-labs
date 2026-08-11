# Lab 04 - 정적 라우팅(Static Routing)

## 실습 목표

Cisco Packet Tracer에서 두 개의 서로 다른 LAN을 구성하고,
두 Router 사이에 Static Route를 설정하여 서로 다른 네트워크 간 통신을 구현했습니다.

이번 실습에서는 단순히 Static Route만 설정하는 것이 아니라,
`show ip interface brief`, `show cdp neighbors`, `show ip route` 등의 명령어를 이용해
실제 통신 장애의 원인을 확인하고 해결하는 과정까지 진행했습니다.

---

## 네트워크 구성도

![Static Routing Topology](./topology.png)

```text
PC0
192.168.10.10/24
GW 192.168.10.1
        |
     Switch0
        |
Router0 G0/0
192.168.10.1/24
        |
Router0 G0/1
10.0.0.1/30
        |
        | 10.0.0.0/30
        |
Router1 G0/0
10.0.0.2/30
        |
Router1 G0/1
192.168.20.1/24
        |
     Switch1
        |
PC1
192.168.20.20/24
GW 192.168.20.1
```

---

## IP 주소 구성

| 장비 | 인터페이스 | IP 주소 | 서브넷 마스크 | 기본 게이트웨이 |
|---|---|---|---|---|
| PC0 | FastEthernet0 | 192.168.10.10 | 255.255.255.0 | 192.168.10.1 |
| Router0 | G0/0 | 192.168.10.1 | 255.255.255.0 | - |
| Router0 | G0/1 | 10.0.0.1 | 255.255.255.252 | - |
| Router1 | G0/0 | 10.0.0.2 | 255.255.255.252 | - |
| Router1 | G0/1 | 192.168.20.1 | 255.255.255.0 | - |
| PC1 | FastEthernet0 | 192.168.20.20 | 255.255.255.0 | 192.168.20.1 |

---

## 네트워크 주소 정리

### LAN 1

- 네트워크 주소: `192.168.10.0/24`
- Router0: `192.168.10.1`
- PC0: `192.168.10.10`

### Router 간 연결 네트워크

- 네트워크 주소: `10.0.0.0/30`
- Network Address: `10.0.0.0`
- Router0: `10.0.0.1`
- Router1: `10.0.0.2`
- Broadcast Address: `10.0.0.3`

`/30` 네트워크는 전체 주소가 4개이며,
실제로 장비에 할당할 수 있는 Host Address는 2개입니다.

```text
10.0.0.0 → Network Address
10.0.0.1 → Router0
10.0.0.2 → Router1
10.0.0.3 → Broadcast Address
```

### LAN 2

- 네트워크 주소: `192.168.20.0/24`
- Router1: `192.168.20.1`
- PC1: `192.168.20.20`

---

## Router0 설정

```cisco
enable
configure terminal

no ip domain lookup

interface gigabitethernet 0/0
 ip address 192.168.10.1 255.255.255.0
 no shutdown
exit

interface gigabitethernet 0/1
 ip address 10.0.0.1 255.255.255.252
 no shutdown
exit

ip route 192.168.20.0 255.255.255.0 10.0.0.2

end
```

---

## Router1 설정

```cisco
enable
configure terminal

no ip domain lookup

interface gigabitethernet 0/0
 ip address 10.0.0.2 255.255.255.252
 no shutdown
exit

interface gigabitethernet 0/1
 ip address 192.168.20.1 255.255.255.0
 no shutdown
exit

ip route 192.168.10.0 255.255.255.0 10.0.0.1

end
```

---

## 정적 라우팅 설정

Router0에서 LAN 2로 가는 경로를 설정했습니다.

```cisco
ip route 192.168.20.0 255.255.255.0 10.0.0.2
```

의미:

```text
목적지 네트워크 : 192.168.20.0/24
다음 홉(Next Hop) : 10.0.0.2
```

즉 Router0에게 다음과 같이 알려주는 설정입니다.

```text
192.168.20.0/24 네트워크로 가려면
다음 Router인 10.0.0.2로 패킷을 전달한다.
```

Router1에서는 반대로 LAN 1로 가는 경로를 설정했습니다.

```cisco
ip route 192.168.10.0 255.255.255.0 10.0.0.1
```

의미:

```text
목적지 네트워크 : 192.168.10.0/24
다음 홉(Next Hop) : 10.0.0.1
```

양방향 통신을 위해서는 목적지로 가는 경로뿐만 아니라
응답 패킷이 돌아올 수 있는 Return Path도 필요합니다.

---

## 라우팅 테이블 확인

사용 명령어:

```cisco
show ip route
```

주요 Routing Code:

```text
C = Connected
L = Local
S = Static
```

예시:

```text
S    192.168.20.0/24 [1/0] via 10.0.0.2
```

각 항목의 의미:

- `S` = Static Route
- `192.168.20.0/24` = 목적지 네트워크
- `1` = Administrative Distance
- `0` = Metric
- `10.0.0.2` = Next-Hop Router

Static Route의 기본 Administrative Distance는 `1`입니다.

---

## 통신 확인

### Router0에서 Router1 확인

```cisco
ping 10.0.0.2
```

결과:

```text
Success
```

### Router1에서 Router0 확인

```cisco
ping 10.0.0.1
```

결과:

```text
Success
```

### PC0에서 PC1 확인

```text
ping 192.168.20.20
```

Static Route 설정 후 통신 성공.

### PC1에서 PC0 확인

```text
ping 192.168.10.10
```

통신 성공.

---

## 패킷 경로 확인

PC0에서 다음 명령어를 사용했습니다.

```text
tracert 192.168.20.20
```

패킷 이동 경로:

```text
PC0
 ↓
Router0
 ↓
Router1
 ↓
PC1
```

이를 통해 서로 다른 두 LAN 사이의 패킷이
Router0과 Router1을 거쳐 목적지까지 전달되는 것을 확인했습니다.

---

## 장애 해결 과정

### 문제 1 - Router0과 Router1 사이 Ping 실패

초기 테스트:

```cisco
Router0# ping 10.0.0.2
```

결과:

```text
Success rate is 0 percent (0/5)
```

Router0과 Router1이 직접 연결되어 있음에도 Ping이 실패했습니다.

먼저 다음 명령어를 사용했습니다.

```cisco
show ip interface brief
```

Router0:

```text
GigabitEthernet0/0   192.168.10.1   up   up
GigabitEthernet0/1   10.0.0.1       up   up
```

Router1:

```text
GigabitEthernet0/0   192.168.20.1   up   up
GigabitEthernet0/1   10.0.0.2       up   up
```

모든 관련 인터페이스가 `up/up` 상태였기 때문에
단순한 `shutdown` 상태나 물리 연결 문제일 가능성은 낮았습니다.

---

### 문제 2 - 실제 연결된 인터페이스 확인

인터페이스 상태는 정상이었지만 실제 연결 포트가 예상과 다를 가능성이 있어
다음 명령어를 사용했습니다.

```cisco
show cdp neighbors
```

Router0 결과:

```text
G0/0 → Switch0
G0/1 → Router1
```

Router1 결과:

```text
G0/0 → Router0
G0/1 → Switch1
```

실제 연결은 다음과 같았습니다.

```text
Switch0
   |
Router0 G0/0

Router0 G0/1
   |
Router1 G0/0

Router1 G0/1
   |
Switch1
```

하지만 Router1의 IP Address를 반대로 설정해 둔 상태였습니다.

잘못된 설정:

```text
Router1 G0/0 = 192.168.20.1/24
Router1 G0/1 = 10.0.0.2/30
```

실제 연결에 맞는 올바른 설정:

```text
Router1 G0/0 = 10.0.0.2/30
Router1 G0/1 = 192.168.20.1/24
```

IP 주소를 올바른 인터페이스로 이동한 후
Router0과 Router1 사이의 Ping이 정상적으로 성공했습니다.

---

### 문제 3 - Overlapping Network 오류

Router1의 IP 주소를 변경하는 과정에서 다음 오류가 발생했습니다.

```text
% 10.0.0.0 overlaps with GigabitEthernet0/1
```

원인은 기존 `G0/1`에 이미 `10.0.0.2/30`이 설정되어 있는 상태에서
`G0/0`에도 같은 `10.0.0.0/30` 네트워크의 주소를 설정하려고 했기 때문입니다.

따라서 기존 인터페이스에서 IP 주소를 먼저 제거했습니다.

```cisco
interface gigabitethernet 0/1
 no ip address
```

이후 올바른 인터페이스에 주소를 설정했습니다.

```cisco
interface gigabitethernet 0/0
 ip address 10.0.0.2 255.255.255.252
```

이를 통해 서로 다른 Layer 3 Router Interface에
서로 겹치는 IP Network를 일반적으로 동시에 설정할 수 없다는 점을 확인했습니다.

---

## 사용한 주요 명령어

### Privileged EXEC Mode 진입

```cisco
enable
```

### Global Configuration Mode 진입

```cisco
configure terminal
```

### 인터페이스 상태 확인

```cisco
show ip interface brief
```

용도:

- 인터페이스별 IP Address 확인
- Interface Status 확인
- Line Protocol 확인
- `up/up`, `down/down`, `administratively down/down` 등의 상태 확인

### 라우팅 테이블 확인

```cisco
show ip route
```

용도:

- Connected Route 확인
- Local Route 확인
- Static Route 확인
- 목적지 네트워크에 대한 경로 존재 여부 확인

### 직접 연결된 Cisco 장비 확인

```cisco
show cdp neighbors
```

용도:

- 현재 Router/Switch에 직접 연결된 Cisco 장비 확인
- Local Interface 확인
- 상대 장비의 Interface 확인
- 실제 케이블 연결 구조 확인

### Ping 테스트

```cisco
ping <IP Address>
```

용도:

- Layer 3 통신 가능 여부 확인

### 경로 추적

```text
tracert <IP Address>
```

용도:

- 목적지까지 패킷이 거치는 Layer 3 경로 확인

### 인터페이스 활성화

```cisco
no shutdown
```

용도:

- Administratively Down 상태의 Router Interface 활성화

### IP 주소 제거

```cisco
no ip address
```

용도:

- 해당 인터페이스에 설정된 IPv4 Address 제거

### DNS Lookup 비활성화

```cisco
no ip domain lookup
```

용도:

Cisco IOS에서 잘못 입력한 명령어를 Domain Name으로 인식하여
DNS 조회를 시도하는 현상을 방지합니다.

---

## 이번 실습에서 이해한 핵심 개념

### 1. Connected Route

Router Interface에 IP Address를 설정하고 Interface가 활성화되면
Router는 해당 Network를 자동으로 Routing Table에 등록합니다.

```text
C = Connected
```

예:

```text
C 10.0.0.0/30
```

---

### 2. Local Route

Router 자신의 Interface IP Address에 대한 Route입니다.

```text
L = Local
```

예:

```text
L 10.0.0.1/32
```

`/32`는 특정 IPv4 Address 하나만을 의미합니다.

---

### 3. Static Route

Router가 자동으로 알 수 없는 Remote Network에 대해
관리자가 직접 경로를 등록하는 방식입니다.

```text
S = Static
```

예:

```text
S 192.168.20.0/24 [1/0] via 10.0.0.2
```

---

### 4. Next Hop

현재 Router가 목적지 네트워크로 패킷을 전달할 때
다음으로 보내야 하는 Router의 IP Address입니다.

예:

```text
10.0.0.2
```

---

### 5. Return Path

PC0에서 PC1까지 가는 경로만 있다고 통신이 완성되는 것은 아닙니다.

```text
PC0 → Router0 → Router1 → PC1
```

응답도 다시 돌아와야 합니다.

```text
PC1 → Router1 → Router0 → PC0
```

따라서 양쪽 Router가 서로의 Remote Network에 대한 경로를 알고 있어야 합니다.

---

### 6. up/up의 의미

```text
Status   : up
Protocol : up
```

인 경우 Interface의 물리적 연결과 Data Link 상태가 정상임을 의미합니다.

하지만 `up/up`이라고 해서
해당 Interface가 내가 의도한 장비나 네트워크에 연결되어 있다는 뜻은 아닙니다.

이번 실습에서도 모든 Interface가 `up/up` 상태였지만
Router1의 IP Address가 실제 연결 구조와 맞지 않아 통신이 실패했습니다.

---

### 7. CDP

CDP(Cisco Discovery Protocol)를 통해
직접 연결된 Cisco 장비와 Interface 정보를 확인할 수 있습니다.

```cisco
show cdp neighbors
```

이번 실습에서는 이 명령어를 이용해
Router1의 실제 연결 Interface를 찾아낼 수 있었습니다.

---

## 장애 확인 순서

이번 실습에서 사용한 기본적인 Troubleshooting 흐름입니다.

```text
1. Ping으로 통신 실패 확인
        ↓
2. show ip interface brief
        ↓
3. Interface IP / up-up 상태 확인
        ↓
4. show cdp neighbors
        ↓
5. 실제 장비 연결 Interface 확인
        ↓
6. IP Address 설정 오류 발견
        ↓
7. 설정 수정
        ↓
8. Router 간 Ping 성공 확인
        ↓
9. show ip route 확인
        ↓
10. Static Route 설정
        ↓
11. PC 간 Ping 확인
        ↓
12. tracert로 실제 경로 확인
```

---

## 이번 실습에서 배운 점

- Router는 직접 연결된 Network를 자동으로 학습한다.
- Router는 직접 연결되지 않은 Remote Network를 자동으로 알 수 없다.
- Static Route를 이용해 Remote Network로 가는 경로를 직접 설정할 수 있다.
- Static Route에서 Next-Hop IP는 다음 Router의 IP Address를 의미한다.
- 양방향 통신에는 Return Path가 필요하다.
- `/30` Network는 사용 가능한 Host Address가 2개이다.
- `show ip route`를 통해 Router가 알고 있는 경로를 확인할 수 있다.
- `show ip interface brief`를 통해 Interface IP와 상태를 빠르게 확인할 수 있다.
- Interface가 `up/up`이더라도 의도한 장비에 연결되어 있다는 뜻은 아니다.
- `show cdp neighbors`를 통해 실제 연결된 Cisco 장비와 Interface를 확인할 수 있다.
- 서로 겹치는 IP Network를 서로 다른 Router Interface에 일반적으로 동시에 설정할 수 없다.
- `ping`으로 통신 가능 여부를 확인할 수 있다.
- `tracert`를 통해 목적지까지 패킷이 거치는 Router를 확인할 수 있다.
- Routing 문제를 해결하기 전에 직접 연결된 구간부터 확인하는 것이 중요하다.

---

## 실습 파일

```text
04-static-routing/
├── README.md
├── topology.png
└── 04-static-routing.pkt
```

---

## 실습 완료 항목

- [x] PC IPv4 Address 설정
- [x] Router Interface IPv4 Address 설정
- [x] Router Interface 활성화
- [x] `/30` Router-to-Router Network 구성
- [x] Router 간 직접 통신 확인
- [x] Static Route 설정
- [x] Routing Table 확인
- [x] PC0 ↔ PC1 End-to-End Ping 확인
- [x] `tracert`를 통한 경로 확인
- [x] `show ip interface brief`를 이용한 장애 확인
- [x] `show cdp neighbors`를 이용한 연결 구조 확인
- [x] Interface/IP Address 불일치 문제 해결
- [x] Overlapping Network 오류 확인 및 해결

---

## Lab Status

**완료 (Completed)**