# Network 04

Date: June 27, 2024
Files & media: AWS_ClientVPN%25EA%25B5%25AC%25EC%2584%25B1%25ED%2595%2598%25EA%25B8%25B0.pdf, VPN(Virtual_Private_Network).pdf

## ACL inbound outbound 실습

ACL은 인바운드 아웃바운드 규칙 모두 허용시켜야 함.

보안그룹은 들어왔다 나가는 것을 자동 허용.

보안그룹은 들어오는 것에 대해서만.

![Untitled](Network%2004%2005bfbb53870e4c5d916b95b1ddd48ec2/Untitled.png)

위 두 개는 식별용도.

차단할 때는 ip로.

- NAT_RTR에서 ACL 적용
192.168.1.0/24 네트워크에서는 서버 100.100.100.200에 웹접속만 가능
서버로 가는 나머지 트래픽 모두 차단. 그 외 트래픽 모두 허용

```bash
ip access-list extended Permit-Outbound
permit tcp 192.168.1.0 0.0.0.255 host 100.100.100.200 eq 80
deny ip 192.168.1.0 0.0.0.255 host 100.100.100.200
permit ip any any
```

172.16.2.0/24 네트워크에서는 서버 100.100.100.200에 FTP접속만 가능
서버로 가는 나머지 트래픽 모두 차단. 그 외 트래픽 모두 허용

```bash
permit tcp 172.16.2.0 0.0.0.255 host 100.100.100.200 eq 21
deny ip 172.16.2.0 0.0.0.255 host 100.100.100.200
permit ip any any
```

두 코드를 아래와 같이 하나로 합침.

```bash
ip access-list extended Permit-Outbound
permit tcp 192.168.1.0 0.0.0.255 host 100.100.100.200 eq 80
deny ip 192.168.1.0 0.0.0.255 host 100.100.100.200
permit tcp 172.16.2.0 0.0.0.255 host 100.100.100.200 eq 21
deny ip 172.16.2.0 0.0.0.255 host 100.100.100.200
permit ip any any
```

출발지가 다른 것을 하나의 ACL에 넣었음. → out으로 처리.

각각 처리해도 됨.

![Untitled](Network%2004%2005bfbb53870e4c5d916b95b1ddd48ec2/Untitled%201.png)

![Untitled](Network%2004%2005bfbb53870e4c5d916b95b1ddd48ec2/Untitled%202.png)

![Untitled](Network%2004%2005bfbb53870e4c5d916b95b1ddd48ec2/Untitled%203.png)

위 이미지는 실습 결과.

`ACL 설정 시 순서 중요. IP가 작은 범위부터 넓은 범위로`

[VPN(Virtual Private Network).pdf](Network%2004%2005bfbb53870e4c5d916b95b1ddd48ec2/VPN(Virtual_Private_Network).pdf)

## VPN 터널링 실습

![Untitled](Network%2004%2005bfbb53870e4c5d916b95b1ddd48ec2/Untitled%204.png)

```bash
R1)
conf t
int f0/0
ip add 192.168.1.1 255.255.255.0
no sh

int s1/0
ip add 3.3.12.1 255.255.255.0
no sh

ip route 0.0.0.0 0.0.0.0 s1/0 3.3.12.2

R2)
conf t
int s1/0
ip add 3.3.12.2 255.255.255.0
no sh

int s1/1
ip add 3.3.23.2 255.255.255.0
no sh

R3)
conf t
int s1/1
ip add 3.3.23.3 255.255.255.0
no sh

int f0/0
ip add 192.168.2.1 255.255.255.0
no sh

ip route 0.0.0.0 0.0.0.0 s1/1 3.3.23.2

PC1)
ip 192.168.1.10 192.168.1.1

PC2)
ip 192.168.2.20 192.168.2.1
```

```bash
R1)
int tun 1
ip unnumbered f0/0
tunnel source s1/0
tunnel destination 3.3.23.3

ip route 192.168.2.0 255.255.255.0 tunnel 1

R3)
int tun 3
ip unnumbered f0/0
tunnel source s1/1
tunnel destination 3.3.12.1

ip route 192.168.1.0 255.255.255.0 tunnel 3
```

![Untitled](Network%2004%2005bfbb53870e4c5d916b95b1ddd48ec2/Untitled%205.png)

터널링으로 PC1에서 PC2로 ping을 보냈을 때 통신이 가능함.

터널링

- 통과할 수 없는 네트워크를 통과할 수 있도록 해줌.

## VPN IPSec 실습

### `Site-to-Site VPN 설정 전 터널 삭제`

```bash
R1)
no int tun1

R3)
no int tun3
```

---

### `Site-to-Site VPN 설정`

- `1단계 정책 설정: 암호화에 사용되는 키 교환을 위한 암호채널 구성`

```bash
crypto isakmp policy 10
	encryption aes
	authentication pre-share
	group 2
	exit
	
	crypto isakmp key cisco address 3.3.23.3
```

