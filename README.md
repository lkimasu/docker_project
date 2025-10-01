

🚀 Docker Compose 기반 고가용성(HA) 웹 서비스 및 MySQL 복제 환경 구축
🎯 프로젝트 개요 및 목적
이 프로젝트는 Docker Compose를 활용하여 엔터프라이즈 환경에서 요구되는 고가용성(High Availability, HA) 및 무중단 운영 체계를 컨테이너 기반으로 구축하고, 통합 모니터링 시스템을 연동하여 운영 가시성을 확보하는 것을 목표로 합니다.

핵심 목표: 단일 장애점(SPoF) 제거, 서비스 안정성 확보, DevOps/운영 역량 증명.

주요 기술 성과: Nginx Active-Active 로드 밸런싱, MySQL Master-Slave 복제, Prometheus/Grafana 통합 모니터링 구축.

🏗️ 아키텍처 및 기술 스택
1. 시스템 구성도
시스템의 전체적인 구조와 컴포넌트 간의 데이터 흐름을 시각적으로 보여줍니다.

2. 기술 스택 상세 역할
구분	기술 스택	사용 목적 및 역할
오케스트레이션	Docker Compose	다중 컨테이너 환경의 일관된 정의 및 관리
LB & Gateway	Nginx	Active-Active 로드 밸런싱, SSL/TLS Termination을 통한 HTTPS 통신 구현
웹 서비스	Apache, PHP	로드 밸런싱 대상 웹 애플리케이션 서버 (Web 1, Web 2)
데이터베이스	MySQL	Master-Slave 복제를 통한 HA 및 읽기 부하 분산(Read Scale-out) 기반 마련
모니터링	Prometheus, Grafana	메트릭 수집 및 대시보드 시각화 (운영 가시성 확보)
수집기	Node Exporter, Mysql Exporter	Host OS 및 MySQL 서버 성능 지표 수집

Sheets로 내보내기
⚙️ 프로젝트 설치 및 실행 (Quick Start Guide)
프로젝트를 실행하고 핵심 기능을 바로 확인할 수 있는 가이드입니다.

1. 필수 요구사항
Docker Engine 및 Docker Compose V2+ 설치

Host OS: Ubuntu (VMware 환경)

2. 실행 명령어
Bash

# 1. GitHub에서 프로젝트 클론
git clone [Your Repository URL]
cd [project-directory]

# 2. 모든 서비스 컨테이너 실행 (Docker Network 자동 구성)
docker-compose up -d --build

# 3. 서비스 상태 확인
# 모든 서비스(7개)가 'Up' 상태인지 확인합니다.
docker-compose ps
3. 접속 정보
서비스	접속 URL (Host IP 기준)	기본 인증 정보
웹 서비스	https://[Host IP]	(N/A)
Grafana 대시보드	http://[Host IP]:3000	admin / admin (또는 설정한 값)
Prometheus UI	http://[Host IP]:9090	(N/A)

Sheets로 내보내기
🛠️ 핵심 기능 작동 및 증명 방법 (Usage Scenarios)
1. 고가용성(HA) 및 로드 밸런싱 증명
로드 밸런싱 확인: https://[Host IP]에 접속하여 새로고침 시, 웹 페이지 출력 서버가 "Webserver1"과 "Webserver2"로 번갈아 가며 출력되는지 확인합니다. 
(Nginx Round Robin 증명)

HA 장애 대응 확인:

Bash

# Webserver1 강제 종료
docker stop web1
Web 접속 후 새로고침 시, 서비스가 중단되지 않고 Webserver2로만 자동 우회됨을 확인합니다. (단일 서버 장애 대응 능력 증명)

2. MySQL Master-Slave 복제 상태 확인
Slave 컨테이너 접속:

Bash

docker exec -it db_slave mysql -u root -p
복제 상태 확인 명령어 실행:

SQL

SHOW SLAVE STATUS\G
결과 강조: 출력 결과에서 다음 핵심 값들을 확인하여 복제 지연 없이 정상 작동함을 증명합니다.

Slave_IO_Running: Yes

Slave_SQL_Running: Yes

Seconds_Behind_Master: 0

3. SSL 인증서 추가

🔐 Ubuntu 환경에서 SSL 인증서 생성 단계
이 단계들은 Ubuntu 호스트 OS의 터미널에서 진행되며, 생성된 파일은 Docker Compose로 Nginx 컨테이너에 주입됩니다.

1. 인증서 저장 디렉토리 생성
프로젝트 디렉토리 내에 인증서와 키를 저장할 디렉토리를 만듭니다.

Bash

# 프로젝트 루트 디렉토리로 이동 (예: cd ~/my-ha-project)

# Nginx 설정과 인증서를 저장할 디렉토리 생성
mkdir -p ./nginx/ssl 
2. OpenSSL을 사용하여 자체 서명 인증서 및 키 생성
OpenSSL을 사용하여 **개인 키(nginx.key)**와 인증서(nginx.crt) 파일을 동시에 생성합니다.

