**1. Nginx 설치**

* 명령어 = **sudo apt install 프로그램명**

  * 시스템 변경이 필요하므로 sudo를 붙임
  * 설치 전, sudo apt update/upgrade로 최신 목록을 확인하고, 최신 버전으로 업데이트를 함
  * apt는 프로그램을 설치, 업데이트, 삭제 등 할 수 있도록 도와주는 프로그램 (PlayStore와 유사)





**2. Nginx 서비스가 정상적으로 실행 중인지 확인 (Day1에서 배운 systemctl 지식 활용)**

* 전체 훑기 = systemctl list-units --state=running

  * 특정 서비스 상태 보기 = **systemctl status 서비스명**
* Nginx = Master process(Worker 관리) + Worker process → 가용성

> nginx.service - A high performance web server and a reverse proxy server

&#x20;    Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; preset>

&#x20;    **Active: active (running)** since Thu 2026-07-09 16:25:41 KST; 6min ago

&#x20;      Docs: man:nginx(8)

&#x20;   Process: 1029 ExecStartPre=/usr/sbin/nginx -t -q -g daemon on; master\_p>

&#x20;   Process: 1036 ExecStart=/usr/sbin/nginx -g daemon on; master\_process on>

&#x20;  Main PID: 1106 (nginx)

&#x20;     Tasks: 13 (limit: 9361)

&#x20;    Memory: 9.1M (peak: 20.5M)

&#x20;       CPU: 74ms

&#x20;    CGroup: /system.slice/nginx.service

&#x20;            ├─1106 "nginx: master process /usr/sbin/nginx -g daemon on; ma>

&#x20;            ├─1108 "nginx: worker process"

&#x20;            ├─1109 "nginx: worker process"

&#x20;            ├─1110 "nginx: worker process"

&#x20;            ├─1111 "nginx: worker process"

&#x20;            ├─1112 "nginx: worker process"

&#x20;            ├─1113 "nginx: worker process"

&#x20;            ├─1114 "nginx: worker process"

&#x20;            ├─1115 "nginx: worker process"

&#x20;            ├─1116 "nginx: worker process"

&#x20;            ├─1117 "nginx: worker process"

&#x20;            ├─1118 "nginx: worker process"

&#x20;            └─1119 "nginx: worker process"



Jul 09 16:25:41 YUHU systemd\[1]: Starting nginx.service - A high performanc>

Jul 09 16:25:41 YUHU systemd\[1]: Started nginx.service - A high performance>





**3. 기본 정적 웹페이지(HTML)를 만들어서 배포**

* 명령어 = curl 주소 (Command Line URL)

  * 용도 : 텍스트 기반의 웹브라우저 \& 데이터를 잘 주고받는지 테스트
  * localhost = 127.0.0.1(IP) → LoopBack으로 80번 포트로 감(웹사이트 전용) → Nginx worker process가 요청을 받음 → 규칙에 따라 응답

><!DOCTYPE html>

<html>

<head>

<title>Welcome to nginx!</title>

<style>

html { color-scheme: light dark; }

body { width: 35em; margin: 0 auto;

font-family: Tahoma, Verdana, Arial, sans-serif; }

</style>

</head>

<body>

<h1>Welcome to nginx!</h1>

<p>If you see this page, the nginx web server is successfully installed and

working. Further configuration is required.</p>



<p>For online documentation and support please refer to

<a href="http://nginx.org/">nginx.org</a>.<br/>

Commercial support is available at

<a href="http://nginx.com/">nginx.com</a>.</p>



<p><em>Thank you for using nginx.</em></p>

</body>

</html>



(2) 파일 내용 변경

* index.html 파일위치 찾기 = cat /etc/nginx/sites-available/default
* 파일 수정 = sudo nano /var/www/html/index.html (ctrl + o, ctrl + x 등..)

※ sudo는 소유자가 root일 경우 pc인 내가 권한을 얻기 위해 적어야 함 ※



\-----------------------



(3) 새로운 url 생성

* 폴더 생성 = sudo mkdir -p /var/www/myhomepage
* 파일 생성 = sudo nano /var/www/myhomepage/index.html 		>  		cat으로 확인
* 설정 관리 = sudo nano /etc/nginx/sites-available/talju

>server {									// 하나의 웹사이트 선언

&#x20;   listen 80;						// 80번 포트로 들어오는 신호를 대기하겠다

&#x20;   server\_name talju.local;			// talju.local이라고 입력하고 들어올 때 적용

&#x20;   root /var/www/talju;				// 폴더 경로

&#x20;   index index.html;				// 주소 뒤에 특정 파일 이름 안 적고 그냥 들어왔을 때, 가장 먼저 보여줄 기본 페이지



&#x20;   location / {

&#x20;       try\_files $uri $uri/ =404;		// 특정 페이지 요청했을 때, 처리규칙

&#x20;   }

}