- `2단계 정책 설정: 보호 트래픽, 데이터를 보호할 정책`

```bash
ip access-list extended R1R3
	permit ip 192.168.1.0 0.0.0.255 192.168.2.0 0.0.0.255

crypto ipsec transform-set r1-policy esp-aes esp-md5-hmac
```

- `크립토 맵 작성 후 적용`

```bash
crypto map vpn 10 ipsec-isakmp
	match address R3R1
	set peer 3.3.12.1
	set transform-set r1-policy

int s1/1 
crypto map vpn
```

---

### `실습 코드`

```bash
R1)
crypto isakmp policy 10
	encryption aes
	authentication pre-share
	group 2
	exit

crypto isakmp key cisco address 3.3.23.3

ip access-list extended R1R3
	permit ip 192.168.1.0 0.0.0.255 192.168.2.0 0.0.0.255

crypto ipsec transform-set r1-policy esp-aes esp-md5-hmac

crypto map vpn 10 ipsec-isakmp
	match address R1R3
	set peer 3.3.23.3
	set transform-set r1-policy

int s1/0 
crypto map vpn

R3)
crypto isakmp policy 10
	encryption aes
	authentication pre-share
	group 2
	exit

crypto isakmp key cisco address 3.3.12.1

ip access-list extended R3R1
	permit ip 192.168.2.0 0.0.0.255 192.168.1.0 0.0.0.255

crypto ipsec transform-set r1-policy esp-aes esp-md5-hmac

crypto map vpn 10 ipsec-isakmp
	match address R3R1
	set peer 3.3.12.1
	set transform-set r1-policy

int s1/1 
crypto map vpn
```

![Untitled](Network%2004%2005bfbb53870e4c5d916b95b1ddd48ec2/Untitled%206.png)

![Untitled](Network%2004%2005bfbb53870e4c5d916b95b1ddd48ec2/Untitled%207.png)

![Untitled](Network%2004%2005bfbb53870e4c5d916b95b1ddd48ec2/Untitled%208.png)

`vpn 설정 후 ping을 던졌을 때 ESP로 암호화 되었음을 알 수 있음`

`해시 알고리즘 - SHA(Default), MD5`

`덮어쓰기를 하는 것이 아니므로 no로 삭제를 해줘야 함.`

### `Authentication`

![Untitled](Network%2004%2005bfbb53870e4c5d916b95b1ddd48ec2/4adc068d-5f32-4e9c-a454-aee8de47a930.png)

### `Encryption Algorithms`

![Untitled](Network%2004%2005bfbb53870e4c5d916b95b1ddd48ec2/Untitled%209.png)

---

<aside>
💡 PKI(Public Key Infrastructure)란 디지털 인증서 사용을 위한 기반구조를 가리킨다. 즉, 디지털 인증서의 생성, 관리, 배포, 사용, 저장 및 해제를 위하여 필요한 하드웨어,소프트웨어, 정책 및 절차를 총칭하는 용어이다.

</aside>

<aside>
💡 디지털 인증서란 인증기관(CA, certificate authority)이 소유자의 신분과 그의 공개키 정`보를 보증하기 위해 발급하는 전자문서이다. 즉, CA는 사용자의 공개키 인증서를 전자서명함으로써, 사용자 인증서가 진짜라는 것과 내용이 변조되지 않았다는 것을 보증한다.

</aside>

`CA에서 인증서 발급. 보증을 해줌.`

---

### `1. NTP 설정`

`시간 동기화 설정.`

![Untitled](Network%2004%2005bfbb53870e4c5d916b95b1ddd48ec2/Untitled%2010.png)

```bash
CA)
conf t
clock timezone KOR +9 -> timezone 설정
ntp master

R1)
conf t
ntp server 3.3.24.4 -> master ip

R3)
conf t
ntp server 3.3.24.4
```

### `2. 디지털 인증서 서버 설정`

<aside>
✅ 참고) X.500 포맷

C=country
CN=common name:CA 이름
E=email
L=locality CA의 위치
O=organization: 회사명
OU=organizational unit: CA내의 부서명 
ST= state:CA의 시도 또는 주 정보

</aside>

```bash
CA)
conf t
ip http server

crypto pki server CertSVR
issuer-name CN=CertServer, L=Incheon, C=KR
grant auto
no sh
패스워드 설정(7자리 이상)

