# Network 05

Date: June 28, 2024
Files & media: AWS_Network.pdf, 0628-lab.pkt, 0628(DHCP).txt

## Client VPN 구성 실습

### `서버 인증서`

```bash
cd ~/keys/pki/issued
cat server.crt
```

### `서버 개인키`

```bash
cd ~/keys/pki/private
cat server.key
```

### `CA 인증서`

```bash
cd ~/keys/pki
cat ca.crt
```

### `Windows에서 ubuntu로 파일 복사`

```bash
scp -i .\kyt-keypair2.pem kyt-keypair2.pem [ubuntu@54.180.251.22](mailto:ubuntu@54.180.251.22):/home/ubuntu
```

### `Client VPN Endpoint 생성`

![Untitled](Network%2005%20d28e8a243c424c67b896e2b8a940331f/Untitled.png)

![Untitled](Network%2005%20d28e8a243c424c67b896e2b8a940331f/Untitled%201.png)

![Untitled](Network%2005%20d28e8a243c424c67b896e2b8a940331f/Untitled%202.png)

![TCP - 443, UDP-1194](Network%2005%20d28e8a243c424c67b896e2b8a940331f/c7e6e521-b0ea-42cd-9115-52444f924909.png)

TCP - 443, UDP-1194

### `Download Client config`

