# Network 02

GNS3와 연동하려면 NPcap이 아닌 WinPcap 설치 필요.

![Untitled](Network%2002/Untitled.png)

![Untitled](Network%2002/Untitled%201.png)

![Untitled](Network%2002/Untitled%202.png)

![Untitled](Network%2002/Untitled%203.png)

![Untitled](Network%2002/Untitled%204.png)

ping을 했을 때 패킷이 감. 윈도우와 이더넷의 통신이 원활함을 알 수 있음.

---

## VLSM

한 번 서브넷팅한 것을 또 한번 서브넷팅.

![Untitled](Network%2002/Untitled%205.png)

AWS CLI

![Untitled](Network%2002/Untitled%206.png)

윈도우에는 192.168.1.100에 대한 정보가 없으므로 가지 않음.

윈도우에서 보내는 것은 R1을 지나서 가야 함

![Untitled](Network%2002/Untitled%207.png)

라우팅 테이블에 경로 추가.

![Untitled](Network%2002/Untitled%208.png)

```bash
R1)
conf t
int f0/0
ip add 172.16.1.254 255.255.255.0
no sh
int f0/1
ip add dhcp
no sh
int s1/0
ip add 10.1.12.1 255.255.255.0
no sh
ip route 192.168.1.0 255.255.255.0 s1/0 10.1.12.2
ip route 192.168.2.0 255.255.255.0 s1/0 10.1.12.2 → static route 설정

R2)
conf t
int s1/0
ip add 10.1.12.2 255.255.255.0
no sh
int f0/0
ip add 192.168.1.254 255.255.255.0
no sh
int f0/1
ip add 192.168.2.254 255.255.255.0
no sh
ip route 172.16.100.0 255.255.255.0 s1/0 10.1.12.1
ip route 0.0.0.0 0.0.0.0 s1/0 10.1.12.1

R3)
conf t
int f0/0
ip add 192168.1.100 255.255.255.0
no sh
no ip routing
ip default-gateway 192.168.1.254
PC1)
ip 192.168.2.20  192.168.2.254
```

컴퓨터는 L7 장치.

컴퓨터도 라우팅 정보가 있음.

![Untitled](Network%2002/Untitled%209.png)

HTTP server에 접속할 수 있도록 함.

GNS - config 상태에서는 do를 붙여서 명령어 실행.

```bash
ip route [ip주소] [서브넷 마스크] [보내려는 곳] [보내는 곳 ip주소]
```

ping을 보내는 시작 주소 바꾸기

```bash
ping [ip 주소] source [시작 주소]
```

![Untitled](Network%2002/Untitled%2010.png)

ip add dhcp → 자동할당받도록 설정.

![Untitled](Network%2002/Untitled%2011.png)

위에 해당이 안 되는 ip는 모두 0.0.0.0으로 감.

```bash
ip route 0.0.0.0 0.0.0.0 s1/0 10.1.12.1
```

라우팅 테이블 안에 해당되는 주소가 없으면 10.1.12.1로 통해 외부로 나감.

0.0.0.0은 모든 DNS 주소를 말함.

라우팅을 했다고 해서 목적지로 간다는 보장X.

관리자에 의해 관리가 가능하여 static도 많이 사용.

정보입력.

중간에 거쳐가는 것은 통신의 목적이 아니라면 정보를 입력할 필요X.

```bash
R1)
no ip route 192.168.1.0 255.255.255.0 s1/0 10.1.12.2
no ip route 192.168.2.0 255.255.255.0 s1/0 10.1.12.2

                      0000 00 01
                      0000 00 10
                      1111 11 00      192.168.0.0/22

ip route 192.168.0.0 255.255.252.0 s1/0 10.1.12.2
```

Dynamic Routing Protocol

1. OSPF(Open Shortest Path First)

R1)

sh run | i ip route ⇒ show running-config|include ip route