-인증서 서버 확인
show crypto pki server
```

![Untitled](Network%2004%2005bfbb53870e4c5d916b95b1ddd48ec2/Untitled%2011.png)

### `3. 공개키/개인키 생성`

- RSA 키 생성

```bash
ip domain-name cloudwave.com
crypto key generate rsa general-key modulus 2048
```

![Untitled](Network%2004%2005bfbb53870e4c5d916b95b1ddd48ec2/Untitled%2012.png)

- 공개키 확인

```bash
show crypto mypubkey rsa
```

![Untitled](Network%2004%2005bfbb53870e4c5d916b95b1ddd48ec2/Untitled%2013.png)

### `4.CA인증서 다운로드`

CA와 사용자 사이에 인증서를 등록하고 CRL(certificate revocation list)를 요청할 때 SCEP(Simple certificate enrollment protocol)을 사용.

SCEP는 읹으서 등록시에는 http를 사용하고, CRL확인시에는 http 또는 ldap을 이용함.

```bash
R1)
-CA 지정
crypto pki trustpoint CA
enrollment url http://3.3.24.4
subject-name cn=R1, l=Seoul, c=KR
exit

-CA인증서 다운 및 인증
crypto pki authenticate CA

- 라우터 R1의 인증서 요청
crypto pki enroll CA

-인증서 정보 확인
show crypto pki certificates
```

![Untitled](Network%2004%2005bfbb53870e4c5d916b95b1ddd48ec2/Untitled%2014.png)

## AWS ClientVPN 설정

### `Window ubuntu 설치`

```powershell
관리자 모드
wsl —install
재시작 후
wsl --install을 다시 입력하여 ubuntu 시작
```

설치 전 windows 기능 켜기/끄기에서 Linux용 Windows 하위 시스템에 체크가 되어있는지 확인 → 체크가 되어있으면 해제

[AWS_ClientVPN구성하기.pdf](Network%2004%2005bfbb53870e4c5d916b95b1ddd48ec2/AWS_ClientVPN%25EA%25B5%25AC%25EC%2584%25B1%25ED%2595%2598%25EA%25B8%25B0.pdf)

```bash
sudo apt update
sudo apt-get install easy-rsa
make-cadir $HOME/keys && $HOME/keys
cd $HOME/keys
vi vars or nano vars 로 편집.
```

```bash
Easy-rsa 이용하여 
hyemi@KHP18:~$ sudo apt install easy-rsa
[sudo] password for hyemi:
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  libccid libpcsclite1 opensc opensc-pkcs11 pcscd
Suggested packages:
  pcmciautils
The following NEW packages will be installed:
  easy-rsa libccid libpcsclite1 opensc opensc-pkcs11 pcscd
0 upgraded, 6 newly installed, 0 to remove and 108 not upgraded.
Need to get 1484 kB of archives.
After this operation, 5033 kB of additional disk space will be used.
Do you want to continue? [Y/n] y
Get:1 http://archive.ubuntu.com/ubuntu jammy/universe amd64 libccid amd64 1.5.0-2 [83.1 kB]
Get:2 http://archive.ubuntu.com/ubuntu jammy-updates/main amd64 libpcsclite1 amd64 1.9.5-3ubuntu1 [19.8 kB]
Get:3 http://archive.ubuntu.com/ubuntu jammy-updates/universe amd64 pcscd amd64 1.9.5-3ubuntu1 [58.1 kB]
Get:4 http://archive.ubuntu.com/ubuntu jammy/universe amd64 opensc-pkcs11 amd64 0.22.0-1ubuntu2 [933 kB]
Get:5 http://archive.ubuntu.com/ubuntu jammy/universe amd64 opensc amd64 0.22.0-1ubuntu2 [346 kB]
Get:6 http://archive.ubuntu.com/ubuntu jammy/universe amd64 easy-rsa all 3.0.8-1ubuntu1 [44.1 kB]
Fetched 1484 kB in 5s (272 kB/s)
Selecting previously unselected package libccid.
(Reading database ... 24208 files and directories currently installed.)
Preparing to unpack .../0-libccid_1.5.0-2_amd64.deb ...
Unpacking libccid (1.5.0-2) ...
Selecting previously unselected package libpcsclite1:amd64.
Preparing to unpack .../1-libpcsclite1_1.9.5-3ubuntu1_amd64.deb ...
Unpacking libpcsclite1:amd64 (1.9.5-3ubuntu1) ...
Selecting previously unselected package pcscd.
Preparing to unpack .../2-pcscd_1.9.5-3ubuntu1_amd64.deb ...
Unpacking pcscd (1.9.5-3ubuntu1) ...
Selecting previously unselected package opensc-pkcs11:amd64.
Preparing to unpack .../3-opensc-pkcs11_0.22.0-1ubuntu2_amd64.deb ...
Unpacking opensc-pkcs11:amd64 (0.22.0-1ubuntu2) ...
Selecting previously unselected package opensc.
Preparing to unpack .../4-opensc_0.22.0-1ubuntu2_amd64.deb ...
Unpacking opensc (0.22.0-1ubuntu2) ...
Selecting previously unselected package easy-rsa.
Preparing to unpack .../5-easy-rsa_3.0.8-1ubuntu1_all.deb ...
Unpacking easy-rsa (3.0.8-1ubuntu1) ...
Setting up libccid (1.5.0-2) ...
Setting up opensc-pkcs11:amd64 (0.22.0-1ubuntu2) ...
Setting up libpcsclite1:amd64 (1.9.5-3ubuntu1) ...
Setting up easy-rsa (3.0.8-1ubuntu1) ...
Setting up opensc (0.22.0-1ubuntu2) ...
Setting up pcscd (1.9.5-3ubuntu1) ...
Created symlink /etc/systemd/system/sockets.target.wants/pcscd.socket → /lib/systemd/system/pcscd.socket.
pcscd.service is a disabled or a static unit, not starting it.
Processing triggers for libc-bin (2.35-0ubuntu3.4) ...
Processing triggers for man-db (2.10.2-1) ...
hyemi@KHP18:~$ make-cadir $HOME/keys && $HOME/keys
-bash: /home/hyemi/keys: Is a directory
hyemi@KHP18:~$ ls
keys
hyemi@KHP18:~$ cd $HOME/keys
hyemi@KHP18:~/keys$ vi vars
hyemi@KHP18:~/keys$ ls
easyrsa  openssl-easyrsa.cnf  vars  x509-types
hyemi@KHP18:~/keys$ nano vars
hyemi@KHP18:~/keys$ ./easyrsa init-pki

