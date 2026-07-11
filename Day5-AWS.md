**<네트워크 구조 설계>**

**1. Public Subnet / Private Subnet**

* 웹서버  → Public Subnet (이유 = 브라우저 접속은 공개)
* DB 서버 → Private Subnet (이유 = 정보는 숨겨놓아야함)



**2. 웹서버의 Security Group 규칙**

* 브라우저 → 80(보안X) \& 443(보안O) 포트 → 0.0.0.0/0 (누구나)
* 관리자 SSH → 22 → 내 컴퓨터 IP만



**3. DB 서버의 Security Group 규칙**

* DB 접속 → 3306(MySQL) \& 5432(PostgreSQL) → 웹서버 Security Group
* 웹서버 IP가 아니라 Security Groupt으로 지정 → 고정된 주소가 아닌, 역할로 규칙을 관리
* 문제점 : DB서버가 Private Subnet에 있을 경우 관리자 조작 불가능
* 해결 : 웹서버를 경유해도 되지만, AWS Systems Manager로 경로를 분리함. 유지보수 용이.



**4. 문제풀이**

* 인터넷의 사용자 → 웹 서버 443 접속: 허용/차단?		허용
* 인터넷의 사용자 → DB 서버 3306 접속: 허용/차단?		차단
* 웹 서버 → DB 서버 3306 접속: 허용/차단?			허용
* 관리자 내 컴퓨터 → 웹 서버 22 접속: 허용/차단?		허용
* 관리자 내 컴퓨터 → DB 서버 22 접속: 어떻게 설계할지?	관리자 → 웹서버 22포트 → DB접속



