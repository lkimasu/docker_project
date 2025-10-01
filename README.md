
-----

# 🚀 Docker Compose 기반 고가용성(HA) 웹 서비스 및 MySQL 복제 환경 구축

## 🎯 프로젝트 개요 및 목적

이 프로젝트는 **Docker Compose**를 활용하여 엔터프라이즈 환경에서 요구되는 **고가용성(High Availability, HA)** 및 **무중단 운영 체계**를 컨테이너 기반으로 구축하고, 통합 모니터링 시스템을 연동하여 운영 가시성을 확보하는 것을 목표로 합니다.

  * **핵심 목표:** 단일 장애점(SPoF) 제거, 서비스 안정성 확보, DevOps/운영 역량 증명.
  * **주요 기술 성과:** Nginx Active-Active 로드 밸런싱, MySQL Master-Slave 복제, Prometheus/Grafana 통합 모니터링 구축.

-----

## 🏗️ 아키텍처 및 기술 스택

### 1\. 시스템 구성도

시스템의 전체적인 구조와 컴포넌트 간의 데이터 흐름을 시각적으로 보여줍니다.

### 2\. 기술 스택 상세 역할

| 구분 | 기술 스택 | 사용 목적 및 역할 |
| :--- | :--- | :--- |
| **오케스트레이션** | `Docker Compose` | 다중 컨테이너 환경의 일관된 정의 및 관리 |
| **LB & Gateway** | `Nginx` | **Active-Active 로드 밸런싱**, SSL/TLS Termination을 통한 HTTPS 통신 구현 |
| **웹 서비스** | `Apache`, `PHP` | 로드 밸런싱 대상 웹 애플리케이션 서버 (Web 1, Web 2) |
| **데이터베이스** | `MySQL` | **Master-Slave 복제**를 통한 HA 및 읽기 부하 분산(Read Scale-out) 기반 마련 |
| **모니터링** | `Prometheus`, `Grafana` | 메트릭 수집 및 대시보드 시각화 (운영 가시성 확보) |
| **수집기** | `Node Exporter`, `Mysql Exporter` | Host OS 및 MySQL 서버 성능 지표 수집 |

-----

## ⚙️ 프로젝트 설치 및 실행 (Quick Start Guide)

### 1\. 필수 요구사항

  * `Docker Engine` 및 `Docker Compose` V2+ 설치
  * Host OS: Ubuntu (VMware 환경)

### 2\. SSL 인증서 생성 (필수 사전 작업)

Ubuntu 호스트 OS에서 자체 서명 인증서를 생성하고 Nginx 컨테이너에 마운트합니다.

```bash
# 프로젝트 루트 디렉토리로 이동 (예: cd ~/my-ha-project)
# 1. Nginx 설정 및 SSL 파일을 위한 디렉토리 생성
mkdir -p ./nginx/ssl 

# 2. OpenSSL을 사용하여 자체 서명 인증서와 개인 키 생성
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout ./nginx/ssl/nginx.key \
    -out ./nginx/ssl/nginx.crt
```

> **💡 참고:** 인증서 생성 시 **Common Name**은 접속할 **Host IP 주소** 또는 `localhost`로 입력하십시오.

### 3\. Docker Compose 실행

```bash
# 1. GitHub에서 프로젝트 클론
git clone [Your Repository URL]
cd [project-directory]

# 2. 모든 서비스 컨테이너 실행 (Docker Network 자동 구성)
docker-compose up -d --build

# 3. 서비스 상태 확인
# 모든 서비스(7개)가 'Up' 상태인지 확인합니다.
docker-compose ps
```

### 4\. 접속 정보

| 서비스 | 접속 URL (Host IP 기준) | 기본 인증 정보 |
| :--- | :--- | :--- |
| **웹 서비스** | `https://[Host IP]` | (N/A) |
| **Grafana 대시보드** | `http://[Host IP]:3000` | admin / admin (또는 설정한 값) |
| **Prometheus UI** | `http://[Host IP]:9090` | (N/A) |

-----

## 🛠️ 핵심 기능 작동 및 증명 방법 (Usage Scenarios)

### 1\. 보안 통신 (SSL/TLS) 구현 확인

  * **접속:** `https://[Host IP]`로 접속합니다.
  * **증명:** 브라우저 주소창의 **자물쇠 아이콘**을 클릭하여 Nginx에 적용된 SSL/TLS 인증서가 정상적으로 인식됨을 확인합니다.

### 2\. 고가용성(HA) 및 로드 밸런싱 증명

  * **로드 밸런싱 확인:** 웹 페이지 새로고침 시 출력 서버가 "Webserver1"과 "Webserver2"로 **번갈아 가며 출력**됨을 확인합니다.
  * **HA 장애 대응 확인:**
    ```bash
    # Webserver1 강제 종료
    docker stop web1
    ```
    Web 접속 후 새로고침 시, 서비스 중단 없이 **Webserver2로만 자동 우회**됨을 확인하여 단일 서버 장애 대응 능력을 증명합니다.

### 3\. MySQL Master-Slave 복제 상태 확인

1.  **Slave 컨테이너 접속:**
    ```bash
    docker exec -it db_slave mysql -u root -p
    ```
2.  **복제 상태 확인 명령어 실행:**
    ```sql
    SHOW SLAVE STATUS\G
    ```
3.  **결과 강조:** 다음 핵심 값들을 확인하여 복제 지연 없이 정상 작동함을 증명합니다.
      * `Slave_IO_Running`: **`Yes`**
      * `Slave_SQL_Running`: **`Yes`**
      * `Seconds_Behind_Master`: **`0`**

### 4\. 통합 모니터링 확인

  * **Grafana 대시보드**에 접속하여 **Node Exporter**를 통한 Host OS의 리소스 사용량과 **Mysql Exporter**를 통한 DB의 성능 및 복제 상태를 실시간으로 확인합니다.

-----

## 📈 향후 계획 (Future Roadmap)

1.  **DB 백업 자동화 및 재해 복구(DR) 대비**
      * `mysqldump` 또는 `Percona XtraBackup` 스크립트 작성 및 `crontab` 등록을 통한 자동화 구현.
      * 백업 파일을 **AWS S3** 등 외부 스토리지로 이관하여 \*\*클라우드 기반 재해 복구(DR)\*\*에 대비.
2.  **자동 알림 및 장애 대응 시스템 구축**
      * **Prometheus Alertmanager**를 사용하여 특정 임계치 초과 또는 에러 로그 발생 시 **Slack/E-mail**로 알림 전송 파이프라인 구축.
3.  **컨테이너 개별 성능 모니터링 심화**
      * **cAdvisor**를 Exporter로 추가하여 **각 Docker 컨테이너별** 세분화된 리소스 사용량을 추적 및 최적화.

-----

## 🧹 프로젝트 종료 (Cleanup)

```bash
# 모든 컨테이너, 네트워크 및 볼륨 삭제 (완벽한 초기화)
docker-compose down -v
```