init-pki complete; you may now create a CA or requests.
Your newly created PKI dir is: /home/hyemi/keys/pki

hyemi@KHP18:~/keys$ ./easyrsa build-ca
Using SSL: openssl OpenSSL 3.0.2 15 Mar 2022 (Library: OpenSSL 3.0.2 15 Mar 2022)

Enter New CA Key Passphrase:
Re-Enter New CA Key Passphrase:
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Common Name (eg: your user, host, or server name) [Easy-RSA CA]:

CA creation complete and you may now import and sign cert requests.
Your new CA certificate file for publishing is at:
/home/hyemi/keys/pki/ca.crt

hyemi@KHP18:~/keys$ ls $HOME/keys/pki
ca.crt           index.txt.attr       private  revoked
certs_by_serial  issued               renewed  safessl-easyrsa.cnf
index.txt        openssl-easyrsa.cnf  reqs     serial
hyemi@KHP18:~/keys$ ls $HOME/keys/pki/private
ca.key
hyemi@KHP18:~/keys$ ./easyrsa build-server-full server nopass
Using SSL: openssl OpenSSL 3.0.2 15 Mar 2022 (Library: OpenSSL 3.0.2 15 Mar 2022)
.+.+.....+...+.......+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*..+.+.........+........+......+.+.....+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*....+......+.........+...+..........+........+.+...+..............................+........+....+........+.......+.....+.+..+................+..+....+.....+.+............+........+...+.......+...+...........+...+.+......+......+...+.....+......+....+.....+......+....+......+......+........+.+......+..............+......+.......+.....+.+.....+.+.........+......+........+.........................+..+.+........+....+...+..............+...............+..........+......+.....+.........+.+..+.+...........+.+..+..................+...+....+..+.+...............+.....+....+........+...+..........+...+......+.....+..........+........+.+.....+....+..+.+........+.+.....+....+.....+......+......+....+........+.+.....+................+........+.+.................+.......+........+....+......+........................+.....+....+............+..+......+.......+...+.....+............+.........+..........+...+..+......+...+....+...+..+.+...........+......+...+.+...+......+.....+....+............+.........+.....+....+...+..+............+...............+....+..+....+...............+......+...+...............+...............+..+....+...+..+...+......+........................+.+.........+..+....+.........+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
...+....+..+....+..+.........+...+.......+...+........+.......+........+....+...+...+..................+..+.......+..+.+..+............+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*.......+...+..................+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*.+........+..........+........+.......+..+.............+........+...+....+.....+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
-----
Using configuration from /home/hyemi/keys/pki/easy-rsa-1319.fYBk9X/tmp.PUIp83
Enter pass phrase for /home/hyemi/keys/pki/private/ca.key:
40E79D10837F0000:error:0700006C:configuration file routines:NCONF_get_string:no value:../crypto/conf/conf_lib.c:315:group=<NULL> name=unique_subject
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
commonName            :ASN.1 12:'server'
Certificate is to be certified until Sep 30 07:44:18 2026 GMT (825 days)

Write out database with 1 new entries
Data Base Updated

hyemi@KHP18:~/keys$ ./easyrsa build-client-full client1.yhm.tld nopass
Using SSL: openssl OpenSSL 3.0.2 15 Mar 2022 (Library: OpenSSL 3.0.2 15 Mar 2022)
.+......+.........+.+.....+......+..........+......+.................+.+..............+.......+......+..+.+..+....+........+............+.+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*....+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*......+...+...+.......+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
...+.+..+...+...+.+........+......+....+...+............+..+.+.....+...............+....+......+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*.+.+..+.+..+....+.....+..........+......+...+..+...+......+...+.+..+...............+...+.+......+.....+.......+.....+.......+......+..+......+.+...+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*......+.+..+...+..........+......+.....+...+.+.....+.+........+......+......+....+..+...................+...+......+.....+....+............+...+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
-----
Using configuration from /home/hyemi/keys/pki/easy-rsa-1394.hUV8fX/tmp.OUllwU
Enter pass phrase for /home/hyemi/keys/pki/private/ca.key:
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
commonName            :ASN.1 12:'client1.yhm.tld'
Certificate is to be certified until Sep 30 07:45:11 2026 GMT (825 days)

