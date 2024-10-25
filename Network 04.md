# Network 04

## ACL inbound outbound 실습

ACL은 인바운드 아웃바운드 규칙 모두 허용시켜야 함.

보안그룹은 들어왔다 나가는 것을 자동 허용.

보안그룹은 들어오는 것에 대해서만.

![Untitled](Network%2004/Untitled.png)

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

![Untitled](Network%2004/Untitled%201.png)

![Untitled](Network%2004/Untitled%202.png)

![Untitled](Network%2004/Untitled%203.png)

위 이미지는 실습 결과.

`ACL 설정 시 순서 중요. IP가 작은 범위부터 넓은 범위로`

## VPN 터널링 실습

![Untitled](Network%2004/Untitled%204.png)

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

![Untitled](Network%2004/Untitled%205.png)

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

![Untitled](Network%2004/Untitled%206.png)

![Untitled](Network%2004/Untitled%207.png)

![Untitled](Network%2004/Untitled%208.png)

`vpn 설정 후 ping을 던졌을 때 ESP로 암호화 되었음을 알 수 있음`

`해시 알고리즘 - SHA(Default), MD5`

`덮어쓰기를 하는 것이 아니므로 no로 삭제를 해줘야 함.`

### `Authentication`

![Untitled](Network%2004/4adc068d-5f32-4e9c-a454-aee8de47a930.png)

### `Encryption Algorithms`

![Untitled](Network%2004/Untitled%209.png)

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

![Untitled](Network%2004/Untitled%2010.png)

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

![Untitled](Network%2004/Untitled%2011.png)

### `3. 공개키/개인키 생성`

- RSA 키 생성

```bash
ip domain-name cloudwave.com
crypto key generate rsa general-key modulus 2048
```

![Untitled](Network%2004/Untitled%2012.png)

- 공개키 확인

```bash
show crypto mypubkey rsa
```

![Untitled](Network%2004/Untitled%2013.png)

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

![Untitled](Network%2004/Untitled%2014.png)

## AWS ClientVPN 설정

### `Window ubuntu 설치`

```powershell
관리자 모드
wsl —install
재시작 후
wsl --install을 다시 입력하여 ubuntu 시작
```

설치 전 windows 기능 켜기/끄기에서 Linux용 Windows 하위 시스템에 체크가 되어있는지 확인 → 체크가 되어있으면 해제

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
...
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
                    ...
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
        ...
-----BEGIN CERTIFICATE-----
...
-----END CERTIFICATE-----
hyemi@KHP18:~/keys/pki/issued$ cd ..
hyemi@KHP18:~/keys/pki$ ls
ca.crt           index.txt.attr.old   private  safessl-easyrsa.cnf
certs_by_serial  index.txt.old        renewed  serial
index.txt        issued               reqs     serial.old
index.txt.attr   openssl-easyrsa.cnf  revoked
hyemi@KHP18:~/keys/pki$ cat ca.crt
-----BEGIN CERTIFICATE-----
...
-----END CERTIFICATE-----
```

![스크린샷 2024-06-24 152258.png](Network%2004/%25EC%258A%25A4%25ED%2581%25AC%25EB%25A6%25B0%25EC%2583%25B7_2024-06-24_152258.png)

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
ubuntu@ip-172-16-16-135:~$ chmod 400 hm-keypair.pem -> 권한 축소
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
