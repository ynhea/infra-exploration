\# < 장애 대응 트리아지(Triage) 순서 >

* ping (L3, 서버 생사)
* &#x20; → 포트 확인 (L4, 서비스 문이 열려있나)
* &#x20;   → 프로세스 확인 (systemctl status, 실제 프로그램이 떠있나)
* &#x20;     → 컨테이너 내부 확인 (docker ps / docker logs, 컨테이너 자체 문제인가)



\---



\# <서비스 운영 체크리스트>



> 대상 서비스: 호스트 Nginx (Reverse Proxy) → Docker 컨테이너(nginxDocker)

> 작성 목적: 담당자가 바뀌어도 동일한 절차로 장애를 진단/복구할 수 있도록 함

> 최종 갱신: 2026-07-11



\---



\## 0. 서비스 구조 요약



```

사용자 브라우저 (talju.local)

&#x20;       ↓

호스트 Nginx (80번 포트, systemd로 관리)

&#x20;       ↓ proxy\_pass http://localhost:8080

Docker 컨테이너 nginxDocker (8080:80, docker로 관리)

```



\- 호스트 Nginx: `systemctl`로 관리 (OS 부팅 시 자동 시작 - enabled)

\- Docker 컨테이너: `docker` 명령으로 관리 (수동 시작 필요, systemd에 등록 안 되어 있음 ⚠️)



\---



\## 1. 서버가 살아있는가? (네트워크 레벨)



```bash

ping <서버 IP>

```

\- 성공 → 서버 전원/네트워크 인터페이스는 정상 (L3)

\- 실패 → 서버 다운 또는 네트워크 단절. 여기서 더 못 내려감. 물리/클라우드 콘솔 확인 필요.



> ⚠️ 주의: 같은 PC 안(localhost/WSL)에서 ping하면 항상 성공하므로 판단 불가.

> 반드시 \*\*다른 PC/외부\*\*에서 ping해야 의미 있음. (Day2 디버깅 사례)



\---



\## 2. 서비스 포트가 열려있는가? (L4)



```powershell

Test-NetConnection -ComputerName <서버 IP> -Port 80

```

\- `TcpTestSucceeded : True` → 포트까지는 정상 도달

\- False → 방화벽 / 서비스 다운 / Security Group(AWS) 확인 필요



\---



\## 3. 프로세스가 정상 작동 중인가? (OS 레벨)



\### 3-1. 호스트 Nginx

```bash

systemctl status nginx

```

\- `Active: active (running)` 확인

\- 죽어있으면: `sudo systemctl restart nginx` → 이후 `sudo nginx -t`로 설정 문법 검사



\### 3-2. Docker 컨테이너

```bash

docker ps          # 실행 중인 것만

docker ps -a       # 정지된 것 포함 전체

```

\- `docker ps`에 안 보이는데 `docker ps -a`엔 보임 → 정지 상태

\- 재시작: `docker start nginxDocker`

\- 완전히 새로 만들어야 할 때만: `docker run ...` (컨테이너 자체가 삭제된 경우)



> ⚠️ 알아둘 점: Docker Desktop(WSL 환경)이 재시작되면 컨테이너는 \*\*자동으로 살아나지 않음\*\*.

> 호스트 nginx는 systemd `enabled`라 자동 복구되지만, Docker 컨테이너는 수동으로 `docker start` 필요.



\---



\## 4. "접속이 안돼요" 신고 시 확인 순서 (로그 레벨)



\*\*진단 순서 (바깥 → 안쪽으로 좁혀간다):\*\*



1\. `ping` → 서버 생사

2\. 포트 확인 → 서비스 문이 열려있는지

3\. `systemctl status nginx` → 호스트 nginx 생사

4\. `docker ps -a` → 컨테이너 생사

5\. `docker logs nginxDocker --tail 20` → 컨테이너 내부 프로그램(nginx) 관점의 로그

6\. `docker inspect nginxDocker` (State 섹션) → Docker가 기록한 종료 사유

&#x20;  - `OOMKilled: true` → 메모리 부족

&#x20;  - `Error`에 값이 있음 → Docker 엔진이 판단한 종료 사유

&#x20;  - 전부 비어있음 → 외부 요인(환경 자체가 내려감) 의심

7\. 호스트 로그 확인

&#x20;  ```bash

&#x20;  sudo tail -n 20 /var/log/nginx/access.log

&#x20;  sudo tail -n 20 /var/log/nginx/error.log

&#x20;  ```



\*\*복구 후 검증 (반드시 "증거"로 확인, 화면만 믿지 않기):\*\*

```bash

curl -H "Host: talju.local" localhost

docker logs nginxDocker --tail 5

```

\- curl 응답이 왔어도, `docker logs`에 방금 요청(User-Agent: curl/...)이 `172.17.0.1`로 찍혔는지까지 봐야

&#x20; "진짜 reverse proxy를 거쳐 컨테이너까지 도달했다"는 확실한 증거가 됨.

&#x20; (host nginx의 default 페이지와 컨테이너 nginx의 default 페이지는 겉보기에 동일하므로 화면만으로는 구분 불가)



\---



\## 5. 이 서버의 sudo 권한자는 누구인가? (권한 레벨)



```bash

grep 'sudo' /etc/group

id <계정명>

```

\- sudo 그룹에 속한 계정 = 이 서버를 망가뜨릴 수도, 살릴 수도 있는 사람

\- 신규 서버 인수인계 시 반드시 최우선으로 확인



\---



\## 6. 알려진 장애 사례 (Postmortem)



\### 사례 1: Docker Desktop 종료로 인한 전체 서비스 다운 (2026-07-11)

\- \*\*증상\*\*: talju.local 접속 불가, Docker UI에서 컨테이너가 정지 상태로 표시

\- \*\*원인\*\*: Docker Desktop 종료 → WSL 환경 재시작 → 그 위 모든 프로세스(호스트 nginx, Docker 엔진, 컨테이너) 강제 종료

&#x20; - 호스트 nginx: systemd `enabled` 설정 덕분에 WSL 재시작 시 자동 복구됨

&#x20; - Docker 컨테이너: 자동 복구 설정이 없어 `Exited (255)` 상태로 남음

\- \*\*판단 근거\*\*: `docker logs`에 에러 없음 + `docker inspect`의 `OOMKilled: false`, `Error: ""` → 컨테이너 내부 문제가 아닌 외부(환경) 요인으로 추정

\- \*\*복구\*\*: `docker start nginxDocker`

\- \*\*검증\*\*: curl 요청 후 `docker logs`에서 동일 요청(172.17.0.1, curl UA) 확인

\- \*\*재발 방지 고려사항\*\*: 운영 환경에서는 `docker run --restart unless-stopped` 옵션 또는 `docker compose`의 restart policy로 컨테이너 자동 복구 설정 필요 (이번 실습 범위 밖, 추후 학습 포인트)

