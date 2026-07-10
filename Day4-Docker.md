# Docker 및 Nginx Reverse Proxy 실습 정리

## 1. Docker 핵심 개념

### Image (이미지)

프로그램과 실행 환경이 함께 저장된 **읽기 전용 설계도**입니다.

### Container (컨테이너)

이미지를 실행한 **격리된 프로세스(실체)**입니다.

### Volume (볼륨)

컨테이너는 삭제되면 내부 데이터도 함께 사라집니다. 이를 보완하기 위해 데이터를 **호스트(컨테이너 외부)에 영구 저장**하는 기능입니다.

### Network (네트워크)

컨테이너와 컨테이너, 또는 컨테이너와 호스트가 서로 통신할 수 있도록 연결해 주는 환경입니다.

> 💡 **Why Docker?**
>
> 개인 개발 환경이나 서버마다 라이브러리와 프로그램 버전이 다르면 서비스 간 충돌이 발생할 수 있습니다.
> Docker는 프로그램 실행에 필요한 모든 요소를 하나의 이미지로 패키징하여 격리하므로, **어디서나 동일한 환경에서 안정적으로 실행**할 수 있습니다.

---

# 2. Docker 컨테이너 실행

다음 명령어를 사용하여 Nginx 웹 서버 컨테이너를 생성하고 실행합니다.

```bash
docker run -p 8080:80 -d --name nginxDocker nginx
```

## 옵션 상세 설명

| 옵션                   | 설명                                                                              |
| -------------------- | ------------------------------------------------------------------------------- |
| `docker run`         | 컨테이너를 새로 생성하고 동시에 실행합니다.                                                        |
| `-p 8080:80`         | 포트 포워딩 설정입니다. 호스트의 **8080 포트**로 들어온 요청을 컨테이너 내부의 **80 포트(Nginx 기본 포트)**로 전달합니다. |
| `-d`                 | 컨테이너를 **백그라운드(데몬) 모드**로 실행합니다. 이 옵션이 없으면 포어그라운드로 실행되어 터미널에 실시간 로그가 출력됩니다.       |
| `--name nginxDocker` | 컨테이너 이름을 `nginxDocker`로 지정합니다.                                                  |
| `nginx`              | 실행할 이미지 이름입니다.                                                                  |

---

## 최초 실행 시 내부 동작

로컬 환경에 `nginx` 이미지가 없다면 Docker는 자동으로 Docker Hub에서 이미지를 다운로드(Pull)합니다.

```text
Unable to find image 'nginx:latest' locally
latest: Pulling from library/nginx
e95a6c7ea7d4: Pull complete
fcb6fd84b2a0: Pull complete
1b30016634d5: Pull complete
acf093e7a04f: Pull complete
cd9307c9ecd8: Pull complete
1645c1e06f46: Pull complete
df68ee7e7a00: Pull complete
1cf7d051b485: Download complete
e2c07e54e55a: Download complete
Digest: sha256:ec4ed8b5299e5e90694af7750eb6dffd2627317d30544d056b0371f8082f7bce
Status: Downloaded newer image for nginx:latest
8d5f643002029a26e1b9f7c690013313b5ff0dab169937b98ba01d77d06f11d7
```

---

## 실행 중인 컨테이너 확인

```bash
docker ps
```

---

# 3. 호스트 Nginx의 Reverse Proxy 설정

외부 요청을 받아 Docker 컨테이너로 전달하기 위해 호스트 환경의 Nginx를 Reverse Proxy로 설정합니다.

## 설정 파일 편집

```bash
sudo nano /etc/nginx/sites-available/talju
```

기존의 정적 페이지 설정(`root`, `index`, `404 처리` 등)을 제거하고, Docker 컨테이너로 요청을 전달하는 `proxy_pass`를 작성합니다.

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name talju.local;

    location / {
        proxy_pass http://localhost:8080;
    }
}
```

---

# 4. 네트워크 요청 흐름 및 로그 분석

## 전체 데이터 흐름

```text
사용자 브라우저 (talju.local 입력)
        │
        ▼
호스트 Nginx (80번 포트 수신)
        │
        │ proxy_pass http://localhost:8080
        ▼
Docker Bridge (docker0)
        │
        ▼
Docker 외부 포트 (8080)
        │
        │ 포트 포워딩
        ▼
Docker 컨테이너 내부 Nginx (80)
```

---

## Reverse Proxy 구조의 한계점

Reverse Proxy 환경에서는 **호스트 Nginx가 모든 요청을 대신 받아 컨테이너로 전달**합니다.

따라서 컨테이너 내부의 Nginx는 실제 사용자의 IP가 아니라 **Docker Bridge 네트워크(172.17.0.1)**를 요청자로 인식하게 됩니다.

---

## Docker 컨테이너 실시간 로그

```bash
docker logs nginxDocker
```

실제 접속 로그를 확인하면 모든 요청이 `172.17.0.1`에서 온 것처럼 기록됩니다.

```text
# 정상 접근 로그
172.17.0.1 - - [10/Jul/2026:09:11:04 +0000] "GET / HTTP/1.1" 200 896 "-" "Mozilla/5.0..."
172.17.0.1 - - [10/Jul/2026:10:32:51 +0000] "GET / HTTP/1.0" 200 896 "-" "Mozilla/5.0..."

# favicon 요청으로 인한 404 예시
2026/07/10 09:11:04 [error] 29#29: *1 open() "/usr/share/nginx/html/favicon.ico" failed (2: No such file or directory), client: 172.17.0.1, server: localhost, request: "GET /favicon.ico HTTP/1.1", host: "localhost:8080"
```

### 문제점

모든 요청이 동일한 IP(`172.17.0.1`)로 기록되기 때문에 다음과 같은 문제가 발생합니다.

* 개별 사용자를 구분할 수 없음
* IP 기반 접근 제어 적용 불가
* 접속 통계 및 로그 분석의 정확도 저하

---

# 5. 최종 해결 방안 (클라이언트 원본 IP 전달)

이 문제를 해결하기 위해 호스트 Nginx가 프록시 요청을 전달할 때 **원본 클라이언트 IP를 HTTP 헤더에 함께 전달**하도록 설정합니다.

수정된 Nginx 설정은 다음과 같습니다.

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name talju.local;

    location / {
        proxy_pass http://localhost:8080;

        # 원본 클라이언트 정보 전달
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
    }
}
```

설정을 저장한 후 Nginx를 재시작합니다.

```bash
sudo systemctl restart nginx
```

이후에는 컨테이너 내부 애플리케이션에서도 `X-Real-IP` 또는 `X-Forwarded-For` 헤더를 통해 **실제 클라이언트의 IP 정보를 확인**할 수 있습니다.

---

## 정리

* Docker는 애플리케이션과 실행 환경을 하나의 이미지로 묶어 어디서나 동일하게 실행할 수 있도록 해준다.
* `docker run -p 8080:80`을 통해 호스트와 컨테이너의 포트를 연결할 수 있다.
* 호스트 Nginx는 Reverse Proxy 역할을 수행하여 외부 요청을 Docker 컨테이너로 전달한다.
* 기본 Reverse Proxy 환경에서는 컨테이너가 실제 사용자 IP를 알 수 없어 모든 요청이 Docker Bridge IP(`172.17.0.1`)로 기록된다.
* `proxy_set_header`를 이용해 원본 클라이언트 IP를 전달하면 정확한 로그 분석과 보안 정책 적용이 가능해진다.