```
client
dev tun
proto udp
remote yhm.cvpn-endpoint-0cd9d2eb50c728770.prod.clientvpn.ap-southeast-1.amazonaws.com 1194
remote-random-hostname
resolv-retry infinite
nobind
remote-cert-tls server
cipher AES-256-GCM
verb 3
<ca>
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

</ca>

<cert> # 클라이언트 인증서
-----BEGIN CERTIFICATE-----
MIIDXjCCAkagAwIBAgIRAPVZuICVkJSOIUf7LN8AN1AwDQYJKoZIhvcNAQELBQAw
FjEUMBIGA1UEAwwLRWFzeS1SU0EgQ0EwHhcNMjQwNjI3MDc0NTExWhcNMjYwOTMw
MDc0NTExWjAaMRgwFgYDVQQDDA9jbGllbnQxLnlobS50bGQwggEiMA0GCSqGSIb3
DQEBAQUAA4IBDwAwggEKAoIBAQCnD8tIv3zsH/LUcXAIeMR6y4FGqZ6jEO+HZeKm
OQhga/Wk63UIvf73It23UYkiTwl8slDneLCtfduRuB1AMMyHyEeIgj5czhtlEb2B
CJH1NcXuDEwTxEyp+E7zYpWt3ES/gziWlDKFFzwBQo5JGTU3VCTHotQvWwJMGmGX
/87FsSoihj40f4NpxkGBTdDdPaFU/PAPMlIbCRh0EqhXwTgtwJcQeadoK840rIC+
XfUyU2maalew71golSkbkkl044BpN/l6eS5VuSutHHIhO9eD+rm6rI8iBpqprceO
pEWqtt8Nf1bS2+TAgC09ukAlxqq+vr/SQMSJ8FEiRpxcUfMTAgMBAAGjgaIwgZ8w
CQYDVR0TBAIwADAdBgNVHQ4EFgQU2/L/Yg/Vu/vmeGmOHasAorO38X4wUQYDVR0j
BEowSIAUc+h+JWpNHRbojQypJ3UUrIIedIahGqQYMBYxFDASBgNVBAMMC0Vhc3kt
UlNBIENBghQUjVBx1/1kpISmbWG2fB8AAUiPGDATBgNVHSUEDDAKBggrBgEFBQcD
AjALBgNVHQ8EBAMCB4AwDQYJKoZIhvcNAQELBQADggEBAJHvWC5fNfnhZBnI32Jq
QkAgdknEu0yBT+iAI6rdzJvgARdyoIw8daVL3+R7A5Tp60/Phmp5vgPY4axbbo32
cnk/6MGfHr/jYXtL/gRpC0ZlvCpwQeCyjA18TZucCdDHg0kUxL12tSjOPXWOga6j
wSmWdZC02G3I9BTI6F9H1TxY3eANBWPtBCeQpEQs4Xbe4jaqV8gZtZ9hZX1+SD+a
lvQoCxP0xVPzUgkitS7lSWh0k3vUVbSL4IdWWfm+bFElnG/SHx6/g6+diXI/kasG
jjdFN4k2kzQv99PYWQxYKLgesQpMlCztmNX7EcYgmX4OnRgUTCllmwf7udU2wWAb
L44=
-----END CERTIFICATE-----
</cert>

<key> # 클라이언트 
-----BEGIN PRIVATE KEY-----
MIIEvQIBADANBgkqhkiG9w0BAQEFAASCBKcwggSjAgEAAoIBAQCnD8tIv3zsH/LU
cXAIeMR6y4FGqZ6jEO+HZeKmOQhga/Wk63UIvf73It23UYkiTwl8slDneLCtfduR
uB1AMMyHyEeIgj5czhtlEb2BCJH1NcXuDEwTxEyp+E7zYpWt3ES/gziWlDKFFzwB
Qo5JGTU3VCTHotQvWwJMGmGX/87FsSoihj40f4NpxkGBTdDdPaFU/PAPMlIbCRh0
EqhXwTgtwJcQeadoK840rIC+XfUyU2maalew71golSkbkkl044BpN/l6eS5VuSut
HHIhO9eD+rm6rI8iBpqprceOpEWqtt8Nf1bS2+TAgC09ukAlxqq+vr/SQMSJ8FEi
RpxcUfMTAgMBAAECggEAI7skDnYuQM6AgGGWxakW1PCrwLJMpLHdmUb4V/cAzKaq
jlBx4qHIY5FIfInq27St4ca9uRQ+LY0qphsL3ovRYwU6D39T2UOX5mEJ+AweM6UB
pW1p5BxrDTt3n2SpgNsItTFarBIcUAztH5PdnNLQZ/oON8iiu1Iqxp4p23mx/uQR
lf3NkHJ7vhvHmZHtzlV/twTUalxRVJE9K+y6K2bEFD5lQzzS3ornRt+3xhAcHZJZ
JoEFO+vfe9y76ibamF1MG9MLWpa5GkhxAJRT7dXamzneBi+lp3jnwDQx7xirZBgv
dRx4Viv8aVd8bUptH6WOq7HhTdH1cJ91HnV3HszhEQKBgQC5slqfd5u7dS3GKz8R
r5W/uq2ZD9xdSzht/gtinKm+ZBbRXgBdSB/iK1qYr0N2J1c71SSny94Igk37Wdzy
6xs42eO4AtjK9VtdeOpMgAg9exXYD7QvBTWrd2P9s0H7TwI23U6aO8eSNq5TvjoA
VP34YerVulOEwAxmNXD7HxncWwKBgQDmT1f9hWLRmyZJA4PYgUwWfAmMA1J0kTvb
/SAp8xtuHf7aUsofPFdH2e0ZIxqs9pgw5Xe75R7WRl9Kagwvj1FyxOlbIl70oFC9
6ZVTtKNPysC7wHRCiq3rSTptJVvHnSbUnjYUaGRTBoZoO06/XeTVxWBhGqlXMR4g
1zMllxNhqQKBgQCqaEaLYmqyGvU3jsYgx72nJXtbI9f4cSfGoEP4aYXHpvfdh+LT
t3UyytAnn2ZMDgwaZx1tMkN+EX3Yc/2zmhbwE9vKKCqkDYabf3ObYj6tlL9V+AtE
vwYiZLyIuzCIm1eP24TfOo6I2c8eIEn/tf8eKVUkbwlnoAmA3ffhKwhDUwKBgAQ8
Dz0dF70M4j5Tbzgq4HBFZZOKeBimlvgrhkzfKHD04ju3a/n5DQ3IJwDgUVw+yHuV
rIbq5o9ACf5/RqwVMYUvs662ShKrY/Cp+9HnL12HN9X76udST3JVDVr2ZeKctBsS
saHcbYPNSeletcr0+gEoOOZwLA7rW0d1TX5ycavJAoGAK4FeZEp3uItGrPH93WjK
MqCisgqHSeFar2r7p9exiXUfOzSNjTku0X0SH7vi3IQXgA8HCnUw3EX7nggdxzCq
OLwt9K+TltkCmHV04eJlKIpOIhkoze+0LO9qCo7NFh9G97jBzEtMZFKZBfhnKuFo
j1poczFioxLFdJZfwFmixio=
-----END PRIVATE KEY-----
</key>

reneg-sec 0

verify-x509-name yhm.server name
```

