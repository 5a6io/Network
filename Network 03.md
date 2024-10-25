# Network 03

Date: June 26, 2024
Files & media: 0626-lab.pkt

## VPC 실습

172.16.0.0/16 → 172.16.0.0/22

        000000
        000001
        000010
           |
255.255.111111 00.0

필요한 서브넷 4개 /22
172.20.0.0/22
172.20.4.0/22
172.20.8.0/22
172.20.12.0/22

172.20.16.0/22 → 더 작은 서브넷이 필요할 때 이것을 나눠서 쓰면 됨.

172.20.16.0/24
172.20.17.0/24
172.20.18.0/24
172.20.19.0/24

![Untitled](Network%2003%20cd3de9872b53409eb102330fe34ef900/87de600e-0bad-43ae-be9b-69a1ac4e35f0.png)

R1에서 routing.

```bash
R1)
conf t
int f0/0
ip add 172.20.0.1 255.255.252.0
no sh

int f0/1
ip add 172.20.4.1 255.255.252.0
no sh

int f2/0
ip add 172.20.8.1 255.255.252.0
no sh

int f3/0
ip add 172.20.12.1 255.255.252.0
no sh
```

NAT를 쓰지 않으면 EC2에서 Linux를 게이트웨이로 쓸 수 있음.

![Untitled](Network%2003%20cd3de9872b53409eb102330fe34ef900/Untitled.png)

```bash
VPC2)  172.31.0.0/16

/24 서브넷 2개 생성

172.31.0.0/24
172.31.1.0/24

R2) 
conf t
int f0/0
ip add 172.31.0.1 255.255.255.0
no sh

int f0/1
ip add 172.31.1.1 255.255.255.0
no sh

PC5) 
ip 172.31.0.50 255.255.255.0 172.31.0.1

PC6) 
ip 172.31.1.60 172.31.1.1 
```

## Static NAT 실습

![Untitled](Network%2003%20cd3de9872b53409eb102330fe34ef900/Untitled%201.png)

Static NAT는 local IP주소와 global IP주소가 1대 1 맵핑되어있기 때문에 내부에서 외부로 외부에서 내부로 모든 트래픽이 허용됨.

local IP주소만큼 global IP주소가 필요하므로 내부 시스템의 인터넷을 위한 용도로는 사용하지 않음.

또한 외부에서 내부로도 모든 트래픽이 허용됨으로 보안에 취약함.

- Portforwarding: Static NAT의 특별한 형태. TCP, UDP로 서비스를 구분해서 외부에서 내부의 특정 서비스에 접속할 수 있도록 하는 NAT
ex) ip nat inside source static tcp 192.168.1.100 21 3.3.12.1 2121

```bash
ip nat inside source static tcp 192.168.1.100 21 3.3.12.1 21
ip nat inside source static tcp 192.168.1.100 80 3.3.12.1 8080
no ip nat inside source static tcp 192.168.1.100 80 3.3.12.1 8080
ip nat inside source static tcp 192.168.1.100 80 200.200.200.1 8080
```

## Dynamic NAT 실습

- Dynamic NAT: 내부 클라이언트가 인터넷이 가능하도록 공인주소 1개 또는 몇개 중에 바꿔주는 기능
클라이언트 주소를 식별하기 위해서 ACL을 사용함.
- 설정순서
1. ACL 작성: 변환할 네트워크 대역을 ACL로 작성
   access-list 1 permit 192.168.1.0 0.0.0.255
2. NAT문 작성
  1) 인터페이스 IP주소로 변환하는 경우
     ip nat inside source list 1 interface g0/0 [overload]

     overload 옵션은 기본옵션으로 NAT → PAT로 변환
     주소 변환 시 포트번호까지 사용
  2) 공인주소풀에서 변환하는 경우
     - NAT Pool 생성
       ip nat pool TestPool 200.200.200.1 200.200.200.2 netmask 255.255.255.252
     - NAT 문 작성
       ip nat inside source list 1 pool TestPool overload → overload를 무조건 붙여서 PAT로 만들어 사용.
3. NAT 적용
   int g0/1
   ip nat inside
   int g0/0
   ip nat outside

