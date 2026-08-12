# Lab 07 - EtherChannel / LACP

<p align="center">
  <img src="./topology.png" alt="EtherChannel LACP Topology" width="1000">
</p>

## 실습 목표

Cisco Packet Tracer에서 Switch 두 대 사이에 여러 개의 물리적 Ethernet Link를 구성하고,
STP가 중복 Link를 어떻게 처리하는지 먼저 확인했습니다.

이후 LACP를 사용하여 두 개의 GigabitEthernet Link를 하나의
논리적인 Port-channel로 묶어 EtherChannel을 구성했습니다.

또한 EtherChannel Member Link 장애,
설정 불일치, LACP Mode, STP와 EtherChannel의 관계,
Load Balancing 방식까지 직접 확인했습니다.

이번 Lab에서는 단순히 EtherChannel을 구성하는 것뿐만 아니라,

```text
Configuration
→ Verification
→ Failure Test
→ Troubleshooting
→ Recovery
```

과정을 직접 실습하는 것을 목표로 했습니다.

---

# 네트워크 구성

```text
PC0                                         PC1
192.168.1.10/24                       192.168.1.20/24
 |                                            |
 | Fa0                                     Fa0 |
 |                                            |
Fa0/1                                      Fa0/1
Switch0 ================================= Switch1
         Gi0/1                 Gi0/1
         Gi0/2                 Gi0/2

              LACP EtherChannel
                 Port-channel1
```

Switch 두 대 사이에 두 개의 GigabitEthernet Link를 구성했습니다.

```text
Switch0 Gi0/1 ↔ Switch1 Gi0/1
Switch0 Gi0/2 ↔ Switch1 Gi0/2
```

PC 연결:

```text
PC0 Fa0 ↔ Switch0 Fa0/1
PC1 Fa0 ↔ Switch1 Fa0/1
```

---

# PC Addressing

PC0:

```text
IP Address  : 192.168.1.10
Subnet Mask : 255.255.255.0
```

PC1:

```text
IP Address  : 192.168.1.20
Subnet Mask : 255.255.255.0
```

두 PC는 같은:

```text
192.168.1.0/24
```

Network에 있기 때문에 별도의 Default Gateway는 설정하지 않았습니다.

---

# EtherChannel이 필요한 이유

두 Switch 사이에 여러 개의 물리적 Link를 연결하면
Network Redundancy를 확보할 수 있습니다.

하지만 EtherChannel을 구성하지 않고 같은 Switch 사이에
여러 Layer 2 Link를 연결하면 STP는 이를 중복 경로로 판단합니다.

예:

```text
          Gi0/1
Switch0 ========= Switch1
          Gi0/2
```

STP 입장에서는 두 Link를 모두 Forwarding 상태로 사용하면
Layer 2 Loop가 발생할 수 있습니다.

따라서 한 Link는:

```text
Forwarding
```

다른 Link는:

```text
Blocking
```

상태로 만들 수 있습니다.

즉 물리적인 Link가 두 개 있어도
실제 Forwarding에는 하나만 사용될 수 있습니다.

---

# EtherChannel

EtherChannel은 여러 개의 물리적 Ethernet Link를
하나의 논리적인 Interface로 묶는 기술입니다.

```text
Gi0/1 ────────────── Gi0/1
         +
Gi0/2 ────────────── Gi0/2

          ↓

     Port-channel1
```

Switch와 STP는 개별 Member Link 대신:

```text
Po1
```

이라는 하나의 논리 Link로 처리할 수 있습니다.

---

# EtherChannel의 장점

EtherChannel을 이용하면:

```text
여러 물리 Link
       ↓
하나의 논리 Link
       ↓
대역폭 활용 + Redundancy
```

가 가능합니다.

Member Link 하나에 장애가 발생하더라도
다른 정상 Member Link가 남아 있다면 Port-channel 자체는
계속 사용할 수 있습니다.

---

# LACP

LACP는:

```text
Link Aggregation Control Protocol
```

의 약자입니다.

여러 개의 Ethernet Link를 EtherChannel로 묶기 위해
장비 간 협상을 수행하는 표준 Protocol입니다.