라우팅 프로토콜은 AS 내부에서 도달가능정보를 주고받기 위해 사용하는 프로토콜(IGP, Interior Gateway Protocol)과 외부의 AS와 통신을 하고자 할 때 도달가능정보를 교환하기 위해 사용하는 프로토콜(EGP, Exterior Gateway Protocol)로 나누어짐. IGP의 예로는 RIP, IGRP, EIGRP, OSPF 등이 있고, EGP에는 BGP 등이 있음.

```bash
- OSPF 라우팅 설정
R1)
conf t
router ospf 1
network 172.16.100.0 0.0.0.255 area 0
network 10.1.12.0 0.0.0.255 area 0

R2)
conf t
router ospf 1
network 10.1.12.0 0.0.0.255 area 0
network 192.168.1.0 0.0.0.255 area 0
network 192.168.2.0 0.0.0.255 area 0
```

```bash
R1)
conf t
int lo0
ip add 1.1.1.1 255.255.255.0

int s1/0
ip add 2.2.12.1 255.255.255.0
no sh

R2)
conf t
int lo0
ip add 2.2.2.2 255.255.255.0

int s1/0
ip add 2.2.12.2 255.255.255.0
no sh

int s1/1	
ip add 2.2.23.2 255.255.255.0
no sh

R3)
conf t
int lo0
ip add 3.3.3.3 255.255.255.0

int s1/1
ip add 2.2.23.3 255.255.255.0
no sh

int s1/2	
ip add 2.2.34.3 255.255.255.0
no sh

R4)
conf t
int lo0
ip add 4.4.4.4 255.255.255.0

int s1/2
ip add 2.2.34.4 255.255.255.0
no sh

int s1/3	
ip add 2.2.45.4 255.255.255.0
no sh

R5)
conf t
int lo0
ip add 5.5.5.5 255.255.255.0

int s1/3
ip add 2.2.45.5 255.255.255.0
no sh
```

```bash
- AS200에 OSPF 설정

R2)
router ospf 1
network 2.2.23.0 0.0.0.255 area 0

R3)
router ospf 1
network 2.2.23.0 0.0.0.255 area 0
network 2.2.34.0 0.0.0.255 area 0

R4)
router ospf 1
network 2.2.34.0 0.0.0.255 area 0
```

virtual pc는 save. router는 wr로 저장.

## BGF

![Untitled](Network%2002/Untitled%2012.png)

```bash
Rsh ip int b → show ip interface brief
```

```bash
R1)
conf t
router bgp 100
neighbor 2.2.12.2 remote-as 200 => eBGP
network 1.1.1.0 mask 255.255.255.0

R2)
conf t
router bgp 200
neighbor 2.2.12.1 remote-as 100
neighbor 2.2.23.3 remote-as 200 => iBGP
neighbor 2.2.23.3 next-hop-self
network 2.2.2.0 mask 255.255.255.0

R3) 
conf t
router bgp 200
neighbor 2.2.23.2 remote-as 200
neighbor 2.2.34.4 remote-as 200
neighbor 2.2.23.2 route-reflector-client
neighbor 2.2.34.4 route-reflector-client
network 3.3.3.0 mask 255.255.255.0

R4)
conf t
router bgp 200
neighbor 2.2.34.3 remote-as 200
neighbor 2.2.34.3 next-hop-self
neighbor 2.2.45.5 remote-as 300
network 4.4.4.0 mask 255.255.255.0

R5)
conf t
router bgp 300
neighbor 2.2.45.4 remote-as 200
network 5.5.5.0 mask 255.255.255.0
```

no router bgp 200을 입력하면 bgp 200에 설정했던 모든 것을 한 번에 지움.

bgp는 neighbor를 해야지만 메세지 전송 가능.

bgp와 remote-as가 같으면 iBGP, 다르면 eBGP

![Untitled](Network%2002/Untitled%2013.png)

…..표시는 request time out

R5가 2.2.12.1에 대한 정보를 가지고 있지 않으므로 time out이 발생.

![Untitled](Network%2002/Untitled%2014.png)

R5가 1.1.1.1에 대한 정보를 가지고 있으므로 time out이 발생하지 않음.

routing이 왜 필요한지.

static routing이 무엇인지.