### `OpenVPN3 설치`

```bash
hyemi@KHP18:~$ sudo mkdir -p /etc/apt/keyrings && curl -fsSL https://packages.openvpn.net/packages-repo.gpg | sudo tee /etc/apt/keyrings/openvpn.asc
-----BEGIN PGP PUBLIC KEY BLOCK-----
Version: GnuPG v1

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
R0bbxjM9S40=
=ntdv
-----END PGP PUBLIC KEY BLOCK-----
hyemi@KHP18:~$ DISTRO=$(lsb_release -c | awk '{print $2}')
hyemi@KHP18:~$ echo "deb [signed-by=/etc/apt/keyrings/openvpn.asc] https://packages.openvpn.net/openvpn3/debian $DISTRO main" | sudo tee /etc/apt/sources.list.d/openvpn-packages.list
deb [signed-by=/etc/apt/keyrings/openvpn.asc] https://packages.openvpn.net/openvpn3/debian jammy main
hyemi@KHP18:~$ sudo apt update
Hit:1 http://archive.ubuntu.com/ubuntu jammy InRelease
Get:2 http://security.ubuntu.com/ubuntu jammy-security InRelease [129 kB]
Get:3 https://packages.openvpn.net/openvpn3/debian jammy InRelease [6254 B]
Get:4 http://archive.ubuntu.com/ubuntu jammy-updates InRelease [128 kB]
Get:5 https://packages.openvpn.net/openvpn3/debian jammy/main amd64 Packages [4160 B]
Hit:6 http://archive.ubuntu.com/ubuntu jammy-backports InRelease
Get:7 http://archive.ubuntu.com/ubuntu jammy-updates/main amd64 Packages [1775 kB]
Err:7 http://archive.ubuntu.com/ubuntu jammy-updates/main amd64 Packages
  Hash Sum mismatch
  Hashes of expected file:
   - Filesize:1775244 [weak]
   - SHA256:7858ba1c03987621f873bab328ee6a8765780832f3a87c6ecc70542cdeb60308
   - SHA1:bdb7cd50b0d947a8fbb929aeb6dc119f22b9c984 [weak]
   - MD5Sum:c49341ab1c44836f4b38b1c9d5ae7fb4 [weak]
  Hashes of received file:
   - SHA256:25b21c2815056b3170eca6bcbeac43813abd22ca97597e6db4d862bea402024f
   - SHA1:f7305fc93a3d76dd8cfacfd5d3c1f6ec82d10228 [weak]
   - MD5Sum:246ffbd670fcaef08a71aaf65f2e0e0f [weak]
   - Filesize:1775244 [weak]
  Last modification reported: Fri, 28 Jun 2024 01:21:16 +0000
  Release file created at: Fri, 28 Jun 2024 01:20:01 +0000
Get:8 http://archive.ubuntu.com/ubuntu jammy-updates/universe amd64 Packages [1091 kB]
Err:8 http://archive.ubuntu.com/ubuntu jammy-updates/universe amd64 Packages

Fetched 3134 kB in 3s (1100 kB/s)
Reading package lists... Done
E: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/jammy-updates/main/binary-amd64/by-hash/SHA256/7858ba1c03987621f873bab328ee6a8765780832f3a87c6ecc70542cdeb60308  Hash Sum mismatch
   Hashes of expected file:
    - Filesize:1775244 [weak]
    - SHA256:7858ba1c03987621f873bab328ee6a8765780832f3a87c6ecc70542cdeb60308
    - SHA1:bdb7cd50b0d947a8fbb929aeb6dc119f22b9c984 [weak]
    - MD5Sum:c49341ab1c44836f4b38b1c9d5ae7fb4 [weak]
   Hashes of received file:
    - SHA256:25b21c2815056b3170eca6bcbeac43813abd22ca97597e6db4d862bea402024f
    - SHA1:f7305fc93a3d76dd8cfacfd5d3c1f6ec82d10228 [weak]
    - MD5Sum:246ffbd670fcaef08a71aaf65f2e0e0f [weak]
    - Filesize:1775244 [weak]
   Last modification reported: Fri, 28 Jun 2024 01:21:16 +0000
   Release file created at: Fri, 28 Jun 2024 01:20:01 +0000
E: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/jammy-updates/universe/binary-amd64/by-hash/SHA256/cae58ea068b31c56397d94006b8ef6aec0d545aa7a191a3bf76acff7d8d91b20
E: Some index files failed to download. They have been ignored, or old ones used instead.
hyemi@KHP18:~$ sudo apt install openvpn3
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  libjsoncpp25 libprotobuf23 libtinyxml2-9
Recommended packages:
  kmod-ovpn-dco
The following NEW packages will be installed:
  libjsoncpp25 libprotobuf23 libtinyxml2-9 openvpn3
0 upgraded, 4 newly installed, 0 to remove and 47 not upgraded.
Need to get 2360 kB of archives.
After this operation, 7990 kB of additional disk space will be used.
Do you want to continue? [Y/n] y
Get:1 https://packages.openvpn.net/openvpn3/debian jammy/main amd64 openvpn3 amd64 21-1+jammy [1370 kB]
Get:2 http://archive.ubuntu.com/ubuntu jammy/main amd64 libjsoncpp25 amd64 1.9.5-3 [80.0 kB]
Get:3 http://archive.ubuntu.com/ubuntu jammy-updates/main amd64 libprotobuf23 amd64 3.12.4-1ubuntu7.22.04.1 [877 kB]
Get:4 http://archive.ubuntu.com/ubuntu jammy/universe amd64 libtinyxml2-9 amd64 9.0.0+dfsg-3 [32.5 kB]
Fetched 2360 kB in 3s (791 kB/s)
Selecting previously unselected package libjsoncpp25:amd64.
(Reading database ... 24410 files and directories currently installed.)
Preparing to unpack .../libjsoncpp25_1.9.5-3_amd64.deb ...
Unpacking libjsoncpp25:amd64 (1.9.5-3) ...
Selecting previously unselected package libprotobuf23:amd64.
Preparing to unpack .../libprotobuf23_3.12.4-1ubuntu7.22.04.1_amd64.deb ...
Unpacking libprotobuf23:amd64 (3.12.4-1ubuntu7.22.04.1) ...
Selecting previously unselected package libtinyxml2-9:amd64.
Preparing to unpack .../libtinyxml2-9_9.0.0+dfsg-3_amd64.deb ...
Unpacking libtinyxml2-9:amd64 (9.0.0+dfsg-3) ...
Selecting previously unselected package openvpn3.
Preparing to unpack .../openvpn3_21-1+jammy_amd64.deb ...
Unpacking openvpn3 (21-1+jammy) ...
Setting up libprotobuf23:amd64 (3.12.4-1ubuntu7.22.04.1) ...
Setting up libtinyxml2-9:amd64 (9.0.0+dfsg-3) ...
Setting up libjsoncpp25:amd64 (1.9.5-3) ...
Setting up openvpn3 (21-1+jammy) ...
Processing triggers for man-db (2.10.2-1) ...
Processing triggers for dbus (1.12.20-2ubuntu4.1) ...
Processing triggers for libc-bin (2.35-0ubuntu3.8) ...
hyemi@KHP18:~$ openvpn3 session-start --config downloaded-client-config.ovpn
Using configuration profile from file: downloaded-client-config.ovpn
Session path: /net/openvpn/v3/sessions/7aceebf3se603s4246s9392s8a582c6f5c4b
Connected
```

