1\. 서버의 IP 확인

* IP: 호스트를 식별하는 주소
* 명령어 = ip a \& ia addr

1: **lo**: <LOOPBACK,UP,LOWER\_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000

&#x20;   link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00

&#x20;   inet 127.0.0.1/8 scope host lo

&#x20;      valid\_lft forever preferred\_lft forever

&#x20;   inet 10.255.255.254/32 brd 10.255.255.254 scope global lo

&#x20;      valid\_lft forever preferred\_lft forever

&#x20;   inet6 ::1/128 scope host

&#x20;      valid\_lft forever preferred\_lft forever

* **루프백 인터페이스 (Loopback Interfae)**

  * 운영체제에서 존재하는 가상 네트워크 인터페이스 (물리적인 HW 없음)
  * 목적 : 자가 테스트 (네트워크 통신 스택(TCP/IP 스택)이 정상적으로 작동하는지 확인)
  * localhost하는 호스트 이름의 127.0.0.1 IP주소 대역이 할당됨
  * 일반적인 경로: 프로그램 → OS 네트워크 스택 → 네트워크 카드(하드웨어) → 인터넷 선 → 외부 PC
  * 루프백 경로: 프로그램 A → OS 네트워크 스택(출력부) → \[여기서 유턴] → OS 네트워크 스택(입력부) → 프로그램 B



2: **eth0**: <BROADCAST,MULTICAST,UP,LOWER\_UP> mtu 1500 qdisc mq state UP group default qlen 1000

&#x20;   link/ether 00:15:5d:a5:4c:66 brd ff:ff:ff:ff:ff:ff				MAC주소

&#x20;   inet 172.26.112.139/20 brd 172.26.127.255 scope global eth0	IPv4주소 =  네트워크 상에서 컴퓨터의 현재 위치를 나타내는 가변 주소

&#x20;      valid\_lft forever preferred\_lft forever					속한 네트워크에 동시에 데이터를 보낼 때 사용하는 주소

&#x20;   inet6 fe80::215:5dff:fea5:4c66/64 scope link				IPv6 주소 = IPv4가 작아서 만듦

&#x20;      valid\_lft forever preferred\_lft forever

* Linux의 외부와 통신하는 데 사용하는 가상 네트워크 인터페이스





2\. 서버가 네트워크 상에 살아있는가?

* 명령어 = ping IPv4주소



3\. 포트가 열려있는 지 확인

* 명령어 = Test-NetConnection -ComputerName IPv4주소 - Port 22(기본 ssh주소)

&#x09;(UBUNTU = sudo systemctl enable --now ssh = 자동 켜지기 등록 \& 즉시 실행)

ComputerName     : 172.26.112.139

RemoteAddress    : 172.26.112.139

RemotePort       : 22

InterfaceAlias   : vEthernet (WSL (Hyper-V firewall))

SourceAddress    : 172.26.112.1

TcpTestSucceeded : True



* SSH (Secure SHell) (보안셀)

  * 멀리 떨어져 있는 컴퓨터(서버)에 인터넷을 통해 안전하게 원격 접속하기 위해 사용하는 프로그램
* DNS: 이름을 IP로 바꿔주는 과정

  * 특정 도메인의 IPv4주소를 파악할 때 사용
  * 명령어 : nslookup 도메인
* 포트: 같은 서버 안에서 서비스를 구분하는 번호



4\. SSH 접속 시도

* 명령어 = ssh 사용자명@IPv4주소