LACP Mode:

```text
active
passive
```

---

# LACP Active

```text
active
```

Mode는 LACP Negotiation을 적극적으로 시작합니다.

쉽게 표현하면:

```text
"EtherChannel 만들자."
```

라고 먼저 상대 Switch에 LACP 메시지를 보내는 쪽입니다.

---

# LACP Passive

```text
passive
```

Mode는 상대방이 LACP Negotiation을 시작하면 응답합니다.

쉽게 표현하면:

```text
"네가 먼저 협상하면 나는 응답할게."
```

라고 이해할 수 있습니다.

---

# LACP Mode 조합

정상적인 LACP 조합:

```text
active  + active  = EtherChannel 가능
active  + passive = EtherChannel 가능
```

정상적으로 EtherChannel이 형성되지 않는 조합:

```text
passive + passive = EtherChannel 형성 불가
```

양쪽 모두 Passive이면 아무 장비도 적극적으로
LACP Negotiation을 시작하지 않기 때문입니다.

---

# EtherChannel 적용 전 STP 확인

EtherChannel 설정 전에 Switch 두 대 사이에
Gi0/1과 Gi0/2 Link를 모두 연결했습니다.

Switch1:

```cisco
show spanning-tree vlan 1
```

결과:

```text
Gi0/2   Altn BLK 4
Gi0/1   Root FWD 4
Fa0/1   Desg FWD 19
```

Switch1의:

```text
Gi0/1
```

은 Root Port로 선택되어 Forwarding 상태였습니다.

반면:

```text
Gi0/2
```

는 중복 경로로 판단되어:

```text
Altn BLK
```

상태가 되었습니다.

---

# EtherChannel 적용 전 구조

```text
Switch0
 ROOT
  ||
  || Gi0/1
  || Forwarding
  ||
Switch1

Gi0/2
Alternate / Blocking
```

즉 두 개의 물리 Link를 연결했지만
STP가 Loop 방지를 위해 하나의 경로를 차단했습니다.

---

# LACP EtherChannel 구성

Switch0은 LACP Active Mode로 설정했습니다.

## Switch0

```cisco
enable
configure terminal

interface range gigabitethernet 0/1 - 2
 channel-group 1 mode active

end
```

Switch1은 Passive Mode로 설정했습니다.

## Switch1

```cisco
enable
configure terminal

interface range gigabitethernet 0/1 - 2
 channel-group 1 mode passive

end
```

구성:

```text
Switch0                     Switch1

Gi0/1 ==================== Gi0/1
Gi0/2 ==================== Gi0/2

ACTIVE                     PASSIVE

           LACP
            ↓
           Po1
```

---

# interface range

다음 명령어:

```cisco
interface range gigabitethernet 0/1 - 2
```

를 사용하면:

```text
Gi0/1
Gi0/2
```

두 Interface를 동시에 선택할 수 있습니다.

따라서 동일한 EtherChannel 설정을 여러 Interface에
한 번에 적용할 수 있습니다.

---

# channel-group

다음 명령어:

```cisco
channel-group 1 mode active
```

에서:

```text
1
```

은 EtherChannel Group Number입니다.

이번 실습에서는:

```text
Channel-group 1
```

을 사용했으며 논리적인 Interface:

```text
Port-channel1
```

이 생성되었습니다.

Cisco IOS에서는 보통:

```text
Port-channel1
```

을 짧게:

```text
Po1
```

로 표시합니다.

---

# EtherChannel 상태 확인

EtherChannel 상태 확인:

```cisco
show etherchannel summary
```

정상 결과:

```text
Group  Port-channel  Protocol    Ports

1      Po1(SU)       LACP        Gig0/1(P) Gig0/2(P)
```

---

# EtherChannel Flag 해석

이번 실습에서 확인한 주요 Flag:

```text
S = Layer 2
U = in use
P = in port-channel
D = down
I = stand-alone
s = suspended
```

정상적인 EtherChannel:

```text
Po1(SU)
```

의 의미:

