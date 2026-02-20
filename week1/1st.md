## 🚀 실습 시나리오: Airflow & Postgres 서비스 구축
목표: 도커 네트워크를 생성하고, 이 네트워크 안에서 Postgres DB와 Airflow 웹 서버 컨테이너를 실행하여 서로 통신하게 만든다.
* 네트워크 이름: airflow-net
* DB 컨테이너:
    * 이미지: postgres:13
    * 이름: postgres-db
    * 환경변수: 유저/비번/DB명 모두 airflow로 설정
* App 컨테이너:
    * 이미지: 3.1.7-python3.13 (사용자 지정 이미지)
    * 이름: airflow
    * 포트: 호스트 8080 <-> 컨테이너 8080

---
## 📝 실습 퀴즈 (Step-by-Step)

### Step 1. 네트워크 생성

```
두 컨테이너가 DNS(컨테이너 이름)를 통해 서로 통신할 수 있도록 사용자 정의 브리지 네트워크를 만들어야 합니다.
Q1. airflow-net이라는 이름의 브리지 네트워크를 생성하는 명령어를 작성하세요.
```

### Step 2. 데이터베이스 컨테이너 실행

```
데이터베이스는 백그라운드에서 실행되어야 하며, Airflow가 접속할 수 있도록 계정 정보를 설정해야 합니다.
Q2. 다음 조건에 맞춰 Postgres 컨테이너를 실행하는 명령어를 완성하세요.
```

* 네트워크: Step 1에서 만든 airflow-net 사용
* 모드: 백그라운드 실행 (Detached)
* 이름: postgres-db
* 환경변수 (-e): POSTGRES_USER=airflow, POSTGRES_PASSWORD=airflow, POSTGRES_DB=airflow
* 이미지: postgres:13
* 포트: 호스트의 5432 포트를 컨테이너의 5432 포트와 연결 (Dbeaver에서 보기 위해서)
* 볼륨: postgres-data라는 이름의 도커 볼륨을 컨테이너의 /var/lib/postgresql/data 경로에 마운트

### Step 3. Airflow 컨테이너 실행

```
이제 DB와 연결되는 Airflow 컨테이너를 띄웁니다. 외부에서 웹 브라우저로 접속해야 하므로 포트 설정이 중요합니다.
Q3. 다음 조건에 맞춰 Airflow 컨테이너를 실행하는 명령어를 작성하세요.
```

* 네트워크: airflow-net 사용 (DB와 같은 네트워크여야 함)
* 모드: 백그라운드 실행
* 이름: airflow-webserver
* 포트 연결: 호스트의 8080 포트를 컨테이너의 8080 포트와 연결
* 볼륨: 호스트의 현재 디렉토리($PWD)/dags 폴더를 컨테이너의 /opt/airflow/dags 경로에 마운트 (DAG 파일 개발용)
    * dags 폴더를 생성해야 합니다.
* 환경변수 (e): AIRFLOW__DATABASE__SQL_ALCHEMY_CONN=postgresql+psycopg2://airflow:airflow@{무엇이 들어와야할까요?}/airflow
* 이미지: 3.1.7-python3.13
* CMD: airflow standalone

### Step 4. 실행 확인

```
Q4. 두 컨테이너가 정상적으로 'Up' 상태인지 확인하는 명령어는 무엇인가요?
```

### Step 5. dags/ 폴더 아래에 Test Dag 추가 및 확인

```
Q5. 호스트의 $(pwd)/dags 폴더 아래에 하기 test.py 파일을 추가해주세요.
파일 추가 후, airflow container에 들어가 /opt/airflow/dags에서 ls 명령어를 실행해, container와 호스트가 잘 bind-mount되었는지 확인해주세요.
```


```
# 호스트 에서 파일 위치
{current_path}/dags
└── test.py
```
* test.py



### Step 6. 서비스 확인

```
Q6. Dbeaver를 통해 Postgres DB에 Airflow의 MetaDB가 정상적으로 initialize 되었는지 확인하고, Airflow 웹 UI(localhost:8080)에 접속 가능한지 확인하세요.
추가적으로 test_dag가 잘 보이는지 확인하세요.
참고로 airflow 로그인을 위한 계정과 비밀번호는 container의 로그를 보면 알 수 있습니다.
```