### `OpenVPN 설치 후 ping 결과`

```bash
hyemi@KHP18:~$ ping 172.16.20.205
PING 172.16.20.205 (172.16.20.205) 56(84) bytes of data.
64 bytes from 172.16.20.205: icmp_seq=1 ttl=63 time=77.4 ms
64 bytes from 172.16.20.205: icmp_seq=2 ttl=63 time=76.1 ms
64 bytes from 172.16.20.205: icmp_seq=3 ttl=63 time=78.9 ms
64 bytes from 172.16.20.205: icmp_seq=4 ttl=63 time=76.0 ms
64 bytes from 172.16.20.205: icmp_seq=5 ttl=63 time=93.5 ms
64 bytes from 172.16.20.205: icmp_seq=6 ttl=63 time=84.5 ms
64 bytes from 172.16.20.205: icmp_seq=7 ttl=63 time=76.9 ms
64 bytes from 172.16.20.205: icmp_seq=8 ttl=63 time=98.1 ms
64 bytes from 172.16.20.205: icmp_seq=9 ttl=63 time=86.9 ms
64 bytes from 172.16.20.205: icmp_seq=10 ttl=63 time=81.4 ms
64 bytes from 172.16.20.205: icmp_seq=11 ttl=63 time=86.2 ms
64 bytes from 172.16.20.205: icmp_seq=12 ttl=63 time=76.2 ms
64 bytes from 172.16.20.205: icmp_seq=13 ttl=63 time=99.4 ms
64 bytes from 172.16.20.205: icmp_seq=14 ttl=63 time=76.5 ms
64 bytes from 172.16.20.205: icmp_seq=15 ttl=63 time=78.5 ms
64 bytes from 172.16.20.205: icmp_seq=16 ttl=63 time=80.0 ms
64 bytes from 172.16.20.205: icmp_seq=17 ttl=63 time=77.2 ms
64 bytes from 172.16.20.205: icmp_seq=18 ttl=63 time=78.5 ms
64 bytes from 172.16.20.205: icmp_seq=19 ttl=63 time=75.6 ms
64 bytes from 172.16.20.205: icmp_seq=20 ttl=63 time=77.0 ms
64 bytes from 172.16.20.205: icmp_seq=21 ttl=63 time=78.2 ms
64 bytes from 172.16.20.205: icmp_seq=22 ttl=63 time=76.2 ms
64 bytes from 172.16.20.205: icmp_seq=23 ttl=63 time=75.5 ms
64 bytes from 172.16.20.205: icmp_seq=24 ttl=63 time=82.4 ms
64 bytes from 172.16.20.205: icmp_seq=25 ttl=63 time=77.3 ms
64 bytes from 172.16.20.205: icmp_seq=26 ttl=63 time=82.6 ms
64 bytes from 172.16.20.205: icmp_seq=27 ttl=63 time=81.2 ms
64 bytes from 172.16.20.205: icmp_seq=28 ttl=63 time=77.1 ms
64 bytes from 172.16.20.205: icmp_seq=29 ttl=63 time=79.2 ms
64 bytes from 172.16.20.205: icmp_seq=30 ttl=63 time=81.7 ms
64 bytes from 172.16.20.205: icmp_seq=31 ttl=63 time=77.9 ms
64 bytes from 172.16.20.205: icmp_seq=32 ttl=63 time=79.0 ms
64 bytes from 172.16.20.205: icmp_seq=33 ttl=63 time=77.0 ms
64 bytes from 172.16.20.205: icmp_seq=34 ttl=63 time=76.7 ms
64 bytes from 172.16.20.205: icmp_seq=35 ttl=63 time=79.2 ms
^X64 bytes from 172.16.20.205: icmp_seq=36 ttl=63 time=76.5 ms
64 bytes from 172.16.20.205: icmp_seq=37 ttl=63 time=75.4 ms
^C
--- 172.16.20.205 ping statistics ---
37 packets transmitted, 37 received, 0% packet loss, time 36062ms
rtt min/avg/max/mdev = 75.411/80.214/99.387/5.788 ms
```

