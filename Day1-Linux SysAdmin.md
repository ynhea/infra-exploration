1\. 파일 시스템: 디스크가 어떻게 나뉘어 있고, 사용률은 어느 정도인가?

* 디스크 분할 = Linux에서 하나의 물리적 디스크를 독립된 여러 개의 논리적 공간(파티션)으로 나누어 사용하는 구조
* 명령어 : df (디스크 전체의 사용률 현황) \& du (특정 폴더/파일이 차지하는 용량)

**<df -h 결과>**

Filesystem      Size  Used Avail Use% Mounted on

none            3.9G     0  3.9G   0% /usr/lib/modules/6.6.87.2-microsoft-standard-WSL2

none            3.9G  4.0K  3.9G   1% /mnt/wsl

drivers         469G  243G  226G  52% /usr/lib/wsl/drivers

***/dev/sdd       1007G  2.1G  954G   1% /				WSL2 리눅스 루트 파일시스템이 저장된 가상 디스크 장치 이름***

none            3.9G   76K  3.9G   1% /mnt/wslg

none            3.9G     0  3.9G   0% /usr/lib/wsl/lib

rootfs          3.9G  2.7M  3.9G   1% /init

none            3.9G  528K  3.9G   1% /run

none            3.9G     0  3.9G   0% /run/lock

none            3.9G     0  3.9G   0% /run/shm

none            3.9G   72K  3.9G   1% /mnt/wslg/versions.txt

none            3.9G   72K  3.9G   1% /mnt/wslg/doc

***C:\\             469G  243G  226G  52% /mnt/c			C 드라이브 연결됨***

***D:\\             7.0G  6.6G  481M  94% /mnt/d			D 드라이브 연결됨***

tmpfs           782M   20K  782M   1% /run/user/1000

* 나머지는 실제 디스크가 아닌, 메모리 기반 가상 파일시스템.



2\. 사용자 및 권한: 어떤 사용자가 있고, 나(현재 로그인 계정)는 어떤 권한을 가지고 있는가?

* cat (파일 내용 확인)/etc/passwd

root:x:0:0:root:/root:/bin/bash

***daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin***

bin:x:2:2:bin:/bin:/usr/sbin/nologin

sys:x:3:3:sys:/dev:/usr/sbin/nologin

sync:x:4:65534:sync:/bin:/bin/sync

games:x:5:60:games:/usr/games:/usr/sbin/nologin

man:x:6:12:man:/var/cache/man:/usr/sbin/nologin

lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin

mail:x:8:8:mail:/var/mail:/usr/sbin/nologin

news:x:9:9:news:/var/spool/news:/usr/sbin/nologin

uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin

proxy:x:13:13:proxy:/bin:/usr/sbin/nologin

***www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin***

backup:x:34:34:backup:/var/backups:/usr/sbin/nologin

list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin

irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin

\_apt:x:42:65534::/nonexistent:/usr/sbin/nologin

nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin

systemd-network:x:998:998:systemd Network Management:/:/usr/sbin/nologin

systemd-timesync:x:996:996:systemd Time Synchronization:/:/usr/sbin/nologin

dhcpcd:x:100:65534:DHCP Client Daemon,,,:/usr/lib/dhcpcd:/bin/false

messagebus:x:101:101::/nonexistent:/usr/sbin/nologin

syslog:x:102:102::/nonexistent:/usr/sbin/nologin

systemd-resolve:x:991:991:systemd Resolver:/:/usr/sbin/nologin

uuidd:x:103:103::/run/uuidd:/usr/sbin/nologin

landscape:x:104:105::/var/lib/landscape:/usr/sbin/nologin

polkitd:x:990:990:User for polkitd:/:/usr/sbin/nologin

**pc	:	x	:	1000:1000	:	,,,	:	/home/pc	:	/bin/bash**

* **계정명 : x(비밀번호 x로 숨김처리) : UID(User ID) : GID(Group ID) : 설명 : 홈디렉토리(절대경로) : 로그인쉘(터미널 종류)**

  * /bin/bash 쉘 = 로그인해서 명령어를 칠 수 있음	

&#x20;    		(/bin = 실행파일(명령어)들이 모여있는 폴더, bin = 프로그램 이름)

&#x09;- user/sbin/nologin = 절대 터미널에 로그인 못 함 → 특정 서비스 실행을 위한 전용 계정

&#x20;     - 즉, 서비스별로 계정을 분리함

* 현재 로그인한 계정(PC)이 가진 권한 → id(PC가 속한 그룹목록) \& grep 'sudo' /etc/group(sudo 권한 그룹만 골라서 보여줌)

<id 결과>

uid=1000(pc) gid=1000(pc) groups=1000(pc),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),100(users),1001(docker)

<grep 'sudo' /etc/group 결과>

sudo:x:27:pc

* sudo = 관리자 권한 쓸 수 있는 그룹
* adm = 시스템 로그 파일을 읽을 수 있는 그룹
* docker = Docker을 sudo없이 실행할 수 있는 그룹
* 서버를 넘겨받으면 SUDO권한을 가지고 있는 계정을 꼭 확인 해야함!! 곧 이 서버를 망가뜨릴 수 있는 사람이라는 뜻이기 때문!@!



3\. 프로세스: 현재 가장 리소스를 많이 쓰는 프로세스는 무엇인가?

* ps aux : 시스템의 모든 프로세스를 CPU 및 메모리 사용량과 함께 보여줌
* top : ps aux를 실시간으로 보여줌
* htop : top의 예쁜 버전 (설치 필요)

root        1264  0.3  0.1 1756620 12672 ?       Ssl  19:06   0:00 /usr/libexec/wsl-pro-service

pc          1289  0.7  0.0   6072  5120 pts/0    Ss   19:06   0:00 -bash



4\. 서비스(systemd): 현재 활성화(running)되어 있는 서비스는 무엇인가?

* systemctl list-units --state=running : 전체 프로세스 중 running상태인 것만 골라 볼 수 O
* systemctl list-units --type=service
* systemctl list-units --failed : failed된 서비스를 확인O → 죽은 서비스를 파악해 문제를 빨리 해결할 수 있음 / 문제있는데 failed에 안 나올 경우, systemd 문제가 아니라 다른 곳일 문제임!

(systemd = Linux 컴퓨터가 켜질 때부터 꺼질 때까지 백그라운드에서 실행되는, 프로그램들을 관리하는 시스템)



5\. 정리

* 학습 목표 : 서버 파악하는 방법 (서버 온보딩 체크리스트 이해)
* df → /etc/passwd/id → ps/top → systemctl --failed
* 디스크 사용률 → 사용자 파악 \& 권한 확인(sudo) → 프로세스 → active/failed 서비스 확인











