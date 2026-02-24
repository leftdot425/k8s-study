# minikube 설치

```bash
brew install minikube
minikube start
```

# 6 쿠버네티스 시작하기
## 6.2 포드(Pod): 컨테이너를 다루는 기본 단위

### 6.2.1 포드 사용하기

```
# nginx-pod.yaml
kubectl apply -f nginx-pod.yaml

kubectl get pods

kubectl describe pod my-nginx-pod

# curl {ip_of_nginx} # 클러스터의 노드 중 한 곳에서 실행, Mac Minikube에서는 안됨

# 클러스터 노드로 접근 불가능하다면 테스트용 포드 생성
kubectl run -i --tty --rm debug --image=alicek106/ubuntu:curl --restart=Never -- sh

kubectl delete -f nginx-pod.yaml # kubectl delete pod my-nginx-pod
```

### 6.2.2 포드 vs 도커 컨테이너

```
# nginx-pod-with-ubuntu.yaml
kubectl apply -f nginx-pod-with-ubuntu.yaml

kubectl exec -it my-nginx-pod -c ubuntu-sidecar-container -- bash
# 컨테이너 안에서 실행
# 포드 내의 컨테이너들이 네트워크 네임스페이스 등과 같은 리눅스 네임스페이스를 공유해서 사용하기에 nginx 서버로 접근 가능함
> curl localhost

kubectl delete -f nginx-pod-with-ubuntu.yaml
```

### 6.2.3 완전한 애플리케이션으로서의 포드
* 하나의 포드는 하나의 완전한 애플리케이션이며, 로그 수집 등이 필요할 때 사이드카 컨테이너를 생성해서 사용함

## 6.3 레플리카셋(Replica Set): 일정 개수의 포드를 유지하는 컨트롤러
* 사용이유: 정해진 수의 동일한 포드가 항상 실행되도록 관리(e.g. 노드 장애 등의 이유로 포드를 사용할 수 없을 때 다른 노드에서 포드를 다시 생성)

### 6.3.2 레플리카셋 사용하기

```
kubectl apply -f replicaset-nginx.yaml

kubectl delete -f replicaset-nginx.yaml
# 위와 같음
kubectl delete replicaset replicaset-nginx
kubectl delete rs replicaset-nginx
```

### 6.3.3 레플라케셋의 동작 원리
* 레플리카셋과 포드는 라벨 셀렉터를 통해 느슨한 연결을 유지하고 있음

### 6.3.4 래플리케이션 컨트롤러 vs 레플리카셋
* 래플리케이션 컨트롤러은 deprecated 되었으며, 레플리카셋부터 matchExpressions를 지원

## 6.4 디플로이먼트(Deployment): 레플리카셋, 포드의 배포를 관리
* 여기부터!