```text
S
→ Layer 2 EtherChannel

U
→ 현재 Port-channel 사용 중
```

---

# Member Port 정상 상태

```text
Gig0/1(P)
Gig0/2(P)
```

에서:

```text
P
```

는 해당 Interface가 정상적으로
Port-channel에 Bundle되어 있다는 의미입니다.

따라서:

```text
Po1(SU)

Gi0/1(P)
Gi0/2(P)
```

상태가 정상적인 EtherChannel 구성입니다.

---

# EtherChannel 적용 후 STP 변화

EtherChannel 적용 전 Switch1:

```text
Gi0/1 Root FWD
Gi0/2 Altn BLK
```

EtherChannel 적용 후:

```cisco
show spanning-tree vlan 1
```

결과:

```text
Fa0/1   Desg FWD 19
Po1     Root FWD 3
```

기존의:

```text
Gi0/1
Gi0/2
```

가 STP 출력에서 개별적으로 보이지 않고:

```text
Po1
```

이라는 하나의 논리 Interface로 보였습니다.

---

# STP와 EtherChannel의 관계

EtherChannel 적용 전 STP:

```text
Gi0/1
Gi0/2

→ 서로 다른 두 개의 Layer 2 Link
```

EtherChannel 적용 후:

```text
Gi0/1 ┐
      ├── Po1
Gi0/2 ┘

→ 하나의 논리 Link
```

따라서 STP는 EtherChannel Member Link 각각을
별개의 중복 경로로 처리하는 것이 아니라:

```text
Port-channel1
```

을 하나의 Interface로 처리합니다.

---

# STP Cost 변화

EtherChannel 적용 전:

```text
GigabitEthernet
Cost = 4
```

EtherChannel 구성 후:

```text
Po1
Cost = 3
```

을 확인했습니다.

이번 Packet Tracer 환경에서는
두 개의 1 Gbps Member Link가 정상적으로 Bundle된 상태에서
Port-channel의 STP Cost가 3으로 계산되었습니다.

```text
Gi0/1 = 1 Gbps
Gi0/2 = 1 Gbps

        ↓

EtherChannel
Aggregate Bandwidth 증가

        ↓

Po1 STP Cost = 3
```

STP에서는 더 낮은 Cost의 경로가 우선적으로 선택됩니다.

---

# EtherChannel Member Link 장애 실험

EtherChannel의 Redundancy를 확인하기 위해
Switch0의 Member Link 하나를 수동으로 비활성화했습니다.

Switch0:

```cisco
enable
configure terminal

interface gigabitethernet 0/1
 shutdown

end
```

---

# 장애 후 EtherChannel 상태

```cisco
show etherchannel summary
```

결과:

```text
Po1(SU)   LACP   Gig0/1(D) Gig0/2(P)
```

해석:

```text
Gi0/1(D)
→ Interface Down

Gi0/2(P)
→ 정상적으로 Port-channel 참여 중

Po1(SU)
→ Port-channel 자체는 계속 정상 사용 중
```

---

# Member Failure 구조

장애 전:

```text
          Gi0/1 ✅
Switch0 ================= Switch1
          Gi0/2 ✅

            Po1 ✅
```

장애 후:

```text
          Gi0/1 ❌
Switch0 ================= Switch1
          Gi0/2 ✅

            Po1 ✅
```

Member Link 하나가 Down 상태가 되었지만
다른 Link가 정상적으로 유지되어 Port-channel 자체는
계속 사용할 수 있었습니다.

---

# 장애 후 STP Cost 변화

Member Link 두 개가 정상일 때:

```text
Po1 Cost = 3
```

Gi0/1 장애 후:

```text
Po1 Cost = 4
```

를 확인했습니다.

즉 사용 가능한 EtherChannel Member가:

```text
2개
↓
1개
```

로 감소하면서 논리 Interface의 사용 가능한 대역폭도 감소했고,
Packet Tracer의 STP Cost가 이에 따라 변경되었습니다.

---

# 장애 상황에서 Ping 확인

PC0에서:

```text
ping 192.168.1.20
```