```bash
NAT_RTR#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
NAT_RTR(config)#ip nat pool TestPool 200.200.200.1 200.200.200.2 netmask
% Incomplete command.
NAT_RTR(config)#ip nat pool TestPool 200.200.200.1 200.200.200.2 netmask 255.255.255.0
NAT_RTR(config)#ip nat inside source list 1 pool TestPool overload
NAT_RTR(config)#end
NAT_RTR#
%SYS-5-CONFIG_I: Configured from console by console

NAT_RTR#sh run |i ip nat
                ^
% Invalid input detected at '^' marker.
	
NAT_RTR#sh run | i ip nat
 ip nat outside
 ip nat inside
ip nat pool TestPool 200.200.200.1 200.200.200.2 netmask 255.255.255.0
ip nat inside source list 1 pool TestPool overload
ip nat inside source static tcp 192.168.1.100 80 3.3.12.1 8080 
ip nat inside source static tcp 192.168.1.100 80 200.200.200.1 8080 
NAT_RTR#sh ip nat translation
Pro  Inside global     Inside local       Outside local      Outside global
tcp 200.200.200.1:8080 192.168.1.100:80   ---                ---
tcp 200.200.200.1:8080 192.168.1.100:80   192.168.1.10:1027  192.168.1.10:1027
tcp 3.3.12.1:8080      192.168.1.100:80   ---                ---

NAT_RTR#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
NAT_RTR(config)#no ip nat inside source static tcp 192.168.1.100 80 200.200.200.1 8080
NAT_RTR(config)#end
NAT_RTR#
%SYS-5-CONFIG_I: Configured from console by console

NAT_RTR#clear ip nat transl *
NAT_RTR#
```

```bash
sh ip nat traslation
sh run | i ip nat
```

dynamic nat 내부에서 외부로 나가는 용도.

‘>’모양인 경우 enable사용해서 관리자 모드로 전환

다음 IP주소의 네트워크 주소와 브로드캐스트 주소는?

219.94.56.100/21

0011 1 000
——————
네트워크 주소
00111111 → 브로드캐스트 주소

네트워크 주소: 네트워크 비트까지는 일치해야 하고 나머지 비트는 모두 0인 주소

브로드캐스트 주소: 네트워크 비트까지는 일치해야 하고 나머지 비트는 모두 1인 주소

네트워크 주소 - 219.94.56.0/21

브로드캐스트 주소 - 219.94.63.255

Standard ACL

출발주소가 사설IP주소대역일 경우 차단하는 ACL

access-list 10 deny 10.0.0.0 0.255.255.255

access-list 10 deny 172.16.0.0 0.15.255.255 →172.16.0.0 ~ 172.32.0.0을 나타냄.

access-list 10 deny 192.168.0.0 0.0.255.255

access-list 10 permit any

access-list 10 deny any → 전부 차단하는 ACL

int g0/0

ip access-group 10 out → 위의 ip를 보았을 때 출발 주소지를 검사하므로 out으로 작성

```bash
NAT_RTR>en
NAT_RTR#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
NAT_RTR(config)#access-list 10 deny 10.0.0.0 0.255.255.255
NAT_RTR(config)#
NAT_RTR(config)#access-list 10 deny 172.16.0.0 0.15.255.255 172.16.0.0 ~ 172.32.0.0 .
                                                            ^
% Invalid input detected at '^' marker.
	
NAT_RTR(config)#
NAT_RTR(config)#access-list 10 deny 192.168.0.0 0.0.255.255
NAT_RTR(config)#
NAT_RTR(config)#access-list 10 permit any
NAT_RTR(config)#
NAT_RTR(config)#access-list 10 deny any
NAT_RTR(config)#int g0/0
NAT_RTR(config-if)#ip access-group 10 out
```

## ACL 실습

![Untitled](Network%2003%20cd3de9872b53409eb102330fe34ef900/Untitled%202.png)

![Untitled](Network%2003%20cd3de9872b53409eb102330fe34ef900/Untitled%203.png)

ACL로 차단해서 통신이 안 됨.

## Extended ACL

INSICE_RTR에서 172.16.1.0/24 네트워크에 있는 단말에서 INSIDE_SVR의 서비스 중에 웹 접속 가능하도록 ACL 적용.

```bash
ip access-list extended ALLOW-HTTP → Named ACL
permit tcp 172.16.2.0 0.0.0.255 host 192.168.1.100 eq [http or 80] -> 허용
deny ip 172.16.2.0 0.0.0.255 host 192.168.1.100
deny ip any any -> 출발지 목적지 따지지 않고 모두 차단.
permit ip any any

int g0/0
ip access-group ALLOW-HTTP in
```