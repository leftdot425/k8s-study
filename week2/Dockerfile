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