을 수행했습니다.

Gi0/1 Member Link가 Down 상태임에도
Gi0/2를 통해 통신이 계속 가능했습니다.

즉:

```text
Member Link 장애
       ↓
나머지 Member Link 유지
       ↓
Po1 유지
       ↓
End-to-End 통신 성공
```

을 확인했습니다.

---

# Member Link 복구

Switch0:

```cisco
enable
configure terminal

interface gigabitethernet 0/1
 no shutdown

end
```

복구 후:

```cisco
show etherchannel summary
```

결과:

```text
Po1(SU)   LACP   Gig0/1(P) Gig0/2(P)
```

두 Interface 모두 다시 정상적으로 Bundle되었습니다.

---

# STP Learning 상태 확인

Interface 복구 직후:

```cisco
show spanning-tree vlan 1
```

에서 일시적으로:

```text
Po1 Desg LRN
```

상태를 확인했습니다.

```text
LRN
→ Learning
```

입니다.

Topology 변화 직후 STP가 상태를 다시 계산하는 과정에서
Learning 상태를 거친 후 최종적으로:

```text
Po1 Desg FWD
```

상태가 되었습니다.

---

# LACP Passive + Passive 테스트

LACP 동작을 확인하기 위해 양쪽 Switch를 모두:

```text
passive
```

로 설정하는 실험을 수행했습니다.

실제 Cisco LACP 동작 원리에서는:

```text
passive + passive
```

조합은 양쪽 모두 Negotiation을 먼저 시작하지 않기 때문에
EtherChannel이 정상적으로 형성되지 않아야 합니다.

---

# Packet Tracer에서 확인한 동작 차이

이번 Packet Tracer 실습에서는
양쪽 Switch의 Running Configuration에서 실제로:

```text
channel-group 1 mode passive
```

가 설정되어 있는 것을 확인했음에도:

```cisco
show etherchannel summary
```

에서:

```text
Po1(SU)
Gig0/1(P)
Gig0/2(P)
```

상태가 유지되는 현상을 확인했습니다.

즉 Packet Tracer에서는 기존 LACP EtherChannel 상태가
계속 유지되는 시뮬레이션 동작 차이가 발생했습니다.

따라서 실제 Cisco LACP 개념은:

```text
active + active   = 가능
active + passive  = 가능
passive + passive = 불가
```

로 이해하고,
이번 결과는 Packet Tracer의 시뮬레이션 구현 차이 또는 상태 갱신 한계로 기록했습니다.

---

# show lacp neighbor 지원 확인

다음 명령어도 테스트했습니다.

```cisco
show lacp neighbor
```

Packet Tracer에서 사용한 Switch IOS에서는:

```text
% Invalid input detected at '^' marker.
```

가 출력되었습니다.

따라서 해당 Packet Tracer 장비에서는
`show lacp neighbor` 명령어를 지원하지 않는 것을 확인했습니다.

---

# EtherChannel 설정 불일치 Troubleshooting

이번에는 Switch1의 Gi0/2 Interface만
EtherChannel에서 의도적으로 제거했습니다.

Switch1:

```cisco
enable
configure terminal

interface gigabitethernet 0/2
 no channel-group 1

end
```

이 상태에서는 양쪽 Switch의 EtherChannel Member 구성이
서로 일치하지 않게 됩니다.

```text
Switch0                     Switch1

Gi0/1 Po1 ================= Gi0/1 Po1

Gi0/2 Po1 ================= Gi0/2
                             ↑
                     Channel-group 없음
```

---

# 설정 불일치에서도 Ping이 가능한 이유

설정 불일치 상태에서도:

```text
PC0 → PC1
```

Ping이 성공할 수 있었습니다.

이는 정상 상태의 Member Link 또는 STP가 허용한 경로가
여전히 존재하기 때문입니다.

따라서:

```text
Ping Success
```

만으로 EtherChannel 전체가 정상이라고 판단할 수 없습니다.

---

# 중요한 Troubleshooting 원칙

```text
Ping 성공
≠
EtherChannel 전체 구성 정상
```