Write out database with 1 new entries
Data Base Updated
```

### `AWS 외부에서 인증서 가져오기`

```bash
hyemi@KHP18:~/keys/pki/private$ cat yhm.server.key
-----BEGIN PRIVATE KEY-----
MIIEvQIBADANBgkqhkiG9w0BAQEFAASCBKcwggSjAgEAAoIBAQDG8iN1lO9/W49g
5wKKkt/2bwfn3fV1PZCtgnDR2eA8GWgu8LSm9bqYE/k7/dFTUb/j5IDr1zOnPq4D
v4Bqn+jfnHYPyj1CGZycFtYsvGM25SvNHqBylrGgHcSLi6SncurpKI7eVuhcCwiq
OERFvUXW+PnmSSsTDKaSkxRoYvZ/36kNIwGDqlPKsNYNnQAMJ4o3lnmrawfq5SdO
QZy5RdZ5EGXyS0c57KuUeIZzqqyDgfVrP/x8qSZfjZx00r6Jhwiq8InYGm/wIbcp
SfDXzgsD1rleclYMiLLnqC2NcTQwiiu06XfgjBNFfvYCcy1eIXrZlC+9+Qyh68zu
XSWtDP9PAgMBAAECggEATEkJA8tKUsGfJv8t4EkVi/9jPqvDtWMYGRBNhopPC3yK
kIVIIEVVeX8fMLvRCmvscsxqCwUID3dfbpx992S9/RCXzNI1zyTXAptXIOxT9vbF
Zu/5gjb6gXUoqoGvb24HWcRtlCArFTA98FeHBl7fauEpof4ogDN3o/i1+JkKAFnr
9r7CQHLL/5KRAot/16JuZofXkFQOEAE6DJGkNR9kgLXzBmxuwNVxXttwswp9Ch3o
Ec8dwVAllrpN+9709OnFPwRGZ+Zl/MkHZXEwWktIt8iw87UWnL1OddbkPYtQbawT
a8a2zdv4RxbvlQcQkt7BBzztOaEtnqdvVena0hczsQKBgQDlNW1BvX7By3LyJ+bo
PZFx83eUHr33eI2pxUTeBS74gdwZuVYHZukohL2VGzXlGbmSfobrt6Nudt+pkUYQ
DJMDE1gpFwHv4P6qujSR/7JN4b5hrxikeWlJVvWt3q/GLrVep8ZJKhiucvnPribQ
hNSl6OuQpNLx6OIcwOaOp03qSwKBgQDeMylnCGVD29ilRsnaFRketwHsfs4gmkg6
yrJfKWg+riNpDEiBjvZeXV4VE6QQ+k6Bxh9Ty78smkLGJUl6wnuB977J9tBbWnhF
efpGa8NI42j+8689tY5BYDvmi/tK1NxO0K+tERZIc2wKw3kRQwHwFBG5/H+yXlzQ
f1rVPtBcjQKBgQCu4VOqK/3RuPvLvRwVqBwX2E4tSkNg1K8pkCTaTRQocVRQoDL+
VMuiqZzIbklxsm3/UuB4atWcS8Cc7QWK6z0jxJeoSjClKILGGmpP1srhV1Ldzy27
GBN37Ixoi5aLXEnvnYzRd/f66iimB1cAE8j3iT5qTwfPoQMcMyX2Q7pT/wKBgDSP
CYYTmFB62j4OBoUNZIm9ZDkarYtMszUk6RhVZREeg8W/YA81T9V2ZGC76p0ReCx+
Pr7FfQ0B2DWicEUXZ7uQbJK9TP+u4LAecDLkHqdJE3brEVKZdXLFXqXkCqbivtHt
zwAzAIBWvQG2xxZsMTMmrCLANTxt0aqH1WaHmyWpAoGAJXAbbe7aJKECTDQbbsce
Vbir77wYm6GU7p/4CxO1jd5EyXxU4EaYmM+AoulhXhEmX2Zne7G4yqAFSgMTeATy
/IEeZWJkZjTEs2lQH6iY2+yg/AAl/kmp7wT96ho4yCcE/Jz/+WcWD0+EN5qANTRy
a6+KA7Bh2wHprKunnTDMqYM=
-----END PRIVATE KEY-----
hyemi@KHP18:~/keys/pki/private$ cd ..
hyemi@KHP18:~/keys/pki$ ls
ca.crt           index.txt.attr.old   private  safessl-easyrsa.cnf
certs_by_serial  index.txt.old        renewed  serial
index.txt        issued               reqs     serial.old
index.txt.attr   openssl-easyrsa.cnf  revoked
hyemi@KHP18:~/keys/pki$ cd issued
hyemi@KHP18:~/keys/pki/issued$ ls
client1.yhm.tld.crt  yhm.server.crt
hyemi@KHP18:~/keys/pki/issued$ cat yhm.server.crt
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            b0:ba:88:72:b9:b0:85:e0:78:4b:77:e2:d1:67:ec:14
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: CN=Easy-RSA CA
        Validity
            Not Before: Jun 27 08:14:43 2024 GMT
            Not After : Sep 30 08:14:43 2026 GMT
        Subject: CN=yhm.server
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
                Modulus:
                    00:c6:f2:23:75:94:ef:7f:5b:8f:60:e7:02:8a:92:
                    df:f6:6f:07:e7:dd:f5:75:3d:90:ad:82:70:d1:d9:
                    e0:3c:19:68:2e:f0:b4:a6:f5:ba:98:13:f9:3b:fd:
                    d1:53:51:bf:e3:e4:80:eb:d7:33:a7:3e:ae:03:bf:
                    80:6a:9f:e8:df:9c:76:0f:ca:3d:42:19:9c:9c:16:
                    d6:2c:bc:63:36:e5:2b:cd:1e:a0:72:96:b1:a0:1d:
                    c4:8b:8b:a4:a7:72:ea:e9:28:8e:de:56:e8:5c:0b:
                    08:aa:38:44:45:bd:45:d6:f8:f9:e6:49:2b:13:0c:
                    a6:92:93:14:68:62:f6:7f:df:a9:0d:23:01:83:aa:
                    53:ca:b0:d6:0d:9d:00:0c:27:8a:37:96:79:ab:6b:
                    07:ea:e5:27:4e:41:9c:b9:45:d6:79:10:65:f2:4b:
                    47:39:ec:ab:94:78:86:73:aa:ac:83:81:f5:6b:3f:
                    fc:7c:a9:26:5f:8d:9c:74:d2:be:89:87:08:aa:f0:
                    89:d8:1a:6f:f0:21:b7:29:49:f0:d7:ce:0b:03:d6:
                    b9:5e:72:56:0c:88:b2:e7:a8:2d:8d:71:34:30:8a:
                    2b:b4:e9:77:e0:8c:13:45:7e:f6:02:73:2d:5e:21:
                    7a:d9:94:2f:bd:f9:0c:a1:eb:cc:ee:5d:25:ad:0c:
                    ff:4f
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Basic Constraints:
                CA:FALSE
            X509v3 Subject Key Identifier:
                D3:58:A4:15:B5:E8:EF:AF:DD:F0:AE:F1:41:2A:0D:F7:8B:1D:11:5B
            X509v3 Authority Key Identifier:
                keyid:73:E8:7E:25:6A:4D:1D:16:E8:8D:0C:A9:27:75:14:AC:82:1E:74:86
                DirName:/CN=Easy-RSA CA
                serial:14:8D:50:71:D7:FD:64:A4:84:A6:6D:61:B6:7C:1F:00:01:48:8F:18
            X509v3 Extended Key Usage:
                TLS Web Server Authentication
            X509v3 Key Usage:
                Digital Signature, Key Encipherment
            X509v3 Subject Alternative Name:
                DNS:yhm.server
    Signature Algorithm: sha256WithRSAEncryption
    Signature Value:
        a4:78:1d:55:ab:ce:af:eb:0e:e8:bb:19:33:43:40:ab:5b:b3:
        ac:86:ba:60:68:04:16:ba:58:7f:c2:8f:2b:a9:0c:e2:3d:9f:
        be:09:ed:8a:de:9c:bf:ca:91:99:04:e9:8e:7c:c8:0c:d4:eb:
        fa:85:a7:5d:c4:48:9e:38:75:37:2f:5a:0f:8f:e9:e7:e6:fe:
        57:63:1f:8e:d2:c9:41:83:68:55:ba:07:5d:42:f8:b9:8a:a3:
        49:d0:ef:67:4d:e3:78:89:00:47:0f:37:6e:98:61:c0:fa:5a:
        61:c3:b1:83:55:e8:33:d1:f4:11:11:02:c3:20:97:a3:a9:18:
        58:73:d7:b1:82:67:a3:70:b5:fa:8f:f7:63:ed:e1:46:5c:25:
        e9:b4:e1:b4:c7:30:15:86:d2:f5:41:b0:d1:ea:33:8d:c3:65:
        18:eb:6a:14:2b:18:87:7c:01:d5:41:92:76:e5:4f:6b:99:6c:
        7e:3e:40:f9:b9:7b:ff:cf:5b:a6:f5:44:ce:72:8c:b7:e0:77:
        84:98:8a:0e:5c:af:a2:10:fc:21:52:9a:63:37:0d:4d:1f:c5:
        1b:ed:13:c3:cb:6f:a4:46:41:14:70:c2:dd:ad:db:57:8f:6b:
        f6:60:b6:36:f5:07:fa:94:33:3f:44:b8:13:4d:29:80:45:7a:
        fa:78:48:ad