* symlink 걸기 = sudo ln -s 원본경로 바로가기경로 (Liunk SymbolicLink)

  * symbolic link(바로가기) 생성 → 유지관리가 쉬워짐
  * sites-available = 설정 파일 보관소 (전체 목록, 당장 안 쓰는 것도 둠)
  * sites-enabled = 현재 실제로 켜져 있는 것들만 모아둔 곳
* 문법 검사 = sudo nginx -t (Test Configuration)
* nginx 재실행 = sudo systemctl reload nginx



\-----------



(4) IPv6 listen 누락 디버깅

* **sudo nginx -T(전체텍스트 중) | grep -A 5 "server\_name" (server\_name 이후 5줄 보여주셈)**

>nginx: the configuration file /etc/nginx/nginx.conf syntax is ok

nginx: configuration file /etc/nginx/nginx.conf test is successful

&#x20;       # server\_names\_hash\_bucket\_size 64;

&#x20;       # server\_name\_in\_redirect off;



&#x20;       include /etc/nginx/mime.types;

&#x20;       default\_type application/octet-stream;



&#x20;       ##

\--

&#x20;       **server\_name \_;**



&#x20;       **location / {**

&#x20;               **# First attempt to serve request as file, then**

&#x20;               **# as directory, then fall back to displaying a 404.**

&#x20;               **try\_files $uri $uri/ =404;**

\--

\#       server\_name example.com;

\#

\#       root /var/www/example.com;

\#       index index.html;

\#

\#       location / {

\--

&#x20;       **server\_name talju.local;**

&#x20;       **root /var/www/talju;**

&#x20;       **index index.html;**



&#x20;       **location / {**

&#x20;               **try\_files $uri $uri/ =404;**



* **curl -v(상세히verbose) -H "Host: talju.local" localhost(헤더로 서버 설정) 2>\&1(오류포함) | head -10 (앞에서 10줄만)**

>\* Host localhost:80 was resolved.

\* IPv6: ::1

\* IPv4: 127.0.0.1

&#x20; % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current

&#x20;                                Dload  Upload   Total   Spent    Left  Speed

&#x20; 0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0\*   Trying \[::1]:80...

**\* Connected to localhost (::1) port 80**

> GET / HTTP/1.1

> Host: talju.local

> User-Agent: curl/8.5.0



* 원인 : talju는 IPv4만 리스닝하지만, default는 IPv4,6 모두 리스닝. curl은 localhost로 이동 → 헤더를 보고 worker process가 방향을 수정. **talju는 IPv4만 듣고 있기에 IPv6를 못 잡음**
* 수정방법 = listen \[::]:80; 추가 (→ 문법검사 → 재실행)





4\. Nginx 로그 위치 찾아서 확인

* 명령어 = sudo tail -n 10 /var/log/nginx/access.log

>::1 - - \[09/Jul/2026:17:01:32 +0900] "GET / HTTP/1.1" 200 615 "-" "curl/8.5.0"

::1 - - \[09/Jul/2026:17:11:06 +0900] "GET / HTTP/1.1" 200 409 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/149.0.0.0 Safari/537.36"

**::1 - - \[09/Jul/2026:17:11:06 +0900] "GET /favicon.ico HTTP/1.1" 404 196 "http://localhost/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/149.0.0.0 Safari/537.36"**

::1 - - \[09/Jul/2026:17:37:42 +0900] "GET / HTTP/1.1" 200 118 "-" "curl/8.5.0"

::1 - - \[09/Jul/2026:18:24:49 +0900] "GET / HTTP/1.1" 200 118 "-" "curl/8.5.0"

::1 - - \[09/Jul/2026:18:25:06 +0900] "GET / HTTP/1.1" 200 118 "-" "curl/8.5.0"

::1 - - \[09/Jul/2026:18:27:24 +0900] "GET / HTTP/1.1" 200 118 "-" "curl/8.5.0"

**127.0.0.1 - - \[09/Jul/2026:18:28:45 +0900] "GET / HTTP/1.1" 200 181 "-" "curl/8.5.0"**

::1 - - \[09/Jul/2026:18:57:21 +0900] "GET / HTTP/1.1" 200 181 "-" "curl/8.5.0"

* 클라이언트IP  인증  시간  요청내용  상태코드  응답크기  Referer  User-Agent



* 에러 로그 명령어 = sudo tail -n 10 /var/log/nginx/error.log

  * /var/... = 절대경로
  * var/... = 상대경로

>2026/07/09 16:25:41 \[notice] 1106#1106: using inherited sockets from "5;6;" (소켓상속)