EtherChannel 상태를 확인할 때는 반드시:

```cisco
show etherchannel summary
```

를 통해 Member Interface가 실제로:

```text
(P)
```

상태인지 확인해야 합니다.

정상 기대값:

```text
Po1(SU)

Gi0/1(P)
Gi0/2(P)
```

---

# 설정 불일치 복구

Switch1 Gi0/2를 다시 LACP Passive 상태로
EtherChannel에 추가했습니다.

```cisco
enable
configure terminal

interface gigabitethernet 0/2
 channel-group 1 mode passive

end
```

복구 후:

```cisco
show etherchannel summary
```

를 통해:

```text
Po1(SU)
Gi0/1(P)
Gi0/2(P)
```

상태로 돌아온 것을 확인했습니다.

---

# EtherChannel Load Balancing

EtherChannel이 여러 Member Link를 가지고 있다고 해서
하나의 Frame이나 하나의 단일 통신을 단순히 절반씩 나누어
두 Link에 동시에 보내는 것은 아닙니다.

Switch는 Hash 기반 Load Balancing 방식을 이용하여
각 Traffic Flow를 Member Link에 분산합니다.

---

# Load Balancing 확인

Switch에서:

```cisco
show etherchannel load-balance
```

를 실행했습니다.

결과:

```text
EtherChannel Load-Balancing Operational State (src-mac):

Non-IP: Source MAC address
IPv4:   Source MAC address
IPv6:   Source MAC address
```

따라서 이번 Switch의 EtherChannel Load Balancing 기준은:

```text
Source MAC Address
```

였습니다.

---

# src-mac Load Balancing

현재 방식:

```text
src-mac
```

은 Source MAC Address를 이용하여
어떤 EtherChannel Member Link를 사용할지 결정합니다.

개념적으로:

```text
Traffic A → Gi0/1
Traffic B → Gi0/2
Traffic C → Gi0/1
Traffic D → Gi0/2
```

와 같은 형태로 여러 Traffic Flow가 Member Link들에
분산될 수 있습니다.

실제 Member 선택은 Switch의 Hash Algorithm에 의해 결정됩니다.

---

# EtherChannel 대역폭에 대한 주의점

예를 들어:

```text
Gi0/1 = 1 Gbps
Gi0/2 = 1 Gbps
```

를 EtherChannel로 구성하면 전체 Aggregate Capacity는
두 Link를 활용할 수 있게 됩니다.

하지만 이것이:

```text
단일 PC → 단일 PC의 하나의 Flow
= 항상 2 Gbps
```

라는 의미는 아닙니다.

EtherChannel은 Hash를 이용해 Traffic Flow를
특정 Member Link에 할당하기 때문에
하나의 Flow는 특정 Link 하나를 계속 사용할 수 있습니다.

따라서 EtherChannel의 장점은:

```text
여러 Traffic Flow가 존재할 때
전체 Link 자원을 분산하여 활용
```

하는 데 있습니다.

---

# 최종 EtherChannel 구성

Switch0:

```text
Gi0/1
channel-group 1 mode active

Gi0/2
channel-group 1 mode active
```

Switch1:

```text
Gi0/1
channel-group 1 mode passive

Gi0/2
channel-group 1 mode passive
```

---

# 최종 상태 확인

```cisco
show etherchannel summary
```

정상 결과:

```text
Po1(SU)   LACP   Gig0/1(P) Gig0/2(P)
```

STP:

```cisco
show spanning-tree vlan 1
```

에서 Port-channel 자체가:

```text
Po1
```

으로 표시되는 것을 확인했습니다.

PC 통신:

```text
PC0
192.168.1.10/24
      ↕
PC1
192.168.1.20/24
```

Ping 정상.

---

# 주요 Verification 명령어

## EtherChannel 상태

```cisco
show etherchannel summary
```

가장 중요한 EtherChannel 확인 명령어입니다.

정상적인 Member Port는:

```text
(P)
```

상태인지 확인합니다.

---

## STP 상태

```cisco
show spanning-tree vlan 1
```