### `OpenVPN을 통해 Private Server 접속`

```bash
hyemi@KHP18:~$ chmod 400 yhm-keypair.pem
hyemi@KHP18:~$ ssh -i yhm-keypair.pem ubuntu@172.16.20.205
The authenticity of host '172.16.20.205 (172.16.20.205)' can't be established.
ED25519 key fingerprint is SHA256:sE3+J1IJgVehlztJHtIt5kbpOvOFIt7OALwx2XtV3nI.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '172.16.20.205' (ED25519) to the list of known hosts.
Welcome to Ubuntu 22.04.4 LTS (GNU/Linux 6.5.0-1017-aws x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

  System information as of Fri Jun 28 02:19:09 UTC 2024

  System load:  0.0               Processes:             96
  Usage of /:   20.8% of 7.57GB   Users logged in:       0
  Memory usage: 21%               IPv4 address for eth0: 172.16.20.205
  Swap usage:   0%

Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status

The list of available updates is more than a week old.
To check for new updates run: sudo apt update
Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings

Last login: Fri Jun 28 00:46:35 2024 from 172.16.16.92
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

ubuntu@ip-172-16-20-205:~$
```

### `Windows에서 직접 EC2로 연결`

![Untitled](Network%2005%20d28e8a243c424c67b896e2b8a940331f/Untitled%203.png)

