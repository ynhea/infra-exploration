# Day 3 - Web Server(Nginx) 실습

## 1. Nginx 설치

### 설치 명령어

```bash
sudo apt install nginx
```

### 왜 `sudo`를 사용하는가?

* 시스템에 프로그램을 설치하거나 변경하는 작업은 관리자 권한(root)이 필요하다.
* 따라서 `sudo`를 사용해 관리자 권한으로 실행한다.

### 설치 전 수행

```bash
sudo apt update
sudo apt upgrade
```

* `apt update` : 설치 가능한 패키지 목록을 최신 상태로 갱신
* `apt upgrade` : 설치된 패키지를 최신 버전으로 업데이트
* `apt`는 프로그램 설치, 삭제, 업데이트를 관리하는 패키지 관리자이다. (Play Store와 비슷한 역할)

---

# 2. Nginx 서비스 실행 확인

### 실행 중인 서비스 확인

```bash
systemctl list-units --state=running
```

### 특정 서비스 상태 확인

```bash
systemctl status nginx
```

### 확인한 내용

```
Active: active (running)
```

* Nginx 서비스가 정상적으로 실행되고 있음을 확인했다.
* Day1에서 학습한 `systemctl` 명령을 실제 서비스 관리에 활용했다.

### Nginx 프로세스 구조

* Master Process

  * 설정 파일을 읽는다.
  * Worker Process를 생성하고 관리한다.
* Worker Process

  * 실제 클라이언트의 HTTP 요청을 처리한다.

즉, 사용자의 요청은 Worker Process가 처리하며, Master Process는 이를 관리하는 역할을 한다.

---

# 3. 기본 웹페이지 배포

## (1) 기본 페이지 확인

```bash
curl localhost
```

### curl이란?

Command Line URL의 약자로, 터미널에서 웹 서버에 HTTP 요청을 보내는 도구이다.

동작 과정

```
localhost
→ 127.0.0.1
→ 80번 포트
→ Nginx Worker Process
→ index.html 응답
```

기본 Welcome 페이지가 정상적으로 출력되는 것을 확인하였다.

---

## (2) 기본 페이지 수정

현재 어떤 HTML을 제공하는지 확인하였다.

```bash
cat /etc/nginx/sites-available/default
```

웹 페이지 수정

```bash
sudo nano /var/www/html/index.html
```

※ `/var/www/html`의 소유자가 root이므로 `sudo`가 필요하다.

---

## (3) 새로운 사이트 생성

### 디렉터리 생성

```bash
sudo mkdir -p /var/www/talju
```

### HTML 작성

```bash
sudo nano /var/www/talju/index.html
```

### 사이트 설정 작성

```bash
sudo nano /etc/nginx/sites-available/talju
```

설정 예시

```nginx
server {
    listen 80;
    server_name talju.local;

    root /var/www/talju;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

### 각 설정의 의미

* `listen 80`

  * HTTP 80번 포트에서 요청을 받는다.

* `server_name`

  * 어떤 도메인으로 접속했을 때 이 설정을 사용할지 결정한다.

* `root`

  * 웹 문서가 저장된 위치

* `index`

  * 기본으로 보여줄 파일

* `location`

  * 요청을 어떻게 처리할지 정의한다.

* `try_files`

  * 요청한 파일이 존재하면 제공하고, 없으면 404를 반환한다.

---

### Symbolic Link 생성

```bash
sudo ln -s 원본경로 바로가기경로
```

Symbolic Link는 실제 파일을 복사하지 않고 바로가기만 만드는 방식이다.

```
sites-available
→ 모든 설정 파일 저장

sites-enabled
→ 현재 활성화된 설정만 존재
```

설정을 활성화한 후

```bash
sudo nginx -t
```

문법 검사

```bash
sudo systemctl reload nginx
```

재시작 없이 설정만 다시 불러왔다.

---

# 4. IPv6 문제 디버깅

현재 적용된 설정 확인

```bash
sudo nginx -T | grep -A 5 "server_name"
```

HTTP 요청 확인

```bash
curl -v -H "Host: talju.local" localhost
```

### 원인

* `localhost`는 IPv6(::1)를 우선 사용하였다.
* 하지만 `talju` 서버는 IPv4만 listen하고 있었다.
* 따라서 요청이 원하는 서버까지 전달되지 않았다.

### 해결 방법

```nginx
listen [::]:80;
```

IPv6도 함께 listen하도록 설정하였다.

이후

```bash
sudo nginx -t
sudo systemctl reload nginx
```

를 수행하여 정상 동작을 확인하였다.

---

# 5. Nginx 로그 확인

### Access Log

```bash
sudo tail -n 10 /var/log/nginx/access.log
```

로그에서 확인할 수 있는 정보

* 클라이언트 IP
* 요청 시간
* 요청 URL
* HTTP 상태 코드
* 응답 크기
* Referer
* User-Agent

예시

```
127.0.0.1
GET /
200
```

→ 로컬에서 정상적으로 페이지를 요청했고, HTTP 200 OK를 반환하였다.

또한

```
GET /favicon.ico
404
```

도 확인하였다.

이는 브라우저가 자동으로 favicon을 요청했지만 해당 파일이 존재하지 않아 발생한 정상적인 404이다.

---

### Error Log

```bash
sudo tail -n 10 /var/log/nginx/error.log
```

예시

```
using inherited sockets
```

이는 Nginx를 재시작하거나 Reload할 때 기존 소켓을 이어받았다는 의미의 Notice 로그이며, 오류가 아니라 정상적인 동작이다.

---

# 실습을 통해 배운 점

* `systemctl`을 이용해 웹 서버 서비스를 관리하는 방법
* `curl`로 HTTP 요청을 테스트하는 방법
* Nginx의 기본 동작 구조(Master / Worker Process)
* Virtual Host와 `server_name` 설정 방법
* Symbolic Link를 이용한 사이트 활성화 방식
* IPv4와 IPv6 설정 차이로 발생하는 문제와 해결 방법
* Access Log와 Error Log를 활용한 웹 서버 점검 방법