EtherChannel 구성 전에는 물리 Interface가 개별적으로 보이지만,
구성 후에는 Port-channel이 하나의 논리 Interface로 보이는 것을
확인할 수 있습니다.

---

## Load Balancing

```cisco
show etherchannel load-balance
```

EtherChannel의 현재 Load Balancing 기준을 확인합니다.

이번 실습 결과:

```text
src-mac
```

---

## Running Configuration

```cisco
show running-config
```

또는 필요한 항목만:

```cisco
show running-config | include channel-group
```

을 사용하여 Interface에 설정된 LACP Mode를 확인했습니다.

---

# 사용한 주요 Cisco IOS 명령어

## LACP Active 구성

```cisco
configure terminal

interface range gigabitethernet 0/1 - 2
 channel-group 1 mode active

end
```

---

## LACP Passive 구성

```cisco
configure terminal

interface range gigabitethernet 0/1 - 2
 channel-group 1 mode passive

end
```

---

## Channel-group 제거

```cisco
configure terminal

interface gigabitethernet 0/2
 no channel-group 1

end
```

---

## EtherChannel 확인

```cisco
show etherchannel summary
```

---

## STP 확인

```cisco
show spanning-tree vlan 1
```

---

## Load Balancing 확인

```cisco
show etherchannel load-balance
```

---

## EtherChannel Member 장애 생성

```cisco
configure terminal

interface gigabitethernet 0/1
 shutdown

end
```

---

## Member Interface 복구

```cisco
configure terminal

interface gigabitethernet 0/1
 no shutdown

end
```

---

## Channel-group 설정 확인

```cisco
show running-config | include channel-group
```

---

# Troubleshooting / Verification 흐름

```text
Switch 두 대 구성
        ↓
Gi0/1 + Gi0/2 중복 Link 연결
        ↓
STP 상태 확인
        ↓
Gi0/1 Root FWD 확인
        ↓
Gi0/2 Altn BLK 확인
        ↓
LACP EtherChannel 구성
        ↓
Switch0 Active
Switch1 Passive
        ↓
Po1(SU) 확인
        ↓
Gi0/1(P), Gi0/2(P) 확인
        ↓
STP가 Po1 하나로 인식하는 것 확인
        ↓
Po1 STP Cost 3 확인
        ↓
Gi0/1 Shutdown
        ↓
Gi0/1(D)
Gi0/2(P)
        ↓
Po1 계속 UP
        ↓
STP Cost 3 → 4
        ↓
PC0 ↔ PC1 Ping 성공
        ↓
Gi0/1 No Shutdown
        ↓
(P)(P) 정상 복구
        ↓
Passive + Passive 테스트
        ↓
Packet Tracer 동작 차이 확인
        ↓
Running-config 검증
        ↓
Gi0/2 Channel-group 제거
        ↓
EtherChannel 설정 불일치 생성
        ↓
Ping 성공 확인
        ↓
Ping 성공 ≠ EtherChannel 정상
        ↓
show etherchannel summary로 검증
        ↓
Gi0/2 EtherChannel 복구
        ↓
Load Balancing 확인
        ↓
src-mac 확인
        ↓
최종 End-to-End Ping 확인
```

---

# 이번 실습에서 배운 점

EtherChannel을 설정하지 않은 여러 Layer 2 Link는
STP에 의해 중복 경로로 처리되어 하나가 차단될 수 있습니다.

EtherChannel을 이용하면 여러 물리 Interface를:

```text
Port-channel
```

이라는 하나의 논리 Interface로 묶을 수 있습니다.

LACP에서는:

```text
active
passive
```

Mode를 이용해 EtherChannel 협상을 수행합니다.

정상 조합:

```text
active + active
active + passive
```

정상적으로 협상되지 않는 조합:

```text
passive + passive
```

EtherChannel 구성 후 STP는 개별 Member Link가 아닌:

```text
Po1
```

을 하나의 Interface로 처리합니다.

Member Link 하나에 장애가 발생해도
나머지 Member Link가 정상이라면 Port-channel은 계속 사용할 수 있습니다.