-----BEGIN CERTIFICATE-----
MIIDcDCCAligAwIBAgIRALC6iHK5sIXgeEt34tFn7BQwDQYJKoZIhvcNAQELBQAw
FjEUMBIGA1UEAwwLRWFzeS1SU0EgQ0EwHhcNMjQwNjI3MDgxNDQzWhcNMjYwOTMw
MDgxNDQzWjAVMRMwEQYDVQQDDAp5aG0uc2VydmVyMIIBIjANBgkqhkiG9w0BAQEF
AAOCAQ8AMIIBCgKCAQEAxvIjdZTvf1uPYOcCipLf9m8H5931dT2QrYJw0dngPBlo
LvC0pvW6mBP5O/3RU1G/4+SA69czpz6uA7+Aap/o35x2D8o9QhmcnBbWLLxjNuUr
zR6gcpaxoB3Ei4ukp3Lq6SiO3lboXAsIqjhERb1F1vj55kkrEwymkpMUaGL2f9+p
DSMBg6pTyrDWDZ0ADCeKN5Z5q2sH6uUnTkGcuUXWeRBl8ktHOeyrlHiGc6qsg4H1
az/8fKkmX42cdNK+iYcIqvCJ2Bpv8CG3KUnw184LA9a5XnJWDIiy56gtjXE0MIor
tOl34IwTRX72AnMtXiF62ZQvvfkMoevM7l0lrQz/TwIDAQABo4G5MIG2MAkGA1Ud
EwQCMAAwHQYDVR0OBBYEFNNYpBW16O+v3fCu8UEqDfeLHRFbMFEGA1UdIwRKMEiA
FHPofiVqTR0W6I0MqSd1FKyCHnSGoRqkGDAWMRQwEgYDVQQDDAtFYXN5LVJTQSBD
QYIUFI1Qcdf9ZKSEpm1htnwfAAFIjxgwEwYDVR0lBAwwCgYIKwYBBQUHAwEwCwYD
VR0PBAQDAgWgMBUGA1UdEQQOMAyCCnlobS5zZXJ2ZXIwDQYJKoZIhvcNAQELBQAD
ggEBAKR4HVWrzq/rDui7GTNDQKtbs6yGumBoBBa6WH/CjyupDOI9n74J7YrenL/K
kZkE6Y58yAzU6/qFp13ESJ44dTcvWg+P6efm/ldjH47SyUGDaFW6B11C+LmKo0nQ
72dN43iJAEcPN26YYcD6WmHDsYNV6DPR9BERAsMgl6OpGFhz17GCZ6NwtfqP92Pt
4UZcJem04bTHMBWG0vVBsNHqM43DZRjrahQrGId8AdVBknblT2uZbH4+QPm5e//P
W6b1RM5yjLfgd4SYig5cr6IQ/CFSmmM3DU0fxRvtE8PLb6RGQRRwwt2t21ePa/Zg
tjb1B/qUMz9EuBNNKYBFevp4SK0=
-----END CERTIFICATE-----
hyemi@KHP18:~/keys/pki/issued$ cd ..
hyemi@KHP18:~/keys/pki$ ls
ca.crt           index.txt.attr.old   private  safessl-easyrsa.cnf
certs_by_serial  index.txt.old        renewed  serial
index.txt        issued               reqs     serial.old
index.txt.attr   openssl-easyrsa.cnf  revoked
hyemi@KHP18:~/keys/pki$ cat ca.crt
-----BEGIN CERTIFICATE-----
MIIDSzCCAjOgAwIBAgIUFI1Qcdf9ZKSEpm1htnwfAAFIjxgwDQYJKoZIhvcNAQEL
BQAwFjEUMBIGA1UEAwwLRWFzeS1SU0EgQ0EwHhcNMjQwNjI3MDc0MjIyWhcNMzQw
NjI1MDc0MjIyWjAWMRQwEgYDVQQDDAtFYXN5LVJTQSBDQTCCASIwDQYJKoZIhvcN
AQEBBQADggEPADCCAQoCggEBAMYRGRd535Hdq7RgYNMKNkhw2bDv4yQoGnbIWAUB
K1OAsAteXx1Qvv9u24f/dNxm/QrznqAwsehpIv+LP+g/thXZxzotnhg7C5wt52AV
H1D4bqtgX6CbZvEXEwXWYp4PvbsYYrMAtjd58imtj7i/0NseLYR1u5xTHsB5MTcK
rVC9vX30wj7RilTaJnzveNKI3qJPmZAhDYVYAczT9hXzziRlt3vHYdSfd957w+uP
8JeFPI/ssHTbO08baQYCW61DhOExev76ZgzU7zUm9ODY3/nex4GW5EO9kzYGgli/
AivNE7PCl5w2s+04qfJFoWv81A6y0VSUWDRr51IH4s7IRcECAwEAAaOBkDCBjTAd
BgNVHQ4EFgQUc+h+JWpNHRbojQypJ3UUrIIedIYwUQYDVR0jBEowSIAUc+h+JWpN
HRbojQypJ3UUrIIedIahGqQYMBYxFDASBgNVBAMMC0Vhc3ktUlNBIENBghQUjVBx
1/1kpISmbWG2fB8AAUiPGDAMBgNVHRMEBTADAQH/MAsGA1UdDwQEAwIBBjANBgkq
hkiG9w0BAQsFAAOCAQEAI34tir9TCOAOCBj3EXfHwseeffh4Y2Xjbu3KPwgRDCf5
q+Uii9tbDwKeRBgWjd7CnXimaG3khePkC8fVtVYJVsVTqisiAz3gjdP5mDgHo4K4
cVCIhj95MOs5W0xaP45FdNu2h4zXDuSX/vaZosK/qPFpnCDf24tsxOKnv3Vb9ZYC
DkjjvXQrbvK9nPhd+Gouu0RaBF5QHPmpBudK+z96F2lcW+4kwN5JLWkJQXH1zdKg
815I81GgJUUfE2a8PQWPNYO2kavkIdJecJYr96qQ2mvw9BNSXP+u+npItdyFntxr
217rJBngiMGk7bAmO0Uxh1MooUTmbbxDk/tJCq31kQ==
-----END CERTIFICATE-----
```

![스크린샷 2024-06-24 152258.png](Network%2004%2005bfbb53870e4c5d916b95b1ddd48ec2/%25EC%258A%25A4%25ED%2581%25AC%25EB%25A6%25B0%25EC%2583%25B7_2024-06-24_152258.png)

### `SSH를 이용해서 퍼블릭에서 프라이빗으로 접속`

```bash
Authenticating with public key "Imported-Openssh-Key"
    ┌──────────────────────────────────────────────────────────────────────┐
    │                 • MobaXterm Personal Edition v24.1 •                 │
    │               (SSH client, X server and network tools)               │
    │                                                                      │
    │ ⮞ SSH session to ubuntu@13.124.112.96                                │
    │   • Direct SSH      :  ✓                                             │
    │   • SSH compression :  ✓                                             │
    │   • SSH-browser     :  ✓                                             │
    │   • X11-forwarding  :  ✓  (remote display is forwarded through SSH)  │
    │                                                                      │
    │ ⮞ For more info, ctrl+click on help or visit our website.            │
    └──────────────────────────────────────────────────────────────────────┘