Bash

openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout ./nginx/ssl/nginx.key \
    -out ./nginx/ssl/nginx.crt
옵션	설명
-x509	자체 서명 인증서(Self-Signed Certificate)를 생성합니다.
-nodes	개인 키 암호화를 생략하여 Nginx 시작 시 비밀번호를 묻지 않도록 합니다.
-days 365	인증서의 유효 기간을 365일로 설정합니다.
-newkey rsa:2048	2048비트 길이의 RSA 개인 키를 새로 생성합니다.
-keyout	생성될 개인 키 파일의 경로를 지정합니다.
-out	생성될 인증서 파일의 경로를 지정합니다.

Sheets로 내보내기
실행 시 입력해야 할 정보:

명령어를 실행하면 몇 가지 정보를 입력하라는 메시지가 나옵니다. 테스트 목적이므로 정확하지 않아도 되지만, Common Name은 Nginx가 접속할 도메인 또는 IP 주소(예: localhost 또는 Ubuntu 호스트의 IP)로 입력하는 것이 좋습니다.

Country Name (2 letter code) [AU]:KR
State or Province Name (full name) [Some-State]:Seoul
Locality Name (eg, city) []:Seoul
Organization Name (eg, company) [Internet Widgits Pty Ltd]:MyPortfolioProject
Organizational Unit Name (eg, section) []:IT
Common Name (e.g. server FQDN or YOUR name) []:localhost  # or your host IP address
Email Address []:[Your Email]
3. Docker Compose에 SSL 파일 마운트 (Nginx 설정)
이제 생성된 nginx.key와 nginx.crt 파일을 Docker Compose의 Nginx 서비스에 볼륨 마운트하고, Nginx 설정 파일에서 이를 사용하도록 지정해야 합니다.

A. docker-compose.yml (볼륨 마운트)
Nginx 서비스에 ./nginx/ssl 디렉토리를 마운트합니다.

YAML

services:
  nginx:
    # ... (생략)
    volumes:
      # Nginx 설정 파일 마운트
      - ./nginx/conf/nginx.conf:/etc/nginx/nginx.conf:ro 
      # SSL 인증서 및 키 파일 마운트!
      - ./nginx/ssl:/etc/nginx/ssl:ro 
      # ... (다른 마운트 경로 생략)
B. nginx.conf (SSL 설정)
Nginx 설정 파일 내의 server 블록에서 HTTPS 포트(443)를 열고, 마운트된 경로를 지정합니다.

Nginx

server {
    listen 80; # HTTP는 HTTPS로 리다이렉트 (선택 사항)
    listen 443 ssl; # HTTPS 포트
    server_name localhost [YOUR_HOST_IP];

    # SSL 인증서 경로 지정 (컨테이너 내부 경로!)
    ssl_certificate /etc/nginx/ssl/nginx.crt; 
    ssl_certificate_key /etc/nginx/ssl/nginx.key;

    # ... (프록시 설정은 여기에 포함)

    location / {
        # Active-Active 로드 밸런싱 설정
        proxy_pass http://web_upstream;
        # ...
    }
}
이 단계를 완료하면, docker-compose up -d를 실행했을 때 Nginx 컨테이너가 생성된 자체 서명 인증서를 사용하여 HTTPS 요청을 처리하게 됩니다.


4. 통합 모니터링 확인
Grafana 대시보드에 접속하여 Node Exporter를 통해 Host OS의 CPU, Memory, Disk I/O를 시각화하고, Mysql Exporter를 통해 DB의 읽기/쓰기 쿼리 수 및 복제 상태를 실시간으로 확인합니다.

📈 향후 계획 (Future Roadmap)
이 프로젝트는 지속 가능한 운영 환경 구축을 목표로 다음과 같은 기능을 추가할 예정입니다.

DB 백업 자동화 및 재해 복구(DR) 대비

mysqldump 또는 Percona XtraBackup 스크립트 작성 및 crontab 등록을 통한 백업 자동화.

백업 파일을 AWS S3 등 외부 스토리지로 이관하여 **클라우드 기반 재해 복구(DR)**에 대비.

자동 알림 및 장애 대응 시스템 구축

Prometheus Alertmanager를 사용하여 특정 임계치 초과 또는 에러 로그 발생 시 Slack/E-mail로 알림 전송 파이프라인 구축.

컨테이너 개별 성능 모니터링 심화

cAdvisor를 Exporter로 추가하여 각 Docker 컨테이너별 CPU, Memory, Network I/O 등 세분화된 리소스 사용량을 추적 및 최적화.

프로젝트 종료 (Cleanup)
Bash

# 모든 컨테이너, 네트워크 및 볼륨 삭제 (완벽한 초기화)
docker-compose down -v