EtherChannel의 사용 가능한 Member Link 수가 감소하면
STP Cost가 변경될 수 있다는 것도 직접 확인했습니다.

또한:

```text
Ping 성공
```

만으로 EtherChannel 상태가 정상이라고 판단할 수 없으며:

```cisco
show etherchannel summary
```

를 통해 실제 Member 상태를 확인해야 한다는 것을 배웠습니다.

마지막으로 EtherChannel은 하나의 Traffic Flow를 단순히
여러 Link에 나누는 것이 아니라 Hash 기반 Load Balancing을 사용하며,
이번 Switch에서는:

```text
src-mac
```

방식이 사용되는 것을 확인했습니다.

---

# 핵심 개념 요약

```text
EtherChannel
→ 여러 물리 Ethernet Link를 하나의 논리 Link로 묶는 기술

LACP
→ EtherChannel을 동적으로 협상하는 표준 Protocol

Port-channel
→ EtherChannel이 생성한 논리 Interface

Po1
→ Port-channel1

Active
→ LACP Negotiation을 적극적으로 시작

Passive
→ 상대방이 시작하면 응답

(P)
→ Port-channel에 정상 Bundle된 Member Port

(SU)
→ Layer 2 Port-channel이며 정상 사용 중
```

---

# EtherChannel 동작 요약

```text
Gi0/1 ┐
      │
      ├──── Port-channel1 ──── 하나의 논리 Link
      │
Gi0/2 ┘
```

EtherChannel 적용 전:

```text
Gi0/1 → Forwarding
Gi0/2 → Blocking
```

EtherChannel 적용 후:

```text
Gi0/1(P) ┐
         ├── Po1
Gi0/2(P) ┘

STP → Po1 하나로 처리
```

---

# 장애 대응 요약

정상:

```text
Gi0/1 ✅
Gi0/2 ✅
Po1   ✅
```

Member 하나 장애:

```text
Gi0/1 ❌
Gi0/2 ✅
Po1   ✅
```

모든 Member가 장애 상태가 되면
Port-channel도 정상적인 통신을 유지할 수 없습니다.

---

# 실습 파일

```text
07-etherchannel-lacp/
├── README.md
├── topology.png
└── 07-etherchannel-lacp.pkt
```

---

# 실습 완료 항목

- [x] Switch 2대 구성
- [x] 중복 GigabitEthernet Link 구성
- [x] EtherChannel 적용 전 STP 확인
- [x] Root Port 확인
- [x] Alternate / Blocking Port 확인
- [x] EtherChannel 개념 이해
- [x] LACP 개념 이해
- [x] Active Mode 이해
- [x] Passive Mode 이해
- [x] Active + Passive 구성
- [x] Interface Range 사용
- [x] Channel-group 구성
- [x] Port-channel1 생성
- [x] Po1(SU) 확인
- [x] Gi0/1(P) 확인
- [x] Gi0/2(P) 확인
- [x] EtherChannel과 STP 관계 확인
- [x] STP가 Po1을 논리 Link로 인식하는 것 확인
- [x] STP Cost 4 → 3 확인
- [x] EtherChannel Member Link 장애 생성
- [x] Gi0/1(D) 상태 확인
- [x] Gi0/2(P) 유지 확인
- [x] Member 장애 후 Po1 유지 확인
- [x] Member 장애 후 STP Cost 3 → 4 확인
- [x] 장애 상태 End-to-End Ping 확인
- [x] Member Interface 복구
- [x] Po1(SU), (P)(P) 정상 복구
- [x] LACP Passive + Passive 테스트
- [x] Packet Tracer 시뮬레이션 동작 차이 확인
- [x] Running Configuration으로 LACP Mode 확인
- [x] EtherChannel 설정 불일치 생성
- [x] 설정 불일치 상태 Ping 테스트
- [x] Ping 성공 ≠ EtherChannel 정상 확인
- [x] EtherChannel 구성 복구
- [x] Load Balancing 확인
- [x] src-mac 방식 확인
- [x] 최종 End-to-End 통신 검증

---

# Lab Status

**완료 (Completed)**