---
## ✅ 모범 답안 (Model Answer)

터미널에서 아래 순서대로 입력하여 실습을 진행해 보세요.

### A1. 네트워크 생성

```
docker network create --driver bridge airflow-net
```

* 설명
    * docker network create 명령어로 사용자 정의 브리지를 생성합니다. 이 네트워크에 속한 컨테이너들은 IP가 바뀌어도 컨테이너 이름(postgres-db)으로 서로를 찾을 수 있습니다(내장 DNS 활용).

### A2. Postgres 컨테이너 실행

```
docker run \
  -d \
  --name postgres-db \
  --network airflow-net \
  -e POSTGRES_USER=airflow \
  -e POSTGRES_PASSWORD=airflow \
  -e POSTGRES_DB=airflow \
  -v postgres-data:/var/lib/postgresql/data \
  -p 5432:5432 \
  postgres:13
```

* 설명
    * -d: 백그라운드 모드로 실행.
    * --network: 방금 만든 네트워크에 컨테이너를 소속시킵니다.
    * -e: Postgres 이미지에서 요구하는 필수 환경변수(계정, 비밀번호)를 설정합니다.
    * -v: 도커 볼륨(postgres-data)을 컨테이너 내부 경로에 마운트하여 데이터를 영구적으로 보존합니다. (볼륨이 없으면 자동 생성됩니다)
    * -p: 5432:5432 호스트의 5432 포트로 들어오는 요청을 컨테이너의 5432 포트로 전달합니다.

### A3. Airflow 컨테이너 실행

```
docker run \
  -d \
  --name airflow \
  --network airflow-net \
  -e AIRFLOW__DATABASE__SQL_ALCHEMY_CONN=postgresql+psycopg2://airflow:airflow@postgres-db/airflow \
  -v $(pwd)/dags:/opt/airflow/dags \
  -p 8080:8080 \
  apache/airflow:3.1.7-python3.13 \
  airflow standalone
```

* 설명
    * -p 8080:8080: 호스트의 8080 포트로 들어오는 요청을 컨테이너의 8080 포트로 전달합니다.
    * -v: 호스트의 dags 폴더를 컨테이너와 공유하여, 로컬에서 작성한 DAG 파일이 컨테이너에 즉시 반영되도록 합니다.
    * 같은 --network를 쓰기 때문에, Airflow 내부 설정에서 DB 주소를 적을 때 IP 대신 postgres-db라는 이름을 사용할 수 있습니다.
    * -e: PostgresDB에 대한 연결 정보를 넣습니다.
    * airflow standalone: container 시작 시 실행되는 명령어를 설정합니다.

### A4. 컨테이너 목록 확인

```
docker ps # 실행중인 컨테이너만 확인
docker ps -a # 전체
```
* 설명
    * STATUS 항목이 Up으로 되어 있는지 확인합니다. 만약 보이지 않는다면 docker ps -a로 Exited 상태인지 확인하고, docker logs airflow 에러 로그를 확인해야 합니다.
        * A4_1.PNG 참고
    * A4_2.PNG 참고


### A5. dags/ 폴더 아래에 Test Dag 추가 및 확인

* A5.png 참고

### A6. 서비스 확인

* A6.png 참고

---
💡 추가 학습 포인트 (Troubleshooting)
만약 3.1.7-python3.13 이미지가 Airflow 공식 이미지라면, 단순히 실행만 해서는 DB와 연결되지 않고 자체 내장 DB(SQLite)를 쓰거나 초기화가 필요할 수 있습니다. 실제 운영 환경처럼 구성하려면 docker run 시 Airflow의 DB 연결 환경변수를 추가해야 합니다.
예시 (참고용):
-e AIRFLOW__DATABASE__SQL_ALCHEMY_CONN=postgresql+psycopg2://airflow:airflow@postgres-db/airflow
* 여기서 @postgres-db 부분이 우리가 A2에서 설정한 컨테이너 이름(--name)입니다. 도커 네트워크 덕분에 이 이름이 IP 주소처럼 작동합니다.