## DHCP(Dyamic Host Configuration Protocol)

호스트에 IP관련 정보(IP주소, 서브넷마스크, 게이트웨이, DNS주소 등)들을 자동으로 할당할 수 있도록 하는 프로토콜, 서비스.

DHCP는 4가지 메시지를 이용해서 클라이언트와 서버간에 통신을 함.
DHCP는 UDP를 사용하고, 포트번호는 클라이언트(68번), 서버(67번)을 사용함.
DHCP는 목적지주소로 브로드캐스트와 유니캐스트 모두 사용함. DHCP프로그램 설정에 따라서 정해짐.

```
            클라이언트(68번)                서버(67번)

                               Discover ->   : 클라이언트가 서버를 찾기위한 메시지
                               Offer    <-   : 서버가 클라이언트에게 할당할 정보를 담은 메시지
                               Request  ->   : 해당 정보를 사용하겠다는 클라이언트 요청 메시지
                               Ack      <-   : 사용을 승인하는 서버 메시지

```

DHCP 서버는 2개이상 운용가능하고, 이때 클라이언트는 먼저 도착한 정보를 사용하게 된다.

DHCP서비스는 라우터, 멀티레이어 스위치, 각종 서버에서 제공가능.

- 시스코 IOS에서 DHCP 서버 설정

ip dhcp pool pool명
network 네트워크주소 서브넷마스크
default-router 게이트웨이주소
dns-server DNS주소
domain-name 도메인주소
lease 임대기간

ip dhcp excluded-address 시작IP주소 마지막IP주소

- DHCP Relay Agent : DHCP서비스를 받으려는 네트워크 안에 DHCP서비스가 없는 경우 클라이언트 메시지와
서버 메시지를 중계해주는 기능.
DHCP Relay Agent는 DHCP서버를 지정해서 유니캐스트로 전송한다.
ip helper-address x.x.x.x 를 해당 네트워크 인터페이스에 설정한다.

![클라이언트한테 줄 정보 세팅. Service는 On으로 세팅.](Network%2005%20d28e8a243c424c67b896e2b8a940331f/Untitled%204.png)

클라이언트한테 줄 정보 세팅. Service는 On으로 세팅.

![DHCP로 자동할당 받은 결과.](Network%2005%20d28e8a243c424c67b896e2b8a940331f/Untitled%205.png)

DHCP로 자동할당 받은 결과.

![Server1에 DNS를 추가하고 nslookup으로 웹사이트 ip 주소 확인.](Network%2005%20d28e8a243c424c67b896e2b8a940331f/Untitled%206.png)

Server1에 DNS를 추가하고 nslookup으로 웹사이트 ip 주소 확인.