Welcome to Ubuntu 22.04.4 LTS (GNU/Linux 6.5.0-1017-aws x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

  System information as of Thu Jun 27 08:43:50 UTC 2024

  System load:  0.08056640625     Processes:             103
  Usage of /:   20.5% of 7.57GB   Users logged in:       0
  Memory usage: 24%               IPv4 address for eth0: 172.16.16.135
  Swap usage:   0%

Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status

The list of available updates is more than a week old.
To check for new updates run: sudo apt update

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

/usr/bin/xauth:  file /home/ubuntu/.Xauthority does not exist
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

ubuntu@ip-172-16-16-135:~$ ls
hm-keypair.pem
ubuntu@ip-172-16-16-135:~$ ls -l
total 4
-rw-rw-r-- 1 ubuntu ubuntu 1674 Jun 27 08:53 hm-keypair.pem
ubuntu@ip-172-16-16-135:~$ chmod 400 hm-keypair.pem -> 권한 축
ubuntu@ip-172-16-16-135:~$ ssh -i hm-keypair.pem ubuntu@172.16.20.100 -> Private Server로 접속
The authenticity of host '172.16.20.100 (172.16.20.100)' can't be established.

ED25519 key fingerprint is SHA256:PdXu81z15kSfC9pMn+TmXk57NA2jC+LVYD50PtW9VCw.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '172.16.20.100' (ED25519) to the list of known hosts.
Welcome to Ubuntu 22.04.4 LTS (GNU/Linux 6.5.0-1017-aws x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

  System information as of Thu Jun 27 08:56:25 UTC 2024

  System load:  0.0               Processes:             97
  Usage of /:   20.4% of 7.57GB   Users logged in:       0
  Memory usage: 20%               IPv4 address for eth0: 172.16.20.100
  Swap usage:   0%

Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status

The list of available updates is more than a week old.
To check for new updates run: sudo apt update

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

ubuntu@ip-172-16-20-100:~$

```