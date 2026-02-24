# 실습

## 🚀 실습 시나리오: Custom Airflow 이미지 빌드 & Docker Compose로 서비스 구축
목표:
1. Dockerfile을 이용해 기본 Airflow 이미지에 데이터베이스 및 Spark 연동을 위한 추가 패키지를 설치한 커스텀 도커 이미지를 생성한다.
2. 여러 개의 `docker run` 명령어를 하나의 `docker-compose.yml`로 통합하여 인프라를 코드로 선언하고 배포한다.

---

## 📝 실습 퀴즈 (Step-by-Step)

### Step 1. 커스텀 이미지를 위한 Dockerfile 작성
```
Q1. 다음 조건에 맞춰 `Dockerfile`의 내용을 완성하세요.
1. 베이스 이미지로 apache/airflow:3.1.7-python3.13를 사용하세요.
2. ROOT 유저로 다음 패키지 설치하세요.
    * iputils-ping
    * telnet
3. airflow 유저로 다음 파이썬 패키지를 설치하세요.
    * 패키 설치시 requirements.txt 파일을 사용하여 설치하세요.
    * `apache-airflow-providers-common-sql`
    * `apache-airflow-providers-oracle`
    * `apache-airflow-providers-mysql`
    * `apache-airflow-providers-apache-spark`
4. airflow 서비스가 실행되도록 CMD 명령어를 설정하세요.
```

### Step 2. 커스텀 이미지 빌드
```
Q2. 현재 디렉터리(`.`)에 있는 `Dockerfile`을 사용하여 `my-airflow-airflow:1.0` 이라는 이름과 태그로 이미지를 빌드하세요.
```

### Step 3. docker-compose.yml 작성
```
Q3. 이전 실습에서 사용했던 네트워크, 볼륨, 환경변수, 포트 설정 등을 하나의 YAML 파일(docker-compose.yml)로 정의합니다.
```

### Step 4. 도커 컴포즈 실행 및 확인
```
Q4. 작성한 docker-compose.yml 파일을 기반으로 서비스들을 백그라운드 모드(-d)로 실행하세요.
```

### Step 5. 리소스 일괄 정리
```
Q5. 실습이 끝난 후, 도커 컴포즈로 생성된 서비스들을 종료시켜 주세요.
```

--------------------------------------------------------------------------------
## ✅ 모범 답안 (Model Answer)
### A1. Dockerfile 작성

```
# Dockerfile
FROM apache/airflow:3.1.7-python3.13

# 1. OS 패키지 설치를 위해 root 유저로 전환
USER root

# 2. apt-get으로 ping, telnet 설치 (&& 연산자로 레이어 최소화 및 캐시 삭제)
RUN apt-get update && \
    apt-get install -y iputils-ping telnet && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# 3. 파이썬 패키지 설치를 위해 다시 airflow 유저로 전환 (보안 및 권한 이슈 방지)
USER airflow

# 4. requirements.txt 복사 및 pip install
COPY requirements.txt /requirements.txt
RUN pip install --no-cache-dir -r /requirements.txt

# 5. 컨테이너 시작 시 실행될 기본 명령어 설정
CMD ["airflow", "standalone"]
```

```
# requirements.txt
apache-airflow-providers-common-sql
apache-airflow-providers-oracle
apache-airflow-providers-mysql
apache-airflow-providers-apache-spark
```

### A2. 커스텀 이미지 빌드

```
docker build -t my-airflow:1.0 .
```

### A3. docker-compose.yml 작성

```yaml
# docker-compose.yml
services:
  postgres-db:
    image: postgres:13
    container_name: postgres-db
    environment:
      - POSTGRES_USER=airflow
      - POSTGRES_PASSWORD=airflow
      - POSTGRES_DB=airflow
    ports:
      - "5432:5432"
    volumes:
      - postgres-data:/var/lib/postgresql/data
    # networks:
    #   - airflow-net

  airflow:
    image: my-airflow:1.0    # 빌드한 커스텀 이미지 사용
    container_name: airflow
    environment:
      - AIRFLOW__DATABASE__SQL_ALCHEMY_CONN=postgresql+psycopg2://airflow:airflow@postgres-db/airflow
    ports:
      - "8080:8080"
    volumes:
      - ./dags:/opt/airflow/dags    # 로컬의 dags 폴더 마운트
    command: airflow standalone
    # networks:
    #   - airflow-net
    depends_on:
      - postgres-db                 # 컨테이너 실행 순서 제어

# networks:
#   airflow-net:
#     driver: bridge

volumes:
  postgres-data:
```

### A4. 도커 컴포즈 실행

```
docker-compose up
```

### A5. 도커 컴포즈 리소스 삭제
```
docker-compose down
```